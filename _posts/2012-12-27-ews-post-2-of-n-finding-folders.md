---
layout: post
title:  "EWS Post 2 of n - Finding Folders"
date: 2012-12-27 22:21:00 +0000
categories: c# ews exchange-web-services
excerpt_separator: <!--end_excerpt-->
---

The EWS SDK can be found [here](http://download.microsoft.com/download/8/2/5/825231AC-D373-45D4-A644-7AF12340C815/EwsManagedApi.msi).

I suggest using Linqpad to run these code snippets as that is where they are being written.
<!--end_excerpt-->
Just add a reference to **C:\Program Files\Microsoft\Exchange\Web Services\2.0\Microsoft.Exchange.WebServices.dll** and a using statement for **Microsoft.Exchange.WebServices.Data** and you're good to go.

Part 2 of n in my EWS series, this time covering finding folders. This is a two liner, barely worthy of it's own blog post, but it's late and, for something so simple, there seem to be alarmingly few straight forward examples of this on the interwebs, so enjoy.

{% highlight csharp %}
var service = new ExchangeService(ExchangeVersion.Exchange2010);
service.Credentials = new WebCredentials("myusername","mypassword");
service.Url = new Uri("https://mymailserver/ews/exchange.asmx");

var f = new FolderView(100);
var res = service.FindFolders(WellKnownFolderName.PublicFoldersRoot,f);
{% endhighlight %}
