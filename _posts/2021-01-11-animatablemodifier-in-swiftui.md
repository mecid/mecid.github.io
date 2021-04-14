---
title: AnimatableModifier in SwiftUI
layout: post
image: /public/swiftui.png
category: Interactions
---

I have already talked about animations in SwiftUI many times on this blog. But still didn't cover all the opportunities in terms of animation. Today I want to fill another gap and talk to you about the *AnimatableModifier* protocol that opens new horizons for animations.

{% include friends.html %}

We use many modifiers to build a view in SwiftUI like *frame*, *foregroundColor*, *padding*, *background*, etc. The main goal of a modifier is adding some logic to create a slightly modified view.

> To learn more about modifiers, take a look at my ["ViewModifiers in SwiftUI"](/2019/08/07/viewmodifiers-in-swiftui/) post.

The single concern that I have about the *ViewModifier* protocol is the animation opportunity. You simply can't animate the behavior inside a *ViewModifier*. You may ask about the *Animatable* protocol to conform to our custom view modifier, but it doesn't work. It doesn't work, but Apple provides us a way to handle animation in view modifiers, and it is *AnimatableModifier*.

```swift
@available(iOS 13.0, macOS 10.15, tvOS 13.0, watchOS 6.0, *)
public protocol AnimatableModifier : Animatable, ViewModifier {
}
```

*AnimatableModifier* protocol extends from *Animatable* and *ViewModifier* protocols and allows us to animate the changes inside the type that conforms to it. Let's take a look at the quick example.

```swift
struct NumberView: AnimatableModifier {
    var number: Int

    var animatableData: CGFloat {
        get { CGFloat(number) }
        set { number = Int(newValue) }
    }

    func body(content: Content) -> some View {
        Text(String(number))
    }
}
```

As you can see in the example above, we create the *NumberView* type that conforms to *AnimatableModifier*. In the *body* function of our modifier, we render our number using the *Text* view. *AnimatableModifier* runs the *body* function multiple times during the animation and creates a smooth transition from one state to another. 

Usually, we use the content parameter inside the body and apply additional modifiers to it. Here we completely ignore the content and instead create a brand new view. Let's try to use the new *NumberView* modifier.

```swift
struct ContentView: View {
    @State private var number = 0

    var body: some View {
        VStack {
            Text(String(number))
                .modifier(NumberView(number: number))

            Button("Animate") {
                withAnimation(Animation.easeInOut(duration: 2)) {
                    number = 100
                }
            }
        }
    }
}
```

In the example above, we animate the number that appears on the screen. SwiftUI uses *animatableData* and vector arithmetic to interpolate the value of the number variable. SwiftUI presents a new text view on every iteration of the animation.

> If you are not familiar with *Animatable* protocol and vector arithmetic, take a look at my ["The magic of Animatable values in SwiftUI"](/2020/06/17/the-magic-of-animatable-values-in-swiftui/) post.

![video](/public/am.mp4)

#### Conclusion
Today we learned about another hidden gem of SwiftUI. *AnimatableModifier* allows us to animate the things that SwiftUI doesn't allow us to animate out of the box, like text. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this article. Thanks for reading, and see you next week!