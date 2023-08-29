---
title: "What's Up With the Cisco ATA 190 Telephone Adapter ?"
date: 2023-08-28T00:00:00Z
draft: false
toc: true
---

There are times we start new projects and end up choosing some vendors, because they are industry standards, are well known and recognized, and some of those times, we end up having in our hands some pretty neat hardware with the capabilities we were searching for, just to notice that the software they come with are designed to get in the way of any potential use of the system outside of an enterprise network and a designated set of infrastructures.

That was the case when I got my hands on the _infuriating_ **Cisco ATA[^1] 190** for ~35€ secondhand on the internet.

## What's the device capable of ?

The **Cisco ATA 190** is a 2-port analog telephone adapter that works with the **SIP[^2]** protocol. It's capable of handling two phones with a variety of voice codecs, transmitting and receiving Fax, on-the-phone configuration for some settings, has a plethora of voice-enhancement features, supports network VLANs, is low consumption and configurable by your network admin, so you don't have to worry about the technicalities™.

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

The ATA 190's firmware comes in two different flavors that are essentially the same, but bundles differently, a `.k3.cop.sgn` file that is signed and contains the other format, a `.zip` file containing itself a binary signed file as well, they both are supposed to be processed by CUCM before updating the device, but we can manually unpack them.

---

Here we will only explore the `.zip` file as it involves less unpacking and binary modifications to obtain a firmware file processable by the ATA.

{{<highlight sh>}}
$> unzip -d . cmterm-ata190.1-2-2-003_SR2-1.zip
Archive:  cmterm-ata190.1-2-2-003_SR2-1.zip
  inflating: ./ATA190.1-2-2-003_SR2-1.bin.sgn
  inflating: ./ATA190.1-2-2-003_SR2-1.loads
{{</highlight>}}

After unpacking the `.zip` and discarding the `.loads` file as it is only used by Cisco's software, we get our hands on this `.bin.sgn` that is the firmware file, with a signature header, that we'll remove right away.

For this, we use the `grep` utility to get the index of the beginning of the firmware blob.

{{<highlight sh>}}
$> grep -aob 'ATA190  FiRmWaRe' ATA190.1-2-2-003_SR2-1.bin.sgn
424:ATA190  FiRmWaRe
{{</highlight>}}

Then, we use `dd` to skip the signature header (424 bytes in this case) and output the unsigned data to a new file named `ATA190.1-2-2-003_SR2-1.bin`.

{{<highlight sh>}}
$> dd bs=424 skip=1 if=ATA190.1-2-2-003_SR2-1.bin.sgn of=ATA190.1-2-2-003_SR2-1.bin
25040+1 records in
25040+1 records out
10617220 bytes (11 MB, 10 MiB) copied, 0,274267 s, 38,7 MB/s
{{</highlight>}}

We can now confirm that the operation went according to plan by reviewing the start of the file using `xxd` and verifying that the first line matches with the firmware's start string.

{{<highlight sh>}}
$> xxd ATA190.1-2-2-003_SR2-1.bin | head -2
00000000: 4154 4131 3930 2020 4669 526d 5761 5265  ATA190  FiRmWaRe
00000010: 0000 0000 0000 0000 0000 0000 0000 0000  ................
{{</highlight>}}

Now that we're left with an unsigned firmware blob (_isn't it a cute blob ?_) that is processable by the ATA, we can investigate further into it's content.

---

### Investigating the web interface

### Accessing the serial console

## Reviving the beast

[^1]: an **A**nalog **T**elephone **A**dapter is a devices that enables the use of analog telephones (or anything that uses a landline) on **V**oice **o**ver **I**nternet **P**rotocol phone systems.
[^2]: **S**ession **I**nitiation **P**rotocol is a popular protocol used for VoIP communications around the world, see [RFC2543](https://datatracker.ietf.org/doc/html/rfc2543).
[^3]: a **T**rivial **F**ile **T**ransfer **P**rotocol server is a piece of software that allows storing and retrieving files from the network in a technically simple way, see [RFC1350](https://datatracker.ietf.org/doc/html/rfc1350).
