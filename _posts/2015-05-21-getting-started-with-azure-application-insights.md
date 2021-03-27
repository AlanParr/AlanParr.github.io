---
layout: post
title:  "Getting started with Azure Application Insights"
date: 2015-05-21 09:42:00 +0000
categories: appinsights azure
excerpt_separator: <!--end_excerpt-->
---

Now that pricing information has been released for Application Insights, I decided to take the dive and deploy it on a few of the web applications I work on. The documentation is pretty good for your basic scenarios, but it glosses over what I think are probably common use cases.
<!--end_excerpt-->
* The instrumentation key (commonly referred to as the iKey) is located in an applicationinsights.config file, which is not very helpful if you want to change it when deploying to Live or Staging environments.
* If you use any monitoring system, such as Traffic Manager, Web Apps AlwaysOn, or any Web Testing application, this all gets included as "real traffic", which is fair enough as AppInsights has no way of knowing that it isn't real traffic. You may want to see it, but I personally do not so I wanted a way to filter it out.

There is a really great [blog post by Victor Mushkatin](http://blogs.msdn.com/b/visualstudioalm/archive/2015/01/07/application-insights-support-for-multiple-environments-stamps-and-app-versions.aspx) which covers changing where AppInsights looks for the instrumentation key, adding your version number and your own tags to the telemetry so you can filter by version number or any tag you provide. I've added a siteIdentifier tag which contains an id for the particular instance of the application, identifying where in the world it is hosted.

My particular variation of his code adds the SiteIdentifier and the assembly version to the telemetry.

{% highlight csharp %}
    public class AppInfoApplicationInsightsConfigInitializer : IContextInitializer
    {
        public void Initialize(TelemetryContext context)
        {
            context.Properties["SiteIdentifier"] = System.Configuration.ConfigurationManager.AppSettings["SiteIdentifier"];

            try
            {
                var verAtt = (AssemblyInformationalVersionAttribute)Assembly.GetExecutingAssembly().GetCustomAttributes(typeof(AssemblyInformationalVersionAttribute), false)[0];
                context.Component.Version = verAtt.InformationalVersion;
            }
            catch (Exception)
            {
                context.Component.Version = "Application Version Unknown";
            }
        }
    }
{% endhighlight %}

I also have a TelemetryInitializer that finds traffic from load balancers and web testers as explained at the top of this post, and reclassifies it as synthetic traffic, making it easier to exclude from charts and reports. I found that this traffic shows up as a different user every time, making my user count orders of magnitude higher than it should be.

{% highlight csharp %}
    public class SyntheticSourceInitializer : ITelemetryInitializer
    {
        public void Initialize(Microsoft.ApplicationInsights.Channel.ITelemetry telemetry)
        {
            if (HttpContext.Current == null)
                return;

            //Set traffic manager check and web test request to synthetic.
            if (HttpContext.Current.Request.Url.ToString().EndsWith("/monitoring"))
            {
                telemetry.Context.Operation.SyntheticSource = "AzureAliveCheck";
            }

            //Set Azure Web Apps AlwaysOn pings to synthetic.
            if (HttpContext.Current.Request.UserAgent == "AlwaysOn")
            {
                telemetry.Context.Operation.SyntheticSource = "AzureAliveCheck";
            }
        }
    }
{% endhighlight %}

You've got full access to the current request so you can identify the traffic however you need to (referrer, headers, request url, etc)
I then tell AppInsights to use these classes with the below entries in Application_Start() in global.asax.cs.

{% highlight csharp %}
            //Configure application insights.
            TelemetryConfiguration.Active.InstrumentationKey = System.Configuration.ConfigurationManager.AppSettings["iKey"];

            TelemetryConfiguration.Active.ContextInitializers.Add(new AppInfoApplicationInsightsConfigInitializer());
            TelemetryConfiguration.Active.TelemetryInitializers.Add(new SyntheticSourceInitializer());
{% endhighlight %}

As I dig further in to Application Insights, if I find more examples of useful overrides for default behaviour, I will add additional blog posts detailing these.
