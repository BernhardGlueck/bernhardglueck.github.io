---
layout: post
title:  "Interesting benchmark results"
date:   2010-03-20 19:59:34
categories: NET Java C# Performance
---

I recently had some discussions about .NET performance, JIT generated code performance in general, and the applicability of managed code platforms to “number crunching” domains, like scientific computing or game development. After some research I found that there are nearly no current benchmark results posted anywhere on up to date versions of different runtimes/platforms. So I decided to do my own investigation. For the pretty narrow domain of “number crunching” I choose the well known SciMark2 in its vanilla Java and Ansi C version, as well as it’s C# port that lives in the pnetmark project ( original version from Cornell university ). Please keep in mind that this is no general benchmark that should be used in “my runtime is faster than yours” contests, but a narrow view on some performance characteristics for the above mentioned problem domains.

The test machine was: Dual CPU i7 920 (8 physical, 16 virtual cores), 16 GB 1600 MHZ DDR3 Ram, Windows 7 Ultimate 64 bit. ( Note that scimark only uses one thread at the moment so it’s a single core benchmark ) The test contenders:

NET 4.0 RC1
Visual C 2010
Java 1.6.18
Mono 2.6.1 ( default JIT )
Mono 2.6.1 ( LLVM as JIT )
The binaries for each platform were built using maximum optimizations ( Link Time Code Generation, SSE2 and Maximize Speed for Visual C ) and subsequently run 10 times each, and the results averaged. Here are the results ( sorted from best to worst )

Visual C 2010 925 MFLOPS
Mono 2.6.1 (LLVM) 602 MFLOPS
Java 1.6.18 527 MFLOPS
NET 4 RC1 525 MFLOPS
Mono 2.6.1 420 MFLOPS
Conclusion: Basically Java and .NET offer the same performance, Mono with LLVM seems to be the fastest managed environment and AOT compilers still beat JIT compilers by a big margin. Roughly switching from Native code to Managed code will cost you 45 % performance. If that is worth it for you depends on your needs of course. Again please note that normal application workloads where a lot of other stuff like allocations, branching etc occur will show less of a performance difference since the pure number crunching is offset by other overhead. In the future i would like to test some other JIT based environments, maybe even some bytecode interpreters, and also try out Intel C++ as well as give Mono’s full AOT mode with maximum optimizations a try. Java also requires an upgrade to 1.7 prelease as soon as a real “beta” arrives.