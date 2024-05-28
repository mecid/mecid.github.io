---
title: If and switch expressions in Swift
layout: post
category: Swift Language Features
image: /public/swift.png
---

One of the silent changes in Swift 5.9 was if and switch expressions. I only saw a little about this option, but it can improve your code in many ways. This week, we will learn about if and switch expressions in Swift.

Let's dive into the practicality of these changes with a real-world example from my codebase. We'll be calculating some health-related metrics, a scenario that many of you may find relatable and interesting. This will help you see the immediate benefits of the new if and switch expressions in Swift 5.9.

```swift
let weight = user.weight.doubleValue(for: .gramUnit(with: .kilo))
let height = user.height.doubleValue(for: .meterUnit(with: .centi))
        
let bmr: Double
        
switch user.gender {
case .male:
    bmr = 66.47 + (13.75 * weight) + (5.0003 * height) - (6.755 * user.age)
default:
    bmr = 655.1 + (9.563 * weight) + (1.85 * height) - (4.676 * user.age)
}
        
let factoredByActivityBMR = bmr * activity.factor

update(
    with: .init(
        unit: .largeCalorie(),
        doubleValue: factoredByActivityBMR
    ), 
    toReach: goal
)
```

As you can see in the example above, we define the *bmr* constant and initialize it later in the switch statement because we have to apply some calculations.

This kind of situation has a few problems with readability due to the loss of local reasoning. We define a non-optional constant without the value, which means we have to set the initial value later in the code before first use to make compiler happy. 

Now, we have to scan the whole function to find all the places providing the initial value for our constant. Fortunately, Swift 5.9 introduced a new way of solving this issue. We can use if and switch statements as expressions.

```swift
let weight = user.weight.doubleValue(for: .gramUnit(with: .kilo))
let height = user.height.doubleValue(for: .meterUnit(with: .centi))
        
let bmr = switch user.gender {
case .male:
    66.47 + (13.75 * weight) + (5.0003 * height) - (6.755 * user.age)
default:
    655.1 + (9.563 * weight) + (1.85 * height) - (4.676 * user.age)
}
        
let factoredByActivityBMR = bmr * activity.factor

update(
    with: .init(
        unit: .largeCalorie(),
        doubleValue: factoredByActivityBMR
    ),
    toReach: goal
)
```

As you can see, we refactored our code. In the example above, we define the *bmr* constant and initialize it inline with the switch expression. Now, we win back local reasoning by guaranteeing that the switch expression provides the initial value to our constant.

Let's discuss some points you must remember while using if and switch expressions in Swift. Each branch of the if, or each case of the switch, must be a single expression. Each of these expressions becomes the value of the overall expression if the branch is chosen.

```swift
let weight = user.weight.doubleValue(for: .gramUnit(with: .kilo))
let height = user.height.doubleValue(for: .meterUnit(with: .centi))
        
let bmr = switch user.gender {
case .male where user.age > 30:
    66.47 + (13.75 * weight) + (5.0003 * height) - (6.755 * user.age)
default:
    655.1 + (9.563 * weight) + (1.85 * height) - (4.676 * user.age)
}
        
let factoredByActivityBMR = bmr * activity.factor

update(
    with: .init(
        unit: .largeCalorie(),
        doubleValue: factoredByActivityBMR
    ),
    toReach: goal
)
```

As you can see in the example above, you can still use pattern matching or where clauses while using if and switch as expressions. Let's look at the example where we use the if as an expression.

```swift
let ruleValue = if values.isEmpty { 0.0 } else { values.reduce(0, +) / Double(values.count) }

// let ruleValue = values.isEmpty ? 0 : values.reduce(0, +) / Double(values.count)
```

Using if as the expression looks similar to using the ternary operator at first glance. I prefer the ternary in this particular case, but the real power of the new approach is revealed when you need to use the if with pattern matching in the expression.

> To learn more about pattern matching in Swift, take a look at my dedicated ["Pattern Matching with case let"](/2019/02/06/pattern-matching-with-case-let/) post.

Today, we learned how to use if and switch as expressions in Swift 5.9. Switch as expression will become very handy when you start using it. I already introduced it many times across my codebase and enjoy how it improves the local reasoning of my code. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week!
