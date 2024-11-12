---
title: Introducing Swift Testing. Parameterized Tests.
layout: post
category: Testing
image: /public/testing.png
---

I decided to finalize the topic of the Swift Testing framework with its unique feature called parameterized tests. In a few cases, you need to verify your functions with different inputs, and parameterized tests easily solve this by providing you with a nice overview.

{% include friends.html %}

Let's first talk about why we might need parameterized tests. For the example, I will show you a test from my codebase. In my CardioBot app, I try to divide user heart rate measurements into a few heart rate zones. The heart rate zone formula heavily depends on the user’s age, and the normal heart rate values are unique for every user and based on the user’s birthday date.

```swift
@Test func verifyNormalHeartRate() {
    let age = [18, 30, 50, 70]
    let bpm = [77.0, 73, 65, 61]
        
    for (age, bpm) in zip(age, bpm) {
        let hr = HeartRate(bpm: bpm)
        let context = HeartRateContext(age: age, activity: .regular, heartRate: hr)
        #expect(context.zone == .normal)
    }
}
```

As you can see in the example above, the test verifies the normal heart rate for the users of different ages. It looks simple, but as soon as it fails, it will really be hard to understand what values make it fail. Fortunately, the Swift Testing framework introduces the parameterized tests feature, which we can utilize to rewrite our test case and get nice reports clearly indicating the parameter failing the test.

```swift
@Test(arguments: [18, 30, 50, 70], [77.0, 73, 65, 61])
func verifyNormalHeartRate(age: Int, bpm: Double) {
    let hr = HeartRate(bpm: bpm)
    let context = HeartRateContext(age: age, activity: .regular, heartRate: hr)
    #expect(context.zone == .normal)
}
```

Here we use the *arguments* parameter of the *@Test* macro, allowing us to pass parameters to the test functions. As you can see, we define two parameters on the test function itself. Whenever you declare a test function with parameters, you have to use the *arguments* parameter to provide them.

> To learn more about the basics of the Swift Testing framework, take a look at my ["Introducing Swift Testing. Basics."](/2024/10/22/introducing-swift-testing-basics/) post.

By default, the Swift Testing framework uses a combination of all available parameters and runs our test 25 times with different combinations. This might be good for any other test case, but in our case, we need to apply parameters by pairing them. Fortunately, we can solve it by using the *zip* function.

```swift
@Test(arguments: zip([18, 30, 50, 70], [77.0, 73, 65, 61]))
func verifyNormalHeartRate(age: Int, bpm: Double) {
    let hr = HeartRate(bpm: bpm)
    let context = HeartRateContext(age: age, activity: .regular, heartRate: hr)
    #expect(context.zone == .normal)
}
```

As you can see in the example above, we use the *zip* function while using *arguments* parameter of the *@Test* macro. Now, the Swift Testing framework pairs the arguments and run our test 5 times. In the Test Navigator we can clearly see that the test fails on the first pair.

![parameterized-test-navigator](/public/parameterized-test.png)

We use arrays to pass our arguments, but the Swift Testing framework allows us to use any type conforming to the *Collection* protocol, including the *Range* type, which might be useful when dealing with numeric arguments. It also provides a special option for passing instances of the *Zip2Sequence* type, which we used in our examples.

Today, we learned how to use parameterized tests to verify different inputs of our tests. The clear view in the Test Navigator will help you to deal with your tests and write bug-free code. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week!
