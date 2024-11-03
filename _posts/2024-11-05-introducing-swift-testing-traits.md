---
title: Introducing Swift Testing. Traits.
layout: post
---

The most powerful feature of the Swift Testing framework is the trait system. Traits allow us to annotate a test or test suite to customize its behavior. This week, we will learn how to use built-in trait types to modify tests.

The Test and Suite macros allow us to enable a trait or set of traits related to the test or test suite and pass built-in trait types.

```swift
@Test(.disabled()) func verifyAdd() {
    let result = add(1, 2)
    #expect(result == 3)
}
```

As you can see in the example above, we use the disabled trait to skip the test. Xcode Test Navigator shows disabled tests as skipped by coloring them in gray.

```swift
@Test(.disabled("Disabled for some reason")) func verifyAdd() {
    let result = add(1, 2)
    #expect(result == 3)
}
```

You can also provide a comment to indicate the reason for skipping the test, which will be easily accessible in the Test Navigator. The Swift Testing framework allows us to skip tests conditionally. For example, it might be useful to skip tests for features disabled by a particular flag.

```swift
@Test(
    .enabled(if: FeatureFlag.addition)
) func verifyAdd() {
    let result = add(1, 2)
    #expect(result == 3)
}
```

As you can see, we use the enabled trait with a boolean value to conditionally allow the test. We can use conditions both with disable and enable test traits.

The real power of the traits system appears when you combine multiple traits to achieve the desired behavior. For example, you can use numerous enabled traits to run the test only when all conditions are met.

```swift
@Test(
    .enabled(if: FeatureFlag.addition),
    .enabled(if: FeatureFlag.anotherFlag)
) func verifyAdd() {
    let result = add(1, 2)
    #expect(result == 3)
}
```

Whenever you use multiple conditions, all of them should pass to run the test. Sometimes, you might skip the test because of a bug. For this case, the Swift Testing introduces the bug trait, allowing us to attribute the corresponding bug identifier or URL.

```swift
@Test(
    .enabled(if: FeatureFlag.addition),
    .enabled(if: FeatureFlag.anotherFlag),
    .bug(id: "0001")
) func verifyAdd() {
    let result = add(1, 2)
    #expect(result == 3)
}
```

The Swift Testing framework allows us not only to skip tests but also to limit them by time. For example, you might have a test that takes much time and resources to run. In this case, you can limit it by time to fail on timeout.

```swift
@Suite(.timeLimit(.minutes(1))) struct TestSuite {
    @Test func verifyFoo() {
        // ...
    }
    
    @Test func verifyBoo() {
        // ...
    }
}
```

Whenever you apply the time limit trait to a test suite, it recursively applies to every test.

Suites allow us to organize related tests. In many cases, we might need another organization type that allows merging multiple suites or selecting only crucial functionality. For this purpose, the Swift Testing framework introduces tagging functionality.

```swift
extension Tag {
    @Tag static var crucial: Self
}
```

First of all, we have to define a tag. We can create tags only by defining extensions on the Tag type in pair with the Tag macro. Then, we can use the tag trait to add tags to the particular test or test suite.

```swift
@Test(.tags(.crucial)) func veryImportantVerification() {
    
}

@Test(.tags(.crucial)) func anotherImportantVerification() {
    
}
```

As you can see in the example above, we define the crucial tag and mark two test cases with it. The Test Navigator in Xcode has a special tab showing tags allowing you to run the tests by tag and visualize if all the tests in the tag pass.
