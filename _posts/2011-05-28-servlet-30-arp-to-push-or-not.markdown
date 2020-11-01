---
layout: post
title: 'Servlet 3.0: ARP, to push or not?'
date: '2011-05-28T16:24:00.000+02:00'
author: Koen Serneels
img: 2.png
tags:
- Web
modified_time: '2011-05-28T16:24:52.443+02:00'
blogger_id: tag:blogger.com,1999:blog-3406746278542965235.post-7047439046856591166
category: java EE
blogger_orig_url: http://koenserneels.blogspot.com/2011/05/servlet-30-arp-to-push-or-not.html
---

With the Servlet 3.0 API out now for I while I wanted to check out its ARP (Asynchronous Request Processing) functionality. Briefly: ARP enables us to execute time consuming processing, while not holding on to our Servlet thread. So, I started doing some browsing and after a short while I ended up with more questions then I started with.

This was thanks to a lot of self explanatory words such as: comet, cometd, tektite, meteor, asteroid, cavebear, grizzly, websocket, bayeux, ... ?!?! Btw; can you spot the two words not belonging in the row? To focus back on the Servlet 3.0 ARP functionality, I learned a couple of things: first, it is NOT a solution for server side push ~ Comet (all the other terms ARE related to it). For me this took a while to understand.

If you look up any of the words I mention in combination with ARP, you will most definitely find a how-to that brings ARP in relation with server side push or at least compare them. So that made me confused, thinking that ARP was an implementation for it. Agreed, building a Comet implementation will benefit from ARP (Grizzly Comet API for example). But out of the box ARP is just not Comet and it should  IMHO not be used in the same sentence.

The Servlet 3.0 ARP functionality is limited to, well, asynchronous processing on the server side.If you look in the specification, they do not mention anything about underlying connection mechanisms. Terms as 'long polling' or 'streaming' (which are vital for a Comet implementation) are not mentioned in the specification.

ARP is just a standardized mechanism for offloading your request processing in a different thread then the 'Servlet' thread. When the Servlet thread terminated, the connection is retained somewhere and it gives you a standard mechanism for getting back to the initial Servlet response from the 'asynchronous' thread to write (extra or not) data to the client.

In fact, the client is even not aware that ARP is being used.
From the clients point of view the connection stays open and it takes a long time to get the response (if you have a long running processing, of course). No changes on the client are required when you use ARP. When you use a real comet implementation however, changes are required one way or the other.

Second, I needed to cut down on my enthusiasm  about what it does do.
This is what the specification says about it:

<i style="font-size: smaller;">"Some times a filter and/or servlet is unable to complete the processing of a request without waiting for a resource or event before generating a response. For example, a servlet may need to wait for an available JDBC connection, for a response from a remote web service, for a JMS message, or for an application event, before proceeding to generate a response.
Waiting within the servlet is an inefficient operation as it is a blocking operation that consumes a thread and other limited resources. Frequently a slow resource such as a database  may have many threads blocked waiting for access and can cause thread starvation and poor quality of service for an entire web container.

Servlet 3.0 introduces the ability for asynchronous processing of requests so that the thread may return to the container and perform other tasks. When asynchronous processing begins on the request, another thread or callback may either generate the response and call complete or dispatch the request so that it may run in the context of the container using the AsyncContext.dispatch method"</i>

Mmkay, first this seems weird. We start another thread to do our asynchronous processing, doing so we can release the Servlet thread. Isn't that like &lt;insert funny zero operation analogy here&gt; ?

On resource usage level it is exactly the same. Using ARP you have no extra means in saving resources, like the difference between blocking and non blocking IO containers.The advantage using ARP is that when using a thread pool for its Servlets, it might give another request a chance by freeing up a thread from that pool.
You moved the long running process to a thread from another thread pool. If this thread pool is full, it will not have any effect on (blocking) new requests.

To illustrate this: suppose you have a Servlet thread pool of 100. You fire 200 requests at it at the same time.
The first 100 requests are destined for LongRunningServlet and take a long time to process. The other 100 requests are for DoAlmostNohtingServlet, which processes very fast. Since our Servlet pool is already full with the first 100 (slow running) requests,the other (fast running requests) are queue'd and need to wait until a thread is released to the pool.

If we would be adding ARP with an additional thread pool (lets call this one the ARP pool) of 10 threads, we would have this scenario:

A request ending up in LongRunningServlet publishes an additional processing request to the ARP pool and terminates. The Servlet thread is returned to the Servlet pool almost directly. The result is that our 100 requests for LongRunningServlet are "processed" within seconds, freeing up threads for processing the request for DoAlmostNohtingServlet. The requests for DoAlmostNohtingServlet are then also processed withing some seconds.

<ul>
	<li>The requests DoAlmostNohtingServlet are processed in seconds, they (almost) did not have to wait for a Servlet thread </li>
	<li>After a short time the Servlet thread pool is completely free again, open for serving new requests</li>
	<li>In the mean time there is still a waiting queue for 90 requests for the ARP pool (10 requests are assigned to threads and are already processing).</li>
</ul>
So they will gradually get processed, but while they are processed the servlet thread pool is again completely free for new requests. To make some things clear:

The connections to the clients of LongRunningServlet are still maintained. So the client is waiting untill its request for LongRunningServlet is processed by the ARP pool. Holding the connection (while the thread is busy in the ARP pool) does not consume a thread by itself.ARP will behave exactly the same in a blocking IO container as in a non blocking IO container: For a container using blocking IO, the 'Servlet thread' is the thread that got the Socket connection.

A basic blocking server would do something like this:

<pre class="brush: java;">
	while (true) {
		final Socket socket = serverSocket.accept();
		//Handof the socket to a 'servlet thread'
		threadPool.submit(new Runnable() {
			@Override
				public void run() {
					SomeHttpRequestObject httpRequestObject = parseRequest(socket.getInputStream());
					process(httpRequestObject);}
				});
			}
</pre>

The moral for a blocking IO Servlet container is that the socket (connection) is read in the Servlet thread.This means that a slow connection consumes a thread without anything happening.Suppose you have 1000 slow clients, none of these clients delivered a complete request yet, and at a given point 0 of those clients are sending data: you'll still take up 1000 threads for doing virtually nothing (in the assumption our Servlet thread pool is =&gt; 1000).

With a non block IO server, connection reading takes only resources when there is actually something to read. This reading might happen in the Servlet thread as well, but as long as a request is not completely read and there is nothing more to read at that time, the thread is returned.

So basically a servlet thread is only going to work (for a longer time) when a complete request has been read. Suppose you have 1000 slow clients, none of these clients delivered a complete request yet, and at a given point 0 of those clients are sending data: you'll use (virtually) 0 threads at time.

Now, as you see the blocking/non blocking scenario plays before the request is handed over to the acutal Servlet. So using ARP is not directly influenced by it.

To conclude: ARP allows you to offload a long running process via a separate thread pool. This releases the Servlet thread back to the pool allowing it to service new requests. By doing this your long running processes are not hogging the Servlet thread pool. The connection from the client that triggered the long running process is kept alive and the ARP API gives you the means to get back to that connection to write (additional) data to it when the long running process completed. ARP does not directly saves you any resources, but being able to offload request to a separate thread pool does give you more control about the number of threads maximum in use at a given point, without having other clients to wait to get served.

Also, ARP serves as a scaleable base for possible Comet implementations , since a long polling request (to name one of the Comet connection strategies) does not hog a thread from the Servlet pool while waiting.
