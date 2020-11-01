---
layout: post
title: Thermal shutdown
date: '2013-02-02T20:08:00.000+01:00'
author: Koen Serneels
img: 2.png
tags:
- System
modified_time: '2013-02-03T10:59:46.561+01:00'
thumbnail: http://4.bp.blogspot.com/-_tV1mcSNhDI/UQ1dFdY7UuI/AAAAAAAAAR8/rwgNm7ZZWOs/s72-c/IMG_4442.JPG
blogger_id: tag:blogger.com,1999:blog-3406746278542965235.post-724665278871147230
blogger_orig_url: http://koenserneels.blogspot.com/2013/02/thermal-shutdown.html
---

While running a CPU intensive process last week my laptop decided to instantly power off on me. Since the battery was not installed I thought it was probably a power glitch, so I turned it back on and restarted the process. After a few minutes it powered off again, so far for the power glitch.

Thinking it might be temperature related I installed sensors and Computer Temperature Monitor for Ubuntu. As it turned out my CPU temperature was pretty high. When the process started it was around 90°C, slowly but steadily progressing towards 100°C. After that it was a matter of seconds before it powered off. Google confirmed my suspicion that 100°C is indeed very high for a laptop core. Running the same process on another comparable laptop ran a whopping 20°C cooler. It was now for sure that it was a thermal shutdown.

Eliminating the easiest thing first seemed a sane thing to do; for me this would be any influence imposed by software. First I booted Windows running the same process there again and gathering temperature readings using Sisoft Sandra. The temperature matched with those gathered in Ubuntu, powering of the laptop again after some minutes. My final attempt was to update the BIOS but again without any noticeable differences.

On a side note; the laptop was docked on a docking station when performing these tests.  The docking station allows even more distance between the air-intake (located on the bottom side of the laptop) and the surface. Room temperature was normal and the (one and only) fan was also running and speeding up normally (as far as my audiovisual inspection powers could tell). Giving these standard conditions, and the fact that I was not even using everything (the GPU for example) a thermal shutdown was certainly not to be expected. Next I started out with the hardware. Again the easiest thing first: clean the air vents. Got myself a can with compressed air to clean the vents:

<div class="separator" style="clear: both; text-align: center;">
<a href="{{site.baseurl}}/assets/img/2013-02-02-thermal-shutdown/IMG_4442-large.jpg" imageanchor="1" style="margin-left: 1em; margin-right: 1em;"><img border="0" src="{{site.baseurl}}/assets/img/2013-02-02-thermal-shutdown/IMG_4442-small.jpg"/></a></div>

There was some dust, but not much. As expected this did not solve the heating problem. I repeated the same procedure, but this time with the keyboard and switch/led cover removed for easier access:  

<div class="separator" style="clear: both; text-align: center;">
<a href="{{site.baseurl}}/assets/img/2013-02-02-thermal-shutdown/IMG_4447-large.jpg" imageanchor="1" style="margin-left: 1em; margin-right: 1em;"><img border="0" src="{{site.baseurl}}/assets/img/2013-02-02-thermal-shutdown/IMG_4447-small.jpg"/></a></div>

Next I emptied some air capsules used for inflating the tires of my MTB on the fan and vents. They produce a pretty high amount of air displacement:-) 

<div class="separator" style="clear: both; text-align: center;">
<a href="{{site.baseurl}}/assets/img/2013-02-02-thermal-shutdown/aircan-large.jpg" imageanchor="1" style="margin-left: 1em; margin-right: 1em;"><img border="0" src="{{site.baseurl}}/assets/img/2013-02-02-thermal-shutdown/aircan-small.jpg"/></a></div>

After everything was defrosted again the vents were now clean for sure. But no improvements on the horizon for this skipper!

My next idea was to decided if the fan was behaving properly. Easier said then done, since I was unable to get any RPM readings. ACPI, closed BIOSes and the likes are apparently withholding me from getting that information. My 15 year old self build desktop had RPM readings of the CPU fan, GPU fan and every case fan. But as it seems, we no longer need health indication of our fans these days. The closest thing I was able to do was comparing the air displacement of the fan with the fan of my comparable laptop.

