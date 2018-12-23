---
id: 196
title: Glib, GObject and memory leaks
date: 2008-08-21T21:33:13+00:00
author: tomasz
layout: post
guid: http://myitcorner.com/?p=196
permalink: /2008/08/21/glib-gobject-and-memory-leaks/
excerpt_separator: <!--more-->
categories:
  - Blog Entry
tags:
  - ANSI C
  - Glib
  - GObject
  - memory leaks
  - valgrind
---
I&#8217;m working currently on a server application which uses Glib and GObject system. I wanted to test it with _valgrind_ against memory leaks, but the results had really scared me. I got tens of memory alignment errors and hundreds of bytes that leaked somewhere. After investigating the problem I found possible reasons and a solution.
  
The main problem is related to Glib and the way it allocates memory. It doesn&#8217;t use a pure C-like way which is based only on malloc & free. It rather uses something called &#8220;slice allocator&#8221;.
  
If you don&#8217;t know what the &#8220;slice allocator&#8221; is, I will quote some useful information from the Gnome Reference Manual.

<!--more-->

> Memory slices provide a space-efficient and multi-processing scalable way to allocate equal-sized pieces of memory, just like the original GMemChunks (from GLib <= 2.8), while avoiding their excessive memory-waste, scalability and performance problems. To achieve these goals, the slice allocator uses a sophisticated, layered design that has been inspired by Bonwick's slab allocator. It uses posix_memalign() to optimize allocations of many equally-sized chunks, and has per-thread free lists (the so-called magazine layer) to quickly satisfy allocation requests of already known structure sizes. This is accompanied by extra caching logic to keep freed memory around for some time before returning it to the system. Memory that is unused due to alignment constraints is used for cache colorization (random distribution of chunk addresses) to improve CPU cache utilization. The caching layer of the slice allocator adapts itself to high lock contention to improve scalability. 

If you are interested more information about &#8220;slice allocator&#8221; [can be found here](http://library.gnome.org/devel/glib/stable/glib-Memory-Slices.html).

Unfortunately _valgrind_ has some problems with the above concept and cannot detect if given blocks of memory were actually freed or not. When I tested a simple Glib application using _g\_slice\_alloc_ with valgrind I got tens of memory alignment errors.
  
You can imagine how hard it can be to find some other errors/leaks which are not related to &#8220;slice alloc&#8221;, but are rather the fault of the programmer.
  
Having tens or hundreds of messages printed by _valgrind_ can be quite misleading and in fact can hide some real problems. So how to avoid that? The simplest solution is always the best. Why can&#8217;t we disable &#8220;slice allocator&#8221;? Of course only for testing, later on when we are sure that there are no memory leaks in our application we can turn it on again.

In order to do this we have to set an environmental variable which disables the &#8220;slice allocator&#8221; and forces Glib to use normal mallocs.
  
```
G_SLICE=always-malloc<br />
```
  
There is no need to change anything in a source code, even the program doesn&#8217;t have to be recompiled. 

Unfortunately &#8220;slice allocator&#8221; is not the only problem, the other one lies on the GObject side.
  
I don&#8217;t know why, but somebody invented a really &#8220;nice&#8221; function called _g\_type\_init_, which is responsible for initializing GObject type system. Ok, its great but why there is no g\_type\_deinit? Does anyone know where the whole memory that was previously initialized is freed? _Valgrind_ doesn&#8217;t know that and again it complaints&#8230;
  
I managed to get rid of some of this errors by creating suppression files that are forcing _valgrind_ to omit some of the most common errors. Of course it is not the best solution, but at least it can help to reduce the number of errors that are not the result of our fault. 

The suppression file can be found below:

[glibsupp.txt](/assets/glibsupp.txt)

One thing is certain. The list of suppression will be expanding. The more I work with Glib the more suppressions I have to add. The current version of this file includes suppressions mainly from _GHashTable_, _g\_thread\_init_ and _g\_type\_init_. If I find more possible leaks I will update this file.

Now it&#8217;s time to start _valgrind_ with the whole configuration which allows to get rid of this Glib and GObject errors, the command is as follows:
  
```
G_SLICE=always-malloc valgrind --suppressions=build-aux/glib.supp --leak-check=full --show-reachable=yes   ./our_program<br />
```