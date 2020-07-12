---
layout: post
title:  "API Head-to-head: AWS S3 Vs Windows Azure Table Storage"
date: 2014-08-06 21:52:00 +0000
categories: aws azure s3 azurestorage
---

Recently, I was experimenting with using S3 as a tertiary backup for my photos, an honour which eventually went to Azure as it was cheaper and I am more familiar with the Azure APIs as I use them in my day job.

I thought I’d take a deeper look at both APIs and see how they compare. I’ll go through some standard operations, comparing the amount of code required to perform the operation.

If you want a comparison of features, there are plenty of blog posts on the subject, just [Bingle It](http://www.bingle.nu/results.php?type=www&query=AWS%20S3%20VS%20Azure%20Blob%20Storage)

All the code in this test is being run in [Linqpad](http://www.linqpad.net/), using the [AWS SDK for .Net](https://www.nuget.org/packages/AWSSDK/) and [Windows Azure Storage](https://www.nuget.org/packages/WindowsAzure.Storage/) Nuget packages.

## Create the client

Both Azure and S3 have the concept of a client, this represents the service itself and is where you provide credentials for accessing the service.

### Azure
{% highlight csharp %}
var account = Microsoft.WindowsAzure.Storage.CloudStorageAccount.Parse("connectionstring");
var client = account.CreateCloudBlobClient();
{% endhighlight %}

### S3
{% highlight csharp %}
var client = AWSClientFactory.CreateAmazonS3Client("accessKey", "secret",RegionEndpoint.EUWest1);
{% endhighlight %}

S3 wins on lines of code but I don’t like having to declare the datacenter the account is in. In my opinion, the application shouldn’t be aware of this. 1 point to Azure.

## Creating a container

This is a folder, Azure refers to is a container, S3 calls it a bucket.

### Azure
{% highlight csharp %}
var container = client.GetContainerReference("test-container");
container.CreateIfNotExists();
{% endhighlight %}

### S3
{% highlight csharp %}
try
{         
 client.PutBucket(new PutBucketRequest { BucketName = "my-testing-bucket-123456", UseClientRegion = true});
}
catch (AmazonS3Exception ex)
{
 if(ex.ErrorCode != "BucketAlreadyOwnedByYou") {
  throw;
 }
}
{% endhighlight %}

S3 loses big time on simplicity here. To my knowledge, this is the only way to do a blind create of a container, that is creating it without knowing up front if it already exists. Azure makes this trivial with CreateIfNotExists. 2 points to Azure.

## Uploading a file

### Azure
{% highlight csharp %}
var container = client.GetContainerReference("test-container");
var blob = container.GetBlockBlobReference("testfile");
blob.UploadFromFile(@"M:\testfile1.txt",FileMode.OpenOrCreate);
{% endhighlight %}

### S3
{% highlight csharp %}
var putObjectRequest = new PutObjectRequest {BucketName = "my-testing-bucket-123456", FilePath = @"M:\testfile.txt", Key = "testfile", GenerateMD5Digest = true, Timeout=-1};
var upload = client.PutObject(putObjectRequest);
{% endhighlight %}

They’re pretty much equal here, but the S3 code is more verbose. I like the idea of getting a reference to a blob while not knowing if it actually exists or not.

## List Blobs

### Azure
{% highlight csharp %}
var container = client.GetContainerReference("test-container");
var blobs = container.ListBlobs(null, true, BlobListingDetails.Metadata);
blobs.OfType().Select (cbb => cbb.Name).Dump();
{% endhighlight %}

### S3
{% highlight csharp %}
var listRequest = new ListObjectsRequest(){ BucketName = "my-testing-bucket-123456"};
client.ListObjects(listRequest).S3Objects.Select (so => so.Key).Dump();
{% endhighlight %}

In terms of complexity, they’re pretty even here too. Azure has one more line but it’s not a difficult one. Notice that whereas with Azure, we get a reference to a container and then perform operations against that reference, with AWS all requests are individual so you end up having to explicitly tell the client for every operation what the bucket name is. Point to Azure.

## Delete a Blob

### Azure
{% highlight csharp %}
var dblob = container.GetBlockBlobReference("testfile");
dblob.Delete();
{% endhighlight %}

### S3
{% highlight csharp %}
var delRequest = new DeleteObjectRequest(){ BucketName = "my-testing-bucket-123456", Key="testfile"};
client.DeleteObject(delRequest);
{% endhighlight %}

Neither code is particularly complicated here, but I prefer Azure’s simplicity with the container and blob reference model so point Azure.

## Delete a container

### Azure
{% highlight csharp %}
var container = client.GetContainerReference("test-container");
container.Delete();
{% endhighlight %}

### S3
{% highlight csharp %}
var delBucket = new DeleteBucketRequest(){ BucketName = "my-testing-bucket-123456"};
client.DeleteBucket(delBucket);
{% endhighlight %}

Again, pretty equal. To micro-analyse the lines, you could say that for Azure, you’ve got one potentially reusable line, and one throw-away line. With S3, they’re both throw away. But in reality, unless you’re doing thousands of consecutive operations, it doesn’t really matter.

## Conclusion
In terms of complexity, Azure’s and S3’s APIs are pretty much equal, but it’s easy to see where they each have their uses. Azure’s API is a much thicker abstraction over REST, whereas the S3 API is such a thin-veneer that you could imagine a home-grown API not turning out that differently (but most likely not as reliable).

In my mind, if you’re doing lots of operations against lots of different blobs and containers then S3’s API is more suitable as each operation is self-contained and there are no references to containers or blobs hanging around.

If you’re doing operations which share common elements, such as performing numerous operations on a blob or working with lots of blobs within a few containers, Azure’s API seems better suited as you create the references and then reuse them, reducing the amount of repeated code.

## Bonus Section
If you could be bothered to read past my conclusion, congratulations on your determination! The comparative speed of Azure and AWS has been done to death, but I couldn’t resist getting my own stats.

These are ridiculously simple stats, essentially Stopwatch calls wrapped around the code in this post. The file I am uploading is only 6k. The simple reason for this is that everyone tests how these services handle lots of large objects, but no one seems to cover the probably more common scenario of users uploading very small files. The average size is probably higher than 6kb, but this is what I’ve got hanging around so this is what I’m using.

So here are my extremely simple and probably not at all reliable benchmarks.

|Operation|S3|Azure|
|-|-|-|
|Create Container|573|279|
|Upload 7Kb file|99|55|
|List Blobs (1)|41|103|
|Delete Blob|55|45|
|Delete Container|221|38|

All times are in milliseconds. I’ve got to admit; I was expecting a more even spread here. Azure is significantly faster creating and deleting containers and uploading the file. It is also faster at deleting a blob, but the difference is insignificant. S3 wins significantly listing blobs.

Not covered in this post: Both APIs also have the Begin/End style of async operations and Azure has the bonus of async operations based on the async/await pattern, I may do another post on that in the future.

**TL;DR;** Azure's API is in my opinion a better abstraction and it's faster for most operations.