---
layout: post
title:  "Concurrent collections in .NET 4.0"
date:   2009-06-14 19:59:34
categories: C# Concurrency NET
---

I did some investigation of the new System.Collections.Concurrent namespace today, because I considered using them for my private server project. I know collections are not the place to put synchronization, but in my use case I believe it’s justified, as it is the classic producer/consumer pattern.

My custom server is built to handle a lot of concurrent tcp/ip connections and receive and send data constantly.

It will also likely run on machines with at least 8 cores, possibly a lot more. Thus I need a collection ( mostly a simple queue ) that is high performance and scales well with the number of physical cores. However after looking at the implementation of ConcurrentQueue in the .NET 4.0 Beta1 release I had to rule them out. While certainly a very clever piece of code, they still rely on the the dreaded monitor and spinlocks.

Therefore as soon as you go over the 4 core mark ( depending CPU architecture of course ) they will not scale well. This is of course no problem for software that just uses them for a little amount of data, but in my server they are under high load all the time.

So for now I still have to use my own queue that uses atomic cpu operations for doing implicit synchronization. While this lock free queue implementation certainly has worse performance, it scales very nicely on more than 4 cores under high load.
