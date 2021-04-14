---
layout: post
title:  "Azure downtime"
date: 2013-02-24 21:57:00 +0000
tags:
  - Downtime
  - Azure
category: Azure
excerpt_separator: <!--end_excerpt-->
---

Azure's storage system took a turn for the worse this weekend, reportedly because they forgot to renew the SSL certificate for the storage services.
<!--end_excerpt-->
Just in the comments section of [this](http://www.theregister.co.uk/2013/02/22/azure_problem_that_should_never_happen_ever/) article, there are many comments chastising Microsoft for making such an amateur mistake (and rightly so). But there are also many who use this incident as a reason to write off cloud computing as a whole.

Letting a certificate expire is a massive cock-up, and as a customer I fully expect to see a report on the whys,hows, and what we'll do to stop it happening again from MS. However, let's not kid ourselves that cock-ups don't happen when we host and maintain these services ourselves. I work for a small company that couldn't afford to build and maintain equivalents of the Azure services ourselves. While it is frustrating and laughable that mistakes like this happen, at least when they happen on Azure, there is a whole team of highly intelligent well-paid developers and administrators working on solving the problem and preventing it from occurring again. In the on-site hosted premise, there's me. Now I am certain that if we were self-hosting, I'd be able to solve any problem that came my way although it would almost certainly take longer to fix purely due to available man hours, but I don't always have the time and resources to put in the necessary work to prevent it happening again, especially if it is something that is unlikely to recur.

If anything, incidents like this only serve to reinforce the message that cloud computing is not the silver bullet that it is often portrayed to be. It is not a suitable platform for everything nor is it devoid of fault or error. Just as you take the added expensive of the hardware, staffing, and management when hosting on-site, you also need to accept that things are to an extent out of your control when you move to cloud. Either way downtime will happen, and it will happen because someone made a stupid mistake. But at least if you're on Azure, you've got a team far more expensive than many companies could afford there to fix things when they go wrong, and also the resources to put processes and systems in place to prevent idiotic cock-ups like this from occurring again.
