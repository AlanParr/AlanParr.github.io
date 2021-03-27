---
layout: post
title:  "Server 8, Win 8 and VS 11 - first impressions"
date: 2012-03-03 21:10:00 +0000
categories: 
excerpt_separator: <!--end_excerpt-->
---

It's been a busy week for Microsoft, releasing the first Beta of Windows Server 8, the Consumer Preview of Windows 8, and the beta of Visual Studio 11. But have the changes they've made been positive ones? These are my first impressions.
<!--end_excerpt-->
## Server 8

After experiencing a hiccup with Virtualbox involving only having one CPU assigned to the VM, the install of the Core configuration took less than 15 minutes which I was very impressed by.

The performance is surprisingly good, far exceeding previous experiences with Server 2008 VMs, even Server Core installations. Running a fresh installation running as a domain controller with only 2Gb of RAM results in boot times of less than 15 seconds and really responsive performance from the minimal UI.

I have only one minor gripe from my initial few hours of testing. Despite Microsoft's big push to get admins using Powershell for all their server tasks, therefore eliminating the need for a UI and increasing the server's efficiency, the default shell for server core is still a standard command prompt. I was fully expecting a Powershell Window here. Also, would it kill to make it full screen so I can make full use of my monitor? I know the emphasis is on remoting, but would it kill to support someone logging onto the machine and wanting a full screen command line?

## Windows 8

As much as I hate to say it, I expect Windows 8 to be a flop on the desktop, much as Vista was. The reason for this is that it has a serious split personality disorder. As a tablet OS, I like Metro and am even considering a WinPho for my next upgrade. As a desktop OS, I'm pissed about the disappearance of the Start button but think the changes they have made to Windows Explorer are long overdue and a definite improvement.

The problem here is that Microsoft have tried to make an OS that works on the Tablet AND the desktop, and I think that is a massive mistake. As long as you only need to use either Metro or Classic, it will be a fine OS. If you need to transition between the two, the cracks between Windows 8's dual personalities start to show and things start to look awfully shoddy.

I can only hope that Microsoft resolve this before release or we could see the same story of enterprise holding on to Windows 7 far beyond it's life in exactly the same way as they did with XP.

## Visual Studio 11

It seems as if the Visual Studio team have spent a great deal of time since 2010 was released fiddling with icons and trying to make VS11 as retro as possible. The key differences between 2010 and 11 are simple, toolbar headers filled with colons that look straight out of the early 90s. I'm sure I saw them in an old OS but just can't put my finger on which one! There is also the case of the ALL CAPS WINDOW TITLES WHICH ARE REALLY ANNOYING AND EXTREMELY DISTRACTING. Other than this and changing the icon set, I haven't seen any real differences as yet. I can only hope that VS11 grows on me as I use it more.

Friendly warning, if you are using the built in Windows Setup projects, do not install VS11. My boss hasn't been able to build an installer since he installed this, not even after uninstalling it and uninstalling/reinstalling VS2010 and multiple other components.

## Verdicts

Bit of a mixed bag from Microsoft here. Hopefully the issues with Win8 and VS11 will be resolved by RTM, and Server 8 is looking like a very promising release. Now they just need to make them all play a little nicer with Virtual Machines!