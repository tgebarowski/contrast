---
id: 107
title: Simple Digit Recognizer
date: 2008-08-10T19:31:00+00:00
author: tomasz
layout: post
guid: http://myitcorner.com/?p=107
permalink: /2008/08/10/simple-digit-recognizer/
excerpt_separator: <!--more-->
categories:
  - Projects
tags:
  - "Neural Networks"
  - Java
---
A simple digit recognition application in Java using a concept of Neural Networks and back propagation algorithm.
  
The application is able to detect one character digits or symbols having some minor distortions.
  
My project is based on the algorithm described by Sacha Barber on the  [codeproject.com](http://www.codeproject.com/KB/recipes/NeuralNetwork_1.aspx)

![Image Filter](/assets/digits-2.png)

<!--more-->

**Download**
  
[digitrecognizer.zip](/assets/digitrecognizer.zip) 

It is important that before starting recognizing the digits, a neural network must be trained for a given set of data probes. Predefined probes can be loaded from the &#8220;Trainer&#8221; tab. It is possible to select from three different training sets (decimal, hexadecimal and binary numbers)

![Image Filter](/assets/digits-1.png)

When a network is successfully trained, move to &#8220;Test (Drawing)&#8221; tab where you can draw a digit (possibly with some disortions) and test it against proper recognition. A set of input test probes can be extended in order to add some additional cases. It is important that after that the network must be retrained.

![Image Filter](/assets/digits-3.png)