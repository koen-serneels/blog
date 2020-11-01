---
layout: post
title: "Some Of My Mostly Used Development Mantras"
date: 2014-01-28T16:38:00.000+01:00
author: Koen Serneels
img: 2014-01-28-some-of-my-mostly-used-development/background.jpeg
tags: 
- Methodology
---

<b>Statement: I assume it works this way!</b><br/>
<b>Mantra: assumption is the mother of all fuckups</b>

<a href="{{site.baseurl}}/assets/img/2014-01-28-some-of-my-mostly-used-development/us2.jpeg" style="clear: right; float: right; margin-bottom: 1em; margin-left: 1em;"><img border="0" data-original-height="1345" data-original-width="1600" height="168" src="{{site.baseurl}}/assets/img/2014-01-28-some-of-my-mostly-used-development/us2.jpeg" width="200"/></a>
<br/>
In this case his assumption had a rather lethal outcome which you might probably remember if you have seen the movie. But ok, being a mercenary, his situation might seem different from a day to day developer. So going further with this example, you might remember there was also a computer whiz on the train doing stuff like operating the satellite. Now imagine you're that guy and your asked to startup satellite communications. You quickly write some code and in doing so you write this line:
<pre class="brush: java;">
i = i++;
</pre>

Normally we would write something as 'i++' leaving out the assignment. However, the assignment seems redundant at most. We assume that it does not has any other side effect and we startup our program. But then suddenly....BOOM! Communication fail, Casey Rayback gets his ass kicked (actually I would love to have seen this), nukes are launched and world peace is far away. Our assumption had a rather negative impact.

Ok, let's see what this is about and make a small test. What do you expect it prints out, do you have any idea why it does so?

<pre class="brush: java;">
int i = 0;<br />i = i++;
System.out.println("i:"+ i);
</pre>

While you're thinking about this, let me explain what happens. As can be seen we have an expression, existing of a unary operator and an assignment. As discussed before the assignment is redundant. But besides being redundant it is also causing an unpleasant side effect. It makes this code behave differently then one would expect. If we drop the assignment (just leaving 'i++;') the code would exactly do what you think it would do.

What happens is that when using an assignment, the JVM will put the value of the operands on the so called operand stack before doing something with them. When the expression is evaluated, the value for making the assignment to the left hand variable is popped from the operand stack. That sneaky unary operator follows different rules. It uses the 'iinc' instruction. The iinc instruction takes as parameter a position in the local variable array of the current frame to put it's value.

So, what happens is that "0" is pushed on the local operand stack. Next the variable "i" is incremented by 1 because of the iinc instruction. At this time the local variable 'i' would actually have the value '1'. We can't show this in a normal way, since the assignment/expression is evaluated in a single step. When the assignment is made, it will pop the latest value from the operand stack assigning it to the variable 'i'. This means that 'i' will be overwritten with the value from the operand stack, changing it's value from 1 back to 0.

<i>Note: when you write: 'i = <b>++</b>i;' thinks will work as expected and the assignment is just redundant without side effects. In that case the iinc instruction is executed before pushing the local variable value to the operand stack. The rational is actually no different as when incrementing a variable in a method call; someMethod(++i) or someMethod(i++) . However, with an assignment it seems less clear that a pre or post unary makes any difference, because we tend to relate it to i++ or ++i on a single line where it absolutely makes no difference if the ++ is pre or post.</i>

In nearly every line of code (depending on experience and knowledge) we take assumptions. This works out most of the time because some of the assumptions will be correct. Incorrect assumptions do not necessarily lead to bugs as they might get balanced out by other assumptions. Or, maybe the side effects are never detected because of the boundaries in which our software operates.

A good way to track down incorrect assumptions in code is to write tests. But that alone is not enough. It is also important to accept this rule. No matter how brilliant you are, there is always (a lot) of internals you don't know or fully understand. One needs to remain skeptical, don't be lazy. Always try lookup things as much as possible. Even in the case when there is only a little bit of doubt: look it up and/or make small test case. It does cost time and energy, but the ROI (being able to write working and high quality code) will be well worth it.

<br/>
<b>Statement: don't worry, it's only temporary!</b><br/>
<b>Mantra: there is no such thing as temporary code</b>

<a href="{{site.baseurl}}/assets/img/2014-01-28-some-of-my-mostly-used-development/westvleteren.jpg" style="clear: left; float: left; margin-bottom: 1em; margin-right: 1em;"><img border="0" data-original-height="1345" data-original-width="1600" height="168" src="{{site.baseurl}}/assets/img/2014-01-28-some-of-my-mostly-used-development/westvleteren.jpg" width="200" /></a>

Agreed, a lot of things in life are temporary. The stock of Westvleteren 12 in my fridge is just one example. But in software projects, "temporary" tends to start a life on its own inflicting poorly designed or written code along its way. This makes sense if you think about it; I don't think anyone works with the same amount of energy or dedication if something is considered temporary. The problem is however that temporary code will become permanent before you even realize it.

An example I can share on this topic is a user management system which we designed for a previous project. The system was going to be a "temporary" user management system because the "permanent" system was delayed for some reason. I won't go into detail, but the name of the software needed to reflect that it was temporary. So it was named TUM (Temporary User Management system). And you guessed it; in the end the system existed for years to come and over time became the one and only UAM. After a while it even became hard to convince new colleagues that the "T" actually stood for "temporary".

I'm going to put this a bit harsh, but "temporary" helps no one and gives developers a wild card to justify bad decisions, laziness or incompetence. In such situations chances are high that the design will not be what could have been and the resulting code will be suffering quality issues. So the rule is simple: always design or write code in the assumption it will stick around for quite a while. It will be beneficial in the long run as nearly all code will, in one form or another.

But make no mistake friends, no matter how dedicated or how good you are, the dark side is always waiting around the corner. If you are being tempted, simply ask the question how you would feel if your car mechanic decided to fix your brakes "temporary" because he didn't have any idea at the time on how to fix them properly? Enough said I think.

Like most rules, there are exceptions. It would be naive to think that there are no real temporary software projects in general. Of course there are, and fundamentally it is no big issue if that would be the case. The important part is to keep this decision on project management level. Doing so one avoids infecting the team with it.

Don't forget that in the end you're never sure how temporary a project is going to be. And even if the project is really temporary, it's code might still find it's way to other modules or projects in the future.
