---
layout: post
title:  "New Relic for Azure"
date: 2013-03-08 20:10:00 +0000
categories: .net azure newrelic profiling
excerpt_separator: <!--end_excerpt-->
---

Ever since my company moved our main management system to Azure, I've been slowly working on the ability to monitor the application better in order to find the bottle necks and improve the user experience. To date, I've added:
<!--end_excerpt-->
* Timing on every request so I can query for the slowest pages.
* Azure Diagnostics
* StackExchange MiniProfiler to give me some insight in to every request, this has proved extremely useful in tracking down the exact cause of slow pages.

More recently, I've been planning to add Glimpse as well, as I like the insight it gives you in to the MVC ViewEngine and routing systems.

All these tools are great but each one adds an extra element of complexity to the system that is unavoidable, or at least it was until now.

Yesterday, I discovered New Relic. Their .net agent promises to hook in to your application with no code changes and it actually achieves it!

How?

Shamelessly lifted from the New Relic blog, this is how:

> Code run by the CLR is considered ‘managed’ code, i.e., the CLR provides a managed environment in which memory object garbage collection and other services are ‘managed’ by the CLR. The Profiler API provides a mechanism for a profiler, such as the New Relic .NET agent, to inject code into whatever managed-code functions it desires. These injected bytes are in the form of MSIL, the .NET assembly language.

Personally, I think this is damn impressive, and as I already mentioned it allows the agent to hook in with no code changes. The agent can be installed on any server, but what impresses me is the simplicity with which this can be added to Azure Cloud Services.

The New Relic blog gives step by step instructions on how to add the agent using Nuget [here](http://blog.newrelic.com/2012/08/21/x-ray-vision-into-your-azure-apps/), however this isn't all that is required. I deployed once and there was no data being reported, logging on to the server revealed that the agent had not been installed.

I found that while the nuget package added a newrelic.cmd file to install the agent, it didn't add an entry to the cloud service's servicedefinition.csdef, so the script was never getting fired. After a few attempts, I found that the following entry works.

{% highlight xml %}
<Task commandLine="newrelic.cmd" executionContext="elevated" taskType="foreground" />
{% endhighlight %}

My initial attempt utilised a taskType of background, which meant that the task was processed asynchronously and everything was already initialised by the time the installation had completed - the agent had missed the boat to get it's hooks in.

I contacted New Relic support prior to working out the solution and they suggested that this would help them with a problem they were having with the nuget package (presumably, it was supposed to add it to the servicedefinition.csdef). The knowledge that I have possibly aided in making use of NR even smoother for other users is great, it feels good to contribute.

If you sign up through Azure, you get the Standard account free with a pro trial, all the details are on the aforementioned blog post, link below for those too lazy to scroll back up.

[http://blog.newrelic.com/2012/08/21/x-ray-vision-into-your-azure-apps/](http://blog.newrelic.com/2012/08/21/x-ray-vision-into-your-azure-apps/)

Note: I am not affiliated with New Relic in any way other than being a (free) customer and they have not paid me for this post (why would they, no one will read it!), I just think this is a really great tool.
