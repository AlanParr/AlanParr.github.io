---
layout: post
title:  "Copying records between tables in different Azure accounts"
date: 2014-03-20 02:00:00 +0000
categories: azure nosql table-storage
excerpt_separator: <!--end_excerpt-->
---

Today I had to quickly throw up a new instance of a customer's service in Hong Kong as they've got a big demo event coming up and want things to be as quick as possible. Now I haven't quite got things to a point where I can have multiple geographically distributed instances of the service all happily talking to each other and sharing data so this instance is it's own little island, a completely separate instance to the main one in the EU.
<!--end_excerpt-->
* Deploying the new Cloud Service was easy.
* Taking a backup of the EU database and deploying it to Hong Kong was also easy.

However, recently I've been making increasing use of Azure Table storage for trivial data storage scenarios where the data isn't relational and the data will want to be shared amongst multiple instances eventually without having to wait for a database sync. It was at this point that I realised, I have no way of copying data from one storage account to another.

Time to correct that!

{% highlight csharp %}
public void Transfer<T>(Microsoft.WindowsAzure.Storage.CloudStorageAccount fromAcc, Microsoft.WindowsAzure.Storage.CloudStorageAccount toAcc, string table, Expression<Func<T,bool>> expr) where T: TableServiceEntity {
  
  var fromTC = fromAcc.CreateCloudTableClient();
  var fromT = fromTC.GetTableReference(table);
  
  var toTC = toAcc.CreateCloudTableClient();
  var toT = toTC.GetTableReference(table);
  toT.CreateIfNotExists();
  
  var fromContext = fromTC.GetTableServiceContext();
  var toContext = toTC.GetTableServiceContext();
    
  var fromData = fromContext.CreateQuery<T>(table).Where(expr);

  foreach (var item in fromData)
  {
    toContext.AttachTo(table,item);
    toContext.UpdateObject(item);
  }
  toContext.SaveChangesWithRetries(SaveChangesOptions.ReplaceOnUpdate);
}
{% endhighlight %}

This Transfer method takes in a from account and a to account and the name of the table.

The last parameter is an expression for the where clause. This is for scenarios where the same table contains multiple types of objects and you just want to query out the ones of a particular type for transfer using whatever clause is appropriate.

T must derive from TableServiceEntity and be the type of the object from which the record originated, or one that is similarly shaped.

The method is quite straight forward, it just fires up 2 table clients, gets a reference to the table specified by the table parameter, creates it on the receiving end if it doesn't exist (I think it's safe to assume that it already exists at the source end), queries out the data, then attaches it to the destination context and saves changes.

This upserts all of the data in to the source table.

Usage is simple:
{% highlight csharp %}
var fromAccount = Microsoft.WindowsAzure.Storage.CloudStorageAccount.Parse("DefaultEndpointsProtocol=https;AccountName=accountname;AccountKey=accesskey");
var toAccount = Microsoft.WindowsAzure.Storage.CloudStorageAccount.Parse("DefaultEndpointsProtocol=https;AccountName=accountname;AccountKey=accesskey");

Transfer<MyProject.MyType1>(fromAccount, toAccount, "sharedtable", p=>p.PartitionKey == "Type1");
Transfer<MyProject.MyType2>(fromAccount, toAccount, "sharedtable", p=>p.PartitionKey == "Type2");
Transfer<MyProject.SomeOtherType>(fromAccount, toAccount, "someothertype", p=>p.PartitionKey != "");
{% endhighlight %}

Put all this together in Linqpad and you've got a simple way to transfer records between accounts on an ad hoc basis. As expected, it works with the Storage Emulator so you can use it to clone the contents of a production account down to your local dev machine and vice versa.
