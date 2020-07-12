---
layout: post
title:  "Supporting SameSite None in .Net 4.6 or lower"
date:   2020-07-10 00:00:00 +0000
categories: samesite net46
---

As I write this post, it is 4 days until Chromium begins enforcing the new SameSite rules again.

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

![No SameSite](\images\cookie-nosamesite.png)

