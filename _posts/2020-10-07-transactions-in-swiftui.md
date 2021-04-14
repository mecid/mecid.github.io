---
title: Transactions in SwiftUI
layout: post
image: /public/swiftui.png
category: Interactions
---

Animations play a vital role in SwiftUI. We saw a lot of examples of complex animations that we can easily implement in SwiftUI. The guidance for building fluid animations in SwiftUI has the only one step: mutate your state, and SwiftUI will automatically animate changes in your views. Today we will talk about transactions, which is a hidden gem of SwiftUI.

{% include friends.html %}

#### Basics
Transaction is the context of the current state-processing update. SwiftUI creates a transaction for every state change. Transaction contains the animation that SwiftUI will apply during the state change and the property indicating whenever this transaction disables all the animations defined by the child views. Let's take a look at the quick example.

```swift
import SwiftUI

struct AnimatedView: View {
    let scale: CGFloat

    var body: some View {
        Circle()
            .fill(Color.accentColor)
            .scaleEffect(scale)
            .animation(.spring())
    }
}
```

As we know, animation modifier applies animation to all the child views of the applied view. Apple suggests us to use this modifier on leaf views rather than container views. This approach allows us to specify animation only for the views that we need.

> To learn more about the animation modifier in Swift, look at my ["Animations in SwiftUI"](/2019/06/26/animations-in-swiftui/) post.

```swift
struct ContentView: View {
    @State private var scale = false

    var body: some View {
        AnimatedView(scale: scale ? 0.5 : 1)
            .onTapGesture {
                scale.toggle()
            }
    }
}
```

Using an animation modifier on a child view has one downside. We can't control that animation. For example, we are not able to replace the spring animation with a linear one. This is where we can use transactions to override animations defined in child views.

```swift
struct ContentView: View {
    @State private var scale = false

    var body: some View {
        AnimatedView(scale: scale ? 0.5 : 1)
            .onTapGesture {
                var transaction = Transaction(animation: .linear)
                transaction.disablesAnimations = true

                withTransaction(transaction) {
                    scale.toggle()
                }
            }
    }
}
```

As you can see in the example above, we use the *withTransaction* function to wrap our mutations with a custom transaction. The new transaction disables all the animations defined inside a view hierarchy and enables a linear animation.

#### Transaction modifier
Now we know how to create a custom transaction for a complete view hierarchy. There is also a way to modify the current transaction for a concrete view using the transaction modifier. Let's see how we can use it.

```swift
struct ContentView: View {
    @State private var scale = false

    var body: some View {
        VStack {
            AnimatedView(scale: scale ? 0.5 : 1)
                .transaction { transaction in
                    transaction.animation = .spring()
                }
            AnimatedView(scale: scale ? 0.5 : 1)
                .transaction { transaction in
                    transaction.disablesAnimations = true
                }
        }.onTapGesture {
            scale.toggle()
        }
    }
}
```

The transaction modifier accepts a closure with the *inout* instance of *Transaction* struct. We can modify the current transaction inside this closure as we need it. In the example above, we completely disable animations for one view and replace animation for another view.

#### Transactions during gesture updates
You can find the usage of transactions across many APIs in SwiftUI. For example, you can modify the transaction during gesture updates. It works similarly to the transaction modifier.

```swift
struct ContentView: View {
    @GestureState private var offset: CGSize = .zero

    var body: some View {
        Circle()
            .frame(width: 100, height: 100)
            .offset(offset)
            .gesture(
                DragGesture()
                    .updating($offset) { value, state, transaction in
                        state = value.translation
                        transaction.animation = .interactiveSpring()
                    }
            )
    }
}
```

> To learn more about building interactive views, look at my ["Gestures in SwiftUI"](/2019/07/10/gestures-in-swiftui/) post.

#### Transactions in bindings
You can also provide a custom transaction during binding updates using the *transaction function* on a binding.

```swift
struct ContentView: View {
    @State private var scale = false

    var body: some View {
        // ...
    }

    private var animatedBinding: Binding<Bool> {
        var transaction = Transaction(animation: .interactiveSpring())
        transaction.disablesAnimations = true
        return $scale.transaction(transaction)
    }
}
```

> To learn more about features provided by bindings, look at my ["Binding in SwiftUI"](/2020/04/08/binding-in-swiftui/) post.

#### Conclusion
Today we learned all about transactions in SwiftUI. Understanding transactions opens new doors for building powerful and reusable view components in SwiftUI. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this article. Thanks for reading, and see you next week!
