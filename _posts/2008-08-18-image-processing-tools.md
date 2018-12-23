---
id: 145
title: Image Processing Tools
date: 2008-08-18T22:05:17+00:00
author: tomasz
layout: post
guid: http://myitcorner.com/?p=145
permalink: /2008/08/18/image-processing-tools/
excerpt_separator: <!--more-->
categories:
  - Projects
tags:
  - "Image Processing"
  - Filters
  - FTT
  - Image
  - Java
  - Rotation
---
Useful collection of Image Processing and manipulation algorithms embedded in a GUI application enabling quick previewing and comparison. Application written in Java, using Swing and JUnit tests. 

![Image Filter](/assets/image-filters-1.png)

<!--more-->

**Download**
  
[p_tools.zip](/assets/p_tools.zip) 

The functionality of the application can be divided into several groups:

**Rotation**

  * Implemented using static image matrix rotation (90 degrees rotation, horizontal and vertical flips)
  * Rotation by arbitrary angle using Bi-linear and Nearest Neighbor interpolation methods.

**Filters**

  * Mean filter (possibility to adjust the kernel)
  * Gaussian filter (possibility to defined the kernel)
  * Median filter (possiblity to adjust the matrix)

**Noise**

  * Adding Salt & Pepper noise
  * Adding random noise

**Image enhancement**

  * Basic Moire removal using FTT transform
  * Histogram equalization
  * Color balance equalization
  * Saturation enhancement

**Other**


  * Fast Fourier Transform with plots

![Image Filter](/assets/image-filters-2.png)
![Image Filter](/assets/image-filters-3.png)
![Image Filter](/assets/image-filters-4.png)
![Image Filter](/assets/image-filters-5.png)