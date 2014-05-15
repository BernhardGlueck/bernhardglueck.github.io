---
layout: post
title:  "Lightmoon"
date:   2011-01-10 19:59:34
categories: C# Concurrency Actors NET
---

The last year I tried to develop Web Push Technology for .NET ( Comet Style Http Push + Web Sockets for more modern browsers ). While the actual communication technology was relatively easy to implement, I ended up in a world where my precious web application had to suddenly keep state around about virtual connections or queues in order to deliver the correct data to the correct clients. This ended in a world of hurt because there was no ready made solution for this kind of problem. While certainly one could develop such a specific solution, I started to think further. In the modern world of application development be it in the web space or in backend development, we are taught to make our applications stateless. However in reality most of them are stateful and we only push the state as far back as possible ( usually the Database ) and use mostly stateless parts elsewhere. This essentially is a trick to get parts of our infrastructure to scale easier ( or cheaper ) while the costs for databases skyrocket ( albeit only if you hit a certain amount of data ) in terms of hardware/software and personnel . It also makes the lives of developers a lot harder since we constantly have to use tricks and workarounds to hide the fact that in fact we have state that we would like to access. Be it clever data access strategies, http cookies or Session State wrappers. Wouldn’t it be easier to just not have to think about it, and follow one simple programming model that comes natural to the average OOP developer ( POCOs ) ? Enter LightMoon, an application middleware I have started developing that essentially provides these features, and while doing so does not sacrifice scalability. This technology could be used in a lot of domains, not the least the development of MMO style games ( where my inspiration originally came from ). I plan to release a technology preview sometime in May 2011 ( I know still some time off ) and later on make it a commercial product that is dual licensed under the GPL and a commercial one. Following is a short list of features that will be supported in the initial CTP:

* Simple programming model of disconnected actors that communicate via message passing similar to what Erlang/Scala provide
* Lightweight views on actors ( auto generated from the implementation code, initially for JavaScript via HTTP communication and C# via TCP/IP )
* Support for a huge number of simultaneously connected clients.
* Transparent persistence
* Transparent concurrency
* Easy horizontal scalability

The ideas i follow are not new, there is just no good implementation of them for .NET so far with modern technology backends.
