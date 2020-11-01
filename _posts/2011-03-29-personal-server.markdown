---
layout: post
title: Personal server
date: '2011-03-29T07:38:00.000+02:00'
author: Koen Serneels
img: 2.png
tags:
- System
modified_time: '2011-03-29T08:17:50.111+02:00'
blogger_id: tag:blogger.com,1999:blog-3406746278542965235.post-611254188783933851
category: infra
blogger_orig_url: http://koenserneels.blogspot.com/2011/03/personal-server.html
---

For personal use I need a small sized server that is reachable over the Internet. I use it to manage my personal emails, queue long downloads, serve access services (ftp, http, ...).
My requirements:
<ul>
	<li>Reachability: it should be reachable from almost every location, even behind proxies and firewalls</li>
	<li>Stability: it should be as stable as possible</li>
	<li>Usability: it should be able to run any software that I like, without restrictions</li>
	<li>Security: pretty clear I guess</li>
	<li>Manageability: it should be manageable via different interfaces, offer possibilities to easy backup data etc</li>
	<li>Pay-ability: at the lowest cost possible</li>
</ul>

Bandwidth is not a tight requirement, since it will only be serving, well just me. I might put some pages on the web server, but even then images of heavy content will be placed somewhere on the Internet. Also, high availability is also not important, since it will only be for me (as long as there is enough stability). In the end I decided to install a server at home. Doing this I can also use it as a NAS to serve content, store backups and the likes via the home network.  My standard ISP contract does not include fixed IP addresses. But, they do allow each port to be accessible from the Internet, so thats good (there are ISPs that block ports < 1024 without having an option to open them). The bandwidth is not very high, but that is ok for me. In physical measured speeds, it is about 7Mbps down and 418 Kbps up.   So with that in mind, I arranged the following: 

<ol>
	<li>Registered a .be TLD by a provider that has domain management</li>
	<li>Put an old laptop, aka 'server', somewhere on my desk (who needs an UPS, if you have a laptop)</li>
	<li>Register for a free domain alias service</li><li>Configure laptop with OS and other required software</li>
	<li>Configure home router to make server accessible over the Internet</li>
</ol>

The only extra cost for me is about 15euro/year for the TLD (domain+management), that's it.

Ok, so how did I wire this up:

First, the domain is a normal '.be' domain registered via a provider. The provider (aka registrar) registers the domain with the instance controlling the .be TLD zone's. They supply their own name servers as NS record, so my domain will be resolved by my provider name servers. Next, the provider gives me access to control my domain controlled by their name servers. This access is pretty elaborate, its a web interface right on top of the <a href="http://www.isc.org/software/bind" target="_blank">named</a> zone configuration. So I have full control and can actually configure it as it was running on my own name server, sweet. Since I do not have a fixed IP address, I'm not able to point a host name in my domain directly to my home server. I use a free dns alias service (<a href="http://www.dyndns.com/" target="_blank">for example</a>) that is able to update a host name entry directly when my IP address changes.  Before declaring me foobar, I'll give some clarifications at this point: 

<ul>
	<li>The dnsalias is also a name provider, just as my .be domainname provider. For free, they only give you a host within their own domain (like x.dnsalias.com, y.dynip.org, ...) and limited manageability. If you pay them, you can have both though, they register your domain and you can use their dns services to update it as you want. However, they do not support .be domains, since they are not an official .be registrar.</li>
	<li>I could have dropped the '.be' domain and directly used the dnsalias. Now I have two name resolv's, a host on my TLD resolves to the dnsalias which then resolves to my home IP address. By directly using the dnsalias I save one resolve. But, I like to have my own .be domain. Furthermore, the dnsalias name is not as officially mine as a .be domain is. Even not when I pay for their services. (Now I can always decide to get a fixed IP address and point directly to that from my .be domain)</li>
	<li>I could also have dropped the dnsalias and update my IP directly with my domain service provider. But they don't offer an interface to do that on an automated fashion.</li>
</ul>

It took me 5 minutes to go through the free registration for a dnsalias, thats it. The fact that there are two hosts to be resolved are certainly not noticeable for normal usage. To map a host in my domain to the dnsalias, I cannot use a normal DNS 'A' record mapping that maps a host name to an ip address (or the other way around for reverse zones), since it is not possible to use a host name instead of an IP address in an A record. So luckily there exists something as a 'CNAME' record, which allows a host name in your domain to be linked with another host name. So on my domain provider I have something like this setup in my zone; 

<pre class="brush: shell;">host.domain.be. IN CNAME host.dnsalias.com</pre>

You can also use wildcards, so '*.domain.be. IN CNAME host.dnsalias.com' would resolve everything in *.domain.be to host.dnsalias.com.The dnsalias service has a normal A record mapping to my home ip address wit a very low TTL. If I do a zone scan, it looks like this:  

<pre class="brush: shell;">host.dnsalias.com. 60 IN A w.x.y.z</pre>

