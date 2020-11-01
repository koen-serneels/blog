---
layout: post
title: webOS app development for the Pre2 with JAX-RS and Atmosphere
date: '2011-07-12T17:45:00.000+02:00'
author: Koen Serneels
img: 2.png
tags:
- Mobile
modified_time: '2011-07-13T08:36:16.206+02:00'
thumbnail: http://3.bp.blogspot.com/-c5jLflt7zCM/ThxlRyzhzdI/AAAAAAAAAL4/SHUwUdE-oSI/s72-c/console.JPG
blogger_id: tag:blogger.com,1999:blog-3406746278542965235.post-4841209048958585823
category: mobile
blogger_orig_url: http://koenserneels.blogspot.com/2011/07/webos-app-development-for-pre2-with-jax.html
---

Recently I tried the development possibilities for my Palm Pre2. As it goes, this smartphone runs some modified linux distro (aka webOS). By default you can only access the 'window manager':  the graphical UI which makes your phone, well, 'your phone
So the first thing you need to do if you want to get deeper into the OS is enabling developer mode. This mode allows you to install custom applications, gain console access and the likes. To put the phone in developer mode, see here at the bottom of the page: <a href="https://developer.palm.com/content/index.php?id=4522" target="_blank">webOS 2.0 Devices</a>

The first thing I tried after enabling developer mode was <a target="_blank" href="http://preware.org/#/index/">preware</a>. From their <a target="_blank" href="http://www.webos-internals.org/wiki/Application:Preware#Key_Features">wiki</a>:

<i>Preware is a package management application for the Palm Pre and the Palm Pixi. Preware allows the user to install any package from any of the open standard package repositories on preware.org (or any other location that hosts an open standard package repository). 
Preware relies on a custom written service developed from community research which allows the mojo app to talk to the built-in ipkg tool. </i> Do note that this is a community product and is not part of the 'official SDK' neither do you need it for development. You can basically use it to:
<ul>
	<li>Install/deinstall apps from open repositories (a lot of apps are already available via the preware repo)</li>
	<li>Install updates/check phone info</li><li>Gain terminal access to your pre</li>
	<li>Install the novacom driver, which will also be required for the SDK (preware can install it automatically for you)</li>
	<li>Probably more, see their site</li>
</ul>

