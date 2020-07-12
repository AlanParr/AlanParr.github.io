---
layout: post
title:  "API Head-to-head Update : AWS S3 Vs Windows Azure Table Storage Vs Rackspace Cloud Files"
date:   2020-06-13 13:29:00 +0000
categories: api rackspace azure aws
---

This is an update to my last API head to head from August 2014, I'm nothing if not consistent with my inconsistent posting. I've recently changed jobs to a new company that is moving to Azure but has some legacy Rackspace assets, so I thought it'd be fun to redo the test with Rackspace added. Worth noting that the Rackspace support for C# is completely non-existent. The official Rackspace SDK hasn't been updated since 2013 and this test is using the openstack.net SDK which hasn't been updated since 2018.

## The code

### Initialise Provider

{% highlight csharp %}
CloudIdentity cloudIdentity = new CloudIdentity()
{
 APIKey = "mykey",
 Username = "myusername"
};
var provider = new CloudFilesProvider(cloudIdentity);
{% endhighlight %}

### Create container

{% highlight csharp %}
provider.CreateContainer(containerName);
{% endhighlight %}

### Upload file

{% highlight csharp %}
provider.CreateObjectFromFile(containerName, testFile, blobName);
{% endhighlight %}

### List Blobs (objects in Rackspace parlance)

{% highlight csharp %}
provider.ListObjects(containerName).ToList();
{% endhighlight %}

### Delete file

{% highlight csharp %}
provider.DeleteObject(containerName, blobName);
{% endhighlight %}

### Delete container

{% highlight csharp %}
provider.DeleteContainer(containerName);
{% endhighlight %}

Can't argue with the simplicity of the code, once you've initialised the provider, everything is one line.

## The results

Worth noting that I don't have the original test file, so the file being uploaded here is [this one](/images/odo.jpg) I happened to have lying around. It is only 1k larger, so don't expect it will have a massive effect on the results.

|Operation|S3|Azure|Rackspace|
|-|-|-|-|
|Create Container|847|668|1915|
|Upload 7Kb file|83|92|230|
|List Blobs (1)|40|172|51|
|Delete Blob|47|35|112|
|Delete Container|240|45|68|

I ran the tests a few times and the numbers fluctuated but the positions pretty much stayed the same, except the Upload file winner switched between S3 and Azure.

## Conclusion
Rackspace was slowest, but that isn't terribly surprising considering I was using a third-party 2-year old library, it isn't necessarily a realistic comparison, just for my own amusement.