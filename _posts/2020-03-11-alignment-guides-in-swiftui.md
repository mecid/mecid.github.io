---
title: Alignment guides in SwiftUI
layout: post
image: /public/swiftui.png
category: Layout
---

This week we will talk about another great tool that we have in SwiftUI. The alignment guide is a way that we can use to speak to SwiftUI's layout system. By using alignment guides, we can easily align views that live in different parts of a view hierarchy.

{% include friends.html %}

#### Basics
SwiftUI provides us a few container views that we can use to build our layout. You might already be familiar with *VStack, HStack, and ZStack*. All of these container views use alignments to regulate the position of child views inside the container. Let's take a look at the very basic example.

```swift
VStack(alignment: .leading) {
    Text("Toronto")
    Text("Paris")
    Text("London")
    Text("Madrid")
}
```

In the example above, we have a vertical container view that displays child views from the top to bottom. We set alignment to leading, and it means that *VStack* will use the leading point of every child view to align them. 

*VStack* uses *HorizontalAlignment* enum to define possible alignments. On the other hand, *HStack* uses *VerticalAlignment* enum. It might be strange, but after all, it is logical. We can set the alignment of *VStack* only in the horizontal way because the container view controls the vertical direction.
*ZStack* uses *Alignment* enum, which is the combination of *HorizontalAlignment* and *VerticalAlignment* enums.

#### Overriding alignment guides
SwiftUI allows us to override standard alignments by using the *alignmentGuide* modifier. For example, we might need to align the bottom of *Image* and *Text* views in a horizontal stack. We can face the problem when image has some spacing inside a bitmap, and it looks not aligned very well. This is a perfect case for overriding an alignment guide.

```swift
struct ContentView: View {
    var body: some View {
        HStack(alignment: .bottom) {
            Image(systemName: "zzz")
                .alignmentGuide(.bottom) { d in d[.bottom] + 8 }
            Text("Sleep")
        }
    }
}
```

As you can see in the example above, we use *alignmentGuide* modifier to override the default value for .*bottom* alignment by adding 8 points. We read the default value by using subscript of **d**, which is an instance of *ViewDimensions* struct. *ViewDimensions* struct provides us access to the width and height of the view and default alignment values by using a subscript.

#### Custom alignment guides
We learned how to override default alignments in SwiftUI, but SwiftUI also allows us to create a custom alignment guide that we can use in container views to align its child views. But why we might need it? Custom alignments allow us to align views that live in different container views. Let's take a look at the example below.

```swift
struct ContentView: View {
    var body: some View {
        HStack(alignment: .center) {
            Image(systemName: "star")
            VStack(alignment: .center) {
                Text("Toronto")
                Text("Paris")
                Text("London")
                Text("Madrid")
            }
        }
    }
}
```

We have a horizontal stack that contains an image and vertical stack with a few text views. I might need to align the image with the third text view, but it doesn't look possible with the current configuration, because these views live in different containers. Fortunately, SwiftUI allows us to create a custom alignment and use it in the parent container view.

```swift
extension VerticalAlignment {
    struct CustomAlignment: AlignmentID {
        static func defaultValue(in context: ViewDimensions) -> CGFloat {
            return context[VerticalAlignment.center]
        }
    }

    static let custom = VerticalAlignment(CustomAlignment.self)
}

struct ContentView: View {
    var body: some View {
        HStack(alignment: .custom) {
            Image(systemName: "star")
            VStack(alignment: .leading) {
                Text("Toronto")
                Text("Paris")
                Text("London")
                    .alignmentGuide(.custom) { $0[VerticalAlignment.center] }
                Text("Madrid")
            }
        }
    }
}
```

In the example above, we use *AlignmentID* protocol to create a custom alignment. This protocol has the only requirement that we need to provide. SwiftUI will use the *defaultValue* whenever we leave it without custom value. The interesting fact is that the inner *VStack* didn't specify a value for custom alignment, but it uses the value that provides its child view.

![align](/public/align.png)

#### Conclusion
Today we learned how powerful could be custom alignments in SwiftUI. We also can use the overriding alignment technique to build super custom layouts. For example, we can build a grid view that arranges child views using alignments. I suggest you play around alignments to understand it well. It is something that can take time. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week!