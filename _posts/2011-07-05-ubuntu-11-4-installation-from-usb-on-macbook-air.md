---
id: 472
title: Ubuntu 11.4 installation from USB on MacBook Air
date: 2011-07-05T21:47:02+00:00
author: tomasz
layout: post
guid: http://myitcorner.com/blog/?p=472
permalink: /2011/07/05/ubuntu-11-4-installation-from-usb-on-macbook-air/
excerpt_separator: <!--more-->
categories:
  - Blog Entry
tags:
  - GRUB
  - MacBook Air
  - Ubuntu
  - USB
---
Installing Ubuntu on a MacBook Air which has no CD drive is a tricky task. First of all it is impossible to boot from the USB stick created in Ubuntu, secondly even if you manage to boot from it, the installer will fail as it tries to read from /cdrom anyway. 

Some time ago I found the following  [thread](http://forums.linuxmint.com/viewtopic.php?f=42&t=47434) which described step by step procedure of generating bootable USB stick, recognized as CD drive. In order to make it working, you have to download and extract the mkhybridiso.tar.gz (from the above page) and remaster your Ubuntu ISO with the following command:

<!--more-->
  
```
tar xvf mkhybridiso.tar.gz
cd mkhybridiso
sudo ./mkhybridiso.sh /PATH/your-ubuntu.iso
```
  
After some time a hybrid ISO is generated (with &#8216;hybrid&#8217; suffix), when this is done, copy the ISO onto USB stick:
  
```
sudo dd if=your-ubuntu_hybrid.iso of=/dev/sdb bs=1M
```
  
Note that /dev/sdb corresponds to your USB drive identifier (you can get the proper name by executing dmesg \| less just after inserting the USB stick).
  
When USB stick is ready, you can reboot the MBA and start the installer (for example via rEFIt). It is very probable that the installation hangs on a black screen. In order to avoid that make sure that you add &#8220;nomodeset&#8221; parameter to the initializer list, just before invoking the installer from the purple menu. If all steps are accomplished the installation should end with success.

Unfortunately, the above procedure doesn&#8217;t work well when we have already installed Ubuntu (hence GRUB as well) and we want to boot again from the same Ubuntu USB installer. It doesn&#8217;t matter how many times the Mac is restarted you will always end up with rEFIt executing GRUB and Ubuntu from the HD.
  
In this specific situation, in order to start up from the USB stick installer you may try invoking GRUB command line (by pressing &#8216;c&#8217; i the GRUB menu).
  
When GRUB prompt starts, try executing the following:
  
```
root (hd1)
set gfxpayload=keep
linux /casper/vmlinuz  file=/cdrom/preseed/ubuntu.seed boot=casper only-ubiquity nomodeset --
initrd /casper/initrd.lz
boot
```
  
If the sequence is executed correctly, the Ubuntu installer should start booting.