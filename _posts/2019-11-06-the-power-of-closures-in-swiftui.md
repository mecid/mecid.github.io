---
title: The power of Closures in SwiftUI
layout: post
category: View Composition
---

One of my favorite design patterns in *UIKit* development was a [*Delegate pattern*](/2019/05/29/the-power-of-delegate-design-pattern/). *Delegate pattern* is very straightforward, and everybody knows how to use it. In the *Functional Programming* world, we usually replace delegates with closures. This week we will learn how to use closures to make SwiftUI views composable and decoupled.

{% include friends.html %}

#### Passing closures to child views
I usually build my app screen by implementing one container view which handles all the data-flow related to the screen and a couple of rendering views, which simply represent passed data and propagate user actions to the container view. Let's take a look at an example.

```swift
struct PostContainerView: View {
    @EnvironmentObject var store: Store<State, Action>
    @State private var content: String = ""

    var body: some View {
        PostContentView(
            content: $content,
            postContent: postContent
        )
    }

    private func postContent() {
        store.send(.post(content))
    }
}

struct PostContentView: View {
    @Binding var content: String
    let postContent: () -> Void

    var body: some View {
        VStack {
            TextField("type something...", text: $content)
            Button("post", action: postContent)
        }
    }
}
```

Here we have a container view that handles user actions by providing a closure to the child view. *PostContentView* renders the state provided by a container view and passes user action to the container view. This technique allows us to reuse *PostContentView* across the codebase. We can use it whenever we need to post a comment or some post.

We already discussed the benefits of using container view in SwiftUI on my blog multiple times, take a look at ["Introducing Container views in SwiftUI"](/2019/07/31/introducing-container-views-in-swiftui/) post if you missed it.

#### Extracting navigation into closures
SwiftUI has a pretty declarative way of building navigation between the screens. All you need to do is embedding your view into a *NavigationLink* with a provided destination view. Here is a quick example of using *NavigationLink* in SwiftUI apps.

```swift
struct ItemsView: View {
    let items: [Item]

    var body: some View {
        NavigationView {
            List(items) { item in
                NavigationLink(destination: DetailsView(item: item)) {
                    Text(item.id.uuidString)
                }
            }
        }
    }
}
```

As you can see in the example above, we simply wrap our list item into a *NavigationLink*, which navigates to the *DetailsView* after a click on any list item. The logic here is very straightforward, but it has at least one downside. *ItemsView* knows about *DetailsView* and depends on it, and because of that, we can't reuse it somewhere in our app, or we can't use it with a different destination in other parts of the app. Let's see how we can solve the problem by using closures.

```swift
struct ItemsView<Destination: View>: View {
    let items: [Item]
    let buildDestination: (Item) -> Destination

    var body: some View {
        NavigationView {
            List(items) { item in
                NavigationLink(destination: self.buildDestination(item)) {
                    Text(item.id.uuidString)
                }
            }
        }
    }
}
```

I refactored our *ItemsView* to accept a closure which maps an item to a destination view. It allows us to leave the responsibility of creating a destination view to a parent view. Now we can reuse *ItemsView* with different destinations depending on our use case. Here is a code sample demonstrating the usage of the *ItemsView*.

```swift
struct ItemsContainerView: View {
    @State private var items: [Item] = [.init(), .init(), .init()]
    
    var body: some View {
        ItemsView(items: items) { item in
            // build your destination view here
            Text(item.id.uuidString)
        }
    }
}
```

Here we have *ItemsContainerView*, which handles data-flow for the screen and builds the destination view. It feels very natural by using trailing closure syntax.

Navigation is a crucial topic, and we already covered it in previous posts, to learn more about navigation in SwiftUI take a look at [the dedicated post](/2019/07/17/navigation-in-swiftui/).

#### Conclusion
This week we learned how to use *closures* to extract navigation and user input handling from SwiftUI views. *Closures* allow us to make our views decoupled and respecting the single responsibility principle. We can benefit it to build simple and composable view hierarchies in SwiftUI. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week! 