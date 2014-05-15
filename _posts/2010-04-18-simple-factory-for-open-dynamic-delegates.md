---
layout: post
title:  "Simple factory for open dynamic delegates"
date:   2010-04-18 19:59:34
categories: NET C#
---

During the course of my development adventures I came to a point where I needed to call instance methods dynamically based only on information that can be obtained through reflection. It is well known that MethodInfo.Invoke is quite slow, and so I decided to create and cache delegates based on the given MethodInfo on first use, and use those to invoke the method.

However it would be prohibitive to generate a delegate for every instance of an object, rather it would be nice to create a delegate per Method prototype and decide the instance to call it on during the final invocation. A little known feature that allows us to achieve this, is the ability of the CLR to create so called “open” delegates. In the CLR a call to an instance method is quite similar to the call of a static one, except that the first parameter on the stack is the instance on which to call the method. Open Delegates allow us to create delegates that for an instance method take an additional first parameter for the instance to call the method on, thus neatly wrapping the above concept.

The feature is not common knowledge, basically because neither C# or VB offer the syntax to create such open delegates, one has to always use Delegate.CreateDelegate to create them dynamically. The overload to use is mainly Delegate.CreateDelegate( Type delegateType,MethodInfo method,bool throwOnBindFailure ).

This is quite straightforward to use, a little complication stems from the fact that we need a delegate type to create the delegate instance. I decided to leverage the predefined Action<> and Func<> types for that, and bind the generic arguments to the parameter types of the method.

Here is my simple utility class that wraps the presented ideas. Note that it does not support open generic methods and that it is limited to methods that have up to 16 parameters ( not that you should have methods that have more ). To use just call the CreateOpenDelegate method with the MethodInfo that represents your instance method. The returned delegate can then either be casted to an appropriate Action<> or Func<> or called via DynamicInvoke. Regardless of the call method used, one has to pass in the instance to call the method on as the first parameter. Here is the source code:

{% highlight csharp linenos %}

public static class DelegateFactory
{
    private static readonly Type[] actionTypes = new [] {
        typeof(Action),
        typeof(Action<>),
        typeof(Action<,>),
        typeof(Action<,,>),
        typeof(Action<,,,>),
        typeof(Action<,,,,>),
        typeof(Action<,,,,,>),
        typeof(Action<,,,,,,>),
        typeof(Action<,,,,,,,>),
        typeof(Action<,,,,,,,,>),
        typeof(Action<,,,,,,,,,>),
        typeof(Action<,,,,,,,,,,>),
        typeof(Action<,,,,,,,,,,,>),
        typeof(Action<,,,,,,,,,,,,>),
        typeof(Action<,,,,,,,,,,,,,>),
        typeof(Action<,,,,,,,,,,,,,,>),
        typeof(Action<,,,,,,,,,,,,,,,>),
    };

    private static readonly Type[] functionTypes = new [] {
        typeof(Func<>),
        typeof(Func<,>),
        typeof(Func<,,>),
        typeof(Func<,,,>),
        typeof(Func<,,,,>),
        typeof(Func<,,,,,>),
        typeof(Func<,,,,,,>),
        typeof(Func<,,,,,,,>),
        typeof(Func<,,,,,,,,>),
        typeof(Func<,,,,,,,,,>),
        typeof(Func<,,,,,,,,,,>),
        typeof(Func<,,,,,,,,,,,>),
        typeof(Func<,,,,,,,,,,,,>),
        typeof(Func<,,,,,,,,,,,,,>),
        typeof(Func<,,,,,,,,,,,,,,>),
        typeof(Func<,,,,,,,,,,,,,,,>),
        typeof(Func<,,,,,,,,,,,,,,,,>),
    };

   public static Delegate CreateOpenDelegate( MethodInfo method )
   {
        Contract.Requires(method != null);

        var closedType = GetClosedDelegateType(method);

        return closedType != null ?
             Delegate.CreateDelegate(closedType, method, true)
             : null;
    }

    private static Type GetClosedDelegateType( MethodInfo method )
    {
        Contract.Requires(method != null);

        var openType = GetOpenDelegateType(method);

        if (openType != null)
        {
            var parameterTypes =
                new[] {method.DeclaringType}.Union(method.GetParameters()
                .Select(p => p.ParameterType));

            if (method.ReturnType != typeof (void))
            {
                parameterTypes = parameterTypes.Union(new[] {method.ReturnType});
            }

            return openType.MakeGenericType(parameterTypes.ToArray());
        }

        return null;
    }

    private static Type GetOpenDelegateType( MethodInfo method )
    {
        Contract.Requires( method != null );

        var parameterCount = method.GetParameters().Length + 1;

        if (parameterCount < functionTypes.Length &&
            parameterCount < actionTypes.Length)
        {

            return method.ReturnType != typeof (void)
                   ? functionTypes[parameterCount]
                   : actionTypes[parameterCount];
        }

        return null;
    }
}

{% endhighlight %}
