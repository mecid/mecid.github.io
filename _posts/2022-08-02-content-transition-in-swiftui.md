---
title: Content transition in SwiftUI
layout: post
---

View transitions are available from the very first version of the SwiftUI framework. The framework can apply a particular transition whenever the view is removed or added to the view hierarchy. The latest iteration of the SwiftUI framework brings us a new type of transition called content transitions. It allows us to apply a particular transition to the content of the view whenever it changes. This week we will learn how to use the new API to apply content transition in SwiftUI.

> To learn more about view transitions in SwiftUI, take a look at my ["Animations in SwiftUI"](/2019/06/26/animations-in-swiftui/) post.

In the previous versions of SwiftUI, we couldn't apply transitions to the view's content. And if you run this example on iOS 15, you will see no transition.

```swift
struct ContentView: View {
    @State private var flag = false
    
    var body: some View {
        VStack {
            Text(verbatim: "1000")
                .fontWeight(flag ? .black : .light)
                .foregroundColor(flag ? .yellow : .red)
        }
        .onTapGesture {
            withAnimation(.default.speed(0.1)) {
                flag.toggle()
            }
        }
    }
}
```

The previous version of SwiftUI doesn't support any transition for *Text* view content, and it applies changes immediately without any visual effect. Fortunately, the latest iteration of SwiftUI allows us to apply a content transition to the *Text* view by using the *contentTransition* view modifier.

```swift
struct ContentView: View {
    @State private var flag = false
    
    var body: some View {
        VStack {
            Text(verbatim: "1000")
                .fontWeight(flag ? .black : .light)
                .foregroundColor(flag ? .yellow : .red)
        }
        .contentTransition(.interpolate)
        .onTapGesture {
            withAnimation(.default.speed(0.1)) {
                flag.toggle()
            }
        }
    }
}
```

As you can see in the example above, the only line of code we added is the *contentTransition* view modifier. It accepts an instance of a particular transition that SwiftUI applies to the view whenever content changes. In this case, we use interpolation because transition affects the size and color of the text.

```swift
struct ContentView: View {
    @State private var flag = false
    
    var body: some View {
        VStack {
            Text(verbatim: "1000")
                .fontWeight(flag ? .black : .light)
                .foregroundColor(flag ? .yellow : .red)
        }
        .contentTransition(.opacity)
        .onTapGesture {
            withAnimation(.default.speed(0.1)) {
                flag.toggle()
            }
        }
    }
}
```

In the current example, we use another instance of *ContentTransition* called opacity. In this case, SwiftUI uses fade in/out whenever the text changes.

SwiftUI provides a type of *ContentTransition* that works with numeric text. It works only with numeric text, understands how the number changed, and provides a nice visual effect that changes only the needed part of the *Text* view representing a number.

```swift
struct TextContentView: View {
    @State private var number = "99"
    
    var body: some View {
        Text(verbatim: number)
            .font(.system(size: 36))
            .contentTransition(.numericText())
            .onTapGesture {
                withAnimation(.default.speed(0.2)) {
                    number = "98"
                }
            }
    }
}
```

> To learn more about other options for animating text changes, take a look at my ["AnimatableModifier in SwiftUI"](/2021/01/11/animatablemodifier-in-swiftui/) post.

The *contentTransition* view modifier passes the provided instance of the *ContentTransition* via the SwiftUI environment and allows us to access it via a particular *EnvironmentKey*.

```swift
struct MySuperCustomTextView: View {
    let text: String
    
    @Environment(\.contentTransition) private var transition
    
    var body: some View {
        switch transition {
        case .opacity:
            drawWithOpacity()
        case .interpolate:
            drawWithInterpolation()
        default:
            draw()
        }
    }
    
    // ...
}
```

Here we have our super custom text view that uses the SwiftUI environment to understand which transition should be used while applying the custom drawing technique of the passed text.

There is another content transition-related *EnvironmentKey*, allowing us to control whenever we want to use GPU-accelerated rendering by wrapping the transition content into a drawing group.

```swift
struct ContentView: View {
    @State private var flag = false
    
    var body: some View {
        VStack {
            Text(verbatim: "1000")
                .fontWeight(flag ? .black : .light)
                .foregroundColor(flag ? .yellow : .red)
        }
        .environment(\.contentTransitionAddsDrawingGroup, true)
        .contentTransition(.interpolate)
        .onTapGesture {
            withAnimation(.default.speed(0.1)) {
                flag.toggle()
            }
        }
    }
}
```

Today we learned about new content transitions in SwiftUI. Not so many views currently support them, but their count can change in the future. Feel free to support it in your views by adopting the new APIs we covered today.
