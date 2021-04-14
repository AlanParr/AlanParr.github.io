---
layout: post
title:  "Slimming down your JSON"
date: 2013-07-18 22:18:00 +0000
tags:
  - EntityFramework
  - Json.Net
  - Serialization
category: Serialization
excerpt_separator: <!--end_excerpt-->
---

Newtonsoft JSON.Net is the JSON serialization library that is so good, Microsoft use it over their own.
<!--end_excerpt-->
While converting an existing project from Linq to SQL to EF5 Code First, I hit an issue with the Unit Tests, which use test objects serialized to XML files as the basis of the tests.  This upset the XML Serializer as collections in EF are ICollection<T> as opposed to Linq to SQL's EntitySet<T> – and the XML Serializer can’t handle interfaces.

JSON.Net to the rescue - fortunately it can handle interfaces so I chose to convert the test data to serialized Json instead. This was a relatively trivial task, accomplished by the below Linqpad script (if you're not using [Linqpad](http://www.linqpad.net/), stop reading this blog and go and download it now).

{% highlight csharp %}
void Main()
{
       var filePath = @"C:\users\Alan\Downloads\";
       var filename = "OrderItemTestData.json";
       var deSerializationMode =2;
       //1 for Json deserialize to object, 2 for cast (use for derived types of abstract classes where return type is the base class).
       var update=false;

var result = LoadData<List<OrderItemBase>>(filePath + filename,deSerializationMode);

       var updated = JsonConvert.SerializeObject(result, Newtonsoft.Json.Formatting.Indented,
       new JsonSerializerSettings() { ReferenceLoopHandling = ReferenceLoopHandling.Ignore, 
TypeNameHandling = TypeNameHandling.All, 
NullValueHandling= NullValueHandling.Ignore});
      
       updated.Dump();
if(update){  
       File.WriteAllText(filePath + filename,updated);
}
}
{% endhighlight %}

This simply takes the XML files, deserializes it to the source objects (that’s all LoadData does), and then reserializes it to JSON.

This solved my immediate problem but I was surprised to see the size of the file, in some cases 3 times larger than the equivalent XML file. But JSON is less characters, so how is that possible? A simple comparison of the files shows the problem:

### Excerpt from XML file
{% highlight xml %}
  <Company>
    <Id>201</Id>
    <Name>Company 201</Name>
    <Code>foo</Code>
    <PlaquePrice>5</PlaquePrice>
    <FreeSampleCount>10</FreeSampleCount>
    <CompanyDispensers>
      <CompanyDispenser>
        <Dispenser>
          <CompanyId>101</CompanyId>
        </Dispenser>
      </CompanyDispenser>
    </CompanyDispensers>
  </Company>
{% endhighlight %}

### Excerpt from JSON file
{% highlight json %}
 "$type": "MyCompany.Model.Customer, MyCompany.Model",
      "IsInternal": false,
      "AvailableWorkFlows": {
        "$type": "System.Collections.Generic.List`1[[System.String, mscorlib]], mscorlib",
        "$values": [
          "StandardWorkFlow"
        ]
      },
      "BillingAddress": null,
      "BillingContact": null,
      "PrimaryContact": null,
      "ActiveUsers": {
        "$type": "MyCompany.Model.User[], MyCompany.Model",
        "$values": []
      },
      "Id": 201,
      "Name": "Company 201",
      "Code": "foo",
      "RecordUpdate": "0001-01-01T00:00:00",
      "RecordCreate": "0001-01-01T00:00:00",
      "Orders": {
        "$type": "System.Collections.Generic.List`1[[MyCompany.Model.Order, MyCompany.Model]], mscorlib",
        "$values": []
      },
      "Products": {
        "$type": "System.Collections.Generic.List`1[[MyCompany.Model.Product, MyCompany.Model]], mscorlib",
        "$values": []
      },
      "Users": {
        "$type": "System.Collections.Generic.List`1[[MyCompany.Model.User, MyCompany.Model]], mscorlib",
        "$values": []
      },
      "IsValid": true
    } 
{% endhighlight %}

In this case, the XML file comes out at 1617 characters, the Json equivalent is 48833 characters

The JSON.Net serializer has gone over the objects and serialized every property, even ones that were null or default for their type and also empty collections. This can be easily solved by setting the appropriate properties on the serializer:

{% highlight csharp %}
new JsonSerializerSettings() { ReferenceLoopHandling = ReferenceLoopHandling.Ignore, 
          TypeNameHandling = TypeNameHandling.All, 
          NullValueHandling= NullValueHandling.Ignore, 
          DefaultValueHandling = DefaultValueHandling.Ignore}
{% endhighlight %}

Setting NullValueHandling and DefaultValueHandling to Ignore solves the problem of properties that are at the default for their type, such as datetimes, and null properties. However, this still leaves us with all of the collections, which are initialised in the object’s constructor to new List<T>.

By default, Json.Net can’t be instructed to ignore these empty lists, because ignoring them may not be the correct action in everyone’s case. In ours it is, so we need to tell Json.Net that it is okay to ignore them to reduce our file size.

To do this we need to create a custom DefaultContractResolver, the code is below:

{% highlight csharp %}
public class IgnoreEmptyCollectionsContractResolver : DefaultContractResolver
{
    public new static readonly IgnoreEmptyCollectionsContractResolver Instance = new IgnoreEmptyCollectionsContractResolver();

  protected override JsonProperty CreateProperty(MemberInfo member, MemberSerialization memberSerialization)
  {
    JsonProperty property = base.CreateProperty(member, memberSerialization);

    if ((property.PropertyType.Name.Contains("IEnumerable") || property.PropertyType.Name.Contains("ICollection")) && property.PropertyType.GenericTypeArguments.Count() == 1)
    {
      property.ShouldSerialize = instance =>
         {
         try{
              var cnt = instance.GetType().GetProperty("Count").GetValue(instance,null);
              return (int)cnt > 0;
              }
         catch(NullReferenceException){
         return false;}
         };
    }
    return property;
  }
}
{% endhighlight %}

The <catchyName>IgnoreEmptyCollectionsContractResolver</catchyName> simply checks if the current property is an ICollection or IEnumerable and that it has a single generic argument. It then checks the Count property and instructs Json.Net to serialize/deserialize that property depending on whether or not count is greater than 0. I’m sure this can be done a lot neater, but it solves my problem.

We then simply instruct Json.Net to use this as part of the JsonSerializerSettings object:

{% highlight csharp %}
new JsonSerializerSettings() { ReferenceLoopHandling = ReferenceLoopHandling.Ignore, 
    TypeNameHandling = TypeNameHandling.All, 
    NullValueHandling= NullValueHandling.Ignore, 
    DefaultValueHandling = DefaultValueHandling.Ignore, 
    ContractResolver = new IgnoreEmptyCollectionsContractResolver()});
{% endhighlight %}

Now the serialized Json looks like this:

{% highlight csharp %}
{
      "$type": "MyCompany.Model.Customer, MyCompany.Model",
      "FreeSampleCount": 10,
      "PlaquePrice": 5.0,
      "AllDispensers": {
        "$type": "MyCompany.Model.Dispenser[], MyCompany.Model",
        "$values": []
      },
      "Id": 201,
      "Name": "Company 201",
      "Code": "foo",
      "IsValid": true
    }
{% endhighlight %}

The total size has dropped from nearly 50000 characters to 1495, a much more acceptable size.

I hope this is of use to someone. If the resolver can be done in a better way, let me know.
