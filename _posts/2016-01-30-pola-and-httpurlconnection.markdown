---
layout: post
title: PoLA and HttpURLConnection
date: '2013-01-30T13:34:00.003+01:00'
author: Koen Serneels
tags:
- Java SE
modified_time: '2020-03-01T21:17:02.519+02:00'
img: 3.jpeg
---

If you are a developer like me you probably heard of '<a href="https://en.wikipedia.org/wiki/Principle_of_least_astonishment">PoLA</a>' or the Principle of Least Astonishment. The principle is fairly simple. Never surprise a user. Or in this case even more important, never surprise a developer. Unfortunately I got more then I bargained for last week when one our web service clients was generating rogue requests.

Rogue requests you say? Yes, like in; we have no freaking idea where they're coming from. It's one of those moments where managers start running around in circles, panicking and yelling "we're hacked!!" or "please someone turn off Skynet!!". Anyway, first a little bit of background. In our project we have automatic activity logging which is triggered by aspects when a process starts. This includes the web service client in question and the processing on the endpoint as both of them are part of our system. So at some point we noticed that before the response was sent by the endpoint, a second call from the same web service client came in. This was unexpected as the client is single threaded and no other clients were in the picture. Review and tests pointed out that it was simplyimpossible©® for our client to simultaneous generate another request while the first one was still in progress.

After a long day of debugging and going through too many logs it turned out that the client was in fact disconnected before processing ended on the endpoint. So there requests weren't simultaneous after all. But why did that took us a day to find out? Did we play starcraft2 again instead of working
Well no, it all started with the HTTP read timeout on the endpoint's container being unexpectedly set lower than we thought. The logging on the endpoint indicated that a reply was generated but the client was actually disconnected before that event because of the read timeout. This was of course not logged by our aspects on the endpoint side as this is decided on a lower level (the HTTP stack) rather then our endpoint itself.
Ok, true, I hear you say, but what about the web service client log? The web service client should have thrown a "ReadTimeoutException" or something similar and that should have been written to the log, right? Well, true, but it didn't. And now it comes, as it turned out the real surprises is inside HttpURLConnection (more specifically the default Oracle internal handler sun.net.www.protocol.http.HttpURLConnection)
Did you know that this default impl of HttpURLConnection has a special "feature" which does HTTP retries in "certain situations"? Yes? No? Well, I for once didn't. So what happened was that the timeout exception was indeed triggered on the web service client but silently catched by HttpURLConnection itself by which it decided to do an internal retry on its own. This means that the read() method called the web service on HttpURLConnection remains blocked, like you are still waiting for the response of the first request. But internally HttpURLConnection is retrying the request more then once and thus generating multiple connections. This explained why it took us so long to discover this as the second call was never logged by our code as it is in fact never triggered by our code but by HttpURLConnection internally.</div>Here some code illustrating this:

<pre class='brush: java;'>
import java.net.HttpURLConnection;
import java.net.InetSocketAddress;
import java.net.SocketTimeoutException;
import java.net.URL;
import java.util.concurrent.Executors;

import com.sun.net.httpserver.HttpServer;

/**
 * Created by koen on 30/01/16.
 */
public class TestMe {

 public static void main(String[] args) throws Exception {
  startHttpd();
  HttpURLConnection httpURLConnection = (HttpURLConnection) new URL("http://localhost:8080/").openConnection();

  if (!(httpURLConnection instanceof sun.net.www.protocol.http.HttpURLConnection)) {
   throw new IllegalStateException("Well it should really be sun.net.www.protocol.http.HttpURLConnection. "
     + "Check if no library registered it's impl using URL.setURLStreamHandlerFactory()");
  }
  httpURLConnection.setRequestMethod("POST");
  httpURLConnection.connect();
  System.out.println("Reading from stream...");
  httpURLConnection.getInputStream().read();
  System.out.println("Done");
 }

 public static void startHttpd() throws Exception {
  InetSocketAddress addr = new InetSocketAddress(8080);
  HttpServer server = HttpServer.create(addr, 0);

  server.createContext("/", httpExchange -&gt; {
   System.out.println("------&gt; Httpd got request. Request method was:" + httpExchange.getRequestMethod() + " Throwing timeout exception");
   if (true) {
    throw new SocketTimeoutException();
   }
  });
  server.setExecutor(Executors.newCachedThreadPool());
  server.start();
  System.out.println("Open for business.");
 }
}
</pre>

If you run this, you'll get:
  
<pre class="brush: bash;">
	Open for business.
	Reading from stream...
	------> Httpd got request. Request method was:POST Throwing timeout exception
	------> Httpd got request. Request method was:POST Throwing timeout exception
	Exception in thread "main" java.net.SocketException: Unexpected end of file from server
	 at sun.net.www.http.HttpClient.parseHTTPHeader(HttpClient.java:792)
</pre>

Notice that our httpd got two calls while we only did one? If we re-run this, but this time set the magic property <a href="http://docs.oracle.com/javase/7/docs/technotes/guides/net/properties.html"> **-Dsun.net.http.retryPost=false**</a> we get:

<pre class="brush: bash;">
	 ------> Httpd got request. Request method was:POST Throwing timeout exception
	Exception in thread "main" java.net.SocketException: Unexpected end of file from server
	 at sun.net.www.http.HttpClient.parseHTTPHeader(HttpClient.java:792) ...
</pre>

Putting all this aside, who the hell builds a retry mechanism that isn't really documented nor configurable? Why am I after 15years of Java development (and a network fetish) not aware of this feature? But more over, why the hell is it doing retries on a freaking HTTP freaking POST? PoLA breach detected!

As you probably guessed by now it's a bug (<a href="http://bugs.java.com/bugdatabase/view_bug.do?bug_id=6382788">http://bugs.java.com/bugdatabase/view_bug.do?bug_id=6382788</a>). Not the retry mechanism of course, that's just crap. The bug is that it also happens for a POST (which is by default not idem potent per HTTP RFC). But don't worry, Bill fixed that bug a long time ago. Bill fixed it by introducing a toggle. Bill learned about backward compatibility. Bill decided it's better to leave the toggle 'on' by default, because that would make it bug-backward-compatible. Bill smiles. He can already see the faces of surprised developers around the globe running into this. Please don't be like Bill?
So after some exciting days of debugging the solution was kinda lame. Merely setting the property to false fixed it. Anyway it surprised me enough to write a blog entry on it, so there you have it.

For the sake of completeness: if you run this code inside a container your results may vary. Depending on libraries used and/or your container, other implementations could have been registered which are then used rather than Oracle's internal one (see URL.setURLStreamHandlerFactory()). So now you might be thinking; why is that guy using the default HttpURLConnection then? Does he also drive to work in a wooden soapbox and cut's his grass with scissors? He could better start a fire and use smoke signals instead! Well, I can't blame you for thinking that, but we never deliberate decided on doings this. Our web service in question is a bit special and uses SAAJ SOAPConnectionFactory which on it's turn uses HttpURLConnection, which reverts to the default impl if no one registered another one.
If you use a more managed WS implementation (like Spring WS, CXF or JAX-WS impls) they will probably use something like Apache HTTP client. And of course if you, yourself would make HTTP connections you would opt for the same. Yes, I'm promoting Apache commons HTTP client, that little critter which changes public API more often then an average fashionista changes shoes. But don't worry, I'll stop ranting now.
