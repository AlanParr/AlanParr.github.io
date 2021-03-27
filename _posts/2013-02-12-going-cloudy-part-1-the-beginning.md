---
layout: post
title:  "Going Cloudy Part 1 - The Beginning"
date: 2013-02-12 22:08:00 +0000
categories: azure cloud migration
excerpt_separator: <!--end_excerpt-->
---

If you believe the hype, "Cloud" is the future and everything will end up there eventually.

It's easy to think that "Cloud" is just another term for deploying your applications and services on virtual servers hosted in someone else's data center, mainstream media make this mistake all the time.
<!--end_excerpt-->
Making full use of the cloud, in my opinion anyway, is architecting and designing your system to take advantage of the availability and scalability that the cloud gives you.

Designing for the cloud is one thing, migrating an existing application to the cloud and taking advantage of all it has to offer, is another entirely. Recently, I have been tasked with planning and carrying out a migration to the cloud.

The application in question is currently hosted on a VM running with just 2Gb of RAM. This VM runs IIS, which hosts 3 individual web-based components of this application. It also hosts the database on SQL Server and a number of other additional tools and services. The user base is currently fairly small, limited to a dozen or so users located in the UK, China, USA, and South Africa.

The customer base is expected to expand massively this year, with the number of users from the US pushing over a hundred and a number of users coming on board from other countries around the world.

Clearly, the current server won't handle this, and Cloud is the way forward for this customer.

## Amazon or Azure?
Amazon has a myriad of services which fill different niches and markets, but don't seem very joined up. In my opinion, while Amazon's cloud is much more fully featured and mature than Azure, it is a big bucket of disparate services trying to be everything to everyone.

Azure on the other hand, while being newer and not as feature complete as AWS, does what it does very well. All of Azure's services are aimed at allowing developers to build scalable and highly available services, the developer experience is first class and you can get started in minutes.

Microsoft finally recognizes that enticing developers is the key to success, and they have embraced other languages in Azure, creating SDKs for Node, PHP, Java, and Python. Azure is a great platform to develop for, and not just for .net developers.

So Azure it is.

Next time, we'll study the current architecture of the application and look at how it needs to change in order to make the move and meet the customer's requirements.

## Disclaimer
I'm not a cloud expert so don't take what I am doing as best practice. This series is intended to help others learn from my experiences. If something can be done better, please use the comments to impart your wisdom.
