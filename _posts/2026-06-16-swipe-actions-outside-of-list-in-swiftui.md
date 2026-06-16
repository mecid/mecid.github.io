---
title: Swipe actions outside of List in SwiftUI 
layout: post
category: Mastering SwiftUI views
image: /public/swipe-action.png
---

Swipe actions were a primary reason for using *List* in SwiftUI. As you may recall, I’ve mentioned several times that a scroll view paired with lazy stacks is the preferred approach in most scenarios, except when swipe actions are required. 

{% include friends.html %}

The *swipeActionsContainer* view modifier allows us to use swipe actions without *List*. This week, we will learn how to use *swipeActionsContainer* view modifier to attach swipe actions inside scroll view, lazy stacks, and even custom layouts.

```swift
struct ContentView: View {
    @State private var messages: [Message] = [
        Message(content: "Hello"),
        Message(content: "World")
    ]

    var body: some View {
        NavigationView {
            List {
                ForEach(messages, id: \.id) { message in
                    Text(message.content)
                        .swipeActions(edge: .trailing, allowsFullSwipe: true) {
                            Button("Delete", role: .destructive) {
                                messages.removeAll { $0.id == message.id }
                            }
                        }
                }
            }
            .navigationTitle("List")
        }
    }
}
```

As you can see in the example above, you can attach the *swipeActions* view modifier to the items of a *List* view to configure swipe actions. The *swipeActions* view modifier has been working only inside a *List* container. Fortunately, things have changed and now we can use the swipe actions with any container like a scroll view, lazy stacks, or even our custom layouts.

```swift
struct ContentView: View {
    var body: some View {
        ScrollView {
            LazyVStack {
                Text("Hello, World!")
                    .swipeActions {
                        Button(role: .destructive) {
                            // delete action
                        }
                    }
            }
        }
        .swipeActionsContainer()
    }
}
```

Here is another example where we attach the *swipeActions* view modifier to the items of the scroll view. All you need to make it work outside of *List* is enabling *swipeActions* by modifying the view using the *swipeActionsContainer* view modifier. 

The *swipeActionsContainer* view modifier is crucial and works like a swipe actions enabler. It also controls a few things. For example, it enables only one row’s swipe actions to be revealed at a time. It tracks the scrolling events and dismisses any open actions. It also dismisses actions while tapping outside the active row.

List doesn’t need *swipeActionsContainer* view modifier because it does that job automatically, but anything else than List needs the *swipeActionsContainer* view modifier attached.

```swift
public struct FlowLayout: Layout {
    public func sizeThatFits(
        proposal: ProposedViewSize,
        subviews: Subviews,
        cache: inout ()
    ) -> CGSize {
        // ...
    }
    
    public func placeSubviews(
        in bounds: CGRect,
        proposal: ProposedViewSize,
        subviews: Subviews,
        cache: inout ()
    ) {
        // ...
    }
}

struct ContentView: View {
    var body: some View {
        FlowLayout {
            Text("Hello, World!")
                .swipeActions {
                    Button(role: .destructive) {
                        
                    }
                }
        }
        .swipeActionsContainer()
    }
}
```

Here is another example where we attach the *swipeActions* to the custom layout. I’m sure there is no reason to use flow layout with swipe actions, but you should know that you can use it with any custom layout as soon as you modify it with the *swipeActionsContainer* view modifier.

This is a small but very important improvement because it removes one more reason to reach for *List* when we don’t actually need it. We can keep full control over layout, styling, and performance while still providing native swipe interactions where they make sense. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week!
