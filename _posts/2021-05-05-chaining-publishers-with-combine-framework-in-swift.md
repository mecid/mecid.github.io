---
title: Chaining publishers with Combine framework in Swift
layout: post
image: /public/combine.png
category: Architecture
---

The Combine framework provides you a bunch of operators to map, filter, and chain asynchronous operations. This week I want to focus on the chaining asynchronous jobs using two main operators that the Combine framework provides us. We will learn how to use flatMap and switchToLatest operators to chain asynchronous tasks in a declarative way.

#### Chaining basics with the flatMap operator
The flatMap operator allows us to take the result of one publisher, pass it to another and run the second publisher. We can use it to chain two different publishers. Let's take a look at how we can use it.

=====================================================

As you can see in the example above, we create the SearchViewModel class that conforms to ObservableObject. Here we define two properties query and repos. @Published property wrapper automatically provides a Publisher for property and allows us to access it via projected value using $ sign.

In the SearchViewModel initializer, we use the flatMap operator to pass the value of query publisher to and generate a new search publisher using the provided query. Then we use the assign operator to save the result of the search operation in the repos property.

=====================================================

Now we can use the SearchViewModel in our SwiftUI app to implement a GitHub repos search screen.

=====================================================

Here we have a SwiftUI view that contains a text field for query terms and a list of results. SearchViewModel will make a network request as soon as the user types something in the query text field. Every new character in the text field generates a new API request. Usually, we want to delay requests till user typing. We can achieve this behavior by using debounce operator on query publisher in the SearchViewModel.

=====================================================

The debounce operator blocks the chain for a time interval that you provide and doesn't emit any values in that period of time. It emits the latest value after the timeout that you provide. We can significantly reduce the number of network requests we make using debounce operator.

=====================================================

#### Advanced chaining with switchToLatest operator
Assume that you have a situation where a user types two queries in a sequence, but the network delays the first one, and the second one finishes earlier. The view represents the search result for the second request, but after a decent amount of time, the first query finishes, and the data appears on the screen by replacing the results of the second query.

Usually, we want to present the results of the latest query and ignore the previous attempts. We can't achieve that with the flatMap operator, and especially for this case, the Combine framework provides us the switchToLatest operator.

=====================================================

In the example above, we refactored our SearchViewModel to use the switchToLatest operator. The switchToLatest operator is designed to work with a publisher that emits other publishers. That's why we use the map operator to transform our publisher into the publisher type that emits search publishers. Then we attach the switchToLatest operator flattening the stream of publishers and delivering results only from the latest one.

#### Conclusion
This week we learned about two different ways of chaining asynchronous operations with the Combine framework. Usually, the flatMap operator is what you need to chain multiple operations. When you should do some work as a response to a user action, you can use the combination of map and switchToLatest operators to deliver the latest results.