---
layout: post
title: "Home Automation (Part I)"
date: 2014-07-24T12:03:00.001+02:00
author: Koen Serneels
img: 2014-07-24-home-automation-part-i/background.jpeg
tags: 
- Home automation
---
 
Being a developer with background  in electronics, I sometimes miss the physical aspect of writing  software. I don't mean physical like touching other people or something,  no, more like writing code that does something physically. Like  operating a lamp or a motor. So, some time ago I saw my chances fit for a  new side project: home automation.

While  investigating what I would need, I Quickly became convinced that  besides programming an installing it myself, the biggest challenge would  be to get all bits and pieces together. There are a lot of systems and  vendors and it is hard to get some objective information, or just  information all together. So I decided to start sharing my experience  here. Maybe it might serve to some good for other brave souls crazy  enough to try the same thing for their homes.

But  first, lets start with a good old rant. Why o why is the world of home automation so badly organized? Let me come again; why? If you enter this world, set aside all open source principles. Do not hope that you are going to find centralized/useful documentation on how to do things.

<a href="{{site.baseurl}}/assets/img/2014-07-24-home-automation-part-i/shnitzel.jpg" imageanchor="1" style="clear: right; float: right; margin-bottom: 1em; margin-left: 1em; text-align: center;"><img border="0" data-original-height="1200" data-original-width="1200" height="200" src="{{site.baseurl}}/assets/img/2014-07-24-home-automation-part-i/shnitzel.jpg" width="200" /></a>

Imagine  going back in a time machine, where the Internet is not sometimes  referred to as the 'interwebs'. Where the Internet is still delivered  over slow landlines owned by evil telco's wanting nothing more then your  small fortune. Where marketeers did not yet understand that a platform  can be offered almost free of charge and that there are plenty of ways  to make money. Imagine a world where not everyone speaks German. German you say? Yes, get used to words as 'Schnittstelle' (not to be confused with the well known <a href="http://en.wikipedia.org/wiki/Schnitzel" target="_blank">Schnitzel</a>) or 'Zählerfernauslesesystem'.

Apparently a lot of these companies are German, and while most  documents are indeed also available in English, some are left  untranslated so you are stuck with the German version. Fact is that most  of these sites need to be browsed in English and a second time in  German locale to make sure all information surfaced. Some documents  simple don't appear otherwise. But it is what it is, so lets suck it up  and proceed.

As  with every project it all starts with the requirements. This time being  the customer myself, having bad, or no, requirements is not going to be  an excuse. But here also lies the danger. Home automation is very  subjective. Some people want their eggs to be backed automatically in  the morning, others just want a panic button. For me it is a subtle  mixture between what is doable, what is actually needed and the budget.  The must haves:

  * All light points in the house should be  able to be programmed and controlled via software. And with all lights I  mean all of them, no exceptions
  * Some power circuits needs to be controllable
  * Lights in hallway, toilet, bathroom,  garage, storage space, outside side door, outside house need to be  operated with motion and or presence sensors. These sensors also needs  to be cat proof. I don't want to turn my house into a blinking Christmas  tree if my cats decide to play at night
  * Buttons need RGB LEDs. If the toilet is  occupied I want to see a red light on the switch. Depending on the mode a  'sensor controlled light' is in, I want this being reflect by the LED.  The brightness of the LEDs needs to be adjusted as well depending on the  available light
  *Light sensors with timer functionality.  For example; the roof lights need to turn on when it's getting dark, but  only after 1900h. If it gets dark during daytime (bad weather or  something) the lights should not switch on
  * Scene support: lights in the living-room/kitchen need to be dimmed to preset scenes
  * Blinds for the windows need to be  controlled automatically.They need to be controlled together (the  default) but&nbsp; also individually
  * Ventilation. If the unit would be to loud at some point, it must be able to be controlled remotely, preferably by a phone
  * Support for programming over Ethernet
  * API for controlling the entire system via software. For example an app, or a dedicated touch panel
  * The interconnection of the components should be using a bus architecture with free topology

Nice to haves:

  * Heating. This is a bit of a gray zone, but it at least the possibilities needs to be investigated
  * Metrics for gas/electricity/water

For me this is where the  barrier between useful and affordable automation ends. Audio for  example, sure nice things are possible, but this is better controlled&nbsp;  separately. There is plenty of choice in media system with Spotify  integration at far better prices. The thing is that for audio one wants  to have freedom of choice and not be bound to a single (expensive)  product.

Next the hardware. Easier said then done, so first let's try to identify some categories

<b>PLCs</b>

<a href="{{site.baseurl}}/assets/img/2014-07-24-home-automation-part-i/S71500.jpg" imageanchor="1" style="clear: left; float: left; margin-bottom: 1em; margin-right: 1em; text-align: center;"><img border="0" data-original-height="1200" data-original-width="1200" height="200" src="{{site.baseurl}}/assets/img/2014-07-24-home-automation-part-i/S71500.jpg" width="200" /></a>

A barebone PLC, for example the Simatic S7 from Siemens. They are the  SAP of automation. You can build nearly anything with them and they are  very robust/stable. <br />However, the thing is that they are barebone. For example, programming  is done with STL, which is very low level, comparable to assembler. On the image below you can get an idea&nbsp; how this looks.

