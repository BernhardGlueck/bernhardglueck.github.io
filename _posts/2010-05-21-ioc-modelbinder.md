---
layout: post
title:  "IOC ModelBinder"
date:   2010-05-21 19:59:34
categories: C# IOC NET ASP.NET MVC
---

I had an idea a couple of days ago, that I decided to play around with, and maybe collect opinions on. The ASP.NET MVC ModelBinder infrastructure is quite exhaustive, and allows you to perform almost any action when a something the developer wants to get data from the request into the domain/data/business model. However recently I found myself in a situation where a controller got quite sophisticated and required a lot of dependencies from our Ioc Container of choice ( Autofac ). Since all of those were mandatory dependencies for execution for one or the other action method, our constructor arguments got a little bit out of hand ( I just don’t like constructors with more than, let’s say 4 arguments ). So I decided to build a model binder that injects dependencies into action method calls.

Without further ado, here is the interesting part of the code:

{% highlight csharp linenos %}

public class IocBindAttribute : CustomModelBinderAttribute
{
    public override IModelBinder GetBinder()
    {
        return new IocModelBinder();
    }
}

public class IocModelBinder : IModelBinder
{
    public object BindModel(ControllerContext controllerContext,
                            ModelBindingContext bindingContext)
    {
        try
        {
            return GetContainer().Resolve(bindingContext.ModelName,
                                          bindingContext.ModelType);
        }
        catch (ComponentNotRegisteredException)
        {
            return GetContainer().Resolve(bindingContext.ModelType);
        }
    }

    private IContainer GetContainer()
    {
        return (HttpContext.Current.ApplicationInstance as
                    IContainerProviderAccessor)
                    .ContainerProvider.ApplicationContainer;
    }
}

public class EmailController : Controller
{
    public ActionResult Index()
    {
        return View();
    }

    public ActionResult Send([IocBind]IEmailService emailService,
                         string from, string to, string subject, string text)
    {
        emailService.Send(from, to, subject, text);

        return RedirectToAction("Index");
    }
}

{% endhighlight %}

Basically you use your container as you like and register the given model binder either globally or via the provided ModelBinder attribute on a more granular level. Note that this code is not meant to be a production implementation, it’s just here to show the idea. The same basic idea can be implemented for virtually any container, or using the common service locator if you choose so. However I am a bit torn with this idea in the context of architectural design:

Arguments for using it:

* Reduces a lot of boilerplate code for dependency handling like constructor arguments and storage.
* If you see each action as a distinct piece of code that is grouped by a container, it better reflects the correspondence between your actions and the dependencies they need, you can see at a glance what dependencies are required by each action, which might not be so obvious when you follow the traditional approach.
* In the same vein it eases both refactoring as well as unit testing, because you can now treat each action more easily in isolation, move it around to different controllers, or set it up for a test case.

Arguments against using it:

* If you have a controller whose action methods require such disparate dependencies, then you might have a deeper problem, either your controller is doing to much in terms of the single responsibility principle, or your dependencies are too granular and it might be a good idea to provide some bridge dependency for that case.
* It mixes the concerns of data flow and the business layer of your application.
* So please fire away with the comments to tell me what you think, or what I have done wrong with this.
