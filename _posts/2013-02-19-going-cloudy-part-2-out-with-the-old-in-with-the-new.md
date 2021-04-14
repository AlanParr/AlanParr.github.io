---
layout: post
title:  "Going Cloudy Part 2 - Out with the old, in with the new"
date: 2013-02-19 20:07:00 +0000
tags:
  - Cloud
  - Migration
  - Azure
category: Azure
excerpt_separator: <!--end_excerpt-->
---

In my previous post, I briefly went over the current situation and the rationale for making the move to Microsoft’s Azure service.
<!--end_excerpt-->
I also promised details on the current architecture and what I have determined to be the new architecture. I will also attempt to explain how I got from old to new and why I have made the decisions I have made.

## The old architecture
Here’s a quick diagram:
![Old Architecture](\images\old-architecture.png)

As stated in my previous post, this server is a single VM running 2GB of RAM and a couple of Ghz CPU. With those resources, for the live environment we are running:

* ASP.Net MVC Web site.
* ASMX Web Service.
* ASP.Net MVC-based REST service.
* SQL Server database.

This is duplicated in the staging site which is located on the same server. On top of this, there is also SQL Server itself, an FTP Server running through IIS, an in-house import/export agent which processes all incoming and outgoing data.

There are also a number of support tools, there is Linqpad, SSMS,Notepad++, off-site file backup, and a number of other minor services.

What this all means is that, whenever I RDP on to change anything, this creates a notable strain on the system.

Bear in mind that the current service doesn’t have many users, say a dozen or so. This will increase by 10x this year so, clearly, the current setup just will not do.

## Key requirements

The new infrastructure has a number of key requirements. Other than the obvious speed, reliability,etc, there is also the following.

### Speed must be the same regardless of where the user is.

This means website and web service hosted in both EU and US. This also means two databases which need to be kept in sync.

### Needs to respond to rises and falls in demand.

This is what cloud does best. We can add and remove instances easily in a matter of minutes. There is also the possibility of using the Scaling Application Block to drop instances down to having a single server when it is night time in their region, halving the service costs during those times.

## The new infrastructure

![New Architecture](\images\new-architecture.png)

The new infrastructure is distributed, fairly fault tolerant and should be easy enough to scale - all those cloudy terms that get overused. Over the next few posts, I will go over my reasoning for laying it out this way and problems I have encountered while creating the infrastructure. I hope to do that in approximately this order.

* Changes to URL scheme.
* FTP server and REST service configuration.
* Instance configuration, the Monitoring project and the Azure Traffic Manager.
* SQL Azure and using Data Sync.
