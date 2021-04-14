---
layout: post
title:  "Supporting SameSite None in .Net 4.6 or lower"
date:   2020-07-10 00:00:00 +0000
tags:
  - Samesite
  - Net46
category: Samesite
excerpt_separator: <!--end_excerpt-->
---

As I write this post, it is 4 days until Chromium begins enforcing the new SameSite rules again.
<!--end_excerpt-->
When they first did this in March, it caused a number of issues including breaking website integrations with some payment gateways.

If you're on .Net 4.7 or higher, Microsoft supports setting SameSite to None. The official recommendation is that if you want to use SameSite None, then you need to move up to .Net 4.7.2, which if you are able, you should absolutely do.

However, there are those of us who are stuck on .Net lower than 4.7 and there is nothing we can do about it and our employers want to know that their sites aren't going to start breaking come the 14th of July.

While trying to find a solution to this problem, I stumbled upon what appears to be a possible solution for those of us stuck on lower .Net versions.

{% highlight csharp %}
var cookie = new HttpCookie("myreallyimportantcookie")
            {
                Value = "myreallyimportantcookievalue",
                Secure = true,
                Path = "/",
                HttpOnly = true
            };
{% endhighlight %}

As you'll see from the below image, we have a cookie with the secure attribute and httponly, but no samesite attribute.

![No SameSite](\assets\img\cookie-nosamesite.png)

## Adding SameSite

In .Net 4.7.2, if we want to support SameSite, we simply add the SameSite attribute.

{% highlight csharp %}
var cookie = new HttpCookie("myreallyimportantcookie")
            {
                Value = "myreallyimportantcookievalue",
                Secure = true,
                Path = "/",
                HttpOnly = true, 
                SameSite = SameSiteMode.None
            };
{% endhighlight %}

Of course, we can't do this in .Net 4.5 as the SameSite property doesn't exist. Instead, we can do this somewhat gross thing:

{% highlight csharp %}
var cookie = new HttpCookie("myreallyimportantcookie")
            {
                Value = "myreallyimportantcookievalue" + ";SameSite=None",
                Secure = true,
                Path = "/",
                HttpOnly = true
            };
{% endhighlight %}

And now if we run the application, we can see we have the SameSite attribute set to None.

![SameSite](\assets\img\cookie-samesite.png)

## Disclaimer #1
This solution is completely unsupported and a bit gross. The approved solution is to move to .Net 4.7.2 and if you are able, you should absolutely do that. But sometimes the real world places limits upon us and if you are in that situation, hopefully this will get you out of a bind.

## Disclaimer #2
I have only tested this in an MVC solution in .Net 4.6.1. Theoretically, I think it will probably work in versions lower than that unless the cookie value-handling semantics changed massively at some point in the past. If you want to use this, test for yourself and post a comment if it worked as maybe that will help someone else.