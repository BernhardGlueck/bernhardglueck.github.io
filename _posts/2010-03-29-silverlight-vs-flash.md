---
layout: post
title:  "Silverlight : Flash = 1:0"
date:   2010-03-29 19:59:34
categories: RIA Flash Silverlight
---

Maybe a bit of a strong title, but I have made no secret of my opinion that I think IF there is a need for a “Rich Internet Application” technology in your project and you have to use something more than HTML5/JavaScript then Silverlight is the best choice. ( That is if you plan to write some actual code with it, not just embed a video ) My main reasoning is that Flash in my opinion is still a graphics designer tool that has some half baked version of ECMAScript tacked onto it ( AS3 ) while Silverlight is a developer tool that has additional rich support for design/graphics work.

One big thing I recently found out ( I have to admit my AS3 experience is limited to helping out others debug some nasty problems with it, otherwise I don’t touch it with a 10 foot pole ) is that the Flash Player 10 garbage collector is like a government employee ….. lazy to the max.

Normally a garbage collector keeps a list of “root” objects and maintains a graph of dependent objects for them, in order to figure out what memory can be reclaimed when it is not accessible to user code anymore. Turns out when the Flash garbage collector sees a particularly large subgraph of objects that would take some time to actually analyze … it just says “oh well that’s a lot of work, I am not in the mood, I’ll just say it’s still used nobody will know …”. That’s right the Flash garbage collector stops analyzing larger graphs and just keeps them around, giving you huge memory leaks for free ! Now that’s a feature.