---
layout: post
title:  "EWS Post 1 of n - Sending an email"
date: 2012-11-20 14:46:00 +0000
categories: c# ews exchange-web-services
excerpt_separator: <!--end_excerpt-->
---

I've been digging in to the basics of using Exchange Web Services recently and thought it worthy of a few blog posts. These won't be the perfectly crafted literary masterpieces of the [Hanselmans](http://www.hanselman.com/blog/) of the world, but rather brain farts and code dumps to play with.
<!--end_excerpt-->
The EWS SDK can be found [here](http://download.microsoft.com/download/8/2/5/825231AC-D373-45D4-A644-7AF12340C815/EwsManagedApi.msi).

I suggest using Linqpad to run these code snippets as that is where they are being written.

Just add a reference to **C:\Program Files\Microsoft\Exchange\Web Services\2.0\Microsoft.Exchange.WebServices.dll** and a using statement for **Microsoft.Exchange.WebServices.Data** and you're good to go.

Sending an email using EWS is astonishingly simple and consists of just 8 lines of code:

{% highlight csharp %}
var service = new ExchangeService(ExchangeVersion.Exchange2010);
service.Credentials = new WebCredentials("myusername","mypassword");
service.Url = new Uri("https://mymailserver/ews/exchange.asmx");

EmailMessage message = new EmailMessage(service);
message.Subject = "Interesting";
message.Body = "The proposition has been considered.";
message.ToRecipients.Add("pointhairedboss@thecompany.com");
message.SendAndSaveCopy();
{% endhighlight %}
