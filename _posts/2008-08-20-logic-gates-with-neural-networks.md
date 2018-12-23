---
id: 186
title: Logic gates with Neural Networks
date: 2008-08-20T20:32:50+00:00
author: tomasz
layout: post
guid: http://myitcorner.com/?p=186
permalink: /2008/08/20/logic-gates-with-neural-networks/
excerpt_separator: <!--more-->
categories:
  - Projects
tags:
  - "Neural Networks"
  - Java
  - Logic
---
Simple Java Swing application written to present learning abilities of Neural Networks. Program uses a back propagation algorithm and provides a set of adjustable logic gates which can be used for the testing purposes. 

![image Fuzzy Logic](/assets/logic-gates-1.png)

<!--more-->

**Download**

[logicnn.zip](/assets/logicnn.zip) 

It is important that before testing the Neural Network it must be trained. Some training properties like number of hidden layers or learning (iteration) steps can be set on a separate tab. 

![Logic Gates](/assets/logic-gates-2.png)

On the next tab which is called &#8220;Network Trainer&#8221; it is possible to define so called training set which corresponds to the possible combination of inputs and generated outputs. Based on that set a given Neural Network will be trained.

In order to test the behavior of the Neural Network , a &#8220;Network Tester&#8221; tab should be chosen which enables us to define some input values and to ask the network about its prediction.

![Logic Gates](/assets/logic-gates-3.png)