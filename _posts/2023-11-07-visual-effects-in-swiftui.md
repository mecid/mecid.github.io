---
title: Visual effects in SwiftUI
layout: post
category: Interactions
image: /public/swiftui.png
---

During WWDC 23, SwiftUI introduced a new view modifier called *visualEffect*. This modifier allows us to attach a set of animatable visual effects by accessing layout information of the particular view. This week, we will learn how to use the new *visualEffect* view modifier in SwiftUI.

{% include friends.html %}

Let's start with the most simple example of using the *visualEffect* view modifier.

```swift
struct ContentView: View {
    var body: some View {
        Text("Hello World!")
            .visualEffect { initial, geometry in
                initial.offset(geometry.size)
            }
    }
}
```

As you can see in the example above, we define a text view and attach the *visualEffect* view modifier. Whenever you attach the *visualEffect* view modifier, you should specify the effect closure. This is the place where you apply all the effects that you need.

The effect closure provides you with two parameters. The first is an initial state of the collections of effects attached to the view. It is an instance of the *EmptyVisualEffect* type. We use this instance to attach additional effects. The second parameter is an instance of the *GeometryProxy* type containing all the layout information of the view you might need, like frame, safe area, etc.

> To learn how to use the *GeometryProxy* type, take a look at my ["Mastering ScrollView in SwiftUI"](/2020/09/24/mastering-scrollview-in-swiftui/) post.

What is a visual effect? The visual effect is anything that can change the visual appearance of the view but doesn't affect its layout. In the previous iterations of the SwiftUI framework, we had view modifiers like *scale*, *offset*, *blur*, *contrast*, *saturation*, *opacity*, *rotation*, etc. All of them are visual effects and conform to the *VisualEffect* protocol now. You can use any of them inside the *visualEffect* closure.

```swift
struct ContentView: View {
    
    var body: some View {
        Text("Hello World!")
            .visualEffect { initial, geometry in
                initial
                    .blur(radius: 8)
                    .opacity(0.9)
                    .scaleEffect(.init(width: 2, height: 2))
            }
    }
}
```

Things like *frame* and *padding* are not visual effects, and you can't use them inside the *visualEffect* closure because they modify the layout of the view hierarchy.

The *visualEffect* view modifier is the new way to do old things. We can modify the opacity and offset of the view using old view modifiers. And you can continue to use them if you don't need the layout information. The only difference of the new approach is the way we scope the visual effects of the view by calculating them from the layout information that *GeometryProxy* provides us.

The *visualEffect* view modifier supports animatable values. So you can continue using it to animate your view's visual appearance depending on its frame and bounds in the view hierarchy that you can access via an instance of the *GeometryProxy* type.

```swift
struct ContentView: View {
    @State private var isScaled = false
    
    var body: some View {
        VStack {
            Button("Scale") {
                isScaled.toggle()
            }
            
            Text("Hello World!")
                .visualEffect { initial, geometry in
                    initial.scaleEffect(
                        CGSize(
                            width: isScaled ? 2 : 1,
                            height: isScaled ? 2 : 1
                        )
                    )
                }
                .animation(.smooth, value: isScaled)
        }
    }
}
```

Today, we learned about the benefits and usage of the new *visualEffect* view modifier in SwiftUI. It is not backward compatible with previous versions of Apple platforms, so keep in mind that you can achieve the same effect by using old but gold view modifiers. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week!
