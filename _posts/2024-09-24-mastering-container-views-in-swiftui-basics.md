---
title: Mastering container views in SwiftUI. Basics.
layout: post
category: View Composition
image: /public/container.png
---

Since the very first version of the framework, SwiftUI has had several container views. The most popular ones are *HStack*, *VStack*, *List*, etc. This year, Apple introduced new APIs that allow us to build custom container views in a new way. This week, we will learn about the benefits of SwiftUI's new decomposition APIs.

{% include friends.html %}

What is a container view? It is a view holding other views. We can easily define a container view using *@ViewBuilder* closures. Here is an example.

```swift
struct Card<Content: View>: View {
    @ViewBuilder var content: Content
    
    var body: some View {
        VStack {
            content
        }
        .padding()
        .background(Material.regular, in: .rect(cornerRadius: 8))
        .shadow(radius: 4)
    }
}
```

As you can see in the example above, we create the *Card* view, which is a container view for any SwiftUI view. It wraps the content you make using the *@ViewBuilder* closure with a rounded background and adds the shadow. 

```swift
struct ContentView: View {
    var body: some View {
        Card {
            Text("Hello, World!")
            Text("My name is Majid Jabrayilov")
        }
    }
}
```

Our card type is simple to use. You create a card and provide content using a closure. By embedding different views inside the *Card* container view, you can reuse it across many screens of your app.

This is the main benefit of using container views, as you can reuse them in different places across the app by encapsulating a shared functionality in the container view.

> To learn more about *@ViewBuilder* closures, take a look at my ["The power of @ViewBuilder in SwiftUI"](/2019/12/18/the-power-of-viewbuilder-in-swiftui/) post.

*@ViewBuilder* closures allow us to compose multiple views easily and embed one view into another. But what about extracting child views from the *@ViewBuilder* closure? SwiftUI introduced new APIs, allowing us to recompose views. For example, we can extract child from the content view built with the *@ViewBuilder* closure and place them as we need.

```swift
struct Carousel<Content: View>: View {
    @ViewBuilder var content: Content
    
    var body: some View {
        ScrollView(.horizontal) {
            LazyHStack {
                ForEach(subviews: content) { subview in
                    subview
                        .containerRelativeFrame(.horizontal)
                }
            }
            .scrollTargetLayout()
        }
        .scrollTargetBehavior(.viewAligned)
        .contentMargins(16)
    }
}
```

As you can see in the example above, we use the *ForEach* view with the *subviews* parameter, which allows us to extract the content view's subviews and iterate over them. 

```swift
struct ContentView: View {
    var body: some View {
        Carousel {
            Color.yellow
            Color.orange
            Color.red
            Color.blue
            Color.green
        }
    }
}
```

SwiftUI uses a particular *Subview* type to expose the instance of an extracted view. It conforms to the *View* protocol, so we can still attach additional SwiftUI view modifiers. It also provides us with the *id* property, which is a unique identifier, and container values associated with the particular view. We will talk more about container values in the upcoming post.

![carousel](/public/container1.png)

Another new API allows us to access child views by index instead of iterating them using the *ForEach* view.

```swift
struct Magazine<Content: View>: View {
    @ViewBuilder var content: Content
    
    var body: some View {
        ScrollView {
            Group(subviews: content) { subviews in
                if !subviews.isEmpty {
                    subviews[0]
                        .padding(.horizontal)
                        .containerRelativeFrame(.vertical) { length, _ in
                            return length / 3
                        }
                }
                
                if subviews.count > 1 {
                    ScrollView(.horizontal) {
                        LazyHStack {
                            ForEach(subviews[1...], id: \.id) { subview in
                                subview
                                    .containerRelativeFrame([.horizontal, .vertical])
                            }
                        }
                        .scrollTargetLayout()
                    }
                    .scrollTargetBehavior(.viewAligned)
                    .contentMargins(16)
                }
            }
        }
    }
}
```

In the example above, we use the *Group* view with the *subviews* parameter, which allows us to extract the child views into a collection type called *SubviewsCollection*. The *SubviewsCollection* type conforms to the *RandomAccessCollection* protocol and provides us access by index.

As you can see, we use the *Group* view to decompose the content view and then compose the subviews in another way. We also leverage the power of the *id* parameter, which allows us to use the *ForEach* view with plain data.

```swift
struct ContentView: View {
    var body: some View {
        Magazine {
            Color.yellow
            Color.orange
            Color.red
            Color.blue
            Color.green
        }
    }
}
```

The new container APIs that SwiftUI introduced this year were missing part about creating customizable and reusable views. I really love how easily it allows us to decompose views and compose them in other ways by hiding the implementation details of our container views.

![magazine](/public/container2.png)

Today, we learned how to create custom container views in SwiftUI using new *Group* and *ForEach* APIs. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week!
