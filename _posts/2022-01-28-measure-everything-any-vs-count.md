---
layout: post
title:  "Measure everything - Count() vs Any()"
date:   2022-01-28 00:00:00 +0000
tags:
  - Benchmarking
category: Benchmarking
---

Within any community, there are often a series of untested truths. Within development communities, this often takes the form of X is faster than Y or Z is more efficient than W. While these are often true at some point, we don't often test them to see if they remain true years later.

One such instance is Count() vs Any() on IEnumerables and Collections. The general wisdom as I understand it is that Any() will generally be faster on IEnumerables and Count is generally better for Collections.

Using the power of Linqpad and Benchmark.Net, let's see if that is true.

This is the Linqpad script I used, you'll just need to create a new query, add a reference to Benchmark.Net, and paste this code in:

```csharp
#LINQPad optimize+     // Enable compiler optimizations

void Main()
{
	Util.AutoScrollResults = true;
	BenchmarkRunner.Run<CountVsAny>();
}

[ShortRunJob, MemoryDiagnoser]
public class CountVsAny{
	
	private IEnumerable<int> GetYieldingEnumerable(){
		for (int i = 0; i < 10; i++)
		{
			yield return i;
		}
	}
	
	private IEnumerable<int> NonYieldingEnumerable() => Enumerable.Range(1,10);
	private List<int> GetList() => Enumerable.Range(1,10).ToList();
	
	[Benchmark]
	public void YieldingEnumerableCountMethod(){
		var result = GetYieldingEnumerable().Count() > 0;
	}
	
	[Benchmark]
	public void YieldingEnumerableAnyMethod() {
		var result = GetYieldingEnumerable().Any();
	}

	[Benchmark]
	public void NonYieldingEnumerableCountMethod()
	{
		var result = NonYieldingEnumerable().Count() > 0;
	}

	[Benchmark]
	public void NonYieldingEnumerableAnyMethod()
	{
		var result = NonYieldingEnumerable().Any();
	}

	[Benchmark]
	public void ListCountMethod() {
		var result = GetList().Count() > 0;
	}

	[Benchmark]
	public void ListCountProperty()
	{
		var result = GetList().Count > 0;
	}
	
	[Benchmark]
	public void ListAnyMethod()
	{
		var result = GetList().Any();
	}
}
```

Running this produced the following results:

|                           Method |     Mean |      Error |    StdDev |  Gen 0 | Allocated |
|--------------------------------- |---------:|-----------:|----------:|-------:|----------:|
|    YieldingEnumerableCountMethod | 86.50 ns | 140.361 ns |  7.694 ns | 0.0076 |      32 B |
|      YieldingEnumerableAnyMethod | 32.55 ns |   8.911 ns |  0.488 ns | 0.0076 |      32 B |
| NonYieldingEnumerableCountMethod | 18.03 ns |   9.068 ns |  0.497 ns | 0.0095 |      40 B |
|   NonYieldingEnumerableAnyMethod | 24.26 ns | 123.983 ns |  6.796 ns | 0.0095 |      40 B |
|                  ListCountMethod | 71.39 ns | 187.181 ns | 10.260 ns | 0.0324 |     136 B |
|                ListCountProperty | 63.02 ns | 152.673 ns |  8.369 ns | 0.0324 |     136 B |
|                    ListAnyMethod | 65.34 ns |  25.340 ns |  1.389 ns | 0.0324 |     136 B |

As you can see from the above, the belief that Any() is faster than Count() for IEnumerables and Count is marginally faster than Any() and Count() is correct.

Yes there is a difference, but it is fairly inconsequential. Personally, I prefer the readability of Any() over Count > 0, so I will happily take the 2ns hit for spending a few milliseconds less reading the code.


Benchmarking code used to be something that took a great deal of effort, now it is so easy, next time you wonder if X is faster than Y, why not measure it?

I've not done a lot of benchmarking so if there are any flaws in the above that may be rendering the results inaccurate, please leave a comment.