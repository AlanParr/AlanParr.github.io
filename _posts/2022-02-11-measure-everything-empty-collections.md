---
layout: post
title:  "Measure everything - Empty collections"
date:   2022-02-11 00:00:00 +0000
tags:
  - Benchmarking
category: Benchmarking
---

What's the quickest way to get an empty `IEnumerable<string>`, `List<string>`, or `string[]`? Let's measure it.

## The Test

As usual, code is running in a [Linqpad](https://www.linqpad.net/) script and is using [Benchmark.Net](https://github.com/dotnet/BenchmarkDotNet).

```csharp
#LINQPad optimize+     // Enable compiler optimizations

void Main()
{
	Util.AutoScrollResults = true;
	BenchmarkRunner.Run<EmptyCollectionsBenchmark>();
}

[ShortRunJob, MemoryDiagnoserAttribute]
public class EmptyCollectionsBenchmark 
{
	[Benchmark]
	public void Enumerable_EnumerableEmpty()
	{
		IEnumerable<string> val = Enumerable.Empty<string>();
	}

	[Benchmark]
	public void Enumerable_ArrayEmpty()
	{
		IEnumerable<string> val = Array.Empty<string>();
	}
	
	[Benchmark]
	public void List_EnumerableEmpty()
	{
		List<string> val = Enumerable.Empty<string>().ToList();
	}

	[Benchmark]
	public void List_ArrayEmpty()
	{
		List<string> val = Array.Empty<string>().ToList();
	}

	[Benchmark]
	public void List_NewList()
	{
		List<string> val = new List<string>();
	}
	
	[Benchmark]
	public void Array_ArrayEmpty()
	{
		string[] val = Array.Empty<string>();
	}
	
	[Benchmark]
	public void Array_NewArray_ZeroLength()
	{
		string[] val = new string[0];
	}
	
	[Benchmark]
	public void Array_NewArray_100Length()
	{
		string[] val = new string[100];
	}
}
```

## The Results

|                     Method |       Mean |      Error |    StdDev |     Median |  Gen 0 | Allocated |
|--------------------------- |-----------:|-----------:|----------:|-----------:|-------:|----------:|
| Enumerable_EnumerableEmpty |  0.0262 ns |  0.8292 ns | 0.0455 ns |  0.0000 ns |      - |         - |
|      Enumerable_ArrayEmpty |  0.0312 ns |  0.9871 ns | 0.0541 ns |  0.0000 ns |      - |         - |
|       List_EnumerableEmpty | 16.3021 ns | 18.8314 ns | 1.0322 ns | 15.7586 ns | 0.0076 |      32 B |
|            List_ArrayEmpty | 25.2154 ns | 24.0245 ns | 1.3169 ns | 24.7718 ns | 0.0076 |      32 B |
|               List_NewList |  5.2154 ns |  3.8314 ns | 0.2100 ns |  5.1523 ns | 0.0076 |      32 B |
|           Array_ArrayEmpty |  0.0035 ns |  0.1098 ns | 0.0060 ns |  0.0000 ns |      - |         - |
|  Array_NewArray_ZeroLength |  3.0829 ns |  1.0481 ns | 0.0574 ns |  3.0888 ns | 0.0057 |      24 B |
|   Array_NewArray_100Length | 42.2577 ns | 23.3965 ns | 1.2824 ns | 41.6450 ns | 0.1970 |     824 B |


## The Conclusion
So what is the most efficient way to get an empty collection?

* For an IEnumerable, `Enumerable.Empty<T>()`, which is what I was expecting.
* For a List, `new List<T>()` is the winner. I initially expected one of the `.Empty()` methods to win but now that I think about it, it makes sense that they don't given the `ToList()` call is probably initiating an enumeration or boxing operation.
* For an array, `Array.Empty<T>()` is the winner. Again, no surprise here.

I hope that was informative. If there are any flaws in the above that may be rendering the results inaccurate, please leave a comment.