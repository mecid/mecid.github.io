---
title: Typed throws in Swift
layout: post
category: Swift Language Features
image: /public/swift.png
---

Swift was promoted as a type-safe programming language on its very first day, and it is solid and safe in many aspects. The part of type safety that needed to be added was throwing functions. Swift 6.0 introduces typed throws, and we will learn all about them this week.

{% include friends.html %}

Let's start by exploring the straightforward definition of a throwing function in Swift 5.9 or any previous language version.

```swift
enum FooError: Error {
    case tooBig
    case tooSmall
}

func foo() throws -> Int {
    let value = Int.random(in: 1...100)
    
    guard value < 60 else {
        throw FooError.tooBig
    }
    
    guard value > 20 else {
        throw FooError.tooSmall
    }
    
    return value
}
```

As you can see in the example above, we define the function that may throw an error while returning an integer value. There is no information on what type of error it may throw. You have to manually scan the function implementation to find all usages of the throw keyword.

```swift
func boo() {
    do {
        let value = try foo()
        print(value)
    } catch let error as FooError {
        switch error {
        case .tooBig: print("Too big...")
        case .tooSmall: print("Too small...")
        }
    } catch {
        print("General erro: \(error)")
    }
}
```

Fortunately, Swift 6.0 allows us to define the throwing error type explicitly. I say it allows because it doesn't require defining a type of throwing error.

```swift
enum FooError: Error {
    case tooBig
    case tooSmall
}

func foo() throws(FooError) -> Int {
    let value = Int.random(in: 1...100)
    
    guard value < 60 else {
        throw FooError.tooBig
    }
    
    guard value > 20 else {
        throw FooError.tooSmall
    }
    
    return value
}
```

We changed the function definition to provide a throwing error type. Now, we can handle every case of the error type in the catch block. The compiler understands the throwing type and will warn you whenever a new case is added to the error type.

```swift
func boo() {
    do {
        let value = try foo()
        print(value)
    } catch {
        switch error {
        case .tooBig: print("Too big...")
        case .tooSmall: print("Too small...")
        }
    }
}
```

Let's talk more about code compatibility with previous versions of the Swift language. You can still use the old way of defining throwing functions, and it works with Swift 6.0. The compiler translates your code from throwing functions to typed throws.

```swift
func foo() throws {
    
}

// becomes

func foo() throws(any Error) {
    
}
```

As you can see, the Swift compiler converts throwing functions to typed throwing using any Error type. Note that even nonthrowing functions translated into throwing where the error type is Never.

```swift
func foo() {
    
}

// becomes

func foo() throws(Never) {
    
}
```

Today, we learned how to use new typed throws definitions of functions to make our codebase even more predictable and type-safe. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week!
