---
id: 230
title: AutoTools, tar and paths longer than 99 characters
date: 2008-10-16T14:10:27+00:00
author: tomasz
layout: post
guid: http://myitcorner.com/?p=230
excerpt_separator: <!--more-->
permalink: /2008/10/16/autotools-tar-and-paths-longer-than-99-characters/
categories:
  - Blog Entry
tags:
  - ANSI C
  - autotools
  - POSIX
  - tar
---
I tried to build a Debian package for the project I am currently working on. Unfortunately all the paths I used in the structure of my application were very long. While executing _make dist_ command I got lots of errors similar to the one below:

<!--more-->
  
```
wzt-poc2/media-server/src/[....]wzt-children-manager/tests/: file name is too long (max > 99); not dumped
```
  
I found out that the error is due to the tar archiver. It does not support file names (in fact &#8220;paths&#8221;) longer than 99 characters. This problem can be easily solved by forcing **AutoTools** to use **POSIX** version of _tar_ which does not have this limitation.

It is enough to modify a _configure.ac_ file in the following way:

```
AC_INIT([your-application-name], [0.1.0])
....
AM_INIT_AUTOMAKE([1.9 tar-ustar])
```

The crucial part is inside the _AM\_INIT\_AUTOMAKE_ declaration which changes the default tar archiver.