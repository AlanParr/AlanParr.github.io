---
layout: post
title:  "Copying messages between SQS queues"
date:   2021-09-08 00:00:00 +0000
tags:
  - AWS
  - SQS
  - Linqpad
category: AWS
---

This week I had the need to copy a load of messages from one SQS queue to another, below is the body of the Linqpad script I used to do it.

```csharp
void Main()
{
	var seenIds = new List<string>();
	var sidContainer = new DumpContainer("Seen Ids: ").Dump();
	var sqsConfig = new AmazonSQSConfig { ServiceURL = "AWS_REGION_URL" };
	var client = new AmazonSQSClient("AWSKEY", "AWSSECRET", sqsConfig);

	var sourceQueueUrl = client.GetQueueUrlAsync("NAME_OF_SOURCE_QUEUE").Result.QueueUrl.Dump();
	var destinationQueueUrl = client.GetQueueUrlAsync("NAME_OF_DESTINATION_QUEUE").Result.QueueUrl.Dump();

	var rmr = new ReceiveMessageRequest { QueueUrl = sourceQueueUrl, MaxNumberOfMessages = 10, VisibilityTimeout = 900, WaitTimeSeconds = 10 };
	var messages = client.ReceiveMessageAsync(rmr).Result;

	do
	{
		$"Found {messages.Messages.Count} messages".Dump();
		foreach (var msg in messages.Messages.Where(x => !seenIds.Contains(x.MessageId)))
		{
			SendToDestinationQueue(client, destinationQueueUrl, msg.Body);
			seenIds.Add(msg.MessageId);
			sidContainer.Content = $"Seen Ids: {seenIds.Count}";
		}
		messages = client.ReceiveMessageAsync(rmr).Result;
	} while (messages.Messages.Count > 0);
}

void SendToDestinationQueue(AmazonSQSClient client, string queueUrl, string body)
{
	var response = client.SendMessageAsync(queueUrl, body).Result;
}
```
This just gets the urls for both queues and downloads messages from the source queue in batches of 10 until no more are returned.
For each batch returned, it creates a new message from that message body and sends it to the destination queue.

The seenIds variable just stores all previously seen message ids so if the visibility time expires of any messages that have already been processed while the script is still running, those won't be resubmitted to the destination queue.

Note that in the above I am using all of the Async methods synchronously. For some reason, Linqpad was not executing properly with async on the day I was going this so I just made it synchronous. It seems to be working fine now but I just haven't gotten around to `re-async-ifying` the script. This is just a one-off utility script, please don't do this in Prod!

If you want to use this yourself, you'll want to change the following placeholders in the script:

* **AWS_REGION_URL** - The url for the AWS region you are using.
* **AWSKEY** - Your key for AWS.
* **AWSSECRET** - Your secret for AWS.
* **NAME_OF_SOURCE_QUEUE** - The name of the queue as viewed in the AWS console (i.e. not the full url).
* **NAME_OF_DESTINATION_QUEUE** - The name of the queue as viewed in the AWS console (i.e. not the full url).