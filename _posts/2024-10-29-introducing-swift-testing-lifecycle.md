---
title: Introducing Swift Testing. Lifecycle
layout: post
category: Testing
image: /public/testing.png
---

Any function marked with the *@Test* macro can be a test in the world of the Swift Testing framework. But how do you handle the lifecycle of the tests? How do you define test suites and provide setup and teardown functionality? This week, we will learn how to handle the test lifecycle in Swift Testing.

{% include friends.html %}

Let's take a look at a simple example of a test using the Swift Testing framework.

```swift
@Test func verifyAdd() {
    let result = add(1, 2)
    #expect(result == 3)
}
```

As you can see in the example above, we use the *@Test* macro to mark our *verifyAdd* function. That's it. It is all you need to define a test using the Swift Testing framework.

> To learn more about the basics of the Swift Testing framework, take a look at my ["Introducing Swift Testing. Basics."](/2024/10/22/introducing-swift-testing-basics/) post.

Let's discuss a more complex example where you have a set of tests that you want to group in a single test suite. In this case, you can simply embed a set of test functions into a struct.

```swift
struct MathTests {
    @Test func verifyAdd() {
        let result = add(1, 2)
        #expect(result == 3)
    }
    
    @Test func verifyMultiply() {
        let result = multiply(1, 2)
        #expect(result == 2)
    }
}
```

Xcode allows us to run test suites with a single click and group them in the Test Navigator. Another benefit of using test suites is the setup and teardown functions.

```swift
struct ModelTests {
    let container: ModelContainer
    
    init() throws {
        let config = ModelConfiguration(isStoredInMemoryOnly: true)
        container = try ModelContainer(for: User.self, configurations: config)
    }
    
    @Test func verifyBulkImport() throws {
        User.bulkImport()
        
        let modelContext = ModelContext(container)
        let count = try modelContext.fetchCount(FetchDescriptor<User>())
        
        #expect(count == 100)
    }
}
```

As you can see in the example above, we define the *init* function in the *ModelTests* type. Swift Testing will call this function before every test case. We can use the *init* function to set up a database connection, preload mock data, etc.

Whenever you need the teardown function to run after each test case, you can change the type of the test suite from *struct* to *class* and use the *deinit* function.

```swift
class ModelTests {
    let container: ModelContainer
    
    init() throws {
        let config = ModelConfiguration(isStoredInMemoryOnly: true)
        container = try ModelContainer(for: User.self, configurations: config)
    }
    
    deinit {
        container.deleteAllData()
    }
    
    @Test func verifyBulkImport() throws {
        User.bulkImport()
        
        let modelContext = ModelContext(container)
        let count = try modelContext.fetchCount(FetchDescriptor<User>())
        
        #expect(count == 100)
    }
}

```

In the example above, we change the *ModelTests* type from being a *struct* to a *class*. Now we can define the *deinit* function. The Swift Testing will run the *deinit* function after every test case. In our example, we use it to erase all data after each test.


The Swift Testing framework runs tests in parallel with each other using task groups. You might need to run them serially in some cases, and you can achieve that using the *@Suite* macro.

```swift
@Suite(.serialized) class ModelTests {
    let container: ModelContainer
    
    init() throws {
        let config = ModelConfiguration(isStoredInMemoryOnly: true)
        container = try ModelContainer(for: User.self, configurations: config)
    }
    
    deinit {
        container.deleteAllData()
    }
    
    @Test func verifyBulkImport() throws {
        User.bulkImport()
        
        let modelContext = ModelContext(container)
        let count = try modelContext.fetchCount(FetchDescriptor<User>())
        
        #expect(count == 100)
    }
}
```

As you can see in the example above, we use the *@Suite* macro with the *serialized* option. Swift Testing will run test cases from this particular suite one by one. The Swift Testing framework doesn't require the *@Suite* macro and implicitly creates suites when tests are grouped in a type.

Still, we can use the *@Suite* macro for additional attribution. For example, we can provide a title with the *@Suite* macro to tune the suite's display name in the Test navigator.

```swift
@Suite("Serial model tests", .serialized) class ModelTests {
    let container: ModelContainer
    
    init() throws {
        let config = ModelConfiguration(isStoredInMemoryOnly: true)
        container = try ModelContainer(for: User.self, configurations: config)
    }
    
    deinit {
        container.deleteAllData()
    }
    
    @Test func verifyBulkImport() throws {
        User.bulkImport()
        
        let modelContext = ModelContext(container)
        let count = try modelContext.fetchCount(FetchDescriptor<User>())
        
        #expect(count == 100)
    }
}
```

The *@Suite* macro has powerful functionality that allows us to set traits and tags, which I will cover in the upcoming post. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week!
