---
id: 235
title: Ubuntu 8.10 first (bad)impression
date: 2008-11-01T23:47:35+00:00
author: tomasz
layout: post
guid: http://myitcorner.com/?p=235
excerpt_separator: <!--more-->
permalink: /2008/11/01/ubuntu-810-first-badimpression/
categories:
  - Blog Entry
tags:
  - Linux
  - Ubuntu
---
Yesterday I found out that a new version of Ubuntu was released. Unfortunatelly I&#8217;m always excited in all the latest products and want to test them as fast as possible. That&#8217;s why I started to google for the safest and quickest possible way to upgrade.
  
My previous version of Ubuntu was 8.04 TLS, so I had to change some option in Software Sources dialog in order to make visible the latest version of Ubuntu.

<!--more-->
  
I don&#8217;t know if you knew that but by default in Ubuntu only upgrades to &#8220;Long term&#8221; releases are performed. If you want to upgrade from 8.04 TLS into 8.10 you have to switch to &#8220;Normal releases&#8221; in Software Sources dialog.

If you need more details about this topic you can find it on the following web page: <http://www.ubuntu.com/getubuntu/upgrading>

Ok, but what I&#8217;ve written so far is not related to my impressions, so here I start:

**Installation**

My Update Manager had to work really hard and upgrade more then 1300 packages. All in all it took me about 1:20 h for downloading all of the files (on 2Mb/s connection) and then additional 30-40 minutes for unpackaging and setting them up. Unfortunately I started very late in the evening because I didn&#8217;t expect that it would take such a long time. Finally I ended up at 2:30 A.M. 

**First reboot**

Although the installation took me some time, I didn&#8217;t have much problems with that. But when I restarted the computer the series of strange things started to happen.

 **Long startup**
  
First of all the system initialized for more than 5 minutes. In the beginning I thought that it just hanged. But then it came out that the problem was related to ALSA demon which did not respond for the certain amount of time. After a given timeout it was killed by the system. That caused another problem which resulted in no support for sound.

 **No Sound &#8211; problematic ALSA** 
  
I managed to solve the startup problem temporarily by blocking ALSA daemon. But what&#8217;s obvious it didn&#8217;t help with the sound:( I guess that the new kernel that was installed in the system might cause some conflict with the sound card or sth.

 **NVIdia and Compuz Fusion** 
  
Another problem, that I faced, was related to Compuz Fusion and NVidia drivers. Fortunately all these eye-catching effects work, but sometimes I observe a strange behavior of my window title which blinks and disappear without any reason. This can be seen on pictures below (look at the right top of windows):

![Jacobi](/assets/ubuntu-1.png)

![Jacobi](/assets/ubuntu-2.png)

I can easily reproduce that strange behavior when the window is deactivated (should be transparent) and then activated by moving mouse cursor over the maximize button.

**Firefox**

Another problem was related to Firefox. My Adobe Flash plugin stopped working. Firefox claimed that it was not installed, but when I wanted to do that it complained that it&#8217;s already installed:) Strange. I had to fix it by removing the plugin manually using _apt-get_ and then reinstalling it back using Firefox. 

**Summary**

To sum up my first impressions with new Ubuntu 8.10.
  
I must admit that at a first glance it&#8217;s hard to see any outstanding and innovative modifications in GUI. In my opinion that should at least influence system stability and reliability. Unfortunately even in this matter I&#8217;m not fully satisfied. Maybe my requirements are too high?? Thanks to previous releases of Ubuntu I got used to Linux which works perfectly just out of the box. For me in 8.10 this most crucial rule was violated:(((