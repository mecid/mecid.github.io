---
title: Customizing view content shape in SwiftUI
layout: post
category: Interactions
image: /public/swiftui.png
---

Usually, SwiftUI uses rectangles to render views, but we can control the shape of the view by using the *clipShape* view modifier. This week we will learn how to modify the interactable shape of the view during hit-testing or previewing drag and drop.

Let's start with a simple view that displays text and an image in a vertical stack.

```swift
struct ContentView: View {
    var body: some View {
        VStack {
            Image(systemName: "star")
            Text("Hello World!")
        }
    }
}
```

Now, we want to make this view tappable and provide the hover effect for the iPadOS pointer.

> To learn more about implementing the hover effect on iPadOS, take a look at my dedicated ["Hover effect in SwiftUI"](/2020/03/25/hover-effect-in-swiftui/) post.

```swift
struct ContentView: View {
    var body: some View {
        VStack {
            Image(systemName: "star")
            Text("Hello World!")
        }
        .hoverEffect()
        .onTapGesture {
            print("Super star!")
        }
    }
}
```

Finally, we can customize the shape of the hover effect by using the *contentShape* view modifier.

```swift
struct ContentView: View {
    var body: some View {
        VStack {
            Image(systemName: "star")
            Text("Hello World!")
        }
        .contentShape(.hoverEffect, Circle())
        .hoverEffect()
        .onTapGesture {
            print("Super star!")
        }
    }
}
```

The *contentShape* view modifier changes the shape of the view used for a particular interaction. We change the content shape only for the hover effect in our example. But we can do it for different types of interaction, like drag and drop previews, hit testing, context menu previews, and hover effect.

The *contentShape* view modifier accepts the instance of the *ContentShapeKinds* option set that defines interactions on a view. SwiftUI provides us these options to use:

1. *interaction* - this one offers a shape for hit testing
2. *dragPreview* - this type provides an outline for drag and drop previews
3. *contextMenuPreview* - this type provides shape for rendering context menu previews.
4. *hoverEffect* - this type provides shape for a hover effect on iPadOS

*ContentShapeKinds* struct conforms to *OptionSet* protocol which means you can combine different options together.

```swift
struct ContentView: View {
    var body: some View {
        VStack {
            Image(systemName: "star")
            Text("Hello World!")
        }
        .contentShape([.hoverEffect, .dragPreview], Circle())
        .hoverEffect()
        .onTapGesture {
            print("Super star!")
        }
    }
}
```

> If you are not familiar with OptionSet protocol, take a look at my ["Inclusive enums with OptionSet"](/2019/04/10/inclusive-enums-with-optionset/) post.

Remember that the *contentShape* view modifier affects only the interactable shape of the view. If you need to change the shape while rendering, you should use the *clipShape* view modifier.

> To learn more about clipping and masking in SwiftUI, take a look at my ["Customizing the shape of views in SwiftUI"](/2020/02/12/customizing-the-shape-of-views-in-swiftui/) post.

```swift
struct Triangle: Shape {
    func path(in rect: CGRect) -> Path {
        var path = Path()

        path.move(to: CGPoint(x: rect.minX, y: rect.maxY))
        path.addLine(to: CGPoint(x: rect.midX, y: rect.minY))
        path.addLine(to: CGPoint(x: rect.maxX, y: rect.maxY))
        path.addLine(to: CGPoint(x: rect.minX, y: rect.maxY))

        return path
    }
}

struct ContentView: View {
    var body: some View {
        VStack {
            Image(systemName: "star")
            Text("Hello World!")
        }
        .contentShape([.hoverEffect, .dragPreview], Triangle())
        .hoverEffect()
        .onTapGesture {
            print("Super star!")
        }
    }
}
```

Keep in mind that you can use any shape you need. As you can see in the example above, we use a custom shape that clips the view content to a triangle.

This week we learned how to customize the shape of the view during interactions via the brand new *contentShape* view modifier. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week!
