---
title: Introducing Animatable macro in SwiftUI
layout: post
---

Easy-peasy animations were always one of the strongest points of the SwiftUI framework. This time Apple makes them even easier by introducing a new Animatable macro. This week, we will learn when and how to use the new Animatable macro.

Let’s start first with a simple IntegerView. It takes an integer value and displays it with a Text by formatting the value to a string.

```swift
struct IntegerView: View {
    var number: Int
    
    var body: some View {
        Text(number.formatted())
    }
}

struct ContentView: View {
    @State private var number = 0
    
    var body: some View {
        IntegerView(number: number)
            .animation(.default.speed(0.5), value: number)
        
        Button("Animate") {
            number = 100
        }
    }
}
```

As soon as you want to animate it from 0 to 100, you will notice that SwiftUI doesn’t know how to enumerate the value and simply animates text change. We need somehow to indicate that the number property should be enumerated to its new value. For this particular case, the SwiftUI framework introduced the Animatable protocol that we can use to mark animatable properties of a view type.

With the recent release of the SwiftUI framework, it simplified the usage of the Animatable protocol by introducing the Animatable macro. Now you can simply mark the view or shape type with the Animatable macro, and it will animate all the stored properties.

```swift
@Animatable
struct IntegerView: View {
    var number: Float
    
    var body: some View {
        Text(number.formatted(.number.precision(.fractionLength(0))))
    }
}
```

As you can see, there are a few changes on the IntegerView. First, we have added the Animatable macro. Second, we have changed the type of the number property from Int to Float. We need that because Animatable macro only works with types conforming to the VectorArithmetic protocol.

```swift
struct ContentView: View {
    @State private var number: Float = 0
    
    var body: some View {
        IntegerView(number: number)
            .animation(.default.speed(0.5), value: number)
        
        Button("Animate") {
            number = 100
        }
    }
}
```

Now, we can animate the number from 0 to 100 with a nice animation. By default, the Animatable macro tries to animate all the stored properties of the type, but you can exclude some of them by using the AnimatableIgnored macro.

```swift
@Animatable
struct IntegerView: View {
    var number: Float
    
    @AnimatableIgnored
    var ignoredValue: Float
    
    var body: some View {
        Text(number.formatted(.number.precision(.fractionLength(0))))
    }
}
```

Types annotated with the Animatable macro get conformance to the Animatable protocol automatically; you don’t need to conform manually. The macro doesn’t affect its conformance whenever the type already conforms to the Animatable protocol.

The new @Animatable macro is another great addition to SwiftUI that makes animations even easier to implement. With just a simple annotation, you can unlock smooth transitions for your view’s properties without manually conforming to the Animatable protocol. And when you need more control, the @AnimatableIgnored macro lets you exclude properties from animation. 
