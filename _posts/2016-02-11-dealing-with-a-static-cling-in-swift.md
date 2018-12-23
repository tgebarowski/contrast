---
id: 768
title: Dealing with a static cling in Swift
date: 2016-02-11T21:36:09+00:00
author: tomasz
layout: post
guid: http://codica.pl/?p=768
permalink: /2016/02/11/dealing-with-a-static-cling-in-swift/
excerpt_separator: <!--more-->
categories:
  - Blog Entry
tags:
  - Swift
  - Swift 2
  - TDD
  - testing
---
As programmers we often have to deal with legacy code, usually not written by us, sometimes of bad quality, without unit tests, hard to modify, with high regression risk. One of ubiquitous code smells is so called static cling. It is so frequent because lots of developers &#8220;love&#8221; static functions, singletons and in fact greatly abuse those patterns.
  
We do so, because static code seems easier and faster to write, but in a longer run, the code written using those patterns is hard to test, not reentrant, coupled, maintaining global state and is just badly designed.

<!--more-->

### So what is a static cling?

Have you ever seen code like that?

```swift
func foo(param: String) {
    return Klass.bar()
}
```

or

```swift
func foo(param: String) {
    let obj = Klass.sharedInstance()
    return obj.bar()
}
```



Those are only two frequently met examples of a static cling. Which in other words can be defined as binding result of a function/method to some static dependency. Such a dependency is easy to introduce as invoking a static method seems like a shortcut in many cases. We don&#8217;t have to care about keeping reference to an object, dependency injection and such. Especially when we do not follow TDD practices and don&#8217;t start our design from writing test cases, this may lead us to many places where static method calls are overused.

What happens when later we have to test such a functionality? Mocking/stubbing such a dependency is not trivial. In Objective-C we could try to use 3rd party mocking frameworks (i.e. OCMock) which could provide static method mocking, but this is more like a workaround for a badly designed class. Swift does not support read-write reflections so Swift mocking frameworks are simply not available. That&#8217;s why when we want to stub or mock certain functionality we have to use dependency injection (either manually or via frameworks such as [Typhoon](https://github.com/appsquickly/Typhoon) or [Swinject](https://github.com/Swinject/Swinject)).

Having a static cling in our code is one of the problems we may experience when writing unit tests post-factum. Fortunately there are some refactorization methods which allow to either eliminate a static cling or reduce its impact. There is no perfect solution, allowing us to solve the problem of a static cling without much effort. Certain methods are more invasive, others are simply wrapping the cling into non static code. In this post I will present three refactorization methods allowing to break the static cling and write mock objects for previously static functionality.

### How to refactor a static cling?

Consider a really simple example of a static cling.

```swift
struct UserDAO {
    static func add(name: String) {
        print("Storing user \(name)")
    }
}

struct User {
    
    func add(name: String) {
        UserDAO.add(name)
    }
}
```



#### Method 1: Dependency injection and non-static wrapper

Let us approach the static cling, by injecting UserDAO via constructor of User object.

```swift
struct User {
    
    let userDAO: UserDAO
    
    init (userDAO: UserDAO) {
        self.userDAO = userDAO
    }
    
    init () {
        self.init(userDAO: UserDAO())
    }
    
    func add(name: String) {
        self.userDAO.add(name)
    }
}
```



Note that initialiser with zero parameters was added to provide compatibility with existing code.

But this is not enough. UserDAO needs a non-static add method. We may refactor our code, by providing a non static wrapper for a static logic.


```swift
struct UserDAO {
    static func add(name: String) {
        // Real logic stays here
        print("Storing user \(name)")
    }
    
    func add(name: String) {
        UserDAO.add(name)
    }
}
```

This two step refactorization allowed us to isolate UserDAO object and decouple it from User class.
  
We may go one step further and extract a protocol UserDAOType, to which our UserDAO will conform.

```swift

protocol UserDAOType {
    func add(name: String)
}

extension UserDAO: UserDAOType {}

struct User {
    
    let userDAO: UserDAOType
    
    init (userDAO: UserDAOType) {
        self.userDAO = userDAO
    }
    
    init () {
        self.init(userDAO: UserDAO())
    }
    
    func add(name: String) {
        self.userDAO.add(name)
    }
}
```


Testing such an object is now possible by injecting a mock conforming to UserDAOType protocol. 

#### Method 2: Extract and Override

Let&#8217;s go back to our original example of a static cling, but this time let us assume that we have a class not a struct. We may select our static call to UserDAO.add(name: String), inside User add(name:String) method and extract it to another non-static method.

```swift
class User {
    
    func addImpl(name: String) {
        UserDAO.add(name)
    }
    
    func add(name: String) {
        addImpl(name)
    }
}
```


Now we can mock one layer up, by writing a mock for the User class. Note that we are using inheritance here, so this method wouldn&#8217;t work on Swift structs.

```swift

class UserMock : User {
    
    override func addImpl(name: String) {
        print("Method called with \(name)")
    }
}
```



### Leveraging Swift 2 protocol extensions to break a static cling

Let us assume that we have a Caller class that uses UIApplication to open iPhone&#8217;s dialler to trigger emergency calls

```swift
struct Caller {
    
    let application: UIApplication
    
    init (application: UIApplication) {
        self.application = application
    }
    
    func emergencyCall() {
        application.openURL(NSURL(string: "tel://911")!)
    }
}

let caller = Caller(application: UIApplication.sharedApplication())
caller.emergencyCall()
```


How could we potentially test if correct number is dialled? Obvious choice is to mock UIApplication and inject it to our caller. 

```swift
class MockApplication : UIApplication {
    override func openURL(url: NSURL) -> Bool {
        print("Opened: \(url)")
    }
}
```


Unfortunately this will not work, as we cannot have two instances of UIApplication in our tests.

Swift 2 comes with protocol extensions, which come handy in this particular case. We may define openURL method as a part of Callable protocol and extend UIApplication to conform to this protocol. Later our mock object can implement Callable and after trivial changes in Caller object it may accept Callable dependency.


```swift

protocol Callable {
    func openURL(url: NSURL) -> Bool
}

extension UIApplication: Callable {}

struct Caller {
    
    let application: Callable
    
    init (application: Callable) {
        self.application = application
    }
    
    func emergencyCall() {
        application.openURL(NSURL(string: "tel:/112")!)
    }
}

class MockDialer : Callable {
    
    func openURL(url: NSURL) -> Bool {
        print("Opened: \(url)")
        return true
    }
}

let caller = Caller(application: MockDialer())
caller.emergencyCall()
```


### Summary

Static cling is usually a code smell, in this post I showed three methods allowing to eliminate static code coupling.

I would like to thank Jakub Sitnicki for reading initial version of this post and sharing his feedback.

If you enjoyed this article, please follow me on twitter: [@tgebarowski](https://twitter.com/tgebarowski)