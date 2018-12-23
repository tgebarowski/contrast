---
id: 241
title: 'Ubuntu 8.10  Fixed problem with sound'
date: 2008-11-10T20:10:16+00:00
author: tomasz
layout: post
guid: http://myitcorner.com/?p=241
excerpt_separator: <!--more-->
permalink: /2008/11/10/ubuntu-810-fixed-problem-with-sound/
categories:
  - Blog Entry
tags:
  - ALSA
  - Linux
  - PCM
  - Sound
  - Ubuntu
---
In my previous post I wrote about a strange bahavior of my freshly installed Ubuntu. One of the problems was related to sound. I managed to get it working. It came out that something had muted the PCM channel of my sound card. ( while updating from Ubuntu 8.04)

You can fix it by right clicking on the &#8220;Sound&#8221; icon next to menu bar, choosing &#8220;Open Volume Control&#8221; and moving up the slider next to PCM label.

<!--more-->

I read on some boards that it&#8217;s a common issue on Ubuntu 8.10, and quite a lot of people had problems with that. I hopy that this &#8220;silly&#8221; solution might help someone.