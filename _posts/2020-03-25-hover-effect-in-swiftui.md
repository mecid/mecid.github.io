---
title: Hover effect in SwiftUI
layout: post
image: /public/ipad.jpg
category: Interactions
---
Last week Apple updated iPad Pro and added trackpad support to iPadOS. We finally have Xcode 11.4, which introduces the *onHover* and *hoverEffect* modifiers to help us utilize trackpad and mouse support in SwiftUI. This week we will learn how to be a good iOS citizen and add support for the trackpad to our SwiftUI views.

{% include friends.html %}

#### hoverEffect modifier
SwiftUI provides us the *hoverEffect* modifier that we can attach to any view. This modifier enables the transformation of trackpad or mouse pointer into the covering view shape. It is tough to explain this transformation and better to see. Let's run the example on an iPad simulator.

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

SwiftUI provides us a *HoverEffect* struct that describes three types of transformation of the pointer into a view shape. By default, *hoverEffect* modifier uses the first one, which is called *automatic*. Besides that, we have *highlight* and *lift* transformations. You can use them by only passing it as a parameter to the *hoverEffect* modifier.

```swift
.hoverEffect(.lift)
.hoverEffect(.highlight)
```

Highlight transformation describes an effect that morphs the pointer into a platter behind the view and shows a light source indicating position. On the other hand, lift transformation defines an effect that slides the pointer under the view and disappears as the view scales up and gains a shadow. Usually, we need the *automatic* transformation that attempts to determine the effect automatically.

#### onHover modifier
Now we are familiar with the standard types of hover effect that SwiftUI provides us. But what about custom effects? Happily, SwiftUI enables us to create super custom hover effects by using *onHover* modifier. This modifier allows us to register a closure that will be called whenever the pointer of the trackpad or mouse covers the view. *onHover* modifier enables all the power of animations in SwiftUI that we can use to highlight changes.

```swift
import SwiftUI

struct RootView: View {
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

#### Conclusion
I am pleased to see that iPadOS gains the support of trackpad. I hope we will see Xcode running on iPadOS pretty soon, and iPad will replace Macbook. For now, let's support trackpad and mouse in our apps. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week!