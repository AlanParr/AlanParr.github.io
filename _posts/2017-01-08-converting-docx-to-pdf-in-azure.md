---
layout: post
title:  "Converting docx to PDF in Azure"
date:   2017-01-08 13:26:00 +0000
tags:
  - Azure
  - PDF
category: Azure
excerpt_separator: <!--end_excerpt-->
---

As part of a recent feature, we needed to implement conversion of docx to pdf. A quick look on Nuget revealed [Free Spire.Doc for .Net](http://www.nuget.org/packages/FreeSpire.Doc/). We thought this looked like a great use case for Azure Functions, so the member of the team who was implementing the feature quickly implemented this as an Azure Function, but when we deployed it, it didn't work. After a bit of googling, the cause of this was that, due to the sandboxed environment of Azure App Service, assemblies requiring access to GDI don’t work. Further Googling with Bing revealed that pretty much every .Net docx to PDF conversion library uses GDI, so we couldn’t just switch to a different library.
<!--end_excerpt-->
We needed full access to Windows to do this, which meant a VM. The cheapest Windows VM in azure is a Basic A0 at less than £9 a month, much cheaper than commercial document conversion services I found, which were at least £20 a month and had really weird APIs that were going to be pretty tricky to integrate in to our application.

I implemented a Windows service using Topshelf and the original Free Spire.Doc code for the actual conversion and installed this on to the VM. It simply polls an Azure Storage Queue for a message and deserializes the body to the following class.

{% highlight csharp %}
private class ConversionMessage
{
    public string SourceBlobContainer { get; set; }
    public string SourceBlobName { get; set; }
    public string DestinationBlobContainer { get; set; }
    public string DestinationBlobName { get; set; }
    public string ConversionType { get; set; }
}
{% endhighlight %}

This simply contains the Azure Blob container and the name for the source document, and the destination container and name for the converted document. There is also a ConversionType property which only has one valid value currently, I added this to facilitate adding other conversions in the future. When a message is received, the service then converts the document with freespire and puts the converted document in the destination container. Below is all the code for doing the conversion and saving it.

{% highlight csharp %}
private void Convert(ConversionMessage message)
        {
            Console.WriteLine(message);

            var inputBlob = GetBlobReference(message.SourceBlobContainer, message.SourceBlobName);
            var outputBlob = GetBlobReference(message.DestinationBlobContainer, message.DestinationBlobName);
            
            if(message.ConversionType == "docxtopdf")
            {
                LogInfo("Beginning conversion, type: docxtopdf");
                ConvertDocxToPdf(inputBlob, outputBlob);
            }
            else
            {
                LogError($"Invalid conversion type {message.ConversionType} received");
            }
        }

        private CloudBlockBlob GetBlobReference(string container, string blobName) => _blobClient.GetContainerReference(container).GetBlockBlobReference(blobName);

        private void ConvertDocxToPdf(CloudBlockBlob inputDoc, CloudBlockBlob outputDoc)
        {
            var inputStream = new MemoryStream();
            inputDoc.DownloadToStream(inputStream);

            inputStream.Seek(0, SeekOrigin.Begin);

            var doc = new Spire.Doc.Document();
            doc.LoadFromStream(inputStream, FileFormat.Docx);

            var outStream = new MemoryStream();
            doc.SaveToStream(outStream, FileFormat.PDF);

            outStream.Seek(0, SeekOrigin.Begin);

            outputDoc.UploadFromStream(outStream);

            LogInfo("Conversion successful");
        }
{% endhighlight %}

Holding all of this together is an Azure Function. This function is really simple, it just gets called whenever the docx file is created in Azure Blob Storage and creates the conversion message, and puts it in the queue for the Windows service on the VM to pick up.

{% highlight csharp %}
public static void Run(CloudBlockBlob myBlob, CloudQueue queue, TraceWriter log)
{
    log.Info($"ConvertWordQuoteToPdf function processed: {myBlob.Name}");

    var filename = System.IO.Path.GetFileNameWithoutExtension(myBlob.Name);
    var cm = new ConversionMessage();
    cm.SourceBlobContainer = "docs";
    cm.SourceBlobName = $"{filename}.docx";
    cm.DestinationBlobContainer = "docs";
    cm.DestinationBlobName = $"{filename}.pdf";
    cm.ConversionType = "docxtopdf";

    var msg = new CloudQueueMessage(Newtonsoft.Json.JsonConvert.SerializeObject(cm));
    queue.AddMessage(msg);
}
{% endhighlight %}

One of the coolest things about this in my view, is that all of this required no changes to the main application at all, we just reacted to the creation of the docx file that it was already doing.
