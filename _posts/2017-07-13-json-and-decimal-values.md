---
id: 814
title: JSON and decimal values
date: 2017-07-13T16:28:22+00:00
author: tomasz
layout: post
guid: http://codica.pl/?p=814
permalink: /2017/07/13/json-and-decimal-values/
excerpt_separator: <!--more-->
categories:
  - Blog Entry
tags:
  - iOS
  - JSON
  - Swift
---
In this short post I will explain why floating point or large decimal values should be represented as strings when transferring data using JSON. This post is based on my recent discovery when trying to fix a nasty bug, when running my app on iOS 11 beta3.

<!--more-->

Let&#8217;s assume that we have a simple JSON representation, containing a large decimal value. Let it be 79228162514264337593543950335, which is Decimal.MaxValue (in C#)

```json
{
	"data": 79228162514264337593543950335
}
```

If you want to parse this on iOS 11 beta 3 you will get:



which surprisingly returns a rounded result:

`79228162514264340000000000000`

When you send this again to some C# backend, you will probably get a parsing error as this number is larger than Decimal.MaxValue. 

What may surprise you, this works perfectly on iOS 10:

```swift
let str = "{ \"data\": 79228162514264337593543950335 }" // C# Decimal.MaxValue
let data = str.data(using: .utf8)
let json = try? JSONSerialization.jsonObject(with: data!, options: [])
let dict = json as? [String: AnyObject]
let val = dict!["data"] //79228162514264337593543950335
print(val!.classForCoder) //NSDecimalNumberPlaceholder
let decimal = NSDecimalNumber(string: val?.stringValue) //
decimal.stringValue // 79228162514264337593543950335
```

It looks like between iOS 10 and 11, _JSONSerialization_, under the hood, started to use directly _NSNumber_ instead of _NSDecimalNumberPlaceholder_. _NSNumber_ cannot represent such a big number and the result is not correct.

Hence, if you want to pass a large number in JSON or if precision is of importance, values should be passed as strings.

AFAIK, in JSON numeric precision is not specified and each parser may choose whatever is convenient. If precision is important you should always pass numeric values as strings and convert them to appropriate representation in your client application.

```swift
let str = "{ \"data\": \"79228162514264337593543950335\" }" // C# Decimal.MaxValue
let data = str.data(using: .utf8)
let json = try? JSONSerialization.jsonObject(with: data!, options: [])
let dict = json as? [String: AnyObject]
let val = dict!["data"] as! String //79228162514264337593543950335
let decimal = NSDecimalNumber(string: val)
print(decimal) //79228162514264337593543950335
```

If you enjoyed this short post, please follow me on twitter: [@tgebarowski](https://twitter.com/tgebarowski)

Many thanks to Pawel Kowalczuk [@riamf1](https://twitter.com/riamf1), for his contribution to this post.