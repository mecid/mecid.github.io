---
title: Discovering Swift Async Algorithms package
layout: post
---

Another week on a series of posts about discovering Swift packages. This week, we will discover the Swift Async Algorithms package, allowing us to completely switch from the Combine framework to the Swift Concurrency feature with async/await. We will learn what the Swift Async Algorithms package offers to eliminate the Combine framework.

The Swift Async Algorithms package is another package that Apple maintains and provides us. You can always become a part of this great community by contributing to the package on GitHub.

#### Combining
The Swift Async Algorithms package offers a set of functions allowing us to combine two or three async sequences into a single sequence. For example, you can merge two async sequences in a single one and observe values from the single resulting sequence.

=====================================================

As you can see in the example above, we use the merge function that allows us to create a single sequence and observe and handle day and timezone changes at once. The Swift Async Algorithms package provides not only merge functions but also combineLatest, zip, chain, and join.

=====================================================

The Swift Async Algorithms package also includes grouping and filtering operators in the Swift Algorithms but applies to async sequences like compacted for filtering nil values or chunking and removing duplicates.
> To learn more about the Swift Algorithms package, take a look at my "Discovering Swift Algorithms package" post.

#### Time manipulations
The Swift Async Algorithms package introduces a few operators, allowing us to manipulate the sequence using time, similar to the Combine framework. For example, you can debounce and throttle async sequences.

=====================================================

As you can see in the example above, we use the debounce function to wait for a particular period of time before emitting a value. Another helpful type that we have in The Swift Async Algorithms package is AsyncTimerSequence. It emits the current date at a given interval.

=====================================================

#### AsyncChannel
The AsyncChannel type allows us to replace passthrough subjects from the Combine framework. It is a great way to bridge the part of the code that doesn't support async context with the async context in your app.

=====================================================

As you can see in the example above, we use the send function on an instance of the AsyncChannel type to emit values. Conversely, the AsyncChannel conforms to the AsyncSequence protocol to support for-each loop with the await keyword. Remember to call the finish function on the channel to close the sequence.

There is also the AsyncThrowingChannel type with a similar functionality supporting failing with errors. Whenever you need to close the channel with the error, you can use the fail function on an instance of the AsyncThrowingChannel type.
