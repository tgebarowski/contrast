---
id: 804
title: Closures argument + member function = retain cycle?
date: 2016-12-29T20:15:23+00:00
author: tomasz
layout: post
guid: http://codica.pl/?p=804
permalink: /2016/12/29/closures-argument-member-function-retain-cycle/
excerpt_separator: <!--more-->

categories:
  - Blog Entry
tags:
  - iOS
  - memory leaks
  - Retain Cycle
  - Swift
  - Swift3
  - swiftlang
---
After introducing ARC memory management was simplified a lot, but still could not be forgotten. For sure every developer working with either Objective-C or Swift had to deal with retain cycles. There are lots of in depth posts about this topic and I do not want to rephrase them, so instead I will link to my favourite from [krakendev](https://krakendev.io/blog/weak-and-unowned-references-in-swift). Unfortunately this post does not cover everything and I would like to mention some specific case which I encountered not that long ago.

<!--more-->

Imagine following example, where private member function is passed as a closure argument:

```swift

import Foundation

class RequestSender {
    func sendRequest(success: @escaping () -> Void) {
        DispatchQueue.global(qos: .background).asyncAfter(deadline: .now() + 0.5) {
            
            success()
        }
    }
}

class Dispatcher {
    
    private let requestSender = RequestSender()
    private let dispatchGroup = DispatchGroup()
    
    deinit {
        print("Deinitialized")
    }
    
    func start() {
        dispatchGroup.enter()
        requestSender.sendRequest(success: handleResponse)
    }
    
    func wait() {
        dispatchGroup.wait(timeout: .now() + .seconds(1))
    }
    
    private func handleResponse() {
        print("Response handled")
        dispatchGroup.leave()
    }
}

let dispatcher = Dispatcher()
dispatcher.start()
dispatcher.wait()

```

The problem with the above code is that _sendRequest_ returns its result using @escaping closure from a background queue.
  
Passing a private _handleResponse_ function as a closure argument creates a retain cycle, hence deinit is never called on Dispatcher object.

The problem could be easily solved by either:

  * inlineing handleResponse content withing the closure where self is weak/unowned

  ```swift
  requestSender.sendRequest(success: { [unowned self] in
    print("Response handled")
    self.dispatchGroup.leave()
  })
  ```


  * wrapping handleResponse with another closure where self is weak/unowned

  ```swift
  requestSender.sendRequest(success: { [unowned self] in
      self.handleResponse()				   
  })
  ```


The above code looks ugly and is less compact. What if we created a helper function allowing us to wrap up this code?

```swift
func unownedClosure<T: AnyObject, A: Any>(_ obj: T, _ method: @escaping (T) -> (Void) -> A) -> (Void) -> A {
    return { [unowned obj] in
        method(obj)()
    }
}
```


and then use the following code?

```swift
requestSender.sendRequest(success: unownedClosure(self, Dispatcher.handleResponse))
```


Not perfect, but definitely more compact. Anyway, I think that this is something that could be addressed in new version of Swift.

Check out my GitHub project with [Swift One-Liners](https://github.com/tgebarowski/SwiftOneLiners), where you can find this [exemplary wrapper](https://github.com/tgebarowski/SwiftOneLiners/blob/master/unownedClosure.swift), extended with versions having different closure signatures.

If you enjoyed this article, please follow me on twitter: [@tgebarowski](https://twitter.com/tgebarowski)