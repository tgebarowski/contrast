---
id: 332
title: Random number generator based on normal distribution with Boost
date: 2009-04-26T10:00:20+00:00
author: tomasz
layout: post
guid: http://myitcorner.com/?p=332
excerpt_separator: <!--more-->
permalink: /2009/04/26/random-number-generator-based-on-normal-distribution-with-boost/
categories:
  - Blog Entry
tags:
  - Boost
  - C++
  - Gaussian
  - Normal Distribution
  - Random
---
Yesterday I was looking for some random number generator, based on gaussian distribution. As I don&#8217;t like to reinvent the wheel, I started to look for some already existing solutions. I found out that Boost library provided a very powerful engine for generating random numbers using various algorithms. The whole description of Boost Random Number library is available [here](http://www.boost.org/doc/libs/1_38_0/libs/random/index.html).

<!--more-->

For those who are looking for already existing solutions I attach a small snippet, which generates random numbers based on the normal distribution. As typical for gaussian distribution, the algorithm takes two parameters: mean value and sigma(variance).

```cpp
#include <boost/random.hpp> 
....

double 
GetRandomDoubleUsingNormalDistribution(double mean,
                    double sigma)
{
 typedef boost::normal_distribution<double> NormalDistribution;
 typedef boost::mt19937 RandomGenerator;
 typedef boost::variate_generator<RandomGenerator>, \
                         NormalDistribution> GaussianGenerator;

  /** Initiate Random Number generator with current time */
  static RandomGenerator rng(static_cast<unsigned> (time(0)));

  /* Choose Normal Distribution */
  NormalDistribution gaussian_dist(mean, sigma);
 
  /* Create a Gaussian Random Number generator
   *  by binding with previously defined
   *  normal distribution object
   */
  GaussianGenerator generator(rng, gaussian_dist);
 
  // sample from the distribution
  return generator();
}
```