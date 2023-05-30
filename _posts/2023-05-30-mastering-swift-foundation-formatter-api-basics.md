---
title: Mastering Swift Foundation Formatter API. Basics
layout: post
category: Swift Language Features
image: /public/swift.png
---

One of the significant additions to Swift Foundation was the new Formatter API. After almost two years of using it, I use it in every project, and even more, I try to build my data formatting logic around this API. Today we will learn how to use the new Swift Foundation Formatter APIs.

{% include friends.html %}

#### Old-style formatters

```swift
let shortDateFormatter = DateFormatter()
shortDateFormatter.timeStyle = .none
shortDateFormatter.dateStyle = .short
shortDateFormatter.string(from: Date.now)
```

The old-style formatters have a few problems that make our lives more complicated than they should be. First of all, the process of formatter initializing is a resource-heavy task, and you should cache formatter instances as much as you can. You should avoid the creation of a formatter instance per item in your collection. The second issue is the error-prone class-based API we inherited from the Objective-C era. You can accidentally override the configuration of the formatter instance, and it will affect all the places where you are using it. It is a classical reference type-related issue and why we love Swift value types.

#### Swift Foundation Formatter API
Many types in Swift Foundation provide the new *formatted* function allowing us to format the instance in place. You don't need to initialize any formatter instance or cache them because the Foundation automatically handles it. The new Swift Foundation Formatter API also provides a new value-oriented approach for customizing formatting options.

```swift
Date.now.formatted()
```

As you can see in the example above, we use the *formatted* function on an instance of the *Date* type to format it to a locale-friendly string. 

```swift
["Majid", "Fuad"].formatted(.list(type: .and))
```

You can always try to type *.formatted()* on any instance of the Swift Foundation types in the Xcode to see if it provides formatting logic. You will be surprised how many of them offer formatting opportunities. And the new type-safe and value-oriented format style builders eliminate the need to check the documentation of a formatter instance because they provide very predictable usage.

```swift
Date.now.formatted(.relative(presentation: .named))
Date.now.formatted(date: .abbreviated, time: .omitted)
Date.now.formatted(.dateTime.day(.twoDigits).month(.abbreviated).year(.twoDigits))
Date.now.formatted(.iso8601)
```

As you can see in the example above, there are many options for customization of date formatting logic. You can mix and match many of them to display the needed data in a user-friendly format.

```swift
1_000_000_000.formatted()
1.001.formatted(.number.precision(.fractionLength(1)))
0.1.formatted(.percent)
0.99.formatted(.currency(code: "USD"))
```

Here is another example of the new Swift Foundation Formatter APIs allowing us to display numbers in the locale-friendly format and customize it using format style builders API.

```swift
Measurement<UnitMass>(value: 1, unit: .kilograms)
    .formatted(.measurement(width: .abbreviated))
```

As you can see in the example above, even the *Measurement* type supports the new Formatter API and allows us to present the data in the appropriate format in a few keystrokes. 

Today we learned about the basics of the new Swift Foundation Formatter APIs. In the following posts, we will continue the topic by implementing custom format styles for our own types. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week!
