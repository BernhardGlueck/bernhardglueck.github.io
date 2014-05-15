---
layout: post
title:  "Code snippet of the day"
date:   2011-01-18 19:59:34
categories: C# NET LINQ
---

As I wrote some code for a declarative mapping system of members to certain data sources today, and I wanted to keep things DRY, I needed to have both read and write access to a property which was only given to me by a simple selector style C# Lambda Expression. Of course one could generate the required contrary method based on Reflection.Emit ( IL generation ) or some other way ( CSharpCodeDom etc ) but .NET Expression Trees to the rescue, my job was much easier than I thought. Here is my implementation ( which could use a little better error checking I know ).

{% highlight csharp linenos %}

public static Action<TObject, TProperty> GetterToSetter<TObject,TProperty>(Expression<Func<TObject, TProperty>> getter)
{
    Contract.Requires<ArgumentNullException>(getter != null, "getter");

    var memberAccess = getter.Body as MemberExpression;

    if (memberAccess != null)
    {
     var memberName = memberAccess.Member.Name;
         var property = typeof (TObject).GetProperty(memberName);

         if (property != null && !property.CanWrite)
         {
             throw new ArgumentException("Property it not writeable");
         }

         var member = property ?? (MemberInfo) typeof (TObject).GetField(memberName);

         if (member != null)
         {
             var parameter1        = Expression.Parameter(typeof (TObject), "obj");
             var parameter2        = Expression.Parameter(typeof (TProperty), "value");
             var memberAccessClone = Expression.MakeMemberAccess(parameter1, member);

             var body    = Expression.Assign(memberAccessClone,parameter2);
             var setter  = Expression.Lambda(typeof (Action<TObject, TProperty>), body,new[] {parameter1, parameter2});

             return (Action<TObject, TProperty>) setter.Compile();
         }
    }

    throw new ArgumentException("Invalid getter expression, only simple property/field accesses are supported");
}

{% endhighlight %}

Given an expression that selects a property like “s => s.Id” this will give you an action that accepts both the owning object as well as a new value for the property and assigns it. Also since Expression trees use DynamicMethods for code generation the new function will be defined in the owning objects type scope, so even if the setter of the property is not reachable publicly the returned function will circumvent that members visibility and still work. The end result is a pair of functions that allow read/write access to a property or field of an object instance. Here is a usage sample:

{% highlight csharp linenos %}

public class Simple
{
    public int Id { get; set; }
}

public static void Sample()
{
    var setter = GetterToSetter<Simple, int>( s => s.Id );
    setter( new Simple(), 10 );
}

{% endhighlight %}

My given implementation is not very useful on it’s own, but can be helpful in code that already has the owning objects type as a generic parameter.