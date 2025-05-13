---
title: Optimized mathematical computations in Swift
layout: post
category: Swift Language Features
---

I’m very passionate about my health routine and have built a bunch of health-related apps. Almost all of them are integrated with Apple Health and provide tons of additional calculations over the functionality that the Apple Health app gives us. Today, we will talk about the hidden gem of the on-device calculations - the Accelerate framework.

The Accelerate framework is a huge topic, and it deserves a series of posts to cover at least half of its functionality. I will start with the most common stuff that you might need in your apps. But first, let’s define what the Accelerate framework is. Apple introduced the Accelerate framework many years ago, so it is available almost on every version of Apple platforms. It is a high-performance and energy-efficient way of doing computations using the vector-processing capabilities of the device.

The Accelerate framework contains a collection of APIs for digital signal processing called *vDSP*. It provides tons of highly optimized functions for operations on large data collections. Let’s start with the most basic one. How often do you find yourself writing code like this?

```swift
let values = [6,6,8,7,8,10,10.0]
let sum = values.reduce(0.0, +)
```

It might look simple and fast when you have 10-20 values, but what about millions of data points? We can leverage the power of vector-processing capabilities of the device by using the *vDSP* functions provided by the Accelerate framework.

```swift
import Accelerate

let values = [6,6,8,7,8,10,10.0]
let sum = vDSP.sum(values)
```

The second most common function I use is the *mean* function. Again, you can easily implement it in Swift using reduce and divide by count, but the performance is significantly different on large amounts of data.

```swift
let values = [6,6,8,7,8,10,10.0]
let mean = values.reduce(0.0, +) / Double(values.count)

// Use vDSP instead
let mean = vDSP.mean(values)
```

Let’s move forward to more advanced statistical analysis you might need. Standard deviation is a pretty useful metric not only in health apps, but also in the financial category. Standard deviation is a measure of how dispersed the data is in relation to the mean.

For example, you might have an average sleep duration of 8 hours during the week, but you sleep about 6-7 hours in the first part of the week and 10 hours on weekends. It might look good, because in average you sleep about 8 hours, but it is not, because variation between your asleep duration is too big and you might need to improve your bedtime routine.

The Accelerate framework provides us the *standardDeviation* function allowing us to calculate the standard deviation in a single line.

```swift
let values = [6,6,8,7,8,10,10.0]
let sd = vDSP.standardDeviation(values)

if sd > 0.5 {
    print("improve your bedtime routine!")
} else {
    print("you have a good sleep schedule!")
}
```

The *standardDeviation* function is available only on the most recent platform version. But fortunately, we can implement it using other functions provided by the Accelerate framework. The standard deviation is the root mean square of the difference between values and the mean.

```swift
extension vDSP {
    static func sd(_ vector: some AccelerateMutableBuffer<Double>) -> Double {
        guard vector.count > 1 else {
            return 0.0
        }
        
        let mean = vDSP.mean(vector)
        let subtractedMean = Array(repeating: mean, count: vector.count)
        let differences = vDSP.subtract(vector, subtractedMean)
        return vDSP.rootMeanSquare(differences)
    }
}
```

As you can see in the example above, we leverage the power of *mean*, *subtract*, and *rootMeanSquare* functions to calculate standard deviation in a very performant way.

The Accelerate framework provides us with tons of useful functionality from building on-device machine learning to Fourier transformation. I highly recommend you check out the documentation to find useful stuff for your apps.
