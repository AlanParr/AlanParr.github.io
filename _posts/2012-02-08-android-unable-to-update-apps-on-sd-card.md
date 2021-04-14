---
layout: post
title:  "Android - Unable to update apps on SD Card"
date: 2012-02-08 19:41:00 +0000
tags:
  - Android
  - HTC-Desire
  - Updating-Apps
category: Android
excerpt_separator: <!--end_excerpt-->
---

I've had my HTC Desire for over a year now and, among the many bugs and annoyances that seem to be sneaking their way in to my daily phone-using experience, the most annoying is the recent inability to update any apps I have on my SD Card. Due to the diminutive size of the Desire's internal storage, ALL of my apps are on the SD Card, making this quite a large problem.
<!--end_excerpt-->
My solution up until now has been to move the offending apps back to the phone, update them, and then move them back to the SD Card.

Fortunately, Nick Damoulakis has a [better idea](http://blog.nickdamoulakis.com/2011/06/i-cant-update-apps-installed-on-sd-card.html?m=1).

Summarised below just in case the original becomes unavailable in the future.

* Connect your phone to your PC
* When the connection is detected by the phone, it may ask you if you wish to switch to USB storage. Say yes.
* From your PC, go to the phone's drive (that should have just appeared in Explorer) and navigate to /sdcard/.android_secure/.
* Find the file called smdl2tmp1.asec and delete it
* Disconnect the phone
* Done!!

Thanks a lot Nick, this has saved me a massive amount of time and hair.
