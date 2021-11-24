---
title: Mastering ProgressView in SwiftUI
layout: post
---

Many of our apps do heavy work on background threads like networking or data processing. We usually want to display progress or the activity indicator of the ongoing work. This week we will learn how to use *ProgressView* to present both infinite and finite progress in SwiftUI.

#### Indefinite progress
All you need to do to display indefinite progress is to place *ProgressView* anywhere in your layout. Let's try to do that in a simple example.

```swift
struct ContentView: View {
    var body: some View {
        ProgressView()
    }
}
```

As you can see in the example above, plain *ProgressView* displays an indefinite circular indicator by default. Usually, it means that there is ongoing work in the background, and the user should wait to see some results.

SwiftUI uses a circular activity indicator by default to display indefinite progress, but there is a way to show definite progress using a linear progress indicator.

#### Finite progress
*ProgressView* provides us with a special initializer that allows us to display finite progress. We might use it in the case where we expect the final result by some time. For example, we usually know the final size of the file while downloading it. This is the case where we have to show finite progress. Let's see how we can do that.

```swift
struct ContentView: View {
    var body: some View {
        ProgressView(value: 250, total: 1000)
    }
}
```

As you can see in the example above, we use another initializer of the *ProgressView* to display the current progress and the total value. SwiftUI automatically calculates the percentage of the work done and shows corresponding progress using the linear indicator.

#### Styling
We can easily customize the *ProgressView* using another initializer that accepts *@ViewBuilder* representing the label of the view.

```swift
struct ContentView: View {
    var body: some View {
        ProgressView {
            Text("Loading")
                .font(.title)
        }
    }
}
```

There is also an initializer with *@ViewBuilder* that works with finite progress and allows you to customize the label.


As usual, SwiftUI provides a style protocol that allows us completely redesign the default look and feel. You need to create a type that conforms to the *ProgressViewStyle* and implement the makeBody function.

```swift
struct HorizontalProgressViewStyle: ProgressViewStyle {
    func makeBody(configuration: Configuration) -> some View {
        HStack(spacing: 8) {
            ProgressView()
                .progressViewStyle(.circular)
            configuration.label
        }.foregroundColor(.secondary)
    }
}

extension ProgressViewStyle where Self == HorizontalProgressViewStyle {
    static var horizontal: HorizontalProgressViewStyle { .init() }
}

struct ContentView: View {
    var body: some View {
        ProgressView("Loading")
            .progressViewStyle(.horizontal)
    }
}
```

In the example above, we create a horizontal stack style to display the circular animating indicator and the label. *Configuration* parameter provides us the completed fraction of the progress, which allows us to draw a super-custom progress view. 

```swift
struct CustomProgressView: View {
    let value: Double

    var body: some View {
        // Draw progress here
    }
}

struct CustomProgressViewStyle: ProgressViewStyle {
    func makeBody(configuration: Configuration) -> some View {
        VStack {
            if let value = configuration.fractionCompleted {
                CustomProgressView(value: value)
            }
            Text("Loading...")
        }
    }
}
```

Here we have a custom view that renders the progress. It might be anything you want, from straightforward text to a custom animated path. Keep in mind that *fractionCompleted* might be nil when you use the indefinite progress view.

#### Conclusion
The *ProgressView* might seem to be very simple, but it provides an excellent level of customization and styling options. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week!
