---
layout: post
title:  "Going Cloudy Part 5 - Scalability without breaking the bank"
date: 2013-03-17 16:15:00 +0000
categories: azure cloud vsperf.exe
---

As detailed previously, the application consists of three major components:

* The web site, which predictably is used by everyone.
* The asmx web service which is used by about 80% of users but performs fairly low-resource actions.
* The REST service is hit only by external systems importing/retrieving data on a schedule, so it doesn’t need a lot of resources either.

Going with the default cloud approach of separating everything would be prohibitively expensive. Currently a 2 instance cloud service is about £18 a month. Add on the monitoring project and this means 4 cloud services, giving me an £80 quid bill every month on servers whose utilisation would be pretty poor.

The smart thing to do is merge them all on to a single cloud service. It’s important that merging multiple sites/services on a single cloud service is only done where appropriate. Putting lots of services on the same instance when doing so will result in a shortage of resources defeats the point of moving to Cloud hosting. In this instance, they are all low-resource services which for the foreseeable future will happily sit on the same instance without problem. When traffic increases to the point where either of the services needs to be separated out to an instance of its own, this will be easy to do when required. The details of how to merge all of the services in to a single instance are detailed in my previous post.

 During initial deployment, I was getting some really appalling performance on all services, which I initially put down to the traffic manager. Using Remote Desktop to log on to one of the instances revealed that there was a process called VSPerf.exe running which was sapping all of the server’s CPU and a great deal of its RAM. There was one of these for each site initially (so 4) and then another 1 for each site was added whenever I did an upgrade deployment. This resulted in one instance where I had 16 VSPerfs running, it took me nearly 15 minutes to remote on to the server to kill them!

I could only find some anecdotal references to this problem on the internet which revealed no solutions other than to reboot after every deployment. Eventually I worked out that I had inadvertently turned on profiling at some point. I’d accidentally turned this on and didn’t need it, so I turned it off and the problem was solved. I’m sure the majority of users of profiling do not have this issue, but if you turn on profiling and then your performance falls through the floor, take a look at Task Manager and see if you are one of the unlucky few.