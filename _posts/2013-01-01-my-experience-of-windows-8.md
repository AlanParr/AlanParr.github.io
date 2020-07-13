---
layout: post
title:  "My experience of Windows 8"
date: 2013-01-01 22:12:00 +0000
categories: metro windows windows-8
---

I think it's fair to say that Windows 8 has been the most controversial and negatively received OS since Vista. Every mention of Windows 8 invokes cries about the Metro UI and how the OS is a schizophrenic nightmare. Occasionally, amongst these voices will be a quiet statement of "it's not that bad once you get used to it".

That little voice isn't far wrong.

That's not to say that after using Windows 8 for a while that I have grown to love the Metro UI and think it's the best thing since sliced bread. No, it's just that I've learned to ignore it to the extent that I barely notice it is there.

Worth noting that, despite it allegedly only being a code name, I will be using the term Metro repeatedly in this post, it's just easier to type than Windows 8 Store app.

## Start screen
It wasn't until I had used Windows 8 for a few weeks that I realized how little I used the old Start Menu. Since the ability to pin applications to the taskbar was added in Windows 7 and the addition of a fantastic little application called Bins which allows me to have 4 apps in one 'slot' of the taskbar, everything I need is in the taskbar. This relegates the Start Menu to two things - Search and shutdown.

## Search
One thing that Microsoft have gotten right in Windows 8 is the search experience, which is superior to that of previous Windows versions. I get a big full page list of search results which covers files, settings, and applications and also the ability to search things like eBay from the same screen (not a feature I see myself using, but a nice touch anyway).

## Shutdown
Shutdown/Restart is the biggest thing that has frustrated me about Windows 8. There are a couple of ways to do it:

1. When using a mouse, hover the mouse over a mystical hotspot in the bottom right hand corner, click on Settings, click on power, and then select from shutdown/restart/update. This is such a massive pain in the ass when using a mouse that it is not even really an option.

2. Same as above but the charm bar is invoked using Win+C instead of the mystical hotspot.

3. Alt + F4 when on the Desktop - the preferred option and not a problem for a keyboard/mouse user, but absolutely impossible if you were using a tablet, which is supposed to be Metro's Raison d'être.

My wife, who is fairly technically literate, could not figure out how to turn it off. I had to Google it, expecting that there would be some ridiculously easy method that I was completely overlooking. There was not. In order to make things easy for myself, I have added a shortcut to shutdown.exe on the Metro start screen so I have the options of one-click shutdown if I'm in Metro or Alt+F4 if I'm in classic desktop. Can't see your average Joe user coming up with this solution though and this will be one of the biggest transition headaches in my opinion.

## Metro applications
The other biggest issue with Windows 8 is Metro apps. I work with Windows and C# on a daily basis and I must admit that I have found it really difficult trying to work with Windows' new paradigm. Local database usage is incredibly difficult, with portable libraries not allowing you to use some of the standard data libraries. Microsoft has tried really hard with their samples, but some of them fall short, such as the settings sample which shows how to use the settings flyout but does not show how to retrieve or save settings the "Metro" way.

The expectation is very much that Metro apps will be consuming cloud data sources and so will not need local database access. But cloud does not work for everything, such as business applications or little apps I write to make my life easier. Microsoft have limited the usefulness of Metro apps by not doing the extra leg work to make the development experience equal or superior to writings a standard Windows app. For something so radical to be picked up, they need to make the community want to use it, not try to force them to do it a certain way and remove the tools they are used to using unnecessarily.

From my perspective, I have almost completely ignored the Metro apps as the vast majority are useless to me. I suspect that, unless major improvements are made to the support for standard .net libraries in Windows apps, that Metro apps will be relegated to time-killing games and media consumption, something that the Metro UI is perfect for.

Those are the biggest issues Windows 8 has which, for your average user, are absolutely massive hurdles to overcome and is the reason many businesses will be completely skipping Windows 8 and hoping that a future service pack or Windows 9 corrects some of Windows 8's mistakes.

But of the 3 main issues described above (Start screen, shutdown, Metro apps), 2 are largely ignorable and the other is easily worked around.

### But it's not all bad...

Those not withstanding, Windows 8 has some plus points.

## Upgrade experience
I upgraded from Windows 98 to ME, and regretted it for weeks after until I finally bit the bullet and did a complete reinstall. The upgrade experience from Windows 7 to 8 was as smooth as it could be. The upgrade adviser told me which applications would be incompatible ahead of time and uninstalled them for me as part of the upgrade, the upgrade itself was paid for without ever leaving the upgrade application.

Post-upgrade, everything worked and the only issue I have had is Linqpad not showing up as an installed application in the search screen which was easily solved by a quick re-install.

## Faster than Windows 7
Owing largely to the removal of superfluous visual effects but I suspect there has also been some serious optimization work done under the hood. The improved Task Manager is also a nice touch.

## Hyper-V built in
I haven't made massive use of this yet owing to having an almost-1 year old in the house and generally having less time to tinker than I used to, but when I have done some work requiring a VM, it has been more than adequate. My virtual server hasn't been fired up in almost a year now.

## Great upgrade offer
I haven't paid for a Microsoft OS in a very long time, apart from as part of a new machine purchase. Despite the controversy over Windows 8, the cheap upgrade price of £15 and the possibility of being able to write applications and publish them through a store that would gain my applications exposure and possibly even make me a few quid, was enough for me to bite. The ecosystem possibility seems a little less likely now, but I still think the upgrade was worth it.
