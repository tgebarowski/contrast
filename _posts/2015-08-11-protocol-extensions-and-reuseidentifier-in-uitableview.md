---
id: 708
title: Protocol extensions and reuseIdentifier in UITableView
date: 2015-08-11T21:21:00+00:00
author: tomasz
layout: post
guid: http://codica.pl/?p=708
permalink: /2015/08/11/protocol-extensions-and-reuseidentifier-in-uitableview/
excerpt_separator: <!--more-->
categories:
  - Blog Entry
tags:
  - Extension
  - Mirror
  - Protocol
  - reuseIdentifier
  - Swift
  - Swift 2
  - UITableView
  - UITableViewCell
  - Xcode 7
---
Inspired by protocol extensions in Swift 2 and reflection mechanism, I implemented a simple _UITableView_ extension allowing to forget about reuseIdentifiers when registering and dequeuing _UITableViewCells_.

<!--more-->

One may define a _Reusable_ protocol that will specify a dynamic property named _reuseIdentifier_:

```swift
protocol Reusable {
    static var reuseIdentifier: String { get }
}
```

In Swift 2 we may extend that protocol and provide a default implementation. Because each _reuseIdentifier_ should be unique with respect to the class we may use reflection mechanism to simply map a class name to a string.

```swift
extension Reusable {
    static var reuseIdentifier: String {
        let mirror = Mirror(reflecting: self)
        return String(mirror.subjectType)
    }
}
```

Note that _String(mirror.subjectType)_ returns simply a type name of the class implementing _Reusable_ protocol. Returned value has the following pattern:

```swift
ClassName.Type
```

Because _Reusable_ protocol has already a default implementation of _reuseIdentifier_, it is possible to extend _UITableViewCell_ and _UITableViewHeaderFooterView_ to indicate that both classes implement our Reusable protocol:

```swift
extension UITableViewCell : Reusable {}
extension UITableViewHeaderFooterView: Reusable {}
```

Finally, we can provide helpers methods for registering and dequeuing our _UITableViewCells_.

```swift
extension UITableView {

    enum Type {
        case Cell
        case HeaderFooter
    }

    func registerClass<T: UIView where T: Reusable>(aClass: T.Type, type: Type) {
        switch (type) {
        case .Cell:
            registerClass(aClass, forCellReuseIdentifier: T.reuseIdentifier)
        case .HeaderFooter:
            registerClass(aClass, forHeaderFooterViewReuseIdentifier: T.reuseIdentifier)
        }
    }

    func registerNib<T: UIView where T: Reusable>(aNib: UINib, aClass: T.Type, type: Type) {
        switch (type) {
        case .Cell:
            registerNib(aNib, forCellReuseIdentifier: T.reuseIdentifier)
        case .HeaderFooter:
            registerNib(aNib, forHeaderFooterViewReuseIdentifier: T.reuseIdentifier)
        }
    }

    func dequeueReusableCell<T: UIView where T: Reusable>(aClass: T.Type, type: Type) -&gt; T {
        switch (type) {
        case .Cell:
            return dequeueReusableCellWithIdentifier(T.reuseIdentifier) as! T
        case .HeaderFooter:
            return dequeueReusableHeaderFooterViewWithIdentifier(T.reuseIdentifier) as! T

        }

    }
}
```

Note: The idea of _reuseIdentifier_ defined in a _Reusable_ protocol was based on the [following](https://github.com/netguru/roomguru) Swift project, that I found interesting. I pushed it a bit further to avoid hardcoding reuseIdentifiers in code.

Code presented in this blog post can be found on my [github account](https://github.com/tgebarowski/UITableView-ReuseIdentifier). It's more like an experiment, not yet used in any app.

If you like this post, please follow me on twitter: [@tgebarowski](https://twitter.com/tgebarowski)