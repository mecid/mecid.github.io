---
title: Hover effect in SwiftUI
layout: post
image: /public/visionOS.webp
category: Interactions
---

Apple introduced the hover effect a few years ago to improve the interaction of the trackpads on iPadOS. Later, it became available on tvOS, producing the same effect while the user navigated through the app using Apple TV Remote. Nowadays, we can use the hover effect in response to eye focus on visionOS. This week, we will learn all about hover effect interaction in SwiftUI.

{% include friends.html %}

#### hoverEffect view modifier
SwiftUI provides us the *hoverEffect* modifier that we can attach to any view. This modifier enables the transformation of eye focus or mouse pointer into the covering view shape. It is tough to explain this transformation and better to see. Let's run the example on an iPad or Vision Pro simulator.

```swift
import SwiftUI

struct RootView: View {    
    var body: some View {
        Text("Hello World!")
            .hoverEffect()
    }
}
```

Fortunately, iPad simulator supports trackpad simulation. You have to enable it by using *I/O -> Input -> Send cursor to Device* menu. Now we can see the pointer on the screen. Let's cover the text view with the pointer.

SwiftUI provides us the *HoverEffect* struct that describes three types of transformation of the pointer into a view shape. By default, *hoverEffect* modifier uses the first one, which is called *automatic*. Besides that, we have *highlight* and *lift* transformations. You can use them by only passing it as a parameter of the *hoverEffect* modifier.

```swift
struct RootView: View {    
    var body: some View {
        VStack {
            Text("Hello")
                .hoverEffect(.lift)
            Text("World")
                .hoverEffect(.highlight)
        }
    }
}
```

Highlight transformation describes an effect that morphs the pointer into a platter behind the view and shows a light source indicating position. On the other hand, lift transformation defines an effect that slides the pointer under the view and disappears as the view scales up and gains a shadow. Usually, we need the *automatic* transformation that attempts to determine the effect automatically.

```swift
struct ContentView: View {
    @State private var isEnabled = false
    
    var body: some View {
        VStack {
            Toggle(isOn: $isEnabled) {
                Text(isEnabled ? "Disable" : "Enable")
            }
            
            Text("Hello World!")
                .hoverEffect(.lift, isEnabled: isEnabled)
        }
    }
}
```

As you can see in the example above, the *hoverEffect* view modifier also allows us to control whenever to turn the effect on or off by using the *isEnabled* parameter.

The only downside of the *hoverEffect* view modifier is that you must apply it to every view you want to enable the hover effect. You can easily forget to add it to a particular view. Fortunately, SwiftUI provides the *defaultHoverEffect* view modifier, allowing us to enable selected hover effects on every view in the hierarchy with a single line of code.

```swift
struct ContentView: View {
    var body: some View {
        VStack {
            Text("Hello")
            Text("World")
        }
        .defaultHoverEffect(.lift)
    }
}
```

Whenever you use the *defaultHoverEffect* on the whole hierarchy, you can use the *hoverEffectDisabled* view modifier to turn off the hover effect on the particular view.

```swift
struct ContentView: View {
    var body: some View {
        VStack {
            Text("Hello")
            Text("World")
                .hoverEffectDisabled(true)
        }
        .defaultHoverEffect(.lift)
    }
}
```

#### onHover view modifier
Now we are familiar with the standard types of hover effect that SwiftUI provides us. But what about custom effects? Happily, SwiftUI enables us to create super custom hover effects by using *onHover* modifier. This modifier allows us to register a closure that will be called whenever the pointer of the trackpad or mouse covers the view. *onHover* modifier enables all the power of animations in SwiftUI that we can use to highlight changes.

```swift
struct CustomView: View {
    @State private var hovered = false
    
    var body: some View {
        Text("Hello World!")
            .scaleEffect(hovered ? 2.0 : 1.0)
            .animation(.default, value: hovered)
            .onHover { isHovered in
                self.hovered = isHovered
            }
    }
}
```

As you can see in the example above, we use *onHover* modifier to register a closure that delivers us a *Boolean* value. This *Boolean* value is *true* whenever the pointer covers the view. We save the value into a state variable and scale our view using default animation.

> To learn more about the power of animation modifier in SwiftUI, take a look at my ["Animations in SwiftUI"](/2019/06/26/animations-in-swiftui/) post.

When you build the custom view, you can use the *isHoverEffectEnabled* environment value to understand whether to apply a custom hover effect.

```swift
struct CustomView: View {
    @Environment(\.isHoverEffectEnabled) var isEnabled
    @State private var hovered = false
    
    var body: some View {
        Text("Hello World!")
            .scaleEffect(hovered && isEnabled ? 2.0 : 1.0)
            .animation(.default, value: hovered)
            .onHover { isHovered in
                self.hovered = isHovered
            }
    }
}
```

#### onContinuousHover view modifier
SwiftUI also provides the *onContinuousHover* view modifier, allowing us to track the hover phases. For example, you can read the hover's location and react whenever it changes.

```swift
struct CustomView: View {
    @State private var scale = 1.0
    
    var body: some View {
        Text("Hello World!")
            .scaleEffect(scale)
            .animation(.default, value: scale)
            .onContinuousHover { phase in
                switch phase {
                case .active(let location):
                    scale = location.y / location.x
                case .ended:
                    scale = 1
                }
            }
    }
}
```

#### Conclusion
I am pleased to see that iPadOS gains the support of trackpad. I hope we will see Xcode running on iPadOS pretty soon, and iPad will replace Macbook. For now, let's support trackpad and mouse in our apps. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week!
