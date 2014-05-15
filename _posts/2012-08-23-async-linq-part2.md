---
layout: post
title:  "Async LINQ part 2"
date:   2012-08-23 19:59:34
categories: C# LINQ Async
---

As i noted in my last post's update, i have decided to implement ASYNC Linq as an open source project, and today i reached the "feature complete" milestone.

All LINQ operators are supported, except "ToLookup" which I never used myself in traditional LINQ and consider a low priority. All operators are implemented asynchronously, and as with traditional LINQ evaluated lazily whenever possible.

The next steps for my library are to increase test coverage and implement a slew of performance optimizations, because currently performance is quite bad, as I did not want to do a lot of optimizations when I was still working on the implementation.

So let's get back to my first post and continue on how to create async enumerables. I have implemented three ways to go about this actually, all suited to particular use cases.

If you think about it a classic IEnumerable where its element is a Task actually represents an async sequence, except that the wait has to be performed on the retrieved current value instead of the actual movement operation.

So the first way to create an async enumerable is to implement/return such an IEnumerable> which is then converted by the library infrastructure into an IAsyncEnumerable. This makes your code quite easy to write as the example below shows:

{% highlight csharp linenos %}

public static async Task<int> GetDataItem(int index)
{
    .... returns an int from somwhere asynchronously
}

public static IEnumerable<Task<int>> EnumerateData()
{
     yield return GetDataItem(1);
     yield return GetDataItem(2);
}

public static void Usage()
{
     IAsyncEnumerable<T> enumerable = EnumerateData().ToAsync();
}

{% endhighlight %}

A few points to note are that in the enumeration method you don't await an async method instead you yield the task returned by it, ( in fact the compiler disallows awaits in yield based enumerator methods ) But the behaviour stays the same as one would expect, the first yield returns a task, which is awaited outside and only after that the method is re-entered and continues before the next yield statement ( in case there is a bit more code there ), so sequencing stays the same as with a regular async/await based method implementation.

However there are some limitations to this approach, namely it's quite tricky to have async calls in such an enumerator implementation that you just want to await but not put their return value into the sequence you're defining.

I will address these limitations with a different concept of defining an Async Sequence in my next post.