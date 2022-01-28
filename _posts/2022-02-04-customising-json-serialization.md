---
layout: post
title:  "Customising JSON serialization with Json.Net"
date:   2022-02-04 00:00:00 +0000
tags:
  - CSharp
  - JSON.Net
category: CSharp
---

While integrating with the API of a popular tropical-sounding online shop, I hit a problem where their Sandbox API was rejecting a perfectly valid request that the main API would most likely be fine with.

In this case, it was objecting to decimals being rendered as `10.0` instead of `10`, likely because the sandbox is auto-generated from the Swagger models and was just doing a string comparison on the request and was expecting `10`.
As soon as we removed the trailing `.0`, the request was accepted.

Rather than just accept the sandbox was broken, I decided to see if it was possible to fix this, so I looked in to Converters so I could intercept serialization of the types and make my tweaks and this is what I came up with:

As per usual, this is in the form of a Linqpad script, so just add the JSON.Net Nuget package and drop in the below code.

```csharp
void Main()
{
	var x = new Foo {Bar = 10.0};
	
	Newtonsoft.Json.JsonConvert.SerializeObject(x, Newtonsoft.Json.Formatting.Indented).Dump();
}

public class Foo{
	[JsonConverter(typeof(DoubleConverter))]
	public double Bar {get;set;}
}

public class DoubleConverter : JsonConverter
{
	public override bool CanConvert(Type objectType)
	{
		return objectType == typeof(double);
	}

	public override object ReadJson(JsonReader reader, Type objectType, object existingValue, JsonSerializer serializer)
	{
		return double.Parse(existingValue.ToString());
	}

	public override void WriteJson(JsonWriter writer, object value, JsonSerializer serializer)
	{
		var asDouble = (double)value;
		var asInt = (int)asDouble;
		
		var hasFraction = asDouble - asInt;
		if(hasFraction > 0){
			writer.WriteValue(asDouble);
		}
		else{
			writer.WriteValue(asInt);
		}
	}
}
```

The converter is quite naive, I haven't yet done the work to fully exercise it to make sure it handles all scenarios.

`CanConvert` is to tell Json.Net if it can convert the incoming type, which will be true if the type is a double.

`ReadJson` is used during deserialization and simply parses the double, working on the assumption that it is valid. I'll likely improve this bit as I use it.

`WriteJson` is used during serialization, this is the bit I was really interested in. To determine if there is a zero fraction, I get the value as a double, then an int, minus the int from the double and see if I have anything left. If I do, write the double version, else write the int version.

Json.Net knows to use this converter from the attribute placed on the double property

```csharp
[JsonConverter(typeof(DoubleConverter))]
public double Bar {get;set;}
```

Without the attribute, the following Json is rendered
```json
{
  "Bar": 10.0
}
```

With the attribute, we now get the following Json
```json
{
  "Bar": 10
}
```

which should make my request succeed.