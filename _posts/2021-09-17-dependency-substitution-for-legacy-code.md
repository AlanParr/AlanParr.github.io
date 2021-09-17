---
layout: post
title:  "Dependency substitution for legacy code"
date:   2021-09-17 00:00:00 +0000
tags:
  - Refactoring
category: Refactoring
---

This post shows an example of how to introduced a form of dependency injection in to legacy code with minimal changes.
I refer to this as dependency substitution because, in my view at least, there is a subtle difference between this and depdendency injection.

With dependency injection, we are *injecting* the dependency via either constructor or property injection. The dependency is set once by our IOC container to whatever type that dependency is set to be set to.

With dependency substitution, we're performing a bait and switch, swapping out the previously hard-coded dependency via a surreptitously snuck in substitute.

It's a small difference admittedly, but as this technique is very much intended as a temporary solution to get tests around your legacy code so it can be modernised safely, I didn't want to give the impression that this technique ticks the `we've added dependency injection` checkbox because in my view it doesn't.

Anyway, on to the code. It is a slightly convoluted example but hopefully demonstrates the technique.

All the code is available on [Github](https://github.com/AlanParr/LegacyAppTesting).

## Before

Program.cs - This instantiates a BoredomSuggestionService, asks the user if they're bored and returns an activity to do until they answer with `N`

```csharp
using System;
using System.Threading.Tasks;

namespace LegacyAppTesting.ConsoleApp
{
    internal class Program
    {
        public static async Task Main(string[] args)
        {
            //Instantiate bored client.
            var service = new BoredomSuggestionService();
            Console.WriteLine("Bored? Would you like a suggestion for something to do? (Y/N): ");
            while (Console.ReadLine() == "Y")
            {
                var randomActivity = await service.FindSomethingToDo();
                Console.WriteLine("  How about " + randomActivity.Activity + "?");
                Console.WriteLine("");
                Console.WriteLine("Still bored?");
            }
                
            Console.WriteLine("So glad you're not bored any more, enjoy your activity!");
        }
    }
}
```

The BoredomSuggestionService is quite simple:
```csharp
public class BoredomSuggestionService
    {
        private BoredClient _client;

        public BoredomSuggestionService()
        {
            _client = new BoredClient();
        }
        public async Task<BoredResponse> FindSomethingToDo()
        {
            return await _client.FindSomethingToDo();
        }
    }
```
It instantiates a BoredClient which talks to the API at [BoredAPI.com](https://www.boredapi.com) to get an activity suggestion.

```csharp
    public class BoredClient
    {
        public async Task<BoredResponse> FindSomethingToDo()
        {
            //Get activity.
            var client = new HttpClient();
            var result = await client.GetAsync("http://www.boredapi.com/api/activity/");
            var bodyString = await result.Content.ReadAsStringAsync();
            return Newtonsoft.Json.JsonConvert.DeserializeObject<BoredResponse>(bodyString);
        }
    }
```

## After
What we are going to do is modify BoredomSuggestionService so we can sneak in a mocked version of BoredClient so we can test it.

We'll go step-by-step over the changes needed to facilitate this.

First, we create an interface for `BoredClient`
```csharp
    public interface IBoredClient
    {
        Task<BoredResponse> FindSomethingToDo();
    }
```

Then we create a really simple factory:
```csharp
    public class BoredClientFactory
    {
        //Static override instance.
        private static IBoredClient _overriddenClient;

        //Protected so only available to classes that inherit from this one.
        protected void SetClientInternalForTestingOnly(IBoredClient boredClient)
        {
            _overriddenClient = boredClient;
        }
        public IBoredClient GetClient()
        {
            //Return either our overridden instance or instantiate a new one as we were before.
            return _overriddenClient ?? new BoredClient();
        }
    }
```

This is the key to getting the mock in. There is a protected method that will allow setting the static overridden `IBoredClient`.
This method is only for testing and is named as such to make it clear to anyone coming across this in the future that they shouldn't use it in Production.

We use the factory in `BoredomSuggestionService` as so, it is a very small change.

This
```csharp
        public BoredomSuggestionService()
        {
            _client = new BoredClient();
        }
```
changes to
```csharp
        public BoredomSuggestionService()
        {
            _client = new BoredClientFactory().GetClient();
        }
```


### Introducing the testing
We've now made all the changes to production code that we need to.

Now we create a test project and in that project, create a class called `TestBoredClientFactory`

```csharp
    public class TestBoredClientFactory: BoredClientFactory
    {
        public void SetClient(IBoredClient boredClient)
        {
            SetClientInternalForTestingOnly(boredClient);
        }
    }
```

and in our test we use this to get the `Mock<IBoredClient>` in to the factory:

```csharp
        [Test]
        public void TestMockBoredClient()
        {
            var mockClient = new Mock<IBoredClient>();
            mockClient.Setup(x => x.FindSomethingToDo()).Returns(Task.FromResult(new BoredResponse()
            {
                Activity = "Extreme Ironing",
                Accessibility = 0.1,
                Key = "EXTREME_IRONING",
                Participants = 1,
                Price = 10,
                Type = "Extreme Sports"
            }));
            //Setup
            new TestBoredClientFactory().SetClient(mockClient.Object);
            var result = new BoredomSuggestionService().FindSomethingToDo().Result;
            
            Assert.AreEqual("Extreme Ironing", result.Activity);
            Assert.AreEqual("EXTREME_IRONING", result.Key);
            Assert.AreEqual("Extreme Sports", result.Type);
        }
```

By calling the `TestBoredClientFactory.SetClient` method, we can push the Mock in to the static `_overriddenClient` which will then be produced by the `BoredClientFactory` in `BoredomSuggestionService`.

As a result of this bait-and-switch we can now substitute the BoredClient with our Mock and get some tests in.

## Conclusion

I've used this technique a few times in cases where DI was not possible and I couldn't risk making large-scale changes without getting at least some tests in place.

It works well, but to reiterate, if true dependency injection is an option that you have available to you, please do that instead.

I hope this is helpful, comments welcome as usual.