This means that if the information is propagated, it will be only be cached for 60 seconds on the intermediaries. Not what you typically want for a busy site. My server will run a small client that updates my IP address with the dns alias service each time it changes. It does that by using an external IP check service, that returns the Internet visible ip address. When that is changed over the last time, it sends out an update to the dnsalias service (+ my account information) with the new IP address. In the above zone snippet, the address w.x.y.z will then be updated.

As operating system I choose Ubuntu (10.04.2 LTS 32bit). Its rock solid, supports all hardware on the server, has a great user support base and its pretty secure out of the box. However, I also like windows for its office and outlook for my emails. And some other windows only programs that just work better under windows. The laptop was originally installed with windows, so it had a windows XP cdkey. Also, windows XP also has nice native remote desktop support.  The RDP protocol shares the clipboard, sounds, it reverse maps your hard drive over the same protocol (so on the VM you see a share of the client's hard drive from which you are connecting) and so on. Some will argue that a remote X server is maybe better, might be true, but an RDP client is available on any windows client. On ubuntu its also available by default. On other linux dists is probably a simple download. Setting up a remote X on a windows client will be more work/require more privileges I think.  To resume, these are the steps I did to configure my ubuntu:

<ol>
	<li>Install <a href="#vmwareserver">VMware server</a>, v2.0.2-203138, used bridge Ethernet connection for VM</li>
	<li>Install windows XP on VM, necessary software, and enabled remote desktop</li>
	<li>Install <a href="#vmwareserver">ddclient</a> for automatic dns update</li>
	<li>Configured my physical <a href="#wireless">wiress ethernet connection to autostart without logging in</a></li>
	<li>Install <a href="#sshd">sshd</a>. Adjusted sshd config file to listen on two ports (22 and 443) and allow it to foward (more on that later)</li>
</ol>

<a name="vmwareserver">VMware server:</a> On previous installs of ubuntu (and VMWare server) I never had problems. However this time the installation failed. Thanks to the community I was able to pickup a patch for that: <a href="http://radu.cotescu.com/how-to-install-vmware-server-ubuntu-fedora-opensuse/" target="_blank">radu.cotescu.com</a>. After that VMware server installed without any problems.

<a name="vmwareserver">DDClient:</a>  

<pre class="brush: shell;">
	sudo apt-get install ddclient
	sudo nano /etc/ddclient.conf
	
	# Configuration file for ddclient generated by debconf
	#
	# /etc/ddclient.conf

	protocol=dyndns2
	use=web, web=checkip.dyndns.com, web-skip='IP Address'
	server=members.dyndns.org
	login=&lt;your login&gt;
	password='&lt;your password&gt;'
	yourhost.dnsalias.com
</pre>

I followed this guide: <a href="https://help.ubuntu.com/community/DynamicDNS" target="_blank">ddns ubuntu</a>

<a name="wireless">Wireless Ethernet connection to autostart:</a>The network manager is only started once you logon in X. So I needed something to connect the server to the wireless network the moment ubuntu was booted. I followed this <a href="http://ubuntuforums.org/showthread.php?t=318539" target="_blank">guide</a>. Basically it came to: 

<pre class="brush: shell;">
	sudo gedit /etc/network/interfaces
	
	auto lo
	iface lo inet loopback
	auto wlan0
	iface wlan0 inet static
	address 192.168.0.2
	gateway 192.168.0.1
	dns-nameservers 195.238.2.22, 195.238.2.21
	netmask 255.255.255.0
	wpa-driver wext
	wpa-conf managed
	wpa-ssid Gateway
	wpa-ap-scan 2
	wpa-proto RSN
	wpa-pairwise CCMP
	wpa-group TKIP
	wpa-key-mgmt WPA-PSK
	wpa-psk &lt;the key&gt;
</pre>

The wpa-psk key is generated by this command: 

<pre class="brush: shell;">
	wpa_passphrase &lt;your_essid&gt; &lt;your_ascii_key&gt;
</pre>

With the command:

<pre class="brush: shell;">iwlist scan</pre>

You should be able to find out the information you need from the AP you want to connect to. After that a network restart enabled my wireless on startup.

<a name="sshd">SSHD:</a>

<pre class="brush: shell;">
	sudo apt-get install sshd
	vi /etc/ssh/sshd_config
</pre>

And add this:

<pre class="brush: shell;">
	# What ports, IPs and protocols we listen forPort 22
	Port 443
	GatewayPorts clientspecified
</pre>

The last option will allow the client to specify target ports to which ssh should forward packets to (by default the sshd can only forward to the host it is running on. To forward to other targets, you need the 'GatewayPorts' as mentioned above).

Ok! The only thing left was the accessibility. I could port map the RDP port from the XP VM directly to the Internet via the router. I could say that 3389 should be port forwarded to 192.168.0.3. However, I only wanted to open one port (besides HTTP) to the Internet, and preferably I want to shield the windows VM completely from the Internet. Also, as I want to access my services from everywhere, sometimes places just give you Internet access via an http proxy. From those places I would not be able to connect directly to the RDP service. To solve this, I tunnel everything I need over SSH. My server exposes only 3 ports: 
<ul>
	<li>22 (SSH)</li>
	<li>80 (http)</li>
	<li>443 (SSH)</li>
</ul>

The router is configured to port forward TCP 22,80 and 443 to the host OS on 192.168.0.2. Instead of running an HTTP SSL acceptor on 443 I configured SSHD to listen on two ports simultaneously; 443 and 22, so its not a typo. When I'm connecting from a remote location (me, being the client) I just need putty. Putty can be configured to connect directly (using port 22) and make a forward tunnel. Doing this I can choose a local port on the client that maps to a port on the target. Even better, I can map it to a port on any target, local network, or even back to the Internet. So, to connect to RDP, I need a tunnel mapping from 

<pre class="brush: shell;">
	&lt;any local port&gt; : 192.168.0.3 : 3389
</pre>

Remember: the Internet does not know (route) 192.168.0.3. But this address is valid in the 'tunnel' that putty sets up. Putty first establishes a connection to ssh.mydomain.be and over that connection it is making a connection to 192.168.0.3 , so requests to 192.168.0.3 are send to the sshd, which then delivers them to the local network.  The total pictures looks like: 

<ul>
	<li>I let putty connect to ssh.mydomain.be at port 22 (or 443, see below)</li>
	<li>Putty creates a forward tunnel from localhost:xxxx (over 192.168.0.2) to 192.168.0.3:3389 via the tunnel (xxxx is 4000 in the screenshot below).</li>
</ul>
In putty that looks like this:   

<table>
	<tr> 
		<td>
			<div class="separator" style="clear: both; text-align: center;">
<a href="{{site.baseurl}}/assets/img/2011-03-29-personal-server/putty1.png" imageanchor="1" style="margin-left: 1em; margin-right: 1em;"><img border="0" data-original-height="558" data-original-width="1090" height="124" src="{{site.baseurl}}/assets/img/2011-03-29-personal-server/putty1.png" width="440" /></a></div>
		</td>
		<td>
<div class="separator" style="clear: both; text-align: center;">
<a href="{{site.baseurl}}/assets/img/2011-03-29-personal-server/putty2.png" imageanchor="1" style="margin-left: 1em; margin-right: 1em;"><img border="0" data-original-height="558" data-original-width="1090" height="124" src="{{site.baseurl}}/assets/img/2011-03-29-personal-server/putty2.png" width="440" /></a></div>
		</td>
	</tr>
</table>

On the client computer from which I'm connecting from, I point my RDP client to localhost:4000 and the connection is established. The nice thing about all of this, is that there is an option in putty to tunnel my tunnel over an http proxy. So if I'm not allowed to go out on 22 directly, I configure putty to talk to the proxy to send out my packets. If you enable HTTP proxying on putty, putty sends an HTTP CONNECT &lt;targethost:port&gt; to the proxy. The proxy will then tunnel your request further to the target.

However, sometimes proxies disallow to tunnel to a target port as '22(ssh)'. An easy trick is spawning the sshd on a second, SSL/TLS port 443. The HTTP handshake for a SSL/TLS connection is the same as it is for a proxied SSH request (its also tunneled by HTTP CONNECT). So actually the proxy thinks you are going to SSL (because of the port), but you are not. To do this, just let putty connect to port 443 instead of 22 (as shown in the first image above). Next you have to tell putty that it should use a proxy:  

<div class="separator" style="clear: both; text-align: center;">
<a href="{{site.baseurl}}/assets/img/2011-03-29-personal-server/putty3.png" imageanchor="1" style="margin-left: 1em; margin-right: 1em;"><img border="0" data-original-height="558" data-original-width="1090" height="124" src="{{site.baseurl}}/assets/img/2011-03-29-personal-server/putty3.png" width="440" /></a></div>

Before declaring me foobar (again), I'll give some clarifications at this point:   

<ul>
	<li>What I'm doing here is building a "poor man's" VPN to a certain extend. However, VPNs work with different protocols, requires a VPN client (and a VPN server). It also requires the network infrastructure to 'allow' to setup a VPN. So suppose you want to connect your (own) office with your home network, a real VPN would definitely be better and more scalable. However, this setup needs to work as lightweight as possible on any type of client (possibly not  managed by me) and preferably on any type of network.</li>
	<li>A really tightend up proxy will disover fast (even if you do it over 443) that you are not SSL/TLS-ing. In that case I could still add an extra module which will tunnel my ssh tunnel in an ssl tunnel tunneled over the proxy. By doing that the proxy is not able to distinguish your session from a SSL/TLS that for example has been started by a clients browser. Since I did not yet meet such tightned proxies, and this setup  would require some additional client software as well, I will leave it like this until its really necessary some day.</li>
</ul>
