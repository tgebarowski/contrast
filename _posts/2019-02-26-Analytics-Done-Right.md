---
id: 1001
title: Analytics done right
author: tomasz
layout: post
permalink: /2019/02/26/analytics-done-right/
excerpt_separator: <!--more-->
categories:
  - Blog Entry
tags:
  - Analytics
  - Dependency Injection
  - Frameworks
  - Modularization
  - Swift
---

Tracking user behaviour and gathering analytics is very important not only for marketing purposes but also to improve the user experience. Especially when having limited resources it allows to concentrate on most important user paths. I've worked on several mobile applications, some of them I've written from scratch, others were maintained by me, for some I did code reviews only. Most of those apps had one thing in common, they implemented some kind of analytics (Firebase, Gemius, Mixpanel or proprietary) which was hard to maintain and seemed like it was there only to satisfy some business requirement. Added without broader perspective, just to satisfy current needs.

<!--more-->

Based on my experience, typical code smells and bad practices when introducing analytics to mobile app included:

- Lack of abstraction and tightly coupling with 3rd party vendors. It was hard to switch to another solution without repetitive changes in lots of places
- Promotion of copy and paste patterns
- Lack of testability due to static cling and no dependency injection
- No support for application modularization
- Either one single enum/struct/class with all events or logic wrapped with hundreds of "descriptive" functions


Let us imagine that we have a Tracking class and several helper methods that are following more or less this pattern:

```swift

enum EventType: String {
   case offerClick
   case offerShow
   case offerBid
   case offerDismiss
...
}

class Tracking {

   static let shared = Tracking()

   func trackOfferClick(id: String) {
       Analytics.logEvent(AnalyticsEventSelectContent, parameters: [
                          AnalyticsParameterItemID: "id-\(id)",
                          AnalyticsParameterItemName: offerClick.rawValue
                          ])
   }
   ...

   func trackOfferShow(id: String) {
       Analytics.logEvent(AnalyticsEventSelectContent, parameters: [
       AnalyticsParameterItemID: "id-\(id)",
               AnalyticsParameterItemName: offerShow.rawValue
                          ])
    }
}
```

So what could we tell about this tiny class that adds analytics functionality to our app? Such a pattern should be familiar to every developer who has spent a reasonable time with mobile apps. Usually, this is the most straightforward abstraction that is added on top of some analytics engine. What's wrong with this code?


First of all, it is tightly coupled with Firebase analytics, switching the library to other vendor will require repetitive and time-consuming changes (a code smell that is frequently referred as copy and paste, with some aspects of shotgun surgery smell). Secondly, the wrapper promotes singleton pattern and probably will be used in lost of places like that:

```swift
Tracking.shared.trackOfferClick(id: "offerId1")
```

resulting in code that is extremely hard to test.


What is more, having a shared *EventType* enum including all possible analytics events makes it hard to split our application into modules. It would require that *EventType* is globally accessible by all dependent modules. Assuming that we split our features into modules, adding tracking to such a new feature would require changes in a shared framework. Hence, the shared framework will have knowledge of all analytics events from features that are not belonging to it. Something that is also a cause of potential source code conflicts (when several people are adding/modifying analytics events).

Finally, wrapping analytics with "descriptive" method names is not a good idea. It will provide only a fake abstraction, a middle man like code smell - won't add any extra functionality to the wrapper class. Imagine that we have 1000 events, this will result in 1000 functions just wrapping analytics logs with some descriptive names.


## How this situation could be improved? 


Let's first abstract the *EventType* enum into protocol allowing to split the conforming data structures into modules.

```swift
protocol EventType {
    static var category: String { get }
    associatedtype N
    associatedtype A
}
```

The protocol will use two *associatedTypes*, one for event name (N) and other for event action (A). It will define a category of event, which will be returned from the default protocol extension as a name of conforming class:

```swift
extension EventType {
    static var category: String {
        return String(describing: self)
    }
}
```

