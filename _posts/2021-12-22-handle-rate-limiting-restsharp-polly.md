---
layout: post
title:  "Handling rate limiting with RestSharp and Polly"
date:   2021-12-22 00:00:00 +0000
tags:
  - Rate-Limiting
  - RestSharp
  - Polly
category: DotNet
---

I was recently working on an integration with an API that has Rate-Limiting, that is if you go over the permitted number of requests within a certain period of time, you'll receive a `429` response along with a header telling you how long you need to wait before trying again.

I've been watching Polly for years but haven't had the chance to use it, so this seemed like the perfect opportunity and it was really quite simple:

Below is a method that returns a policy, which is what Polly uses for a lot of it's retry functionality.

This takes a delegate which returns an `IRestResponse`. It checks the StatusCode property of that reponse and, if it is `429`, it enters the `sleepDurationProvider` method.

In my case, this uses a class called RateStatistics that takes data from the headers of each response, it then returns either the value that was provided by the response of defaults to 5 seconds if it wasn't. I don't expect this will ever happen but wanted to be belt and braces on it and figured 5 seconds is a good number that should cover us in most cases.

It is worth noting that this request is being sent from a background service so there is no user sitting waiting for this request to finished, which would obviously affect our choice of 5 seconds as the fallback value.

```csharp
        private RetryPolicy<IRestResponse> GetPolicy()
        {
            var policy = Policy.HandleResult<IRestResponse>(p => (int)p.StatusCode == 429).WaitAndRetry(
                retryCount: 3,
                sleepDurationProvider: (retryCount, response, context) =>
                {
                    var stats = RateStatistics.FromRestsharpResponse(response.Result);
                    var msToWait = stats.TimeUntilWindowResetMilliseconds ?? 5000;
                    return TimeSpan.FromMilliseconds(msToWait);
                },
                onRetry: (response, timespan, retryCount, context) =>
                {
                    // Logging so we know the retry has happened.
                    _logger.Log(Level.Warning, $"Retry attempt #{retryCount} for {response.Result.ResponseUri} due to rate-limiting.");
                }
            );
            return policy;
        }
```

The `RateStatistics` class is fairly simple. It just receives an IRestResponse and pulls the values we are interested in out of the headers. For the sake of clarity, i've stripped out any properties/values that aren't relevant to this post.

```csharp
    internal class RateStatistics
    {
        public int? TimeUntilWindowResetMilliseconds { get; set; }

        public static RateStatistics FromRestsharpResponse(IRestResponse response)
        {
            var windowTimeRemaining = response.Headers.FirstOrDefault(x => x.Name == "X-Rate-Limit-Time-Reset-Ms")?.Value;

            return new RateStatistics
            {
                TimeUntilWindowResetMilliseconds = windowTimeRemaining == null ? (int?)null : int.Parse(windowTimeRemaining.ToString()),
            };
        }
    }
```

Our code to call our RestSharp client with our retry policy is really simple:

Before adding Polly:
```csharp
var result = client.Execute(request);
```

After adding Polly:
```csharp
var result = GetPolicy().Execute (() => client.Execute(request));
```

Now if we receive a `429` response, Polly will wait the amount of time provided by the API we are calling and will try up to 3 times before allowing the request to fail.