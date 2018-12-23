---
id: 437
title: My first Elisp script
date: 2011-02-03T16:24:20+00:00
author: tomasz
layout: post
guid: http://myitcorner.com/blog/?p=437
permalink: /2011/02/03/my-first-useful-elisp-script/
excerpt_separator: <!--more-->
categories:
  - Blog Entry
tags:
  - Elisp
  - Emacs
  - Glib
---
It&#8217;s already been one year since I switched from Vim to Emacs (thanks Aleksander;) I heard a lot about how powerful Elisp was, but I didn&#8217;t have much motivation to learn it. I started playing with Elisp last weekend and I must say that is much different from everything I used before. Anyway because I work a lot with GObjects and Glib I decided to create some useful script which will speed up my coding. My first script is trivial but it helps to create GList iterator, it acts as a simple code snippet with this difference, that the initial position of the iterator is assigned with the identifier under the cursor.

<!--more-->

Elisp script:

```lisp
(defun azt-glist-iterate-all()
    "Creates a GList iterator for
      the list under the cursor"
    (interactive)
    (setq itr_template "GList *node_itr = vvv;
while (node_itr != NULL)
{
    /* GType *data = (GType *)node_itr->data; */
    node_itr = g_list_next(node_itr);
}")
    ; Replace patter with a word at point
    (setq initializer
             (replace-regexp-in-string "\*" "" 
                   (thing-at-point 'symbol)))
    (setq itr_template 
             (replace-regexp-in-string "vvv" 
                     initializer itr_template))
    (forward-line 1)
    (insert itr_template)
)
```

When the script is loaded into Emacs and when you start typing something like &#8220;GList *my\_list = priv->\_list&#8221; , put your cursor on _my_list_ identifier and _M-x azt-glist-iterate-all_ to generate the snippet below:
```c
GList *node_itr = my_list;
while (node_itr != NULL)
{
    /* GType *data = (GType *)node_itr->data; */
    node_itr = g_list_next(node_itr);
}
```