New events can be defined by conforming to *EventType* protocol:

```swift
struct Messages: EventType {
    typealias N = Name
    typealias A = Action
    
    enum Name: String {
        case title
    }
    enum Action: String {
        case click
    }
}
```

Now let's build a generic wrapper class for reporting events:

```swift
protocol EventReporterPlugin {
    func send(category: String, name: String, action: String, userData: [String: String])
}

final class EventReporter<T: EventType> {
    private let plugins: [EventReporterPlugin]
    
    init(plugins: [EventReporterPlugin] = []) {
        self.plugins = plugins
    }
    
    func send(name: T.N, action: T.A, userData: [String: String] = [:]) {
        plugins.forEach { $0.send(category: T.category, name: "\(name)", action: "\(action)", userData: userData) }
    }
}
```

What's interesting about this class? It introduced an *EventReporterPlugin* protocol that all 3rd party plugins should conform to. With this design, it should be possible to hook up new analytics SDK easily, or even use two analytics engines simultaneously. Secondly, it allows to use a generic *EventType* and has no knowledge of specific events. Together with *EventReporter*, *EventType* protocol can be a part of generic *Tracking.framework* having no knowledge about tracked events. We can specialize EventReporter by providing concrete *EventType* implementations and support only required events in our feature specific frameworks.


By conforming to EventReporterPlugin we can support Firebase analytics or add a dummy reporter plugin just for testing purposes:

```swift

class ConsoleReporter: EventReporterPlugin {
    func send(category: String, name: String, action: String, userData: [String: String]) {
        print("[Tracking] Category: '\(category)' name: '\(name)' action: '\(action)' userData: \(userData)")
    }
}

```

we could add a helper extension to simplify object creation or use dependency injection to promote testability:

```swift
extension EventReporter {
    static var console: EventReporter<T> {
        return EventReporter<T>(plugins: [ConsoleReporter()])
    }
}
```


## Practical use case

Taking into account what was shown above, we can implement tracking that promotes modularization and is easy to maintain:


_Basket.framework_:


```swift
struct Basket: EventType {
    typealias N = Name
    typealias A = Action
    
    enum Name: String {
        case payment
        case add
        case checkout
    }
    enum Action: String {
        case select
        case pay
        case other
    }
    
    // Override default category
    static var category: String = "Cart"
}


let reporter: EventReporter<Basket> = EventReporter.console
reporter.send(name: .payment, action: .select,
              userData: ["cardType": "Mastercard"])

```

Note that if we are not satisfied with what is returned automatically from a category using default protocol extension, the value can be overridden.


_Onboarding.framework_:

```swift
struct Onboarding: EventType {
    typealias N = Name
    typealias A = Action
    
    enum Name: String {
        case registration
        case consents
        case login
    }
    enum Action: String {
        case tapped
        case entered
        case clicked
        case submited
    }
}


let reporter: EventReporter<Onboarding> = EventReporter.console
reporter.send(name: .consents, action: .submited,
              userData: ["legal": "YES", "marketing" : "NO"])

```


## Summary

Proposed solution solves most of the problems covered in the first part:

- ✅ It supports modularization and does not leak feature specific event definitions to shared frameworks
- ✅ For testing purposes, we can use dependency injection and mock the plugins
- ✅ We reduce the number of source code conflicts by avoiding shared enum with event types
- ✅ Copy and paste is reduced to a minimum

I know that this pattern is not perfect. The *EventType* protocol does not support nesting so that *Names* and *Actions* are not directly related. It could happen that from analytics perspective given a combination of *Name* and *Action* does not exist, but the compiler will not complain. Secondly, EventReporter send method's userData parameter accepts weakly typed values in form of a dictionary of strings. This allows flexibility but is also a bit risky. Anyway, I think that there are more pros than cons and this pattern will bring more benefits to your app than potential problems.  


If you enjoyed this post and would like to be informed about new articles, please follow me on twitter: [@tgebarowski](https://twitter.com/tgebarowski)











