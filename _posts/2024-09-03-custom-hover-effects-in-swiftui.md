---
title: Custom hover effects in SwiftUI
layout: post
category: Interactions
image: /public/visionOS.webp
---

Since purchasing Apple Vision Pro, I have been fully immersed in adapting my applications to visionOS. The first thing I noticed on the device was the need to customize hover effects in some views. This week, we will talk about building custom hover effects in SwiftUI.

First of all, hover effects are not specific to visionOS only. The same API is used on tvOS to build interactions while the user navigates using remote and macOS, where the user uses a mouse or trackpad.

> To learn more about the basic hover effects that SwiftUI provides us, take a look at my "Hover effect in SwiftUI" post.

Let's take a look at the simple example of building a custom hover effect. Assume that you have a rounded button and want to scale it a bit whenever the user looks at it or drags the mouse onto it.

```swift
struct ContentView: View {
    var body: some View {
        Button("Play", systemImage: "play.fill") {
            // some action here...
        }
        .labelStyle(.iconOnly)
        .hoverEffect { effect, isActive, geometry in
            effect.scaleEffect(isActive ? 1.1 : 1.0)   
        }
    }
}
```

As you can see in the example above, we use the hoverEffect view modifier to build our custom effect. The hoverEffect view modifier works similarly to the visualEffect view modifier and provides us with a list of parameters we can use to implement our effect.

The first parameter is the empty effect stub, which we can use to add more effects. The second parameter is a boolean value that becomes true whenever the user looks at the view. The third parameter is an instance of the GeometryProxy type, which allows us to read the necessary geometry data and derive its effect.

You may clutter the user experience with many custom hover effects, all of which will move and highlight the user interface while the user is looking around. Fortunately, custom hover effects support animation, which allows us to delay our effects slightly.

```swift
struct ContentView: View {
    var body: some View {
        Button("Play", systemImage: "play.fill") {
            // some actions here...
        }
        .labelStyle(.iconOnly)
        .hoverEffect { effect, isActive, geometry in
            effect.animation(.default.delay(isActive ? 0.8 : 0.2)) {
                $0.scaleEffect(isActive ? 1.1 : 1.0)
            }
        }
    }
}
```

As you can see, we use the animation function on an empty hover effect stub to provide some delay. Animated delays are crucial whenever your custom effect significantly impacts the view, like changing its size to expand the content.

The custom scale hover effect we built above is neat, and I might need to use it in other parts of my app. I want to avoid copying and pasting it across my codebase. For this particular case, SwiftUI introduces the CustomHoverEffect protocol.

```swift
struct ScaleEffect: CustomHoverEffect {
    func body(content: Content) -> some CustomHoverEffect {
        content.hoverEffect { effect, isActive, geometry in
            effect.animation(.default) {
                $0.scaleEffect(isActive ? 1.1 : 1.0)
            }
        }
    }
}

struct ContentView: View {
    var body: some View {
        Button("Play", systemImage: "play.fill") {
            
        }
        .labelStyle(.iconOnly)
        .hoverEffect(ScaleEffect())
    }
}
```

Here, we create the ScaleEffect type that conforms to the CustomHoverEffect protocol. As with many other protocols introduced by SwiftUI, the only requirement is the body function, where you implement your effect's logic.

As you can see, we copy the content of the hoverEffect view modifier from our view and move it inside the ScaleEffect type without any changes.

Today, we learned how to build custom hover effects in SwiftUI and discussed the importance of delaying effects that impact the size of the views.
