---
layout: post
title:  "Upgraded to Azure Storage Emulator 3.2, where have all my tables gone?"
date: 2014-07-18 15:05:00 +0000
tags:
  - StorageEmulator
  - Azure
category: Azure
excerpt_separator: <!--end_excerpt-->
---

In an attempt to solve a 400 error accessing tables on the Azure Storage Emulator 3.0 today, I upgraded to 3.2 using the Web Platform Installer. This resulted in a kind of good news, bad news situation.
<!--end_excerpt-->
**Good** - The error stopped happening.

**Bad** - Where the f**k have all my tables gone!

I'll be buggered if I'm recreating and repopulating them all so I went hunting. I managed to find the emulator database is in C:\Users\<username>\. In that directory you'll find mdf files called WAStorageEmulatorDbXX.mdf where XX is the version number. I had ones ending in 22, 30, 32. Each will be accompanied by a _log file.

I loaded them up in Linqpad and the schemas looked the same, so for a punt I just renamed the files ending in 32 to something else and the renamed the files ending in 30 to 32.

Start up the emulator and everything is present again. That saved me a few hours!