<a href="{{site.baseurl}}/assets/img/2014-07-24-home-automation-part-i/lst.gif" imageanchor="1" style="clear: right; float: right; margin-bottom: 1em; margin-left: 1em; text-align: center;"><img border="0" data-original-height="1200" data-original-width="1200" height="200" src="{{site.baseurl}}/assets/img/2014-07-24-home-automation-part-i/lst.gif" width="200" /></a>
Agreed,  you can also opt for a graphical way of programming, like LAD (ladder)  but it remains pretty low level. But that's not all. These things have a  high cost, which is understandable if you see in which kind of  factories they are being used.There are also other manufacturers which are S7 compliant, such as VIPA, but the price still remains high. But the biggest downside is probably that there are no ready build HOME  components. If you want a push button with LEDs, no problem, but then  you have to build it yourself. I like a challenge but this would be a  bit too much for me.

<b>Mini PLCs</b>

Siemens Logo! is such an example

<a href="{{site.baseurl}}/assets/img/2014-07-24-home-automation-part-i/logo.jpg" imageanchor="1" style="clear: left; float: left; margin-bottom: 1em; margin-left: 1em; text-align: center;"><img border="0" data-original-height="1200" data-original-width="1200" height="200" src="{{site.baseurl}}/assets/img/2014-07-24-home-automation-part-i/logo.jpg" width="200" /></a>

While  they are cheaper, they still suffer the same drawbacks as its bigger  brothers. There are also no real home components and having something  simple as LED feedback on a button (e.g. a 'occupied' LED) means you'll  have to run an extra wire and an need an additional output just to steer  this one LED.
While  there is profibus (really expensive, but ok) for Simatic, Logo! has no  bus system. This means that if you want 16 switches you'll at least need  17 wires running in your 'data' cable. Double that amount if you want  to use LEDs</div><br />Don't get me wrong; Logo! (or PLCs)  do have a place in home automation. First there is the ease of  installation. You don't need cross switches or wiring switches directly  to switching points as with a traditional installation. It will be well  suited for basic stuff like push buttons (instead traditional toggle  switches), programming panic buttons, simple motion detectors etc They  are also more favorable over contactors/relays as they dynamically  programmed and prices will be almost the same

<b>Home automation PLCs</b>

Then there are what I call 'home automation PLCs, one of them is the Loxone Miniserver

<a href="{{site.baseurl}}/assets/img/2014-07-24-home-automation-part-i/loxonems.png" imageanchor="1" style="clear: right; float: right; margin-bottom: 1em; margin-left: 1em; text-align: center;"><img border="0" data-original-height="1200" data-original-width="1200" height="200" src="{{site.baseurl}}/assets/img/2014-07-24-home-automation-part-i/loxonems.png" width="200" /></a>

It  is a CPU just like Logo! or Simatic, but more focused on home  automation out of the box. It comes shipped with a webserver and phone  app allowing instant remote access from your browser/phone. There are  also several already build components which you can use straight away.  And most of all, it comes with KNX/EIB support (more on that later). But  to use it as the main home automation solution it would suffer the same  disadvantages as the previous ones; limited intelligent components(°)  and no bus system.

(°) limited, but there are at least extensions like a an IP cam based intercom, RGBW controllers, wireless extension and more (<a href="http://www.loxone.com/enuk/products/extensions/extension.html%29" target="_blank">check it out</a>) these home components are not available with traditional PLCs

<b>Real home automation</b>

With  "real" I don't mean that these system would be more stable or mature,  not at all. But they are more home oriented and therefore a closer match  as to what one would need for home automation.

<a href="{{site.baseurl}}/assets/img/2014-07-24-home-automation-part-i/nhc_3.jpg" imageanchor="1" style="clear: left; float: left; margin-bottom: 1em; margin-right: 1em; text-align: center;"><img border="0" data-original-height="1200" data-original-width="1200" height="200" src="{{site.baseurl}}/assets/img/2014-07-24-home-automation-part-i/nhc_3.jpg" width="200" /></a>

To make it easy I divided them in the closed systems; such as <a href="http://www.niko.eu/enus/niko/products/niko-home-control/" target="_blank">Niko</a> (Belgian company) home control (NHC) <a href="http://www.bticino.com/" target="_blank">or BTicino</a> (Italian) etc. These are examples of a closed standard and only the manufacturer in question produces components for it.

<a href="{{site.baseurl}}/assets/img/2014-07-24-home-automation-part-i/basalte-deseo-new-1.jpg" imageanchor="1" style="clear: right; float: right; margin-bottom: 0em; margin-left: 1em; text-align: center;"><img border="0" data-original-height="1200" data-original-width="1200" height="200" src="{{site.baseurl}}/assets/img/2014-07-24-home-automation-part-i/basalte-deseo-new-1.jpg" width="200" /></a>

