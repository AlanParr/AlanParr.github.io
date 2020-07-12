---
layout: post
title:  "Making old machines immortal(-ish) with P2V"
date: 2013-04-17 22:35:00 +0000
categories: p2v virtualisation vmware converter
---

The time comes to every PC, when it's reached the end of it's life and it's time to be turned off once and for all...Except, when that PC has old software on it with a non-transferable license. In an ideal world this wouldn't happen, but sometimes it just can't be helped - either the software is no longer available to buy and transferring to an equivalent would cost a bomb, or you'd have to buy a new license for the most recent version which would also cost a bomb.

Fortunately, this is where virtualisation becomes really useful for a small business. While small businesses may not have workloads that warrant massive clusters of VMs spanning multiple centuply redundant clusters on fault tolerant blade servers, they will almost certainly at some point experience the imminent failure of *that* machine - the one machine in the entire company that simply must remain. It cannot be upgraded and it cannot be replaced, it must exist forever.

In short, P2V allows you to take an existing OS install on physical hardware and convert it to run as a virtual machine on a virtual host. It's easy to do, so easy in fact that I'm not going to tell you how to do it. Some detailed instructions for performing a P2V conversion can be found here

[http://www.petri.co.il/virtual_convert_physical_machines_to_virtual_machines_with_vmware_converter.htm](http://www.petri.co.il/virtual_convert_physical_machines_to_virtual_machines_with_vmware_converter.htm)

Some tips when you're doing this:

* Install the converter on the machine you want to convert - I've found the agent can sometimes be difficult to get to work, if you want to get things done quickly, just install on the machine and then uninstall from the converted VM when you're done.
* Make sure you change the disks to Thin Provision - this will save you disk space on your virtual server. Space may not be an issue for companies with SANs, but if you're just using the drives in the server box, it doesn't hurt to save every gigabyte you can.
* Make sure you get the number of CPUs right - I didn't do this on the first XP machine I converted, the physical hardware had 1 core and the virtualised version 2. I ended up in a tricky situation where I had to reactivate XP but couldn't because it couldn't connect to the activation service. Eventually, after numerous attempts, it connected and I could reactivate, but if I'd set the cores I may not have had to.

Virtualisation for me takes between 2 and 3 hours, after which I have a virtualised copy of the hardware machine. As a matter of course I go through a few steps here.

* Create a snapshot before doing anything.
* Uninstall VMWare converter.
* A bit of general cleanup. Take out unnecessary items from services and startup, drop the visual effects (Classic mode for XP). This will make remoting a more pleasant experience as there won't be so many differing colors to transfer over the wire and render on the client side.
* Almost forgot this one. On your physical PC, you probably have various software installed for the graphics card (ATI Catalyst Control Center, etc), network adapters, and any number of esoteric peripherals. The majority of these you will no longer need so remove them and save your new virtual machine some work.

Take a snapshot after each of these steps just in case you get anything wrong. 10 seconds to create a snapshot could save you 2-3 hours starting all over again.