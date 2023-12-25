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

### Unwrapping the update files

The ATA 190's updates bundles come in two different flavors that are essentially the same, a `.k3.cop.sgn` file that is signed and contains the other format, an archive (`.zip` or `.tar.gz`) file containing a signed binary file as well.
They both are supposed to be processed by CUCM before updating the device, however we can manually unpack them if we fiddle around.

Here we will only explore the archive (`.zip` file in this case) as it involves less unpacking and binary modifications to obtain a firmware file processable by the ATA.

Once we downloaded the bundle, we can extract the archive in the current directory using `unzip` as such:

{{<highlight sh>}}
$> unzip -d . cmterm-ata190.1-2-2-003_SR2-1.zip
Archive:  cmterm-ata190.1-2-2-003_SR2-1.zip
  inflating: ./ATA190.1-2-2-003_SR2-1.bin.sgn
  inflating: ./ATA190.1-2-2-003_SR2-1.loads
{{</highlight>}}

After discarding the `.loads` file as it is only used by Cisco's software, we get our hands on the `.bin.sgn` firmware file, with a signature header, that we'll remove right away.

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

### Diving into the firmware

Now that we're left with an unsigned firmware blob (_isn't it a cute blob ?_) that is processable by the ATA, we can investigate further into it's content.

A quick and dirty way of doing this is by using the `strings` command, this will reveal us some of the ASCII strings contained in the binary file. Here we search for some of the web interface files in example.

{{<highlight sh>}}
# Most of the output is truncated, but these look like
# interesting pages to look at later
$> strings ATA190.1-2-2-003_SR2-1.bin | grep -e .asp -e .cgi
# ...
Backup.asp
Diagnostics2.asp
Diagnostics_tab1.asp
Diagnostics_tab2.asp
Factory_Defaults.asp
Factory_Defaults_run.asp
Management.asp
Management2.asp
Management_u.asp
Reboot.asp
Reset_button.aspbH
Restore.asp
Upgrade.aspbM
Upgrade_run.asp
port_setting.aspVC
privilegectl.asp
quick_setup.asp
quicksetup.asp
# ...
{{</highlight>}}

A more efficient way to unpack the firmware's content is to use `binwalk`[^4]. It's an utility able to find start and end markers of some known formats in a binary file and extract them.

{{<highlight sh>}}
$> binwalk --extract ATA190.1-2-2-003_SR2-1.bin

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
388           0x184           uImage header, header size: 64 bytes, header CRC: 0xE7981D4B, created: 2106-02-07 06:28:15, image size: 1572736 bytes, Data Address: 0x20008000, Entry Point: 0x20008000, data CRC: 0xF84AC86, OS: Linux, CPU: ARM, image type: OS Kernel Image, compression type: none, image name: "SP2Xcybertan_rom_bin"
452           0x1C4           Linux kernel ARM boot executable zImage (little-endian)
13592         0x3518          gzip compressed data, maximum compression, from Unix, last modified: 2020-09-07 07:07:53
1573188       0x180144        uImage header, header size: 64 bytes, header CRC: 0x1628277F, created: 2106-02-07 06:28:15, image size: 8998912 bytes, Data Address: 0x0, Entry Point: 0x0, data CRC: 0x7607965A, OS: Linux, CPU: ARM, image type: Filesystem Image, compression type: none, image name: "SP2Xcybertan_rom_bin"

WARNING: Extractor.execute failed to run external extractor 'unsquashfs -d 'squashfs-root' '%e'': [Errno 2] No such file or directory: 'unsquashfs', 'unsquashfs -d 'squashfs-root' '%e'' might not be installed correctly

WARNING: Extractor.execute failed to run external extractor 'sasquatch -p 1 -le -d 'squashfs-root' '%e'': [Errno 2] No such file or directory: 'sasquatch', 'sasquatch -p 1 -le -d 'squashfs-root' '%e'' might not be installed correctly

WARNING: Extractor.execute failed to run external extractor 'sasquatch -p 1 -be -d 'squashfs-root' '%e'': [Errno 2] No such file or directory: 'sasquatch', 'sasquatch -p 1 -be -d 'squashfs-root' '%e'' might not be installed correctly
1573252       0x180184        Squashfs filesystem, little endian, non-standard signature, version 3.1, size: 8998794 bytes, 1044 inodes, blocksize: 131072 bytes, created: 2020-09-07 23:51:21
{{</highlight>}}

Moving on to the destination directory's content, we mainly have a `.squashfs` file containing the firmware's root filesystem. While `unsquashfs` won't be able to unpack this for _unknown reasons_, `7z` seems to be able to handle this job just fine.

{{<highlight sh>}}
$> 7z -osquashfs-root x _ATA190.1-2-2-003_SR2-1.bin.extracted/180184.squashfs

7-Zip (z) 23.01 (x64) : Copyright (c) 1999-2023 Igor Pavlov : 2023-06-20
 64-bit locale=C.UTF-8 Threads:4 OPEN_MAX:1024

Scanning the drive for archives:
1 file, 8998794 bytes (8788 KiB)

Extracting archive: _ATA190.1-2-2-003_SR2-1.bin.extracted/180184.squashfs
--
Path = _ATA190.1-2-2-003_SR2-1.bin.extracted/180184.squashfs
Type = SquashFS
Physical Size = 8998794
Headers Size = 55391
File System = SquashFS-LZMA 3.1
Method = LZMA ZLIB
Cluster Size = 131072
Big-endian = -
Created = 2020-09-08 01:51:21
Characteristics = UNCOMPRESSED_INODES CHECK DUPLICATES_REMOVED EXPORTABLE
Code Page = UTF-8
# ... (stripped out some complaints about insecure symlinks)
{{</highlight>}}

The `squashfs-root` directory is now populated with the ATA's root filesystem content. _Hooray !_

#### Some random discoveries made during the filesystem exploration

- There is a directory located at `/www/spa100_help` containing documentation the **Cisco SPA100** series ATAs, _could this be a repurpose of the firmware, of the hardware ?_
- The files doing actual changes on the configuration of the device are directly bundled in the web server binary (`/usr/sbin/httpd`), making any further research on how to configure the device by attacking the web interface way more complicated.
- There are many leftover files not referenced in the web interface, but totally accessible knowing their path, _more on that later_.
- Some files make reference to a **nvram**(_non-volatile memory_), this is probably the place where the configuration is stored.
- The firmware update routine in the web interface checks for the exact string `ATA190  FiRmWaRe` before allowing the upgrade, but also contains checks for some other models like **SPA112** & **SPA122**.

### Investigating the web interface



### Accessing the serial console

> This section is based on the works made in the following article: https://www.insentricity.com/a.cl/277/unlocking-a-cisco-spa122-for-use-with-any-provider.

## Reviving the beast

[^1]: an **A**nalog **T**elephone **A**dapter is a devices that enables the use of analog telephones (or anything that uses a landline) on **V**oice **o**ver **I**nternet **P**rotocol phone systems.
[^2]: **S**ession **I**nitiation **P**rotocol is a popular protocol used for VoIP communications around the world, see [RFC2543](https://datatracker.ietf.org/doc/html/rfc2543).
[^3]: a **T**rivial **F**ile **T**ransfer **P**rotocol server is a piece of software that allows storing and retrieving files from the network in a technically simple way, see [RFC1350](https://datatracker.ietf.org/doc/html/rfc1350).
[^4]: see https://github.com/ReFirmLabs/binwalk.
