---
id: 361
title: Boost ptr_map iterator incompatibility
date: 2009-08-11T20:46:49+00:00
author: tomasz
layout: post
guid: http://myitcorner.com/?p=361
excerpt_separator: <!--more-->
permalink: /2009/08/11/boost-ptr_map-iterator-incompatibility/
categories:
  - Blog Entry
tags:
  - autotools
  - Boost
  - C++
---
Boost library provides a very powerful set of data structures and algorithms which are intended to be cross platform and rather concise. Unfortunately it happens, that the API of the library changes from version to version. The problem which I faced recently was related to _ptr_map_ container iterator and different methods allowing for accessing keys and values of the container. In boost <=1.33 the mentioned fields could be accessed via _iterator->key()_ and _(*iterator)_ methods respectively. Starting from version 1.34 the authors of the library introduced approach which is compatible with map container from STL and uses _iterator->first_ and _iterator->second_ fields.

<!--more-->

Unfortunately I was forced to use different platforms for development (Ubuntu 9.04 &#8211; Boost 1.39) and deploying (CentOS 5.3 with Boost 1.33), that&#8217;s why some problems appeared.
  
In order to solve them I used some conditional compilation tricks and some basic _AutoConf_ routines.

First of all in _configure.ac_ file I added the following condition, which tries to compile a given instruction using _ptr_map_ iterator from Boost > 1.33.
  
If it succeeds the _HAVE\_PTR\_MAP\_ITR\_SECOND_ value is defined in _config.h_ file and we are sure that the platform supports STL compatible method of accessing map entries.

```
AC_TRY_COMPILE(
[#include ],
[boost::ptr_map::iterator itt;int *i=itt->second;],
AC_DEFINE(HAVE_PTR_MAP_ITR_SECOND,1,
[ptr_map::iterator defines second field]))
```

Later I created a short macro which will be replaced with appropriate code depending on the _HAVE\_PTR\_MAP\_ITR\_SECOND_ condition:

```
#ifdef HAVE_PTR_MAP_ITR_SECOND
#define ptr_map_itr_second(s) ((s)->second)
#else
#define ptr_map_itr_second(s) (&(*s))
#endif
```

The exemplary code showing application of the created macro can be found below:

```
...
ptr_map::iterator it;
it = ports_map_.find("key");

if (it != ports_map_.end())
{
  OffPortRange *port_range =  ptr_map_itr_second(it);
  ....
}
```