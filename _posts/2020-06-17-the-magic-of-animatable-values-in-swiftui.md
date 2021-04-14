---
title: The magic of Animatable values in SwiftUI
layout: post
image: /public/swiftui.png
category: Interactions
---

*WWDC20* is already around the corner, and we are waiting for massive changes and additions to the SwiftUI framework. It is a perfect week to wrap up the season with a post about one of the strongest sides of the SwiftUI framework, which is *animation*. Today we will learn how to build complex animations by using *VectorArithmetic* protocol.

{% include friends.html %}

#### Basics
Let's start with the basics. Usually, we attach the animation modifier to a view and change some state variables. SwiftUI docs say that animation modifier applies the given animation to all **animatable** values within this view. Here is a small example that animates the button on every tap.

```swift
struct RootView: View {
    @State private var scale: CGFloat = 1

    var body: some View {
        Button("Press me") {
            self.scale += 1
        }
        .padding()
        .foregroundColor(.white)
        .background(Color.blue)
        .cornerRadius(8)
        .scaleEffect(scale)
        .animation(.default, value: scale)
    }
}
```

> To learn more about basics of animation in SwiftUI, take a look at my ["Animations in SwiftUI" post](/2019/06/26/animations-in-swiftui/).

But how SwiftUI understand which values are **animatable**? SwiftUI introduces a protocol called *Animatable*. This protocol has the only requirement, *animatableData* property, that describes the changes during an animation. So during the state changes, SwiftUI traverses the view hierarchy and finds all the values that conform to *Animatable* protocol and animate its changes by understanding *animatableData* of a particular item. Let's take a look at another example.

```swift
struct Line1: Shape {
    let coordinate: CGFloat

    func path(in rect: CGRect) -> Path {
        Path { path in
            path.move(to: .zero)
            path.addLine(to: CGPoint(x: coordinate, y: coordinate))
        }
    }
}

struct RootView: View {
    @State private var coordinate: CGFloat = 0

    var body: some View {
        Line1(coordinate: coordinate)
            .stroke(Color.red)
            .animation(Animation.linear(duration: 1).repeatForever())
            .onAppear { self.coordinate = 100 }
    }
}
```

Here we have a *Line* struct that conforms to the *Shape* protocol. All shapes in SwiftUI conform to *Animatable* protocol, but as you can see, there is no animation while running the example. SwiftUI doesn't animate our line because the framework doesn't know how to animate it. 

> To learn more about *Shape API* in SwiftUI, take a look at my ["Building BarChart with Shape API in SwiftUI" post](/2019/08/14/building-barchart-with-shape-api-in-swiftui/).

By default, shape returns the instance of *EmptyAnimatableData* struct as its *animatableData*. SwiftUI allows us to use *EmptyAnimatableData* whenever we don't know how to animate the value. Let's solve the problem by replacing *EmptyAnimatableData* with some value.

```swift
struct Line1: Shape {
    var coordinate: CGFloat

    var animatableData: CGFloat {
        get { coordinate }
        set { coordinate = newValue }
    }

    func path(in rect: CGRect) -> Path {
        Path { path in
            path.move(to: .zero)
            path.addLine(to: CGPoint(x: coordinate, y: coordinate))
        }
    }
}
```

In the example above, we made our *Line* struct animatable by implementing *animatableData* property. But how we can animate multiple properties of the same shape? There is another type called *AnimatablePair* that helps us to animate the paired values. Let's make our *Line* struct animatable in both vertical and horizontal directions.

```swift
struct Line2: Shape {
    var x: CGFloat
    var y: CGFloat

    var animatableData: AnimatablePair<CGFloat, CGFloat> {
        get { AnimatablePair(x, y) }
        set {
            x = newValue.first
            y = newValue.second
        }
    }

    func path(in rect: CGRect) -> Path {
        Path { path in
            path.move(to: .zero)
            path.addLine(to: CGPoint(x: x, y: y))
        }
    }
}
```

