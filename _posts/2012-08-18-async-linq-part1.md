---
layout: post
title:  "Async LINQ part 1"
date:   2012-08-18 19:59:34
categories: C# LINQ Async
---

Update 20th August: I decided due to interest to actually implement the functionality described in this post series as an open source library. The repository is at: GitHub repository and the corresponding nuget package is at: Nuget package

Now that NET 4.5 has hit RTM and is available to MSDN Subscribers, I want to take a look at one of the functionality gaps that are present in this release. The major new feature in .NET 4.5 was the addition of the async/await syntax for developing asynchronous code in a much more concise way than was possible before.

However as maybe some of you have already noticed while there is PLINQ for concurrent LINQ to Objects query processing, "ALINQ" as I would call it for asynchronous queries is sadly absent in this release.

So I decided to implement this missing functionality myself, and I want to walk through it's design in this series of blog posts.

First let's define what ALINQ should provide for us:

* The ability to define asynchronously enumerable sequences
* Async iteration over those sequences
* Common query operators well know from LINQ that also allow us to specify asynchronous selector/predicate functions

In the end we want to be able to execute something like this:

{% highlight csharp linenos %}

AsyncEnumerable.Range(0,100)
    .Select( async i => await DownloadSomething(i) )
    .ForEach( async r => await Output.WriteAsync(r) );

{% endhighlight %}

Let's start with the first building block to achieve something like this. Sequences in .NET are modeled with the IEnumerable interface, but behave always strictly synchronously. We need an equivalent that works asynchronously. ( One could argue that IObservable models this concept, but that is actually not the case since IObservable follows push semantics while we still want to have pull semantics, just working asynchronously )

IEnumerable is defined as follows:

{% highlight csharp linenos %}

public interface IEnumerable<out T> : IEnumerable
{
     IEnumerator<T> GetEnumerator();
}

{% endhighlight %}

Basically .NET defines a stateful cursor concept for sequences encapsulated in the IEnumerator interface that GetEnumerator returns. This abstraction allows multiple enumerations of a sequence to be processed in parallel since the cursor is seperate from the actual underlying sequence.

IEnumerator is defined as:

{% highlight csharp linenos %}

public interface IEnumerator<T> : IEnumerator
{
   T Current { get; }
   bool MoveNext();
   void Reset();
}

{% endhighlight %}

Quite simple actually, MoveNext moves to the next position in the sequence, returning if the move was successful and data is available at that position, Current returns the current item in the sequence and Reset should reposition the enumerator to the beginning of the sequence again.

There is one caveat to note: Reset is basically deprecated since nobody made use of it, and in fact yield based Enumerators throw a NotImplementedException when it is called.

How would we now model this to support async enumerations ?

{% highlight csharp linenos %}

public interface IAsyncEnumerable<out T> : IAsyncEnumerable
{
    IAsyncEnumerator<T> GetEnumerator();
}

{% endhighlight %}

This is basically the same as the standard IEnumerable, we just return a different kind of enumerator. It get's more interesting with the definition of the actual IAsyncEnumerator definition

{% highlight csharp linenos %}

public interface IAsyncEnumerator<T> : IAsyncEnumerator
{
     T Current { get; }
     Task<bool> MoveNext();
}

{% endhighlight %}

As we can see we made the MoveNext method asynchronous and removed the Reset method simply because of the reasons stated before. Current works the same as in the synchronous IEnumerator interface. We now have a sequence that we can get a cursor where the actual move to the next position can be performed asynchronously.

I left out the explanation of the non generic interfaces on purpose, since they don't help to explain the concept and just follow the same patterns anyway. Here is the code so far in full:

{% highlight csharp linenos %}

public interface IAsyncEnumerator
{
    object      Current { get; }
    Task<bool>  MoveNext();
}

public interface IAsyncEnumerator<out T> : IAsyncEnumerator
{
    new T       Current { get; }
}

public interface IAsyncEnumerable
{
    IAsyncEnumerator GetAsyncEnumerator();
}

public interface IAsyncEnumerable<out T> : IAsyncEnumerable
{
    new IAsyncEnumerator<T> GetAsyncEnumerator();
}

{% endhighlight %}

In the next post we will look at how to produce such sequences/cursors in a convenient way.



