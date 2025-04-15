---
title: Introducing Swift Testing. Scoping.
layout: post
category: Testing
image: /public/testing.png
---

Apple recently released Swift 6.1, with most of the changes being cosmetic. However, I particularly like the scoping feature introduced in the Swift Testing framework. This week, we’ll delve into the new test scoping feature and explore how to effectively utilize it in Swift.

{% include friends.html %}

We already talked about test lifecycle in Swift Testing framework in previous posts. We can utilize init and deinit methods on class types to define setup and teardown functions using Swift Testing framework.

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

Assume that you have a bunch of test cases depending on the same ModelContainer, or predefined data. It means you have to write initialization code again and again for every test case. Fortunately, the Swift Testing framework solves it for us by introducing test scoping functionality. Test scoping works in pair with testing traits. Let me revise your memory with the example of testing traits.

```swift
@Test(
    .enabled(if: FeatureFlag.addition)
) func verifyAdd() {
    let result = add(1, 2)
    #expect(result == 3)
}
```

As you can see in the example above, we can enable or disable particular tests or test suites using testing traits. With the most recent update of the Swift Testing framework, we can go further and annotate with test scopes, allowing us to run custom code before and after the test suite or test function execution.

It means the test scoping features allow us to build powerful and reusable mechanisms, allowing us to customize the test or test suite environment.

```swift
struct Environment {
    let search: (String) async throws -> [String]
}

extension Environment {
    static var production: Environment {
        // production-ready environment
    }
    
    static var mock: Environment {
        // mocked environment
    }
}

extension Environment {
    @TaskLocal static var current = Environment.production
}
```

As you can see in the example above, we define the Environment struct holding all the app dependencies. We also define the task local value for the actual instance of the environment. Task local values are not only a great way of providing shared information with default values, but they also allow us to easily replace one value with another when needed.

In our example, we use task local values with default production-ready dependencies, but our final goal is to provide a mocked version of the environment for testing purposes.

```swift
struct MockEnvironmentTrait: TestTrait, SuiteTrait, TestScoping {
    func provideScope(
        for test: Test,
        testCase: Test.Case?,
        performing function: @Sendable () async throws -> Void
    ) async throws {
        try await Environment.$current.withValue(Environment.mock) {
            try await function()
        }
    }
}

extension Trait where Self == MockEnvironmentTrait {
    static var mockedEnvironment: Self { Self() }
}
```

Here we define the MockEnvironmentTrait type conforming to the TestTrait, SuiteTrait, and TestScoping protocols. The TestScoping protocol has the only requirement, which is the provideScope function. It has a few parameters providing you with information about calling the test function and test suite.

The last parameter is the performing function that describes the particular test function or the whole test suite. You should run the performing function to allow test function or test suite execution.

```swift
@Test(.mockedEnvironment) func verifySomething() async throws {
    Environment.current // provides access to the mocked environment
}

@Suite(.mockedEnvironment) struct ExamplesTests {
    func verifySomething() async throws {
        Environment.current // provides access to the mocked environment
    }
}
```

In our example, we run the performing function inside the closure that overrides the task local value. This technique allows us to run the test suite or test function and provide it with a mocked environment. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week!
