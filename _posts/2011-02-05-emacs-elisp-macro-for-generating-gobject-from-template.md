---
id: 447
title: Emacs Elisp macro for generating GObject from template
date: 2011-02-05T14:07:43+00:00
author: tomasz
layout: post
guid: http://myitcorner.com/blog/?p=447
permalink: /2011/02/05/emacs-elisp-macro-for-generating-gobject-from-template/
excerpt_separator: <!--more-->
categories:
  - Blog Entry
tags:
  - ANSI C
  - Elisp
  - Emacs
  - GObject
---
Everyone who has worked with GObjects knows how time consuming is to create a new GObject. In order to make it working you have to generate lot of boilerplate code which is more or less the same for all the objects.
  
In most of the cases people tend to use templates and make several find-replace substitutions to generate the output GObject.

<!--more-->

To simplify everything and to learn more about Elisp I decided to build simple Elisp macros for generating GObjects directly from the Emacs command line. The script prompts for some basic info, like name of the final object (ex. MyExtremelyUselessObject), the output directory where the object is to be generated and some brief description of the object (for Doxygen purpose). While preparing templates I tried to follow GObjects name conventions and used Doxygen for documenting the source code (I know that probably GTK-Doc would be better but in all my current projects I use Doxygen so I know it better). The the set of macros can be downloaded from here: [emacs-gobject-macros.tar.gz](/assets/emacs-gobject-macros.tar.gz) , a detailed installation instruction is inside the Elisp script. Note, that this script can also generate some kind of C-Objects based on normal C-struct&#8217;s.
  
If you happen to make some useful modification to the script, please let me know!