Then there are the open systems such as <a href="http://en.wikipedia.org/wiki/KNX_%28standard%29" target="_blank">KNX</a>/<a href="http://en.wikipedia.org/wiki/European_Installation_Bus" target="_blank">EIB</a>. The latter is an open standard and every one is free to build  components compatible with this standard. This means that you don't have  a vendor lock-in; you can mix different brands and components, as long  as they are KNX/EIB compatible. For example, one can have a <a href="http://www.mdt.de/EN_Push_Buttons.html" target="_blank">MDT pushbutton</a> combined with a <a href="http://www.jung.de/en/online-catalogue/69799111/" target="_blank">Jung dim actuator</a> and a <a href="http://www.basalte.be/en/switches" target="_blank">Basalte thermostat</a>. Just as NHC, KNX uses a bus (EIB) with a free topology.

Ok, then what is the best solution? There is no objective answer in which  is better. It basically depends on budget, taste and mindset. For  example; vendor lock-in is mostly not a good idea. But if you chose a  good vendor that "has it all" it can also have advantages. If you go to  your local Niko dealer, someone can probably explain all components. You  get everything (information, ordering, support) from a single place. No  need to endlessly browse the Internet comparing manufacturers or  products. On the other hand a closed standard might impose future risks:  what if Niko decides to stop producing NHC? They already did that with  it's predecessor: Niko bus. Agreed, they solved it with NHC/NB bridge  which allows to add NHC components to an existing NB installation. But  not all functionality is usable this way.&nbsp; So if you bought Niko bus  some years ago you are already limited in adding new components.

KNX  excels in the fact that it has no central processor (unlike PLCs). It  is message driven and each sensor/actor has it's own processor. They  communicate with each other using telegrams (~messages). A sensor or an  actor generates can subscribe to a so called 'group address' (like a  multicast address) and it will then be able to pick up telegrams sent to  that address by other sensors or actors. Based on the configuration  they can decide to do something with the telegram or ignore it. But  there lies also it's weak point: each  component is responsible for generating or processing telegrams, but it is  limited to the logic offered by that component. For example; a light  point needs to be switch on conditionally based on the value of a light  sensor and the wall-clock time. This clearly involves clock logic. While  perfectly possible with KNX, you'll need a separate actor which is able  to generate telegrams based on clock schedules. This starts to become a  pricing issue as each KNX actor is a processor on its own making them  not cheap.

<a href="{{site.baseurl}}/assets/img/2014-07-24-home-automation-part-i/gira-home-server-1.jpg" imageanchor="1" style="clear: left; float: left; margin-bottom: 1em; margin-right: 1em; text-align: center;"><img border="0" data-original-height="1200" data-original-width="1200" height="200" src="{{site.baseurl}}/assets/img/2014-07-24-home-automation-part-i/gira-home-server-1.jpg" width="200" /></a>

But there is also a solution. The so called KNX/EIB 'servers'. They allow more advanced and centralized programming and form an extension to the traditional KNX sensors and actuators. An example of this is Gira's <a href="http://www4.gira.com/en/gebaeudetechnik/produkte/neuheiten/homeserver-update.html" target="_blank">home server</a>. However, it's price tag is really, really high  at €2.200. Luckily,  this is where Loxone comes in. Loxone offers the same functionality but  at a reasonable price. It can even do more as it is a traditional CPU  with KNX/EIB integration. To program your KNX components via Ethernet,  you also need an IP gateway. This is again an extra (special) actor.  With Loxone you get all of that combined in a single module. So by  adding Loxone we get follow things in return:

  * A 'soft entry' into home automation system. It allows for remote  access via an API and apps thanks to it's built in webserver. Since it's  KNX/EIB compliant, Loxone can be used as the programmatic interface to  EIB, not needing extra KNX components
  * KNX/EIB gateway; using Loxone one can program KNX/EIB components using it as a gateway
  * The inputs on the Loxone are not really usable when using EIB.  However, the outputs are. By using them you are also saving a 8 digital  output KNX actor
  * Actual programming with logic circuits, controllers, smart components, you can even program in pico C if that would be required

A KNX gateway and  8DO KNX actor already cost more then the entire Loxone setup. So it's a  win-win deal: less money for more functionality.

So in the end I decided that an open system (such as KNX/EIB) in  combination with a home automation PLC (Loxone Miniserver) combines the  best of two worlds. KNX/EIB offers an open standard and freedom of  choice in vendors and components. It also decentralized, message driven  and open ended architecture which I felt comfortable with. Loxone adds  the central processing power and interface to other platforms  (web/mobile). It also allows to program all logic you would even need  for your home automation at a reasonable price.

There  is however one thing to keep in mind. While Loxone Miniserver is  KNX/EIB compliant, it cannot be programmed like other KNX components. In  other words, Loxone is programmed separately with different software  and different 'programming techniques'. But this actually makes sense. The way a KNX/EIB is programmed does not lend itself for complex logic.  After all, KNX/EIB programming is more 'configuring'. The Loxone development environment is better suited for this, taking full  advantage of its possibilities. Do note that a Gira Home server for  example is also programmed in a different way than traditional KNX/EIB components, so this is just  natural.

In  the next part I'll show the components I'm planning to use to meet the  requirements and the test board together with the initial programming I  did to be able to test everything from my desk.
