---
title: The power of @ViewBuilder in SwiftUI
layout: post
image: /public/viewbuilder.png
category: Building custom views
---

Last week we started a series of posts about developing interactive components using SwiftUI, where we talked about building the bottom sheet. We need to understand the power of *@ViewBuilder* before moving to the next post about building another interactive view. That's why this week, we will talk about *@ViewBuilder* and its benefits while developing custom views.

{% include friends.html %}

#### Result builders
*@ViewBuilder* is one of the possible result builders. The result builder feature of Swift is described in [Swift Evolution Proposal](https://github.com/apple/swift-evolution/blob/main/proposals/0289-result-builders.md). The main goal of result builder is providing *DSL* like syntax. Let's take a look at a very quick example of *@ViewBuilder* usage.

```swift
import SwiftUI

struct ContentView: View {
    var body: some View {
        HStack{
            Text("hello")
            Text("world")
        }
    }
}
```

We need to go down one layer to understand how *@ViewBuilder* works.

```swift
@inlinable public init(
    alignment: VerticalAlignment = .center,
    spacing: CGFloat? = nil,
    @ViewBuilder content: () -> Content
)
```

Here is the declaration of *HStack* view, as you can see the content closure inside the init method marked with *@ViewBuilder*. It means that expression inside that closure needs to be handled by *@ViewBuilder*. The swift compiler will try to find the static buildBlock method declared in *@ViewBuilder* struct that has two views as parameters. Let's take a look at *@ViewBuilder* struct declaration to find that method.

```swift
@available(iOS 13.0, OSX 10.15, tvOS 13.0, watchOS 6.0, *)
extension ViewBuilder {
    public static func buildBlock<C0, C1>(_ c0: C0, _ c1: C1) -> TupleView<(C0, C1)> where C0 : View, C1 : View
}
```

As you can see, *@ViewBuilder* has a static *buildBlock* method that accepts two views, combine them and return *TupleView*. It also has other declarations of *buildBlock* method, which takes from one to ten child views, and all of them combine child views into a *TupleView*. That's why *@ViewBuilder* can accept only ten views inside the closure.

```swift
struct ContentView: View {
    @ObservedObject var viewModel: ViewModel

    @ViewBuilder
    var body: some View {
        if viewModel.isAuthorized {
            Text("Hello World!")
        } else {
            LoginView(viewModel: viewModel)
        }
    }
}
```

*@ViewBuilder* also has support for conditions via *if* and *switch* expressions.

```swift
struct ContentView: View {
    @ObservedObject var viewModel: ViewModel

    @ViewBuilder
    var body: some View {
        switch viewModel.isAuthorized {
        case true:
            Text("Hello")
        case false:
            LoginView(viewModel: viewModel)
        }
    }
}
```

> To learn how to avoid ten views limitation, take a look at my ["View Composition in SwiftUI"](/2019/10/30/view-composition-in-swiftui/) post.

#### TupleView
```swift
import SwiftUI

struct ContentView: View {
    var body: some View {
        HStack{
            Text("hello")
            Text("world")
        }
    }
}

print(Mirror(reflecting: ContentView().body))
// Mirror for HStack<TupleView<(Text, Text)>>
```

*TupleView* is a view created from a swift tuple of view values. *TupleView* doesn't have any logic inside. It just holds the views. *TupleView* completely transparent and behaves like its parent view. It means when you put it inside the *HStack*, *TupleView* places the views from the tuple in a horizontal direction.

#### Using @ViewBuilder
Now we know all the needed things to build our own custom view container, which uses *@ViewBuilder*. Assume that our app needs a notification view. The notification view should have a consistent design and appear in the top of the screen, but content can be various. It is a perfect use case for *@ViewBuilder*. Let's see how we can utilize it.

```swift
import SwiftUI

struct NotificationView<Content: View>: View {
    let content: Content

    init(@ViewBuilder content: () -> Content) {
        self.content = content()
    }

    var body: some View {
        content
            .padding()
            .background(Color(.tertiarySystemBackground))
            .cornerRadius(16)
            .transition(.move(edge: .top))
            .animation(.spring())
    }
}
```

As you can see, we use *@ViewBuilder* to mark our content closure. It gives us the opportunity to use *NotificationView* in the same way as *VStack* or *HStack*. Here is the example of using *NotificationView*.

```swift
import SwiftUI

struct ContentView: View {
    @State private var notificationShown = false

    var body: some View {
        VStack {
            if self.notificationShown {
                NotificationView {
                    Text("notification")
                }
            }

            Spacer()

            Button("toggle") {
                self.notificationShown.toggle()
            }

            Spacer()
        }
    }
}
```

> We also used the ability to build custom views via *@ViewBuilder* during ["Building Bottom sheet in SwiftUI"](/2019/12/11/building-bottom-sheet-in-swiftui/) post.

#### Conclusion
This week we talked about the benefits of result builders and used *@ViewBuilder* as a concrete example. *@ViewBuilder* allows us to build super reusable SwiftUI views by separating its presentation logic and content. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week! 
