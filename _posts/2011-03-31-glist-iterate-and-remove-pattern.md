---
id: 464
title: GList iterate and remove pattern
date: 2011-03-31T15:03:13+00:00
author: tomasz
layout: post
guid: http://myitcorner.com/blog/?p=464
permalink: /2011/03/31/glist-iterate-and-remove-pattern/
excerpt_separator: <!--more-->
categories:
  - Blog Entry
tags:
  - ANSI C
  - Glib
  - GObject
---
Recently I've came across a problem, when I had to iterate through a GList and remove its elements depending on some condition. 
This is the solution I came up with:

```c
/* Set the iterator to the beginning of our list */
GList *node_itr = priv->_list;
while (node_itr != NULL)
{
   GObject *obj = (GObject *)node_itr->data;
   /* Store next element's pointer before removing it */
   GList *next = g_list_next(node_itr);
   /* Some dummy removal condition */
   if (my_gobject_marked_for_removal(obj))
   {
      g_object_unref(obj);
      priv->_list = g_list_delete_link(priv->_list, node_itr);
   }
   node_itr = next;
}
```