OK, nice. Now we can animate two values of the same shape. But what about the line chart? Assume that you are working on the charting library, and you want to build an animatable line chart? There might be hundreds of values that we want to animate. How could we deal with this challenge?

#### Introducing VectorArithmetic protocol
As you can see, we have already used *CGFloat* and *AnimatablePair* types as animatable data. But it doesn't mean that you can use any type here. *Animatable* protocol has a constraint on *animatableData* property. Any type that conforms to *VectorArithmetic* protocol can be used as *animtableData*. SwiftUI provides us a few types that conform to *VectorArithmetic* protocol out of the box. For example, *Float*, *Double*, *CGFloat*, and *AnimatablePair*. 

Let's back to our line chart idea. I want to make a line chart shape that animates all the values. There are probably could be hundreds of points, and it looks like an excellent candidate for a custom type that conforms to *VectorArithmetic* protocol. *VectorArithmetic* has a few requirements like scaling, adding, subtracting, etc. You should describe how SwiftUI must handle these operations on your type. Here is a drop-in implementation for an array of values.

```swift
import SwiftUI
import enum Accelerate.vDSP

struct AnimatableVector: VectorArithmetic {
    static var zero = AnimatableVector(values: [0.0])

    static func + (lhs: AnimatableVector, rhs: AnimatableVector) -> AnimatableVector {
        let count = min(lhs.values.count, rhs.values.count)
        return AnimatableVector(values: vDSP.add(lhs.values[0..<count], rhs.values[0..<count]))
    }

    static func += (lhs: inout AnimatableVector, rhs: AnimatableVector) {
        let count = min(lhs.values.count, rhs.values.count)
        vDSP.add(lhs.values[0..<count], rhs.values[0..<count], result: &lhs.values[0..<count])
    }

    static func - (lhs: AnimatableVector, rhs: AnimatableVector) -> AnimatableVector {
        let count = min(lhs.values.count, rhs.values.count)
        return AnimatableVector(values: vDSP.subtract(lhs.values[0..<count], rhs.values[0..<count]))
    }

    static func -= (lhs: inout AnimatableVector, rhs: AnimatableVector) {
        let count = min(lhs.values.count, rhs.values.count)
        vDSP.subtract(lhs.values[0..<count], rhs.values[0..<count], result: &lhs.values[0..<count])
    }

    var values: [Double]

    mutating func scale(by rhs: Double) {
        values = vDSP.multiply(rhs, values)
    }

    var magnitudeSquared: Double {
        vDSP.sum(vDSP.multiply(values, values))
    }
}
```

As you can see, I use the *Accelerate* framework that provides us high-performance vector-based arithmetic. By using *AnimatableVector* struct, we can animate as many values as we want, and it will work super fast because it uses the *Accelerate* framework. Now we have all we need to implement an animatable line chart shape.

```swift
import SwiftUI

struct LineChart: Shape {
    var vector: AnimatableVector

    var animatableData: AnimatableVector {
        get { vector }
        set { vector = newValue }
    }

    func path(in rect: CGRect) -> Path {
        Path { path in
            let xStep = rect.width / CGFloat(vector.values.count)
            var currentX: CGFloat = xStep
            path.move(to: .zero)

            vector.values.forEach {
                path.addLine(to: CGPoint(x: currentX, y: CGFloat($0)))
                currentX += xStep
            }
        }
    }
}
```

You can download the source code of *AnimatableVector* through [Github gist](https://gist.github.com/mecid/18a80b18cc9670eef1d8667cf8c886bd).

```swift
struct RootView: View {
    @State private var vector: AnimatableVector = .zero

    var body: some View {
        LineChart(vector: vector)
            .stroke(Color.red)
            .animation(Animation.default.repeatForever())
            .onAppear { 
                self.vector = AnimatableVector(values: [50, 100, 75, 100]) 
            }
    }
}
```
![linechart-animation](/public/linechart.gif)

#### Conclusion
I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week!