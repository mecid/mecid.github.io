---
title: Glassifying custom SwiftUI views. Groups
layout: post
category: Mastering SwiftUI views
image: /public/glass-container-2.png
---

Glassifying custom views can be very easy using the *glassEffect* view modifier. It is a one-shot view modifier that handles everything for you. But things can become quite complicated when you try to glassify a group of views. Today, we will talk about glassifying groups of views in SwiftUI.

{% include friends.html %}

Why is it not enough to use the *glassEffect* view modifier on a group of views?

```swift
struct ContentView: View {
    var body: some View {
        ScrollView {
            Color.red.frame(height: 300)
            Color.yellow.frame(height: 300)
            Color.green.frame(height: 300)
            Color.black.frame(height: 300)
            Color.orange.frame(height: 300)
            Color.blue.frame(height: 300)
            Color.brown.frame(height: 300)
        }
        .safeAreaInset(edge: .bottom) {
            HStack {
                Image(systemName: "pencil")
                    .font(.title)
                    .frame(width: 40, height: 40)
                    .padding()
                    .glassEffect(.regular.interactive())
                
                Image(systemName: "eraser")
                    .font(.title)
                    .frame(width: 40, height: 40)
                    .padding()
                    .glassEffect(.regular.interactive())
            }
        }
    }
}
```

You can apply the *glassEffect* view modifier individually on a per-view basis. Unfortunately, it is not the way to go. Glass should reflect other glass elements around. To achieve that, you should still use the *glassEffect* view modifier, but also wrap the group of views with the *GlassEffectContainer*.

```swift
struct ContentView: View {
    var body: some View {
        ScrollView {
            // ...
        }
        .safeAreaInset(edge: .bottom) {
            GlassEffectContainer {
                HStack {
                    Image(systemName: "pencil")
                        .font(.title)
                        .frame(width: 40, height: 40)
                        .padding()
                        .glassEffect(.regular.interactive())
                    
                    Image(systemName: "eraser")
                        .font(.title)
                        .frame(width: 40, height: 40)
                        .padding()
                        .glassEffect(.regular.interactive())
                }
            }
        }
    }
}
```

All the glass effects inside the *GlassEffectContainer* can interact with each other and reflect the light. *GlassEffectContainer* not only improves the performance of rendering multiple glass effects but also allows us to morph in and out shapes of glasses.

There is a special *spacing* parameter on the *GlassEffectContainer* allowing you to control the distance between two glasses that will morph after this distance.

```swift
struct ContentView: View {
    var body: some View {
        ScrollView {
            // ...
        }
        .safeAreaInset(edge: .bottom) {
            GlassEffectContainer(spacing: 32) {
                HStack {
                    Image(systemName: "pencil")
                        .font(.title)
                        .frame(width: 40, height: 40)
                        .padding()
                        .glassEffect(.regular.interactive())
                    
                    Image(systemName: "eraser")
                        .font(.title)
                        .frame(width: 40, height: 40)
                        .padding()
                        .glassEffect(.regular.interactive())
                }
            }
        }
    }
}
```

The *spacing* parameter allows us to tune the spacing amount in the layout after which glass shapes should morph in or out. SwiftUI allows us to animate the morphing behavior easily, as many other properties of views.

There is another option allowing us to combine glass shapes together without relying on spacing. The *glassEffectUnion* view modifier allows us to combine a set of glass effects with the same identifier. It might be useful whenever views have too big a distance between them to rely on the spacing parameter and you want to manually indicate that glass effects of these particular views must be combined.

```swift
struct ContentView: View {
    @Namespace var tools
    var body: some View {
        ScrollView {
            // ...
        }
        .safeAreaInset(edge: .bottom) {
            GlassEffectContainer {
                HStack {
                    Image(systemName: "pencil")
                        .font(.title)
                        .frame(width: 40, height: 40)
                        .padding()
                        .glassEffect(.regular.interactive())
                        .glassEffectUnion(id: "tools", namespace: tools)
                    
                    Image(systemName: "eraser")
                        .font(.title)
                        .frame(width: 40, height: 40)
                        .padding()
                        .glassEffect(.regular.interactive())
                        .glassEffectUnion(id: "tools", namespace: tools)
                }
            }
        }
    }
}
```

The *glassEffectUnion* view modifier combines glasses only when they have the same effect types, similar shapes, and identifiers. Keep this in mind when you try to combine glass effects manually.

By using *GlassEffectContainer*, you enable interaction between multiple glass views, improving both the visual consistency and rendering performance. Parameters like spacing allow you to fine-tune morphing behavior, while *glassEffectUnion* gives you precise control over how distant views should visually merge. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week!
