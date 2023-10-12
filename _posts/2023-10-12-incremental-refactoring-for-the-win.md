---
layout: post
title:  "Incremental Refactoring for the Win"
date:   2023-10-12 00:00:00 +0000
tags:
  - Refactoring
category: Refactoring
---

We've all encountered those sprawling, unruly code bases that have grown unchecked over the years. We label them as "legacy," but that is often just code for "code that works, pays the bills, and our customers rely on to continue working". The challenge then becomes how to improve these code bases without causing more harm than good.

In this article, we'll explore a pragmatic approach to rejuvenating legacy code through small, incremental changes. This method is all about making minimal, yet meaningful, alterations and getting them into production quickly. While it might appear counterintuitive at first, this strategy can save you time and frustration in the long run. So, let's dive into the art of incremental refactoring.

1. Redefining "Legacy" Code:
Before we dive into the how, let's reconsider what we mean by "legacy" code. It's not just outdated, messy code; it's the code that pays the bills and is relied upon by our users.

2. The Power of Incremental Changes:
Small, incremental improvements are the cornerstone of this approach. Whether it's refactoring a single method, modernizing a class, or cleaning up a snippet of unused code, the goal is to make the smallest possible change and push it to production as quickly as possible. While this may seem like a time-consuming process due to code reviews, QA testing, merging, and publishing, it ultimately saves time by preventing issues that large, sweeping changes can introduce.

3. The Importance of Testing:
In the world of legacy code, poor code coverage is often the norm. Testing may seem daunting, but it's crucial to ensure the reliability of your changes. Start by writing tests for the specific code you're altering, and gradually expand your test suite as you make further improvements. A test that just calls your method and verifies it doesn't throw an exception is better than nothing.

4. Embrace Imperfection:
Your initial refactoring efforts don't have to result in a gold-standard SOLID-adherent perfect vision of code. Refactoring from "terrible" to "bad" is still an improvement. Subsequent iterations can then raise the code's quality from "bad" to "okay" and beyond.

5. The Pitfalls of Radical Overhauls:
Attempting a complete codebase overhaul in one go can be a recipe for disaster. It's highly likely to introduce more bugs and leave you questioning your decisions. Incremental changes, on the other hand, allow you to maintain confidence in your work and monitor the impact of each alteration.

6. Tailoring Change Sizes to Test Coverage:
If your codebase lacks substantial test coverage, stick to small-scale changes to minimize risks. As your testing improves, you can gradually expand the scope of your refactorings.

Let's see this process in action with a simple example:

## Starting Point
```csharp
public class Bad
{
	private DatabaseService _database;
	
	public Bad()
	{
		_database = new DatabaseService(System.Configuration.ConfigurationManager.AppSettings["DatabaseConnectionString"]);
	}
}
```
This class is problematic for a few reasons
* It news up the DatabaseService which means we can't replace it during testing ("new is glue").
* It is coupled to the concrete type which is also a barrier to testing.
* It reaches out in to app configuration to get the connection string for that database. This restricts our options for changing where that configuration comes from and also gives the caller too much knowledge about the internal workings of that class and makes it difficult to make changes to how the class is instantiated.

## Better
```csharp
public class Better
{
	private DatabaseService _database;

	public Better()
	{
		_database = DatabaseServiceFactory.GetDatabase();	
	}	
}
```
This is better as have isolated the logic of building a DatabaseService instance in to a single place. It should also be fairly easy to drop this in the place of the previous code with no other changes.

But it still has some issues:
* We are now bound to a static factory but at least now we no longer know anything about the object that is being created.
* We are still bound to the concrete type.

## Better Still

Ideally you would skip this step and go straight to the Final Form but there are many reasons why this may not be immediately possible, for instance you don't have DI currently, so I wanted to show this step as an example of how to achieve testability when you do not have the benefits of DI.

```csharp
public class BetterStill
{
	private IDatabaseService database;

	public BetterStill()
	{
		database = new DatabaseServiceFactory().GetDatabase();	
	}	
}

public class DatabaseServiceFactory
{
	private static IDatabaseService _testInstance;
	
	public IDatabaseService GetDatabase()
	{
		if(_testInstance != null)
			return _testInstance;
			
		return new DatabaseService(System.Configuration.ConfigurationManager.AppSettings["DatabaseConnectionString"]);
	}
	
	protected void SetTestInstance(IDatabaseService instance)
	{
		_testInstance = instance;
	}
}
```
Now we have an interface. The combination of this and modifying the DatabaseServiceFactory now unlocks substitution of the database service.

Now before anyone gets upset, this is still not good code, it is just a step closer to where we want to be. We have actually introduced a bit of sin in the form of having affordances for testing in a production class (the _testInstance logic in DatabaseServiceFactory) but it is for a good cause. We can now create a TestDatabaseServiceFactory in our tests that will allow us to override the IDatabaseService instance in testing.


```csharp
public class TestDatabaseServiceFactory : DatabaseServiceFactory
{
	public void SetInstance(IDatabaseService instance)
	{
		base.SetTestInstance(instance);
	}
}
```
which we can use as

```csharp
[SetUp]
public void TestSetup()
{
  var databaseInstance = new Mock<IDatabaseService>();
  TestDatabaseServiceFactory.SetInstance(databaseInstance);
}
```
So now we can inject a Mock and start mocking and verifying behaviour of this service.

But don't stop here.

## Final form
```csharp
public class Final
{
	private IDatabaseService _database;
	
	public Final(IDatabaseService database)
	{
		_database = database;
	}
}
```
This is the final form for this example.
* We are no longer newing up the instance directly with it now coming in on the constructor and it is now an interface so we are no longer tied to a concrete type, these 2 things together has unlocked testability in our class.
* In production, if we need to change how the DatabaseService is created we can now do that easily without having to make any changes to its consumers.


## Conclusion
Legacy codebases are not monuments to be razed and rebuilt in a day. They are the foundation of your projects and require a thoughtful, incremental approach to improve them. By making small, well-tested changes and embracing gradual progress, you can bring your legacy code back to life, ensuring its reliability and maintainability while avoiding the pitfalls of radical overhauls.