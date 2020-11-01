---
layout: post
title: "Turning On The Light In 2020"
date: 2020-02-16T00:34:00.001+02:00
author: Koen Serneels
img: 2020-02-16-turning-on-the-light-in-2020/background.jpeg
published: false
tags: 
- Home automation
---

Time had come to replace the power hungry (60watt) light bulb in my sons room with a less power consuming LED (standard E24 fitting). The following smart bulb drew my attention: https://www.amazon.de/gp/
product/B07RR1BYF7/. Non-smart white LEDs go for around 8-10euro, but I thought it would be cool to have colours for a couple of euro's extra. There was one important requirement: it needs to work when power is turned on/off without the use of an app. According to the manual this should be no problem, and indeed, after installation the LED lighted up after turning on the power. What the manual didn't say was that the standard color is blue-white radiation-level intense. Being a good choice for example a surgery room, it isn't so quite for a childs room. No big deal of course, after connecting the unit to WiFi the light could easily be configured to "warm white" via the standard app. After toggling the power a couple of times it was also confirmed the value remained "saved".

What immediately drew my attention when doing this however, was the LED WiFi initalization/reset sequence. The unit has no physical 'reset' switch. It is placed into init mode by toggling power in a pre-defined sequence. The sequence for this unit is rather simple: turning it on/off a couple of times (there are however units/versions with pretty insane sequences: https://youtu.be/1BB6wj6RyKo?t=15). Reset means that it first goes to init mode: starts an accesspoint (AP) and the LED starts blinking, waiting for the WiFi details to be configured. If the power is cycled one more time it will go to factory default: blue-white radiation-level intens.

At this point fear started building up. In case you don't know, 5y olds are the perfect stress and stability testers. Give them something and if it survives their thorn, you're sure you can trust your life with it. So that night after my son went to bed my fears were confirmed. 5 minutes later he came back saying the light was acting weird and that it "suddenly" started blinking and later on it became "very bright", asking if I could make it normal again. Damnit! This was suposed to be easy.

Oh well. Time for plan B. The idea is as follows: currently the light swith (KNX) is operating a power actor (KNX) turning power on and off. The switch could be re-configured (going via Loxone or raspberry) to operate the soft relay of the LED unit to turn the light off (like it can be done via the official app). Of course for this to work integration with the with the LED had to be possible first. Then, in paralel, a time trigger could be activated to turn off the KNX actor. If the LED is toggled rapidly, the trigger would be reset each time meaning the KNX actor would remain in the 'on' state. The unit would in that case only be soft toggled via it's internal relay and thus not going into init mode. The KNX actor would only go to the 'off' state when the light wasn't operated outside of the 'init mode time window'. Btw; turning off the power completely is not a must, but I don't like having a light bulb connected to my wifi when not in use. 

The million dolar question: what would be the easiest way to integrate with the unit and have full API control over it? Just as I was entering my favorite command (sudo wireshark) I found out that a lot of these units share a common platform created by "tuya" (being an IoT turnkey solution) running on an ESP8266. And as it goes, these units can be flashed OTA with "alternative" firmware such as https://tasmota.github.io/docs/#/Home. Great! As I didn't know about this beforehand at all, a quick check on the supported devices page showed that that this particular unit is supported https://templates.blakadder.com/teckin_SB53.html so this was a very pleasant surpise, thanks community!

The 'normal' installation procedure for Tasmota requires some wire soldering on the ESP to get a serial connection, but thanks to https://github.com/ct-Open-Source/tuya-convert you first flash some kind of bootloader (OTA!) and then it can load any supported firmware such as tasmota. So there I was sitting on my sons bed with a laptop, staring anxiously at a light bulb waiting untill it power cycled after having it flashed. Waiting untill I could access its "console" to load the acutal device settings and having it accessible via MQTT so that it finally could do what I want and go to sleep, thinking; why didn't I listen to my wife and simply put back the standard 60watt light bulb?

Luckily for me installing the bootloader worked out just as the manual described and it was a rather fast and hassle free experience. The bootloader will automatically install a 3th party firmware by default you can chose between tasmota and ESPurna, where I chose the first. After flashing the unit will reboort and Tasmota will put it into AP mode allowing another device to connect to it. Once you connect to it's wifi it will open a "sign in" page which the actual tasmota initial setting page:

This page allows to configure the actual AP tasmota will connect to after reboot. In case you run into problems, as of tasmota 7.something, they also implemented a power on/off reset sequence allowing the unit to be reset so you are able to repeat the process (the irony).

After entering the real AP settings tasmota reboots and shows the actual homepage:

Now it's matter of loading the correct template and reboot:

As of tasomate 8 a nice colorpicker is shown allowing to directly operate the unit from the page!

But what about communicating with Tasmota? Well, next to having a MQTT server Tasmota can also be configured as a client to an existing MQTT server. So, how to communicate from KNX with MQTT then? As it turns  out, this could be handled by Loxone. Loxone has a raspberry extension (Loxberry) which comes with an MQTT server (mosquito) and it also acts as a bridge between Loxone and MQTT allowing Loxone to send and receive MQTT messages. In case you don't know: loxone is a separate home automation system, but it's also KNX compatible. In my home automation setup I'm using Loxone as a KNX logic component.

The only problem with this is that LoxBerry is delivered as an image and requires a Raspberry for itself: dedicating a raspberry soley for LoxBerry is not something I want. Someone made an effort of dockerizing Loxberry but it seems rather outdated. So as long as LoxBerry does not support some dockerization on it's own or creates a less intrusive package this is not a real option for me.

The next idea was to look for some MQTT adapter middelware that could adapt http or udp into MQTT and vica versa, as http and udp are the only native possibilties on Loxone. This search turned out fruitless as well. So, to keep things simple I added an adapter myself in the Java home automation project:

It's pretty straight forward: it takes a udp packet containing channel name 


Now, this is cool. This is open. This made me happy :-)








<a href="{{site.baseurl}}/assets/img/2017-07-29-home-automation-heating-with-ebus-and-bulex/EBus_Logo2.png" style="clear: right; float: right; margin-bottom: 1em; margin-left: 1em;"><img border="0" data-original-height="1345" data-original-width="1600" height="168" src="{{site.baseurl}}/assets/img/2017-07-29-home-automation-heating-with-ebus-and-bulex/EBus_Logo2.png" width="200" /></a>


