---
layout: post
title:  "Copying records between tables in different Azure accounts : The Next Generation"
date: 2014-10-09 10:04:00 +0000
categories: azure tablestorage
excerpt_separator: <!--end_excerpt-->
---

A few months back I posted a small piece of code for copying objects between Azure Table Storage Accounts. I have 2 bug-bears with the code I previously posted.
<!--end_excerpt-->
* It's all very custom, you need to give it the type that is in the table and add lambdas to make sure it doesn't try to select the wrong object.
* Due to a change in the Azure Storage Library, it no longer works.

 I hadn't used this script in a while but now I have an impending need for something like it but better that will allow me to copy the contents of all the tables or a defined subset of tables in a given account, enter the DynamicTableEntity which allows you to get an entry from a table as a dynamic object.

To run the code, open up Linqpad and add a reference to the Windows Azure Storage library, best way to do this is using Nuget.

{% highlight csharp %}
void Main()
{
  var srcClient = CreateClient("source account connection string");
  var destClient = CreateClient("destination account connection string");
  
  var mappings = new List<Tuple<string,string>>();
  
  //Manually setup mappings.
  //mappings.Add(new Tuple<string,string>("table1","table1copy"));
  //mappings.Add(new Tuple<string,string>("table2","table2copy"));
  
  //Copy all tables from the src account in to identically named tables in the destination account.
  var tables = srcClient.ListTables(null, new TableRequestOptions(){PayloadFormat = TablePayloadFormat.JsonNoMetadata});
  mappings = tables.Select (t => new Tuple<string,string>(t.Name,t.Name)).ToList();

  Copy(srcClient,destClient,mappings);
}

public void Copy(CloudTableClient src, CloudTableClient dest, List<Tuple<string,string>> mappings) {

  mappings.ForEach(x=>{
    var st = src.GetTableReference(x.Item1);
    var dt = dest.GetTableReference(x.Item2);
    dt.CreateIfNotExists();
    
    var query = new TableQuery<DynamicTableEntity>();

    foreach (var entity in st.ExecuteQuery(query))
    {
      dt.Execute(TableOperation.InsertOrReplace(entity));
    }
  });
}

public CloudTableClient CreateClient(string connString){
  var account = CloudStorageAccount.Parse(connString);
  return account.CreateCloudTableClient();
}
{% endhighlight %}

In the above code, we create CloudTableClients to represent the source and destination accounts, then we build a mapping of source and destination tables.

We can do this manually if we only want to copy some tables and/or we want the destination tables to have different names to the source tables.

Alternatively, we can get a list of all the tables from the source and use that to build a 1:1 map, this will have the result of copying all items in all tables from the source account to the destination.

The Copy method simply takes the clients and mapping and does some iteration to get the items from each table from the source account and save them to the destination account.

**Note:** The code above is horribly inefficient for copying large amounts of data as it inserts each request individually. In a follow up post, I'll make this more efficient by making use of the TableBatchOperation.
