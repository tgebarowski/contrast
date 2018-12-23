---
id: 244
title: 'GObject and protected fields &#8211; simple hack'
date: 2008-12-21T14:30:44+00:00
author: tomasz
layout: post
guid: http://myitcorner.com/?p=244
excerpt_separator: <!--more-->
permalink: /2008/12/21/gobject-and-protected-fields-simple-hack/
categories:
  - Blog Entry
tags:
  - ANSI C
  - GObject
  - OOP
---
Not so long ago I wondered how to add some protected fields into one of my GObject. Those of you who have more experience with Glib and GObject should know a very popular construct allowing to define and then add a private structure to a GObject class definition (with _g\_type\_class\_add\_private_).
  
As opposed to that, there is no such an easy mechanism which simplifies adding protected fields. Unfortunately when we use inheritance and virtual methods sometimes this can lead to some inconvenience and a lot of boilerplate code (which btw. in case of GObject library can reach a significant number of lines)

<!--more-->

I tried to search over the net, but I couldn&#8217;t find any solution to this problem. Having no better solution I invented a simple &#8220;hack&#8221; that mixes the behavior of both private and public fields. Together with some naming conventions it allows to simulate some kind of protected mechanism. Of course the word protected has more intuitive meaning because it violates a lot of constraints that must be satisfied in a real OOP language.
  
The main problem in implementing protected mechanism in a GObject is the scope and visibility of the protected fields. It is impossible to define them in the source file, because then the implementation will be hidden in all the derived GObjects. That&#8217;s why I decided to move it into a header file and wrap all the fields into a struct similar to the one used to define private fields. 

```c
/** Struct representing protected fields of a class */
typedef struct
{
    /* Some protected fields */
    guchar *_raw_header;
    guint     _raw_header_size;
    ....
} MicGenericObjectProtected;
```

Moreover the header file should contain the macro definition allowing to access protected fields from other derived objects. This can be done with the following line:

```c
/** Getter for protected data of an object **/
#define MIC_GENERIC_OBJECT_GET_PROTECTED(o) \
  (G_TYPE_INSTANCE_GET_PRIVATE ((o), \
  MIC_GENERIC_OBJECT_TYPE, MicGenericObjectProtected))
```

Now it&#8217;s time to move into source code of GObject class.
  
In the class_init function we add the protected structure into class definition with previously mentioned _g\_type\_class\_add\_private_ function.
  

  
The code can be similar to the one below:

```c
static void
mic_generic_object_class_init (MicGenericObjectClass *klass)
{
 GObjectClass *object_class = G_OBJECT_CLASS (klass);

 /** Add private structure to the class **/
g_type_class_add_private (klass,                             \
                          sizeof (MicGenericObjectProtected));

object_class->dispose = mic_generic_object_dispose;
object_class->finalize  = mic_generic_object_finalize;

/** Virtual public methods with their implementations **/
....
/** Abstract Method */
....
}
```

That&#8217;s everything. This hack is not perfect because it doesn&#8217;t allow to easily add both private and protected fields in a single GObject. Remember that g\_type\_class\_add\_private can be called only once! The other problem is that protected fields are public to all other code and in fact they can be used from arbitrary object or function. That&#8217;s why I decided to implement some kind of naming convention (&#8220;protected&#8221; suffix) which highlights that a given collection of fields can be used only by derived objects. Of course it is not a protection, it&#8217;s more like an information to the programmer. It&#8217;s his decision whether he takes it into consideration or not.