---
layout: post
title:  "JSON vs XML: Challenging my assumptions"
date: 2013-12-21 15:12:00 +0000
categories: json optimisation xml
---

I was recently (today actually) working on optimising a particular section of my project. This section is basically a Q & A that uses a piece of server-generated XML which is placed in to a Razor view where some Javascript works with it to generate the input fields for the user.

But XML is so 2010 right? If I swapped the XML for some JSON the payload would be smaller, it would generate faster, the javascript would be able to work with it faster, right?

Let's test those assumptions one by one shall we?

To do that, I duplicated the method that populates an object and serializes it to XML and changed the serialization to use JSON instead and ran them both at the same time so I could compare them side by side.

## Payload size

I added a call to Encoding.ASCII.GetByteCount() around the serialized JSON and XML results and dumped this to the Trace windows. The results are below.

|Type|1|2|3|4|5|6|7|
|-|-|-|-|-|-|-|-|
|XML|3338|4133|3487|3255|3465|3194|1138|
|Json|2266|2717|2292|2110|2234|2061|621|

This is in bytes, so the real world difference between the XML and the JSON in this case is at most a single kilobyte. This isn't always the case as I've swapped out XML for JSON in the past where it has been many times smaller.

## Generating the output
Next job is to test how quickly the output is generated. Keep in mind that the method I'm testing includes other data access logic so these timings are not purely serialization. But as both will be doing the same work, it won't hurt to have that included as well to see how each performs in the real world.

|Type|1|2|3|4|5|6|7|Avg|
|-|-|-|-|-|-|-|-|-|
|XML|87.4|31.9|33.6|29.6|31.6|29.6|45.9|41.37143|
|Json|38.9|20.7|27.8|21.2|23.8|24.4|23.5|25.75714|

Across the whole Q & A session, JSON comes out not far off 50% faster. So I should definitely swap it right?

But wait, the method that we're capturing also includes some data access. The JSON method is running second, could it be benefiting from the XML method's work?

To test this out, I swapped them round so the JSON method was first. Below is the aggregated table from both runs.

|Type|1|2|3|4|5|6|7|Avg|
|-|-|-|-|-|-|-|-|-|
|XML First|87.4|31.9|33.6|29.6|31.6|29.6|45.9|41.37143|
|XML Second|39.2|25.3|26.7|24.3|26.5|37.1|21.6|28.67143|
|Json First|83.5|28.9|30.4|30.9|32.7|46.3|47.7|42.91429|
|Json Second|38.9|20.7|27.8|21.2|23.8|24.4|23.5|25.75714|

As you can see from the average, I was right to be suspicious. The first method to run is always slower than the second due to things like EF caching records. In reality, the difference between the 2 is so small that no one is ever going to notice.

These two simple benchmarks which took me about 10 minutes to complete shows that to substitute the current XML payload for JSON would be a micro-optimisation. I've got much bigger fish to fry than this so it is not worth my time to do this work.

 Will swapping out make things faster? Absolutely. Will anyone notice? Not remotely.

You'll notice I didn't perform the third test of measuring how the Javascript handled JSON as opposed to XML. I'm sure that would probably show a modest improvement as well but based on these numbers, my best course of action is to not waste any more time on this path.
