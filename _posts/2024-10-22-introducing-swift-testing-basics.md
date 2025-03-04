---
title: Introducing Swift Testing. Basics.
layout: post
category: Testing
image: /public/testing.png
---

Swift Testing is a new framework with expressive and intuitive APIs that improve your testing experience. It is powered by macros that allow you to organize and assert your tests. This week, we will learn about the basics of the Swift Testing framework and how we can use it side-by-side with XCTest.

{% include friends.html %}

To start using the Swift Testing framework, we only need to import it. Then, we will gain access to all macro types provided by the framework.

```swift
import Testing

func add(_ a: Int, _ b: Int) -> Int {
    a + b
}

@Test func verifyAdd() {
    let result = add(1, 2)
    #expect(result == 3)
}
```

As you can see in the example above, we use the *@Test* macro to annotate our *verifyAdd* function. You don't need to name your test functions with any prefix; you only need to annotate tests with the *@Test* macro. We can also customize the display name to show a different title in the Test navigator.

```swift
@Test("Verify addition function") func verifyAdd() {
    let result = add(1, 2)
    #expect(result == 3)
}
```

You can annotate the functions with *@Test* macro throwing and async. Throwing tests will fail whenever an unhandled error appears. Async test cases allows us to await values and verify the outcomes using the Swift Concurrency feature. You can also use them side-by-side with XCTest classes in a single project.

The *expect* macro allows you to assert the values in your tests. It replaces the whole collection of the assert functions from the XCTest framework with a single one. You place the boolean expression inside the *expect* macro, which will pass whenever it is true and fail whenever it is false.

```swift
@Test func verifyMagicNumber() {
    let number = magicNumber()
    
    #expect(number != 0)
    #expect(number > 10)
    #expect(number <= 100)
}
```

You can use the *expect* macro to verify any boolean expression that you can imagine. As you can see, it replaces many of the *XCAssert* functions that the *XCTest* framework introduced.

Assume that you have a throwing function that might throw an error in a particular case and want to test that behavior. You can still use the *expect* macro to verify the error throw.

```swift
@Test func verifyThrowingFunction() {
    #expect(throws: MyError.self) {
        try throwingFunction()
    }
}
```

Another overload of the *expect* macro allows us to verify the function's throwing behavior and inspect the error itself to confirm that it is the error case you actually desire.

```swift
@Test func verifyThrowingFunction() {
    #expect {
        try throwingFunction()
    } throws: { error in
        guard let myError = error as? MyError else {
            return false
        }
        return myError == .invalidInput
    }
}
```

The second macro type introduced by the Swift Testing framework is the *require*. The *require* macro has the same API as the *expect* macro with two differences. It is a throwing function that throws an error as soon as the boolean expression you pass into is false. Which means it allows us to stop the test when a required condition doesn't meet our expectations.

```swift
@Test func verifyOptionalFunc() throws {
    let result = optionalFunc()
    try #require(result != nil)
    
    #expect(result! > 0)
}
```

Another difference is the ability to unwrap an optional type or stop the test by failing if it is *nil*.

```swift
@Test func verifyOptionalFunc() throws {
    let result = try #require(optionalFunc())
    #expect(result > 0)
}
```

An alternative method to stop the test and document the issue is to use the *record* function of the *Issue* type. This allows us to log the issues in the Test Navigator.

```swift
@Test func verifyOptionalFunc() throws {
    guard let result = optionalFunc() else {
        Issue.record("optional result is nil")
        return
    }
    
    #expect(result > 0)
}
```

The Swift Testing framework integrates well with Xcode, providing a rich, inline representation of test results. It also provides a new way to organize, group, and structure test suites, which I will cover in the next few posts. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week!
