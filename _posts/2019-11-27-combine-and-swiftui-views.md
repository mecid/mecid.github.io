---
title: Combine and SwiftUI views
layout: post
image: /public/combine-and-swiftui-views.png
category: Data Flow
---

*Combine* is one of the new frameworks released during WWDC 2019. It provides a declarative *Swift API* for processing values over time. Today we will talk about one of the hidden features of SwiftUI views, which is *onReceive* modifier. It allows views to subscribe and react as soon as the publisher emits the value.

{% include friends.html %}

#### Combine
We didn't talk much about Combine on my blog, but I mainly use it for handling asynchronous work. Usually, we have a data layer that is responsible for all operations in the app, like fetching or saving, and this is the place where all asynchronous operations take place. To learn more about the modeling app state, please take a look at ["Redux-like state container in SwiftUI" post](/2019/09/18/redux-like-state-container-in-swiftui/).

But sometimes it is very handy to receive some system-wide notifications in the view layer. An excellent example of this type of notifications can be *keyboardWillShowNotification*. Framework emits this notification as soon as the system keyboard appears. By listening to this notification, we can understand when to add some bottom padding to the root view to keep it visible above the keyboard.

Let's take a look at how we can subscribe to the *Notification Center* using the Combine framework and react to the changes.

```swift
NotificationCenter.default
    .publisher(for: UIResponder.keyboardWillShowNotification)
    .compactMap { $0.userInfo?[UIResponder.keyboardFrameEndUserInfoKey] as? CGRect }
    .map { $0.height }
    .sink { print("height: \($0)") }
```

As you can see in the example above, the Combine framework provides an extension for the *Notification Center*, which allows us to receive and handle events in a reactive style. Now let's see how we can use it inside a SwiftUI view.

#### onReceive modifier
SwiftUI views provide the **onReceive** modifier, which has two arguments: the *Publisher* from Combine framework and the closure. SwiftUI subscribes to the publisher and runs passed closure whenever the publisher emits the value. Let's take a look at the sample code now.

```swift
import SwiftUI
import Combine

struct ItemsView: View {
    let items: [String]

    @State private var keyboardHeight: CGFloat = 0
    private var keyboardHeightPublisher: AnyPublisher<CGFloat, Never> {
        Publishers.Merge(
            NotificationCenter.default
                .publisher(for: UIResponder.keyboardWillShowNotification)
                .compactMap { $0.userInfo?[UIResponder.keyboardFrameEndUserInfoKey] as? CGRect }
                .map { $0.height },
            NotificationCenter.default
                .publisher(for: UIResponder.keyboardWillHideNotification)
                .map { _ in CGFloat(0) }
        ).eraseToAnyPublisher()
    }

    var body: some View {
        List(items, id: \.self) { item in
            Text(item)
        }
        .padding(.bottom, keyboardHeight)
        .onReceive(keyboardHeightPublisher) { self.keyboardHeight = $0 }
    }
}
```

In the example above, we ask SwiftUI to update the state with the value emitted by the publisher, and as soon as state changes, SwiftUI updates our view with a new bottom padding value, which keeps our view above the keyboard. We can extract this piece of code into a *ViewModifier* to make it more reusable. 

```swift
import SwiftUI
import Combine

struct KeyboardAwareModifier: ViewModifier {
    @State private var keyboardHeight: CGFloat = 0

    private var keyboardHeightPublisher: AnyPublisher<CGFloat, Never> {
        Publishers.Merge(
            NotificationCenter.default
                .publisher(for: UIResponder.keyboardWillShowNotification)
                .compactMap { $0.userInfo?[UIResponder.keyboardFrameEndUserInfoKey] as? CGRect }
                .map { $0.height },
            NotificationCenter.default
                .publisher(for: UIResponder.keyboardWillHideNotification)
                .map { _ in CGFloat(0) }
        ).eraseToAnyPublisher()
    }

    func body(content: Content) -> some View {
        content
            .padding(.bottom, keyboardHeight)
            .onReceive(keyboardHeightPublisher) { self.keyboardHeight = $0 }
    }
}

extension View {
    func keyboardAwarePadding() -> some View {
        ModifiedContent(content: self, modifier: KeyboardAwareModifier())
    }
}
```

To learn more about [ViewModifiers in SwiftUI, take a look at my dedicated post](/2019/08/07/viewmodifiers-in-swiftui/).

Another good example of handling system-wide notifications can be *userDidTakeScreenshotNotification*. Framework emits this notification as soon the user takes a screenshot of your app. Assume that we are working on the shopping app where we have a product details screen, and we can present a share sheet as soon as the user takes the screenshot of the product. Let's take a look at how we can achieve this behavior in SwiftUI.

```swift
import SwiftUI

struct ProductContainerView: View {
    let product: Product
    
    @State private var shareShown = false
    private var screenshotPublisher: AnyPublisher<Notification, Never> {
        NotificationCenter.default
            .publisher(for: UIApplication.userDidTakeScreenshotNotification)
            .eraseToAnyPublisher()
    }

    var body: some View {
        ProductDetailsView(product: product)
            .onReceive(screenshotPublisher) { _ in self.shareShown = true }
            .sheet(isPresented: $shareShown) { ShareView(url: self.product.url) }
    }
}
```

Here we have another excellent example of using the Combine framework in SwiftUI views to present a share sheet as soon as the user takes a screenshot.

#### Conclusion
Combine framework is a great way to model your data processing, but today we learned about another interesting usage of publishers in combination with SwiftUI views. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week! 