Since I was going to try this on Linux (Ubuntu Maverick), I was already preparing for a long night of debugging, catting var/log/messages, resolving and finding missing dependencies, compiling my kernel for some support that would be missing, searching the web for usb problems and so forth.However, as it turned out, none of this was required (no, it really wasn't). Basically, you first enable developer mode, download preware, extract it, plug your Pre2 into the USB (select 'just charge' on the phone) and run preware:

<pre class="brush: shell;">~/Desktop/Palm Pre 2$ java -jar WebOSQuickInstall-4.2.0.jar</pre>

It will ask you to install novacom on first run and after that you'll get the main screen. 
Launching the terminal looks like this:

<div class="separator" style="clear: both; text-align: center;">
<a href="{{site.baseurl}}/assets/img/2011-07-12-webos-app-development-for-pre2-with-jax/console.jpg" imageanchor="1" style="margin-left: 1em; margin-right: 1em;"><img border="0" data-original-height="558" data-original-width="1090" height="124" src="{{site.baseurl}}/assets/img/2011-07-12-webos-app-development-for-pre2-with-jax/console.jpg" width="440" /></a></div>

I used preware to access the console of the phone linux distro via the build in terminal. After that you can probably install or configure an SSH server on the phone. I did not explore this route any further since I only have one phone and I don't want a lot of services draining my battery, lulzers hacking my phone, or needing to hard reset my phone every week.

Secondly I also installed a terminal app on the phone itself via preware.Currently I have problems in my car when its connected via bluetooth. I can see the phonebook, the network connection etc, but I cannot make phone calls. So when next time I'm stuck traffic I'm planning to do some on the fly debugging to find out whats wrong. The terminal app I installed was 'SDLTerminal', the other ones; terminal and terminus didn't work for me (at least not on a install/run lazy fashion)

What the SDK is concerned, it is really easy and it works perfect on my ubuntu. The steps can be found <a target="_blank" href="https://developer.palm.com/content/resources/develop/developing_with_the_eclipse_ide.html">here</a>. As Java developer I'm using it via eclipse, although other possibilities exist, see their site.

Summarized:

<ul>
	<li>Get the latest Eclipse, they advice you to use the Web developer profile</li>
	<li>Install the webOs plugin via eclipse</li>
	<li>Install the Aptana plugin via eclipse (optional)</li>
	<li>Restart eclipse</li>
</ul>

<a  target="_blank" href="https://developer.palm.com/index.php?option=com_content&view=article&layout=page&id=1788#">Install the SDK</a>  I skipped: Java (allready had Java6), ia32-libs (already installed for some other 32bit comatibility I required), novacom (already installed by preware). I also installed the latest version of virtualbox (v4), and then removed it again to install 3.2, since the SDK must have a virtualbox version: >=3.0 && <=3.2.  Now, after you type:  

<pre class="brush: shell;">palm-emulator </pre>

You'll see virtualbox popping up and booting webOS (the image came along with the SDK):

<div class="separator" style="clear: both; text-align: center;">
<table width="100%">
	<tr> 
		<td width="50%"><div class="separator" style="clear: both; text-align: right;">
<a href="{{site.baseurl}}/assets/img/2011-07-12-webos-app-development-for-pre2-with-jax/vbox1.jpg" imageanchor="1" style="margin-left: 1em; margin-right: 1em;"><img border="0" data-original-height="558" data-original-width="1090" height="24" src="{{site.baseurl}}/assets/img/2011-07-12-webos-app-development-for-pre2-with-jax/vbox1.jpg" width="240" /></a></div>

		</td> 
		<td width="50%"><div class="separator" style="clear: both; text-align: left;">
<a href="{{site.baseurl}}/assets/img/2011-07-12-webos-app-development-for-pre2-with-jax/vbox2.jpg" imageanchor="1" style="margin-left: 1em; margin-right: 1em;"><img border="0" data-original-height="558" data-original-width="1090" height="24" src="{{site.baseurl}}/assets/img/2011-07-12-webos-app-development-for-pre2-with-jax/vbox2.jpg" width="240" /></a></div>

		</td>
	</tr>
</table>
</div>

Virtualbox takes your NIC into account, and sets up a NAT connection. The latter means that your emulated webOS can go out on your network; packets get the source address from your host nic, but does not have an ip on your network. So by default you won't be able to access your webos vm FROM the network. If you want that, the easiest thing is probably to switch the network adapter configuration in virtualbox to "bridged" instead (or setup portforwarding).

To get a console to the webOS vm, you have two options: 
<ul>
	<li>The novacom driver acts as middleware both for you phone connected via USB as well as webOS running in the VM. So you can access the console of the vm webOS via the terminal option in preware, just as you would to to access it on the pre.</li>
	<li>Even though NAT is used by default, there are some port mappings made in the virtualbox image configuration. This means that certain ports on the host are forwarded by virtualbox to the VM, which does allow you to connect to the vm without any extra setup when using NAT. But only for these ports, and only from your machine. For example port 5522 is mapped to the webOS VM on 22.</li>
</ul>

Do note that latter is a difference with the 'production' webOS image on the pre2: there is no such ssh daemon running by default, as I addressed previously. There are other differences between the vm and the one on the phone; like no camera, gravity meter etc.  For example, ssh'ing to your webos VM;  <pre class="brush: shell;">ssh root@localhost -p 5522</pre>(There is no password, just hit enter)

Next I followed <a target="_blank" href="https://developer.palm.com/content/resources/develop/building_your_first_app.html">this howto</a> to create my first app. If you are lazy, you could just:  

<ol>
	<li>Startup the emulator (commandline: 'palm-emulator')</li><li>Startup your webos-plugin-enabled eclipse</li>
	<li>File->New->Palm webOS->Hello World application</li>
	<li>Run->Run</li><li>Look on your virtualbox webOS VM, you'll see your application running</li>
</ol>

To run the appp on your pre2 instead, just plugin the pre2 in the USB, select 'just charge' and  <ol><li>Run->Run</li><li>Look on your pre2</li></ol>Finally, I was eager to find out how hard it is to extend the hello world app and let it do something cool. Following the moto 'go hard or go home', my idea was to extend the example to send a photo (either taken directly or selected from FS) in JSON format to a RESTful webservice via ajax. There would also needed to be a web page that queries the same webservice using Ajax push (Comet). So, the flow should look like this:   

<ol>
	<li>Startup the app in webOS</li>
	<li>Click a button which brings the user to a select/take picture menu</li><li>Select (or select freshly taken) picture</li>
	<li>In the app the picture thumbnail and the path on the device should be shown</li><li>After hitting a send button, the image is send to the RESTful webservice</li>
	<li>A possible browser window connected to the server will get the newly uploaded picture automatically pushed</li>
</ol>

To do this I first created a small RESTful webservice via JAX-RS. I used Athmosphere to do the comet part.  

<pre class="brush: java;">
@Path(&quot;upload&quot;)
@Singleton
public class ImageService {
	private static final String THE_IMAGE_LOC = &quot;/tmp/theimage.jpg&quot;;
	private Broadcaster topic = new JerseyBroadcaster();
	
	@POST
	@Consumes(MediaType.APPLICATION_JSON)
	@Broadcast
	public Broadcastable uploadImage(FileContent fileContent) throws IOException {
		FileUtils.writeByteArrayToFile(new File(THE_IMAGE_LOC),Base64.decodeBase64(fileContent.getData()));
		return new Broadcastable(fileContent.getData() + &quot;END", &quot;&quot;, topic);
	}
		
	@GET
	public SuspendResponse&lt;string&gt; suspend() {
		return new SuspendResponse.SuspendResponseBuilder&lt;string&gt;().broadcaster(topic).outputComments(false).build();
	}
}
</pre>

The web.xml:

<pre class="brush: xml;">
&lt;servlet&gt;
&lt;description&gt;AtmosphereServlet&lt;/description&gt;
&lt;servlet-name&gt;AtmosphereServlet&lt;/servlet-name&gt;        &lt;servlet-class&gt;org.atmosphere.cpr.AtmosphereServlet&lt;/servlet-class&gt;\n  &lt;init-param&gt;\n   &lt;param-name&gt;org.atmosphere.core.servlet-mapping&lt;/param-name&gt;
   &lt;param-value&gt;/ImageService&lt;/param-value&gt;
  &lt;/init-param&gt;
        &lt;load-on-startup&gt;1&lt;/load-on-startup&gt;
    &lt;/servlet&gt;

    &lt;servlet-mapping&gt;
        &lt;servlet-name&gt;AtmosphereServlet&lt;/servlet-name&gt;
        &lt;url-pattern&gt;/ImageService/*&lt;/url-pattern&gt;
    &lt;/servlet-mapping&gt;
&lt;/web-app&gt;
</pre>

The browser will call the 'suspend' method which will block on the topic untill new data is available. When an image is uploaded, the broadcast will trigger the suspend method to resume for possible connected clients. In the result they'll find the image in base64 format.  Atmosphere also comes with a JQuery based plugin, which makes it easy to make an Ajax connection to the webservice. Via the plugin you can easily switch between comet types or even websocket. In this case I've used long polling.  
<pre class="brush: java;">
&lt;script type="text/javascript"&gt;
    $(document).ready(function() {
        var callbackAdded = false;
        var lastRead=0;

        function subscribe() {
            function callback(response) {
           var data = response.responseBody;
           if(data.length &gt; 3 && data.substring(data.length - 3, data.length) == &quot;END&quot;){
          data = data.substring(lastRead, (data.length-3));
          lastRead=lastRead+data.length+3;
           $(&quot;#imageHolder&quot;).replaceWith(&quot;&lt;img width='&quot;+ ( $(window).height())+&quot;' height='&quot;+( $(window).height()/1.2) +&quot;' id='imageHolder' src='data:image/jpg;base64,&quot;+data+&quot;'/&gt;&quot;);
           }
            }
            var location = &quot;ImageService/upload&quot;;
            $.atmosphere.subscribe(location, !callbackAdded ? callback : null, $.atmosphere.request = { transport: &quot;polling&quot; });
            callbackAdded = true;
        }
        subscribe();
    });
&lt;/script&gt;
</pre>
As you can see I directly injected the base64 string in an HTML img tag. I also applied some superior scaling algorithm to make the images a bit smaller

At this point, there were some problems:  

<ul>
	<li>On my setup this was not doing long polling, but streaming. In case of long polling a new connection is made once data has been returned from the server.In my case the connection remained open forever (==streaming). Maybe I need some server side config for long polling? I could not find this anywhere...</li>
	<li>Every time the callback is called, you get the newly written data AND the previously written data back. So when sending a second image, the response still contains the first image. My guess is that this is a side effect of streaming? To get around this I'm keeping a 'lastRead' variable as you can see.</li>
	<li>The callback is called multiple times for different chunks of the received data. This means that I must find a way to denode the end of a transmission. Apparently there are some characters added to the end (CRLF I assume) so I have could used these, but to lazy to find out, so I added a token myself (namely "END").</li>
</ul>
	
For the JS imports, this is all I used: 
	
<pre class="brush: javascript;">&lt;script src=&quot;js/jquery-1.4.2.js&quot; type=&quot;text/javascript&quot;&gt;&lt;/script&gt;
&lt;script src=&quot;js/jquery.atmosphere.js&quot; type=&quot;text/javascript&quot;&gt;&lt;/script&gt;
</pre>

The atmosphere plugin came bundled with JQuery 1.4.2 so I used that version, no idea if it also works with newer ones.  Btw the jquery.atmosphere.js plugin I found it <a target="_blank" href="https://oss.sonatype.org/content/repositories/releases/org/atmosphere/atmosphere-jquery/">here</a> (for the download link of this app see below).

Now, for the Pre2 application: the applications that plugin into webOS are written in Javascript (yes, that 0 == "" language). For the record, you can also develop them in C/C++ if you need speed and very low level access to the hardware.  The developer site explains (in a very annoying and hard to use layout) that there is a broad javascript platform which contains all the components you need to build applications fast. I was quite impressed by the amount of components/services/widgets available, it is like a JQuery/prototype on steroids. There is even test support built in. If you know JQuery or prototype a lot of these things will seem familliar.

All of this allows you to develop via javascript on a higher level.  I knew the Ajax thing was going to be easy, so the first thing I tried out was how to read a (binary) file and transform it to base64. Apparently webOS (2+) uses "Node.js", so thats piece of cake. The hard part was to learn that you can only access the node functionality from a headless service rather then an application (not 100% sure on this, but thats how it looked like).  The next annoying thing to find out was that I could not get the eclipse plugin to package my service along with my app. So I have to manually package and upload the service to the phone.  The service code looks like this:  

<pre class="brush: java;">
var fs = IMPORTS.require('fs');
var ReadFileAssistant = function(){
}
ReadFileAssistant.prototype.run = function(future) { 
	var file = fs.readFileSync(this.controller.args.filePath);
	var fileBase = file.toString(&quot;base64&quot;,0,file.length);
	future.result = { reply: fileBase};
</pre>

I used a hello world service sample (from the SDK docs&samples), removed the hello world stuff and added my code in place. The "buildpackage" file contains the commands you need to build an ipk from it and get that on your phone/emulator. You can run the service, but that is not required. As it is headless it won't show you anything, besides the entry page of the application. From the moment it is installed you'll be able to access the service from any application (at least when it is made "public"). Btw; apparently a service must be package inside an application. 

So in the end I will have two applications: one with the service inside and one with the UI. But this is only because eclipse withheld me from packing them together (and packing the UI manually together with the service would take to much time for testing).

Next we have the UI part which is made out of HTML: 
 
 <pre class="brush: html;">&lt;div id=&quot;main&quot; class=&quot;palm-hasheader&quot;&gt;
 &lt;div class=&quot;palm-header&quot;&gt;Demo app&lt;/div&gt;

 &lt;div x-mojo-element=&quot;Drawer&quot; id=&quot;drawerId&quot; class=&quot;drawerClass&quot; name=&quot;drawerName&quot;&gt;
  &lt;div class=&quot;palm-group&quot;&gt;
   &lt;div class=&quot;palm-group-title&quot; x-mojo-loc=''&gt;Image:&lt;/div&gt;
   &lt;div class=&quot;palm-list&quot;&gt;
    &lt;div class=&quot;first row&quot;&gt;
     &lt;div class=&quot;palm-row-wrapper textfield-group&quot; x-mojo-focus-highlight=&quot;true&quot;&gt;
      &lt;div x-mojo-element=&quot;ImageView&quot; id=&quot;ImageId&quot; class=&quot;ImageClass&quot; name=&quot;ImageName&quot; align=&quot;center&quot;&gt;&lt;/div&gt;
     &lt;/div&gt;
    &lt;/div&gt;

    &lt;div class=&quot;last row&quot;&gt;
     &lt;div class=&quot;palm-row-wrapper&quot;&gt;
      &lt;div id=&quot;File&quot; style=&quot;font-size: 10px&quot; x-mojo-element=&quot;TextField&quot; /&gt;
     &lt;/div&gt;
    &lt;/div&gt;
   &lt;/div&gt;
  &lt;/div&gt;
 &lt;/div&gt;
&lt;/div&gt;
&lt;div id=&quot;Select&quot; name=&quot;Select&quot; x-mojo-element=&quot;Button&quot;&gt;&lt;/div&gt;
&lt;div id=&quot;Send&quot; name=&quot;Send&quot; x-mojo-element=&quot;Button&quot;&gt;&lt;/div&gt;
&lt;div id=&quot;result&quot; class=&quot;palm-body-text&quot; /&gt;
&lt;/div&gt;
</pre>
 
 The "assistent" then, which forms the logic behind the view:  
 
 <pre class="brush: java;">
 FirstAssistant.prototype.handleSelect = function(event) {
 var self = this;
 var params = {
  defaultKind : 'image',
  onSelect : function(file) {
   self.controller.get('File').innerHTML = file.fullPath;
   self.controller.get('ImageId').mojo.centerUrlProvided(file.fullPath);
   self.controller.get('drawerId').mojo.setOpenState(&quot;true&quot;);
  }
 };
 Mojo.FilePicker.pickFile(params, this.controller.stageController);
}

FirstAssistant.prototype.handleSend = function(event) {
 var that = this;
 var filePath =  that.controller.get('File').innerHTML;

 this.controller.serviceRequest(&quot;palm://com.palmdts.servicesample.service&quot;,
   {
    method : &quot;readFile&quot;,
    parameters : {
     &quot;filePath&quot; : filePath
    },
    onSuccess : function(response) {
     var contentToSend = {filePath:filePath, data: response.reply};
     $.ajax({
      type : &quot;POST&quot;,
      contentType:&quot;application/json&quot;,
      url : &quot;http://192.168.0.1:8080/WebOSServer/ImageService/upload&quot;,
      data: Object.toJSON(contentToSend),
      cache : false,
      success : function(result) {
       that.controller.get(&quot;result&quot;).innerHTML = &quot;File Sent!&lt;br/&gt;&quot;+ result;
      },
      error: function(result){
       that.controller.get(&quot;result&quot;).innerHTML = &quot;File sending failed.&quot;;
      }
     });
    },
    onFailure : function(response) {
     that.controller.get(&quot;result&quot;).innerHTML = &quot;FAILURE:&quot;
       + response.reply;
    }
   });
}

FirstAssistant.prototype.setup = function() {
 var libraries = MojoLoader.require({
  name : &quot;mediacapture&quot;,
  version : &quot;1.0&quot;
 });
 this.mediaCaptureObj = libraries.mediacapture.MediaCapture();

 this.controller.setupWidget(&quot;Select&quot;, {}, {
  &quot;label&quot; : &quot;Select&quot;,
  &quot;buttonClass&quot; : &quot;&quot;,
  &quot;disabled&quot; : false
 });
 Mojo.Event.listen(this.controller.get(&quot;Select&quot;), Mojo.Event.tap,
   this.handleSelect.bind(this));

 this.controller.setupWidget(&quot;Send&quot;, {}, {
  &quot;label&quot; : &quot;Send&quot;,
  &quot;buttonClass&quot; : &quot;affirmative&quot;,
  &quot;disabled&quot; : false
 });
 Mojo.Event.listen(this.controller.get(&quot;Send&quot;), Mojo.Event.tap,
   this.handleSend.bind(this));

 this.controller.setupWidget(&quot;File&quot;, {}, {
  &quot;disabled&quot; : true
 });
 
 this.controller.setupWidget(&quot;ImageId&quot;);
 this.controller.setupWidget(&quot;drawerId&quot;,
     this.attributes = {
         modelProperty: 'open',
         unstyled: true
     },
     this.model = {open: false}
   ); 
}

function FirstAssistant() {
 var libraries = MojoLoader.require({
  name : &quot;mediacapture&quot;,
  version : &quot;1.0&quot;
 });
 this.mediaCaptureObj = libraries.mediacapture.MediaCapture();
}

FirstAssistant.prototype.activate = function(event) {
 this.controller.get('ImageId').mojo.manualSize(100,100);
};

FirstAssistant.prototype.deactivate = function(event) {
};

FirstAssistant.prototype.cleanup = function(event) {
};
</pre>
 
 The first function opens the file picker that allows you to select a file (an image in our case) from the file system. This filesystem to which the picker default goes is mounted under '/media'. This is also the place where normally all the user data is located. I was in luck that this file picker automatically shows a  button to take a new picture, so with this I had everything in one. The result (see <a target="_blank" href="https://developer.palm.com/content/api/reference/mojo/classes/mojo-filepicker.html">here</a>) in case of an image contains the 'fullPath' to the image. Next, I set the path in the textfield, and set it on an image viewer which will show the image. Finally I open the 'drawer' so that everything becomes visible.  

The second method uses JQuery to do a POST Ajax call to the RESTFull webservice. It first calls the service to load our image and transforms it to base64, the rest is normal JQuery usage.  Some notes: 

<ul>
	<li>The the webOS application javascript imports (mojo/mojo-loader) have prototype (the JS framwork) build in. I don't know which version or how exactly, but you can easily do Ajax using prototype syntax. Since I'm more familiar with JQuery and I wanted to try if that also worked together, I did it this way (so I explicity imported JQuery in my index.html)</li>
	<li>The file loaded is put into memory. Since its base64 it will probably be around 30% bigger then the initial file. I have no idea how much a JS in webOS (webkit) can handle by default, but I doubt I'll be able to send 100mb files this way.</li>
</ul>

You can download the Pre2 app together with the Java backend here: <a target="_blank" href="https://sourceforge.net/projects/webosphotouploa/files/webos-upload-photo.tar.gz/download">DOWNLOAD</a>

Btw; if you want to try this app (or any other app that requires your computer as server) on your phone, and you don't have any wireless connection, you can use the USB cable. See <a target="_blank" href="http://www.webos-internals.org/wiki/USBnet_Setup">here</a>.
On ubuntu this works out of the box. You just type 'usbnet enable' on the phone, restart it, and ubuntu will automatically add a new NIC (called usb0). You can then reach the host computer from your pre2. If you want to reach the entire network, you'll have to iptables something like this:

<pre class="brush: shell;">iptables --table nat --append POSTROUTING --out-interface eth0 -j MASQUERADE
iptables --append FORWARD --in-interface usb0 -j ACCEPT
echo 1 > /proc/sys/net/ipv4/ip_forward
</pre>

To conclude:

Developing for the Pre2 is really fun. There is a lot of information to be found on the web, and the integration widgets/services are really, really good. As demonstrated, picking an image, taking a new photo, its all there out of the box. Getting the current GPS location, thats also 2 lines of JS etc... You can access almost every part on the Pre2 with some lines of Javascript. The fact that you write something like HTML and Javascript makes that its easy to develop and very lightweight. The SDK is also very good, works out of the box on linux, and the emulator makes it easy to develop without having to use a real phone.

The dislikes: the eclipse plugin is very limited and did not work very stable. Every change you make requires to repackage the app and install it on the emulator or phone. This only takes some seconds, but thats still too much. The debugging capabilities are poor. I was hoping I could put a breakpoint in the JS file in eclipse, but that was not possible. You can put a breakpoint using the CLI and the palm-debug command, but thats not really appealing (and its very time consuming).

Some images :

<table width="100%"><tr> <td colspan="2">Start the app. The right icon is the app in which the service lives. If you launch it you'll just get the empty screen of the application in which the service resides</td> </tr><tr> <td width="50%">
<div class="separator" style="clear: both; text-align: right;">
<a href="{{site.baseurl}}/assets/img/2011-07-12-webos-app-development-for-pre2-with-jax/pre2-1.jpg" imageanchor="1" style="margin-left: 1em; margin-right: 1em;"><img border="0" data-original-height="558" data-original-width="1090" height="24" src="{{site.baseurl}}/assets/img/2011-07-12-webos-app-development-for-pre2-with-jax/pre2-1.jpg" width="220" /></a></div>
</td> <td width="50%">
<div class="separator" style="clear: both; text-align: left;">
<a href="{{site.baseurl}}/assets/img/2011-07-12-webos-app-development-for-pre2-with-jax/pre2-2.jpg" imageanchor="1" style="margin-left: 1em; margin-right: 1em;"><img border="0" data-original-height="558" data-original-width="1090" height="24" src="{{site.baseurl}}/assets/img/2011-07-12-webos-app-development-for-pre2-with-jax/pre2-2.jpg" width="220" /></a></div>
</td> </tr><tr> <td colspan="2">After pushing the 'select' button, the file picker opens. If running on a phone you can either select an existing photo or take a new one. We choose an existing (wallpaper):</td> </tr><tr> <td>
<div class="separator" style="clear: both; text-align: right;">
<a href="{{site.baseurl}}/assets/img/2011-07-12-webos-app-development-for-pre2-with-jax/pre2-3.jpg" imageanchor="1" style="margin-left: 1em; margin-right: 1em;"><img border="0" data-original-height="558" data-original-width="1090" height="24" src="{{site.baseurl}}/assets/img/2011-07-12-webos-app-development-for-pre2-with-jax/pre2-3.jpg" width="220" /></a></div>
</td> <td>
<div class="separator" style="clear: both; text-align: left;">
<a href="{{site.baseurl}}/assets/img/2011-07-12-webos-app-development-for-pre2-with-jax/pre2-4.jpg" imageanchor="1" style="margin-left: 1em; margin-right: 1em;"><img border="0" data-original-height="558" data-original-width="1090" height="24" src="{{site.baseurl}}/assets/img/2011-07-12-webos-app-development-for-pre2-with-jax/pre2-4.jpg" width="220" /></a></div>
</td> </tr><tr> <td colspan="2">After selecting the picture it is shown on the screen. Select it by tapping 'open photo'. After that the image path is passed to our JS. It is then displayed in the image viewer scaled down to a kind of thumbnail:</td> </tr><tr> <td>
<div class="separator" style="clear: both; text-align: right;">
<a href="{{site.baseurl}}/assets/img/2011-07-12-webos-app-development-for-pre2-with-jax/pre2-5.jpg" imageanchor="1" style="margin-left: 1em; margin-right: 1em;"><img border="0" data-original-height="558" data-original-width="1090" height="24" src="{{site.baseurl}}/assets/img/2011-07-12-webos-app-development-for-pre2-with-jax/pre2-5.jpg" width="220" /></a></div>
</td> <td>
<div class="separator" style="clear: both; text-align: left;">
<a href="{{site.baseurl}}/assets/img/2011-07-12-webos-app-development-for-pre2-with-jax/pre2-6.jpg" imageanchor="1" style="margin-left: 1em; margin-right: 1em;"><img border="0" data-original-height="558" data-original-width="1090" height="24" src="{{site.baseurl}}/assets/img/2011-07-12-webos-app-development-for-pre2-with-jax/pre2-6.jpg" width="220" /></a></div>
</td> </tr><tr> <td colspan="2">Tapping 'send' will submit it to our RESTful webservice. The status is shown in text at the bottom of the screen. After some seconds the browser automatically shows the uploaded image:</td> </tr><tr> <td>
<div class="separator" style="clear: both; text-align: right;">
<a href="{{site.baseurl}}/assets/img/2011-07-12-webos-app-development-for-pre2-with-jax/pre2-7.jpg" imageanchor="1" style="margin-left: 1em; margin-right: 1em;"><img border="0" data-original-height="558" data-original-width="1090" height="24" src="{{site.baseurl}}/assets/img/2011-07-12-webos-app-development-for-pre2-with-jax/pre2-7.jpg" width="220" /></a></div>
</td> <td>
<div class="separator" style="clear: both; text-align: left;">
<a href="{{site.baseurl}}/assets/img/2011-07-12-webos-app-development-for-pre2-with-jax/pre2-8.jpg" imageanchor="1" style="margin-left: 1em; margin-right: 1em;"><img border="0" data-original-height="558" data-original-width="1090" height="24" src="{{site.baseurl}}/assets/img/2011-07-12-webos-app-development-for-pre2-with-jax/pre2-8.jpg" width="220" /></a></div>
</td> </tr></table>
