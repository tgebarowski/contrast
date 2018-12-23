---
id: 428
title: Mounting ext3 partition on OS X 10.6 (Snow Leopard)
date: 2011-01-30T13:23:05+00:00
author: tomasz
layout: post
guid: http://myitcorner.com/blog/?p=428
permalink: /2011/01/30/mounting-ext3-partition-on-os-x-10-6-snow-leopard/
excerpt_separator: <!--more-->
categories:
  - Blog Entry
tags:
  - ext2
  - ext3
  - Mac OS X
  - Macfuse
  - Snow Leopard
---
Recently I&#8217;ve faced some problems with mounting a large (~ 850 GB) ext3 partition on OS X 10.6. I didn&#8217;t except any difficulties as I have already mounted ext2 or 3 partitions on my MBP running Leopard. Normally it could be easily done with [Macfuse](http://code.google.com/p/macfuse/) and [fuse-ext2](http://sourceforge.net/projects/fuse-ext2/) extension. Unfortunately in this specific case it simply didn&#8217;t work. I have realized that I&#8217;m running a 64-bit version of Snow Leopard and Macfuse is only officially delivered for 32-bit architecture. This explained a lot! I managed to find an unofficial 64-bit compilation of Macfuse which could be downloaded from [Tomas Carnecky blog](http://caurea.org/2009/09/15/unofficial-macfuse-release-for-64bit-kernels/). I installed the 64-bit version and used the old fuse-ext2 plugin, unfortunately when trying to mount the ext3 partition I got the following error:

```
fuse-ext2 /dev/disk2s2 /Volumes/Private/**

fuse-ext2 /dev/disk2s2 /Volumes/Private/fuse-ext2: version:&#8217;0.0.7&#8242;, fuse_version:&#8217;27&#8217; [main (../../fuse-ext2/fuse-ext2.c:324)]
  
fuse-ext2: enter [do\_probe (../../fuse-ext2/do\_probe.c:30)]
  
fuse-ext2: Error while trying to open /dev/disk2s2 (rc=16) [do\_probe (../../fuse-ext2/do\_probe.c:34)]
  
fuse-ext2: Probe failed [main (../../fuse-ext2/fuse-ext2.c:340)] 
```

<!--more-->

After googling a little bit, I discovered that this error could be caused by some conflict with previously installed [ext2fsx](http://sourceforge.net/projects/ext2fsx/). During my small fight with Snow Leopard and ext3 I tried several possibilities and I totally forgot about them. It seemed that ex2fsx might have caused some conflicts with Macfuse. After uninstalling the ext2fsx the drive could be successfully mounted with the command mentioned above.
  
FYI, the ext2fsx can be easily removed by using uninstallation script delivered in the dmg package. Also when you mount the device make sure that you specify it correctly by referencing to corresponding /dev/diskXXX. The exact name of the device can be read from the Disk Utility application (right clicking on the partition and selecting &#8220;Information&#8221;) 

Note, that the name of fuse-ext2 may suggest that only ext2 partitions are supported, fortunately both ext3 and ext4 partitions should work correctly as well.