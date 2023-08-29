---
title: "What's Up With the Cisco ATA 190 Telephone Adapter ?"
date: 2023-08-28T00:00:00Z
draft: false
toc: true
---

There are times we start new projects and end up choosing some vendors, because they are industry standards, are well known and recognized, and some of those times, we end up having in our hands some pretty neat and functional hardware with the capabilities we were searching for, just to notice, when in hand, that the software they come with are designed to get in the way of any potential use of the system outside of an enterprise network and a designated set of infrastructure.

That was the case when I got my hands on the _infuriating_ **Cisco ATA[^1] 190** for ~35€ secondhand on the internet.

## What's the device capable of ?

The **Cisco ATA 190** is a 2-port analog telephone adapter that works with the **SIP[^2]** protocol, it's capable of handling two phones with a variety of voice codecs, allows transmitting and receiving Fax over the lines, offers on-the-phone configuration for some settings, has plethora of voice-enhancement features, has support for network VLANs, is low consumption and is configurable by your network admin, so you don't have to worry about the technicalities.

![](https://images.pcliquidations.com/images/isaac/59/59345.jpg)
> **Figure 1.** Cisco ATA 190 (Top View)

![](https://images.pcliquidations.com/images/isaac/59/59346.jpg)
> **Figure 2.** Cisco ATA 190 (Rear View)

## The catch

There's always a catch, and today it's that the device is _only_ manageable by your network admin.

But what if you are _your home's own network admin_ you might ask ?  \
The configuration requires a specific piece of software called Cisco Unified Communications Manager (CUCM) which is sold for at least _$125 per device_ at the time of the writing, requires a TFTP[^3] server to host the configuration of the devices and apparently only sells to enterprise customers.

:(

## A desire of revenge

At this point I felt betrayed, I had a fully functionning device in my hands, without any mean of configuring it.
As dumb as this might sound, you cannot set the device up, _or can you ? :)_

### Looking into the firmware files

### Investigating the web interface

### Accessing the serial console

## Reviving the beast

[^1]: an **A**nalog **T**elephone **A**dapter is a devices that enables the use of analog telephones (or anything that uses a landline) on **V**oice **o**ver **I**nternet **P**rotocol phone systems.
[^2]: **S**ession **I**nitiation **P**rotocol is a popular protocol used for VoIP communications around the world, see [RFC2543](https://datatracker.ietf.org/doc/html/rfc2543).
[^3]: a **T**rivial **F**ile **T**ransfer **P**rotocol server is a piece of software that allows storing and retrieving files from the network in a technically simple way, see [RFC1350](https://datatracker.ietf.org/doc/html/rfc1350).
