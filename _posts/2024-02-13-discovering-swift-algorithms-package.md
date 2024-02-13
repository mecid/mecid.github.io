---
title: Discovering Swift Algorithms package
layout: post
category: Swift Language Features
image: /public/spm.jpg
---

Almost every app I built and supported includes the Swift Algorithms package. However, I noticed that only some developers are familiar with it. Today, we will discover what the Swift Algorithms package offers us to write better, safer code for complex algorithms.

{% include friends.html %}

#### Reason
You can write every algorithm from the Swift Algorithms package, but you must maintain and test it in this case. On the other hand, you can depend on the ready-to-use package that you can include in every app you work on without code duplication and be sure that the community members well-tested the package.

> Remember, you can always become a part of this great community by contributing to this package.

#### Setup
First, you must add the package to the project in the Xcode project settings screen. The "Package Dependencies" tab allows you to add or remove any package you need. The [Swift Algorithms](https://github.com/apple/swift-algorithms) package lives on Github, where you can easily follow updates, browse pull requests, and monitor issues.

The Swift Algorithms package contains tons of valuable collections and sequence algorithms. It is nearly impossible to cover them as part of a single post, but I will cover my favorite things.

#### Binary search
I mainly work on health-related apps, and my users' privacy is crucial to me. That's why I make all of my calculations on the devices. Searching a massive array of health-related data for a particular item is a common task.

As you may know, the binary search is the best way to find an item in the sorted data array. Usually, we query HealthKit for sorted data, allowing us to use binary search efficiently. Binary search requires your data to be sorted by the key you seek.

```swift
let someDate = Date.now
let index = heartRates.partitioningIndex { $0.startDate >= someDate }

guard
    index != heartRates.endIndex,
    heartRates[index].startDate == someDate
else {
    return nil
}

return heartRates[index]
```

The Swift Algorithms package provides the partitioningIndex function, the generalized version of the binary search. It uses the same logic as the binary search. 

Still, instead of returning an item, it returns the index of the first item, dividing your collection into two parts where any item from the first part returns false for your predicate, and any item from the second part always returns true for the same predicate. 

We must wrap its results with an additional guard statement verifying them. Whenever the partitioningIndex function can't find the relevant index, it returns the end of the collection.

We also verify that the resulting index divides the array into two partitions, and an item for the first index also equals the item we are looking for. There might be a case where you can find an index dividing the array using your predicate, but the array doesn't contain the value you want.

#### Chunking
Dividing a collection into chunks is another common task in my apps. You might need to divide the collection into chunks of any count or by any additional logic. The Swift Algorithms package provides us with chunking API for this particular case.

```swift
let numbers = [1, 2, 3, 4, 5, 6]
print(numbers.chunks(ofCount: 2))
// [1, 2]
// [3, 4]
// [5, 6]
```

The Swift Algorithms package provides the chunks function, taking a count of items in a single chunk as the parameter and returning the subsequence array.

My app has a more interesting situation where the particular logic should drive chunking. In my case, I need the chunks where items have time intervals between them no longer than one hour. 

```swift
sleepSamples.chunked { $1.startDate.timeIntervalSince($0.endDate) < 3600 }
```

As you can see in the example above, we use the chunked function with the predicate, where we can compare two adjacent elements of the collection and decide whenever we want to put them into the same chunk.

#### Filtering
Almost every app has a situation where you have a collection with optional values, and you need to keep only non-nil values. For this case, the Swift Algorithms package introduces the compacted functions.

```swift
let array: [Int?] = [10, nil, 30, nil, 2, 3, nil, 5]
let withNoNils = array.compacted()
// Array(withNoNils) == [10, 30, 2, 3, 5]
```

Another common task is to remove the duplicates from a collection of elements, and you can easily do it with the help of the unique function.

```swift
let numbers = [1, 2, 3, 3, 2, 3, 3, 2, 2, 2, 1]

let unique = numbers.uniqued()
// Array(unique) == [1, 2, 3]
```

#### Sampling
Another situation I came across in my apps is extracting some minimal or maximal elements from the collection. You can easily do that with the Swift Algorithms package's min, max, or minAndMax functions.

```swift
let numbers = [7, 1, 6, 2, 8, 3, 9]
let smallestThree = numbers.min(count: 3)
// [1, 2, 3]

let numbers = [7, 1, 6, 2, 8, 3, 9]
let largestThree = numbers.max(count: 3)
// [7, 8, 9]
```

How often do you need to get the particular count of the random elements from the collection? The Swift Algorithms package has the randomSample function, taking the count as a single parameter and returning an array of the random elements. 

```swift
let numbers = [7, 1, 6, 2, 8, 3, 9]
let randomNumbers = numbers.randomSample(count: 3)
```

#### Combinations
The Swift Algorithms package provides us with the combinations function, allowing us to combine every element of the collection with each other.

```swift
let colors = ["fuchsia", "cyan", "mauve", "magenta"]
for combo in colors.combinations(ofCount: 3) {
    print(combo.joined(separator: ", "))
}
// fuchsia, cyan, mauve
// fuchsia, cyan, magenta
// fuchsia, mauve, magenta
// cyan, mauve, magenta
```

As you can see in the example above, the combinations function takes only one parameter, defining the number of elements that it should use per combination.
