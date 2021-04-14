---
layout: post
title:  "Simple Search MVC"
date: 2011-12-24 23:31:00 +0000
tags:
  - CSharp
  - Search
category: CSharp
excerpt_separator: <!--end_excerpt-->
---

## Introduction

This article introduces some simple code which allows easy addition of search functionality which can be used to search any objects (files, .net classes) and display the search results together in a single view.
<!--end_excerpt-->
## Background

This demo is presented in ASP.NET MVC so basic MVC knowledge would be useful, although this solution could be applied in other .net environments.

## Using the code

The main work is done in the implementations of the abstract base class Searcher.

In this example, some dummy data is created in the constructor, this is then queried for matching items in the Search method and projected in to a SearchResult.
Note how RouteInformation is populated. The entries in the RouteInformation dictionary will be used to create an ActionLink in the MVC view.

{% highlight csharp %}
   public class ProductSearcher : Searcher  
   {  
     private List<Product> Products { get; set; }  
     public ProductSearcher()  
     {  
       //Dummy up some data here...  
  Products = new List<Product>();  
     }  
     public override IEnumerable<SearchResult> Search(string searchTerm)  
     {  
       var result = Products.Where(p => p.Description.ToLower().Contains(searchTerm.ToLower()) || p.Name.ToLower().Contains(searchTerm.ToLower()))  
         .Select(p => new SearchResult  
         {  
           Category = "Product",  
           Description = p.Name,  
           Id = p.ProductId,  
           OriginatingObject = p  
         }).ToList();  
       result.ForEach(p => p.RouteInformation = GetRouteInfo(p.Id));  
       return result;  
     }  
     private static Dictionary<string, string> GetRouteInfo(int id)  
     {  
       return new Dictionary<string, string>  
                   {  
                     {"controller", "product"},  
                     {"action", "details"},  
                     {"Id", id.ToString()}  
                   };  
     }  
   }  
{% endhighlight %}

SearchCore is initialized with a List of Searcher implementations that it should use to build the search results. When Search is called, it simply iterates over them, calling their Search method and aggregating the result.

{% highlight csharp %}
   public class SearchCore  
   {  
     private List<Searcher> Searchers { get; set; }  
     public SearchCore(List<Searcher> searchers)  
     {  
       Searchers = searchers;  
     }  
     public IEnumerable<SearchResult> Search(string searchTerm)  
     {  
       var result = new List<SearchResult>();  
       foreach (Searcher searcher in Searchers)  
       {  
         result.AddRange(searcher.Search(searchTerm));  
       }  
       return result;  
     }  
   }  
{% endhighlight %}

## The Search Controller

The Search controller just has a single ActionResult which creates instances of our Searchers and adds them to a list which is then passed to SearchCore. It uses a stopwatch to measure the time taken to perform the search.
Caching can be enabled on the Search method to improve performance.

{% highlight csharp %}
     private SearchCore SearchCore { get; set; }  
     public SearchController()  
     {  
       var searchers = new List<Searcher> {new ProductSearcher(), new CompanySearcher()};  
       SearchCore = new SearchCore(searchers);  
     }  
     //Uncomment below to enable caching.  
     //[OutputCache(VaryByParam = "searchTerm", Duration = 180)]  
     [HttpPost]  
     public ActionResult Search(string searchTerm)  
     {  
       var model = new SearchResponse();  
       //Use a stopwatch to measure the time it takes to perform the search.  
       var s = new System.Diagnostics.Stopwatch();  
       s.Start();  
       model.Results = SearchCore.Search(searchTerm);  
       s.Stop();  
       model.OriginalSearchTerm = searchTerm;  
       model.TimeTaken = s.Elapsed;  
       return View(model);  
     }  
{% endhighlight %}

## Search View

The Search view simply prints the original search term and time taken, and interates over the results, printing the link (using the RouteInfoLink extension method) and then calling a DisplayTemplate for the original object (whatever that may be). The Display Templates are stored in the DisplayTemplates subfolder of the Search folder.

{% highlight html %}
@model SimpleSearch.SearchResponse
<html>
<head>
    <title>Search Results</title>
</head>
<body>
    <h2>Search Results</h2>
    @Html.Raw(string.Format("Your search for <strong>{0}</strong> returned {1} 
  results in {2} seconds", Model.OriginalSearchTerm, Model.Results.Count(), 
  Math.Round(Model.TimeTaken.TotalSeconds, 3)))<br />
    @*Only meaningful if caching is enabled on the controller*@
    <em>@String.Format("Cached at {0}", DateTime.Now.ToShortTimeString())</em>
    @foreach (var item in Model.Results)
    {
        <section style="border: 0px none; display: block; float: none; padding: 0px">
            <h3>@item.Category - @Html.RouteInfoLink(item.Description, item.RouteInformation)</h3>
            @Html.DisplayFor(m => item.OriginatingObject)
        </section>   
    }
</body>
</html>
{% endhighlight %}

## Display Templates

The DisplayTemplates are just standard Razor partials named after the type they are strongly-typed to.

{% highlight html %}
@model Search.Models.Product

<section style="border: 0px none; display:block; float:none; padding: 0px">
    @Model.Name<br />
    <b>Short Name: </b>@Model.ShortName<br/>
    @Model.Description<br/>
    <h3>£@Model.Price</h3>

</section>
{% endhighlight %}

## Points of Interest

This is only a very basic implementation but shows how you can do really useful things with relatively little code.
