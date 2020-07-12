---
layout: post
title:  "ProTip: Open Powershell as admin from Powershell"
date: 2015-01-16 10:18:00 +0000
categories: powershell
---

Just a quick one as I find I have been using this trick a lot lately.

If I am in a standard Powershell prompt and need to get an admin one open, I used to search for Powershell, right-click, run as admin. I'd do this even if I was already in a Powershell prompt as I can never remember the syntax for runas.exe.

A much easier way, especially if you are already in a Powershell prompt is:

{% highlight powershell %}
Start-Process powershell -verb runas
{% endhighlight %}

This works for any executable and will pop up UAC appropriately to allow you to enter credentials if you need to, or just run as admin if you are already an Administrator.

Hope this helps some one.

**Edit**

This can be further shortened to

{% highlight powershell %}
Start powershell -verb runas
{% endhighlight %}

Thanks anonymous user!