<div class="separator" style="clear: both; text-align: center;">
<a href="{{site.baseurl}}/assets/img/2013-02-02-thermal-shutdown/IMG_4455-large.jpg" imageanchor="1" style="margin-left: 1em; margin-right: 1em;"><img border="0" src="{{site.baseurl}}/assets/img/2013-02-02-thermal-shutdown/IMG_4455-small.jpg"/></a></div>

As it turned out, on top speed the air displacement was similar between the two laptops. This gave me a satisfactoriness indication that the fan/fan-controller were OK. All other inspections turned out to be OK as well; heat sinks were still fixed and heating up properly, so I was running out of ideas.

Getting annoyed I googled some more and stumbled across a blog about installing a custom heat sink for my model and the steps it involved. One of those steps is as most of you know applying thermal paste between the core and the heat sink. Thermal paste helps in pushing out remaining air between core and heat sink for optimal heat transfer. As a fan issue was now excluded, it could well be that the heat was perhaps not even properly being transfered to be vented out... I also realized that the temperature fluctuations of the CPU were indeed a bit strange. Idle it was around 40°C, but when the process started the temperature went up so fast that the fan lagged behind some seconds before shifting to top speed. When the fan finally reached its top speed the temperature increase was slowed down, but in the mean time the CPU had already ~90°C. 

I decided to renew the thermal paste as a final act of desperation. Again easier said then done, at least for me, since the heat sink screws are torx, dammit! So, first to the home depot buying some torx equipment, and then to the computer shop getting some silver based thermal paste. 

<div class="separator" style="clear: both; text-align: center;">
<a href="{{site.baseurl}}/assets/img/2013-02-02-thermal-shutdown/IMG_4446-large.jpg" imageanchor="1" style="margin-left: 1em; margin-right: 1em;"><img border="0" src="{{site.baseurl}}/assets/img/2013-02-02-thermal-shutdown/IMG_4446-small.jpg"/></a></div>

When I removed the heat sink I saw that they used ceramic based paste and that it was hardened out. A good indication I could be on to something since its not supposed to be hardened out, and certainly not on a only 1.5 year old laptop. Cleaned the paste of first:  

<div class="separator" style="clear: both; text-align: center;">
<a href="{{site.baseurl}}/assets/img/2013-02-02-thermal-shutdown/IMG_4448-large.jpg" imageanchor="1" style="margin-left: 1em; margin-right: 1em;"><img border="0" src="{{site.baseurl}}/assets/img/2013-02-02-thermal-shutdown/IMG_4448-small.jpg"/></a></div>

Added new paste by putting a tiny drop in the middle on each IC surface. This is enough as when you install the heatsink it will (hopefully) spread evenly over its surface. To verify this I installed the heat sink, tightened the screws and then removed it again. In my case the surface is covered without the paste going over it, so that is what we want: 

<div class="separator" style="clear: both; text-align: center;">
<a href="{{site.baseurl}}/assets/img/2013-02-02-thermal-shutdown/IMG_4450-large.jpg" imageanchor="1" style="margin-left: 1em; margin-right: 1em;"><img border="0" src="{{site.baseurl}}/assets/img/2013-02-02-thermal-shutdown/IMG_4450-small.jpg"/></a></div>

As it turned out I was lucky this time, after applying the new thermal paste the CPU temperature was now steady at around 80°C when on full load. Nice. 

<div class="separator" style="clear: both; text-align: center;">
<a href="{{site.baseurl}}/assets/img/2013-02-02-thermal-shutdown/Screenshot-large.png" imageanchor="1" style="margin-left: 1em; margin-right: 1em;"><img border="0" src="{{site.baseurl}}/assets/img/2013-02-02-thermal-shutdown/Screenshot-small.png"/></a></div>
