---
layout: post
title: Oracle Please Stop Shipping Ask.com Toolbar
date: '2013-11-15T12:28:00.001+01:00'
author: Koen Serneels
img: 2.png
tags: 
- Java SE
---

Lately I configured a JRE on some windows machine. It was not a developer machine and the goal was to run some Java, so I installed the JRE. Normally I download everything from oracle.com, but I was lazily this time and google presented java.com as the first result. I never used java.com. I know Java is from Oracle, but the paranoia routine in my brain has not yet white-listed this site as having something to do with Oracle and as such I refuse to associate it with them. But hey as I said, I was lazy and the site seemed legit, so I started downloading.

While going through the install wizard, I suddenly got this screen:

<div class="separator" style="clear: both; text-align: center;"><a class="image-link" href="{{site.baseurl}}/assets/img/2013-11-15-oracle-please-stop-shipping-ask.com-toolbar/asktoolbar.png" imageanchor="1" style="margin-left: 1em; margin-right: 1em;"><img border="0" height="244" src="{{site.baseurl}}/assets/img/2013-11-15-oracle-please-stop-shipping-ask.com-toolbar/asktoolbar.png" width="540" /></a></div>

I was baffled to see this. The first thing that went through my mind was: s*** ! I must have downloaded a compromised installer! hAx0orz in my interwebs! So I immediately executed evasive maneuvers theta 3 for windows users: No time for checksums! Pulled out the power, ripped out ram and hdd and fried them instantly.

After I was calmed down I was curious in finding out which compromised installer I had downloaded. So I used a working machine to go out on the Internet. It turned out I was in for an even bigger surprise. It's just part of the official installer! <a href="http://www.java.com/en/download/faq/ask_toolbar.xml" target="_blank">http://www.java.com/en/download/faq/ask_toolbar.xml</a>

Even more, this is not even breaking news as it has been around there for quite some time (and I'm not the first one writing about this). It always escaped me since it's only in the JRE, and only in the one you download from java.com *sigh*. The JDK and the JRE exe's comming from oracle.com don't contain this crap. 

This is bad. Really bad. Yeah, ok, you can disable it on install, true, but that won't cut it. The fact that it is only in JRE installs via java.com makes it even worse. People working professionally with Java (requiring a JDK install instead) could see through this if it was in that type of installer (but then it would probably don't generate profit either). However, systems using JRE's are end user systems (or end user systems managed by other professionals). These are the people working with the applications we write. If these people starts to associate Java with crapware then we have a problem.

It might well be that this was a decision taken by SUN and that Oracle now has to live with the contract, but then it's time they end it. This should stop as it's damaging the reputation of Java all together! I kindly invite you in joining me signing this petition, at least if you also agree this should stop: <a href="https://www.change.org/petitions/oracle-corporation-stop-bundling-ask-toolbar-with-the-java-installer" target="_blank">https://www.change.org/petitions/oracle-corporation-stop-bundling-ask-toolbar-with-the-java-installer</a>
