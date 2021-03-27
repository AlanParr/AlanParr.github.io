---
layout: post
title:  "A simple WebCache Helper"
date: 2013-07-01 20:25:00 +0000
categories: asp.net azure caching
excerpt_separator: <!--end_excerpt-->
---

As mentioned in most of my previous posts, the main project I work on is due to move to Azure in the future. Among the many gems of Azure is their caching infrastructure, which can either be hosted on a dedicated worker role or instructed to use spare memory on your web roles.
<!--end_excerpt-->
More information and pricing for Azure Caching can be found at [http://www.windowsazure.com/en-us/services/caching/](http://www.windowsazure.com/en-us/services/caching/)

I fully intend to make use of Azure’s in-built caching when we get there, but I can’t wait to start implementing some sort of caching and I don’t want to have to do a big find-replace in the code when we do get there, so I wrote a simple WebCacheHelper which provides easy access to caching anywhere in the application but is also easy to replace when I move to Azure.

The code is below.

{% highlight csharp %}
    public static class 
        WebCacheHelper
    {
        public static T TryGetFromCache<T>(string cacheName, string itemKey) where T:class
        {
            if (HttpContext.Current == null || HttpContext.Current.Session == null) { return null; }
 
            var sessionResult = TryGetFromSessionCache<T>(cacheName, itemKey);
            if (sessionResult != null){return sessionResult;}
 
            var applicationResult = TryGetFromApplicationCache<T>(cacheName, itemKey);
            return applicationResult;
        }
 
        public static T TryGetFromCache<T>(string cacheName, string itemKey,CachingLevel cachingLevel) where T:class
        {
            if (HttpContext.Current == null || HttpContext.Current.Session == null) { return null; }
 
            switch (cachingLevel)
            {
                case CachingLevel.Session:
                    return TryGetFromSessionCache<T>(cacheName, itemKey);
                case CachingLevel.Application:
                    return TryGetFromApplicationCache<T>(cacheName, itemKey);
            }
            return null;
        }
 
        public static void AddToCache(string cacheName, string itemKey, object cacheItem, CachingLevel cachingLevel=CachingLevel.Application)
        {
            if (HttpContext.Current == null || HttpContext.Current.Session == null) { return; }
 
            switch (cachingLevel)
            {
                case CachingLevel.Session: AddToSessionCache(cacheName, itemKey, cacheItem);
                    break;
                case CachingLevel.Application: AddToApplicationCache(cacheName, itemKey, cacheItem);
                    break;
            }
        }
 
        public static void RemoveFromCache(string cacheName, string itemKey)
        {
            if (HttpContext.Current == null || HttpContext.Current.Session == null) { return; }
 
            RemoveFromApplicationCache(cacheName,itemKey);
            RemoveFromSessionCache(cacheName,itemKey);
        }
 
        public static void RemoveFromCache(string cacheName, string itemKey, CachingLevel cachingLevel)
        {
            if (HttpContext.Current == null || HttpContext.Current.Session == null) { return; }
 
            switch (cachingLevel)
            {
                case CachingLevel.Session: RemoveFromSessionCache(cacheName, itemKey);
                    break;
                case CachingLevel.Application: RemoveFromApplicationCache(cacheName, itemKey);
                    break;
            }
        }
 
        private static string GetCacheKey(string cacheName, string itemKey)
        {
            return cacheName + "-" + itemKey;
        }
 
        #region Session Cache
 
        private static void AddToSessionCache(string cacheName, string itemKey, object cacheItem)
        {
            HttpContext.Current.Session[GetCacheKey(cacheName, itemKey)] = cacheItem;
        }
 
        private static void RemoveFromSessionCache(string cacheName, string itemKey)
        {
            var result = HttpContext.Current.Session[GetCacheKey(cacheName, itemKey)];
            if (result != null)
            {
                HttpContext.Current.Session.Remove(GetCacheKey(cacheName, itemKey));
            }
        }
        
        public static T TryGetFromSessionCache<T>(string cacheName, string itemKey) where T:class
        {
            var result = HttpContext.Current.Session[GetCacheKey(cacheName, itemKey)];
            if (result == null)
            {
                return null;
            }
            return (T) result;
        }
 
        #endregion
 
        #region Application Cache
 
        private static void AddToApplicationCache(string cacheName, string itemKey, object cacheItem)
        {
            HttpContext.Current.Application[GetCacheKey(cacheName, itemKey)] = cacheItem;
        }
 
        private static void RemoveFromApplicationCache(string cacheName, string itemKey)
        {
            var result = HttpContext.Current.Application[GetCacheKey(cacheName, itemKey)];
            if (result != null)
            {
                HttpContext.Current.Application.Remove(GetCacheKey(cacheName, itemKey));
            }
        }
 
        public static T TryGetFromApplicationCache<T>(string cacheName, string itemKey) where T:class
        {
            var result = HttpContext.Current.Application[GetCacheKey(cacheName, itemKey)];
            if (result == null)
            {
                return null;
            }
            return (T) result;
        }
 
        #endregion
 
    }
{% endhighlight %}

As you can see there is nothing clever going on here, there is a single AddToCache method for adding data to the cache, with a default parameter for cachingLevel so the calling code can override the default application-level caching to cache at the session level.

There are a couple of RemoveFromCache methods, one where the calling code can specify which cache to remove the item from, the other removes the data from whichever cache it resides in.

There are also two TryGet methods for retrieving from a specified caching level or from any that has a matching key.

CachingLevel is just an enum with items for Session and Application.

{% highlight csharp %}
     public enum CachingLevel
    {
        Application,
        Session
    }
{% endhighlight %}

I also use the following static string class to save having magic strings peppered through the application

{% highlight csharp %}
    public static class WebCacheKeys    {
        public const string Users = "Users";
    }
{% endhighlight %}

I can just add strings as required and change the backing value if I need to without having to make a large number of changes elsewhere. If I had IOC available, I probably wouldn't have made this static and would've abstracted the functionality behind an ICacheHelper interface. The way I look at it, this is the next best thing in terms of ability to make changes in the future. When I move to Azure, I'll post the Azure-centric version of this helper.
