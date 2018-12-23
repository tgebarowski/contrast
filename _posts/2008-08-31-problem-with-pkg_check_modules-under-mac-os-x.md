---
id: 207
title: Problem with PKG_CHECK_MODULES under Mac OS X
date: 2008-08-31T20:36:46+00:00
author: tomasz
layout: post
guid: http://myitcorner.com/?p=207
permalink: /2008/08/31/problem-with-pkg_check_modules-under-mac-os-x/
excerpt_separator: <!--more-->
categories:
  - Blog Entry
tags:
  - aclocal
  - ANSI C
  - autotools
  - Mac OS X
  - pkg-config
---
Yesterday I had some problem with compiling my Glib application under Mac OS X. I knew that everything worked before on Debian and the problem had to be one of this darwin-specific.
  
Just to make myself clear. Here is the error I got after executing: _./configure_
  
```
 checking for pkg-config... pkg-config<br />
./configure: line 3489: syntax error near unexpected token `GLIB,'<br />
./configure: line 3489: `PKG_CHECK_MODULES(GLIB, glib-2.0)'<br />
```
  

<!--more-->


It seemed that there was some problem with _PKG\_CHECK\_MODULES_ &#8211; autoconf&#8217;s macro responsible for detecting if a given library or module was installed in the system (basing on pkg-config).
  
In fact it came out that the macro was not defined at all. Normally after running _aclocal_ command it should be generated in the file aclocal.m4. But in my situation there was no such entry.

Why did that occur? At that time I could not answer that question but at least I had some starting point.
  
You may probably know that _aclocal_ is one of the tools of Autotools package. It creates a file named _aclocal.m4_ which includes a number of Autoconf macros that can be used later by configure. (one of these macros is of course _PKG\_CHECK\_MODULES_)
  
After reading the aclocal info page I found out that the aclocal.m4 file content was based on some external .m4 files and my configure.ac. I read that the _PKG\_CHECK\_MODULES_ macro should be defined inside _pkg.m4_ file, and I really could find it there in my directory _/opt/local/share/aclocal_ .

And here the answer came. I installed _pkg-config_ and Glib using **darwin ports**, where the default prefix for packages was _/opt/local_. But on the other hand all the Autotools programs were located in /usr/bin. It was obvious that the prefix was different, so that _aclocal_ could not detect pkg.m4 file and define _PKG\_CHECK\_MODULES_ macro.

The only missing thing was to inform _aclocal_ about the path of pkg.m4 and all other external .m4 files.
  
It was simple and could be done by adding extra parameter -I:
  
```
aclocal -I /opt/local/share/aclocal<br />
```
  
Now after running my _autogen.sh_ file which more or less looked liked that:
  
```
aclocal -I /opt/local/share/aclocal<br />
autoheader<br />
automake<br />
autoconf<br />
```
  
and later
  
```
./configure<br />
make<br />
```
  
my application was compiled.
  
Of course this can be also achieved in many different ways. To find more solutions I recommend studying _aclocal_info page (especially secion Macro Search Path)