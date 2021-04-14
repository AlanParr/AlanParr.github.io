---
layout: post
title:  "Speeding up Azure debugging your MVC application"
date: 2012-03-24 18:25:00 +0000
tags:
  - Azure
  - Debugging
  - WebRole
category: Azure
excerpt_separator: <!--end_excerpt-->
---

When debugging an application that is intended for an Azure Web Role, you debug using the Compute and Storage emulators in order to give you a realistic Azure experience. These tools used to take a long time to initialize on Build, meaning debugging your app could take significantly longer than normal. Even though the tools have become a lot quicker, there is still an additional delay when debugging using them.
<!--end_excerpt-->
If, like me, you have just migrated a bog-standard web application to Azure and are not yet using any of the specific Azure features, essentially treating Azure as a web server and nothing else, there is something you can do to speed things up.

In your solution, simply change the start up project from the Azure Web Role to the actual MVC project that is going to be deployed in the Azure Web Role. This way you can debug in the normal way and then deploy to Azure when ready.

Note however, that if you start using any of the Azure features such as Blob, Table or Queue storage, then you just need to change the Start Up Project back to the Azure Web Role project

![Azure Compute Emulator](\images\azurecompstor.jpg)
