---
id: 697
title: Swift Behaviors
date: 2015-06-04T21:04:14+00:00
author: tomasz
layout: post
guid: http://codica.pl/?p=697
permalink: /2015/06/04/swift-behaviors/
excerpt_separator: <!--more-->
categories:
  - Blog Entry
---
Recently I've published a set of small uncoupled Swift objects (Behaviors) which may provide various functionality to your apps. The whole concept of Behaviors is inspired by [objc.io #13 article](http://www.objc.io/issue-13/behaviors.html) by [Krzysztof Zabłocki](http://merowing.info). I ported the idea to Swift and extended it a bit to allow modifying UIViews (for animations, transitions etc)

<!--more-->

The whole idea of Behaviors is really powerful. In most of the cases you can extend your apps by simply adding a Behavior to your UIViewController in the Interface Builder, connecting a couple of IBOutlets and that's it. The behavior is invoked when NIB or parent UIView is loaded and adds to your app intended functionality.

In objc.io article all behaviors were derived from KZBehaviour which provided a lifetime binding to owner object. In my Swift implementation I decided to name that class SwiftBehavior. I have also defined another class which may be of potential interest for behaviors that require access to UIView properties (e.g. animations). UIView is not yet initialized, when awakeFromNib is executed in your behavior. For that purpose I added ViewBehavior class, which provides targetView IBOutlet and two methods that can be overriden:

```swift
targetViewDidSet(target: UIView)
targetViewDidLoad(target: UIView)
```

The whole idea is to attach a targetView IBOutlet to a UIView that we want to be sure that is loaded when targetViewDidLoad method is called. With this approach by overriding targetViewDidLoad behaviors that modify UIView properties may be created. Internally ViewBehavior implements this using KVO pattern.

I&#8217;ve published all behaviors that I created so far on my [github account](https://github.com/tgebarowski/SwiftBehaviors).