---
layout: post
title:  "Using Custom Model Binders in ASP.Net MVC"
date: 2014-08-31 21:53:00 +0000
tags:
  - ASP.Net
  - MVC
category: MVC
excerpt_separator: <!--end_excerpt-->
---

I answered a question on Reddit this week from someone starting out in MVC who had read an incorrect article about model binding which was mostly correct, but made using custom binders look like they require more code than they actually do, so I thought it was worth a post to clear that up.
<!--end_excerpt-->
## What is (Custom) Model Binding?
Model Binding is the process through which MVC takes a form post and maps all of the form values in to a custom object, allowing you to have a POST action method which takes in a ViewModel and have it automagically populated for you. Custom Model Binders allow you to insert your own binders for particular scenarios where the default binding won't quite cut it.

## Creating our custom binder
We have the following typical example ViewModel:

{% highlight csharp %}
    public class MyViewModel
    {
        public string MyStringProperty { get; set; }
    }
{% endhighlight %}

It's just a class, nothing special about it at all. Now we want to manually handle the binding of this model because we want to add some text to the end of MyStringProperty when it gets bound. This is unlikely to be something you would want to do in real life, but we're just proving the point here.

This is our binder:

{% highlight csharp %}
    public class MyViewModelBinder:IModelBinder
    {
        protected System.Collections.Specialized.NameValueCollection Form { get; set; }

        private void Initialise(ControllerContext controllerContext, ModelBindingContext bindingContext)
        {
            Form = controllerContext.HttpContext.Request.Form;
        }

        public object BindModel(ControllerContext controllerContext, ModelBindingContext bindingContext)
        {
            Initialise(controllerContext, bindingContext);
            var msp = Form["MyStringProperty"];
            return new MyViewModel {MyStringProperty = msp + " from my custom binder"};
        }
    }
{% endhighlight %}

Model Binders need to implement IModelBinder and have a BindModel method. This gives you access to the controllerContext from which you can access HttpContext and the bindingContext, which admittedly I have never had to use.

In our binder, we just manually pick up the MyStringProperty value from the form, add it to a new instance of our object and return it, adding our incredibly important piece of text to the end of the retrieved value.

## Using our Custom Binder
There are 2 ways we can use our custom binder, which one we use depends on each scenario. If we need to override the binding of a class for a particular Action method, we can use the ModelBinder attribute on the relevant parameter of the Action Method:

{% highlight csharp %}
        [HttpPost]
        public ActionResult Index([ModelBinder(typeof(MyViewModelBinder))]MyViewModel model)
        {
            return View(model);
        }
{% endhighlight %}

This will apply our Custom Binder to this property (MyViewModel) for this action only, no other actions or controllers will be affected.

Alternatively, if we want to apply our custom binder to MyViewModel globally within the application, we can add the following line to Application_Start in global.asax.cs:

{% highlight csharp %}
        protected void Application_Start()
        {
            AreaRegistration.RegisterAllAreas();
            FilterConfig.RegisterGlobalFilters(GlobalFilters.Filters);
            RouteConfig.RegisterRoutes(RouteTable.Routes);
            BundleConfig.RegisterBundles(BundleTable.Bundles);

            ModelBinders.Binders[typeof(MyViewModel)] = new MyViewModelBinder();
        }
{% endhighlight %}

Using this method, everywhere a parameter of type MyViewModel is encountered on an ActionResult, our custom binder will be invoked instead of the standard one. Because this applies globally, we do not need the ModelBinder attribute on our Action Method so the overridden behaviour is completely transparent to the controller, promoting code reuse and keeping model binding logic where it belongs.