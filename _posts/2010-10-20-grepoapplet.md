---
id: 372
title: GRepoApplet
date: 2010-10-20T12:26:38+00:00
author: tomasz
layout: post
guid: http://el-instalacje.pl/myitcorner.com/?p=372
excerpt_separator: <!--more-->
permalink: /2010/10/20/grepoapplet/
categories:
  - Projects
tags:
  - Applet
  - C++
  - GConf
  - GNOME
  - GnomeKeyring
  - libnotify
  - VCS
---
GRepoApplet is a simple GNOME Panel applet for monitoring status of Version Control Systems. Currently only SVN repositories are supported but due to provided backend API it should not be difficult to support more types of VCS&#8217;s or even DVCS&#8217;s. The applet allows to specify arbitrary number of monitored repositories and time interval between each query. All the recent changes are displayed in a nicely looking notification box generate by the libnotify. 

<!--more-->

![image SwiftySettings interface](/assets/grepo-applet-new-repository.png)

![image SwiftySettings interface](/assets/grepo-applet-notify-small.png)


The applet is written in C and is using Glib and GObject libraries. For better integration with GNOME and storing user configuration GConf and GnomeKeyring are used.

<!--more-->

**Project homepage:**
  
 <https://launchpad.net/grepo-applet>