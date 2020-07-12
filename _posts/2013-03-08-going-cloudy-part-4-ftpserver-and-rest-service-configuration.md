---
layout: post
title:  "Going Cloudy Part 4 - FTPServer and REST Service configuration"
date: 2013-03-08 19:29:00 +0000
categories: azure cloud migration
---

The FTP Server is a Windows Server 2012 Extra Small VM, running IIS for FTP and our in-house import/export agent.

The FTP server is used to exchange data between the application and external services. The reason for FTP is that the main external service we deal with only has that capability. We hope that external service can eventually switch to using JMS which can then be bridged to Service Bus, but for now this is all we have.

For the purposes of resilience and quick recovery after a failure, all files including the FTP folders and the executables for the agent are kept on a separate data disk and all of the configuration needed to get from brand new VM to all systems go is scripted with Powershell.

If the server ever dies, it’s just a case of firing up a new VM, adding the data disk, and running the Powershell script. I also intend to image the OS to give me the reimage option as well. The Powershell script is nothing special, but I intend to publish it for the sake of completeness.

You’ll notice from the diagram that the rest service is being directly accessed from rest.domain.com, bypassing the load balancer. You will also notice that only the EU instance is being used, the US instances are sitting there doing nothing.

There is a good reason for this.

The REST service can theoretically be accessed by any external system that is capable (as long as we grant it permission of course). Some of the imports carried out by the REST service are fairly destructive. If traffic was going through the load balancer, there is the possibility that the REST service in the US could be carrying out an import at the same time as the REST service in the EU, having some very scary consequences.

Now, in an ideal world, the REST service and the underlying data model would’ve been built to deal with this possibility. But it wasn’t, so we have to deal with that. Eventually, I intend for the REST service to simply be a receiver of files, which then puts the received data in to a queue to be picked up by a worker process, or worker processes, which WILL be designed to deal with multiple instances working at the same time without screwing everything up.

Having the entirety traffic go straight to the EU instance means that only one REST service will be taking request at any one time. It also means the US ones are doing nothing, but they use little to no resources if they are not being sent any work to do so I don’t think this is a major problem.

The only way around this would be to have a separate service definition set up for the US and other regions, which it seems to me is unnecessary duplication and extra work.

An alternative set up that I am considering is to have a second traffic manager configured for failover which is there just for the REST service. This would allow failover to the US instances if the EU ones ever became unavailable.
