---
title: Inclusive enums with OptionSet
layout: post
category: Swift Language Features
---

Enums are one of the most powerful features of Swift language. It forms Value-Oriented Programming in conjunction with Structs. Enum is the best way to describe the exclusive state in Swift, but what about the case when you need an inclusive state. Today we will talk about OptionSet protocol and how we can achieve inclusive states with it.

{% include friends.html %}

#### Exclusive Enums
Assume that we have some HistoryFetcher class, which can fetch data from a cache or make a network request or both of them. Let's start with describing very simple source enum.

```swift
enum FetchSource {
    case memory
    case disk
    case remote
    case cache
    case all
}
```

Now we can work on our history fetch method which will take a source as a parameter and make request accordingly to the source.

```swift
class HistoryFetcher {
    func fetch(from source: FetchSource = .all, handler: @escaping Handler<History>) {
        switch source {
        case .memory:
            fetchMemory(handler: handler)
        case .disk:
            fetchDisk(handler: handler)
        case .remote:
            fetchRemote(handler: handler)
        case .cache:
            fetchMemory(handler: handler)
            fetchDisk(handler: handler)
        case .all:
            fetchMemory(handler: handler)
            fetchDisk(handler: handler)
            fetchRemote(handler: handler)
        }
    }
}
```

There are possible downsides of this approach.
1. As soon as we increase the count of the sources, we have to add a separated case for that and add it to "all" case handling.
2. We can't easily create some unions of sources, like memory and remote, or disk and remote, etc. We need a lot of additional logic here to make it possible.

#### OptionSet for the rescue

OptionSet is a protocol which represents bitset types, where individual bits represent members of a set. Adopting this protocol in your custom types lets you perform set-related operations such as membership tests, unions, and intersections on those types. 

OptionSet protocol is very straightforward. All we need is rawValue property which should be a type conforming FixedWidthInteger. So basically in most cases, we can use Int type. Next, we have to create unique options using the unique power of two for every case. Here we can use bit shifting operators. Let's refactor our FetchSource enum to use OptionSet.

```swift
struct FetchSource: OptionSet {
    let rawValue: Int

    static let memory = FetchSource(rawValue: 1 << 0)
    static let disk = FetchSource(rawValue: 1 << 1)
    static let remote = FetchSource(rawValue: 1 << 2)

    static let cache: FetchSource = [.memory, .disk]
    static let all: FetchSource = [.cache, .remote]
}
```

As you see above, we can create multiple union members, which contains other members. It brings real power while handling this OptionSets. Here is the refactored version of our HistoryFetcher class.

```swift
class HistoryFetcher {
    func fetch(from source: FetchSource = .all, handler: @escaping Handler<History>) {
        if source.contains(.memory) {
            fetchMemory(handler: handler)
        }

        if source.contains(.disk) {
            fetchDisk(handler: handler)
        }

        if source.contains(.remote) {
            fetchRemote(handler: handler)
        }
    }
}
```

New implementation of HistoryFetcher class is pretty simple. We handle every unique case of FetchSource which is also covering all possible unions of our OptionSet.

#### Conclusion
Today we learn how to use OptionSet protocol and how it can be useful as Enum replacement with some extra features. We will continue to cover small and powerful types from the Swift Foundation in future posts. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading and see you next week!