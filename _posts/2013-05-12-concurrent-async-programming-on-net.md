---
layout: post
title:  "Concurrent/Async programming on .NET"
date:   2013-05-12 19:59:34
categories: C# Concurrency Async
---

By popular request, here is the course material for a course I held at various companies, introducing developers to the world of concurrent and asynchronous programming on the .NET platform. This covers topics like basic CPU operation principles, OS threading basics, optimization topics for parallel processing and of course asynchronous patterns in order to improve scalability or responsiveness of applications. We start how these topics are handled in NET 1.0 and later refined in subsequent versions of .NET up to async/await keyword usage in NET 4.5. Some optional libraries ( Reactive Extensions, TPL Dataflow ) are also discussed.

In addition to the presentation itself, about 80 code samples are included to illustrate various usage patterns.

Download the presentation material here: Presentation & Code Samples

Note: While immutable collections are discussed, the recent BCL Team authored immutable collection library is not mentioned since the presentation is over a year old and interested parties should look at the bcl team blog for information about this: BCL Team Immutable collections library
