---
title: Accessibility actions in SwiftUI
layout: post
category: Accessibility
image: /public/accessibility.jpeg
---

SwiftUI provides us accessible views out of the box, and usually, you don't need to do anything to build an accessible app for your users. But there is always room for improvements and additional functionality that you can create using Accessibility API. This week we will learn how to provide custom accessibility actions for SwiftUI views.

{% include friends.html %}

#### Adjustable action
SwiftUI provides us an adjustable trait that VoiceOver uses to indicate the ability to adjust the view using swipes up or down. Assume that you are working on the *RatingView*. *RatingView* has to present the current 5-star rating and should provide the opportunity to change the rating. Let's see how we can implement this view.

```swift
struct RatingView: View {
    @Binding var rating: Int

    var body: some View {
        HStack {
            ForEach(1..<6) { index in
                Button(action: { rating = index }) {
                    Image(systemName: index <= rating ? "star.fill" : "star")
                }
            }
        }
    }
}
```

As you can see in the example above, the implementation of the *RatingView* is pretty straightforward. The *RatingView* changes the value of rating using binding whenever you press the particular button. But what about accessibility? How does VoiceOver work with the *RatingView*?

> To learn about the basics of accessibility in SwiftUI, take a look at my ["Accessibility in SwiftUI"](/2019/09/10/accessibility-in-swiftui/) post.

Buttons are accessible out of the box, and VoiceOver will focus on the first button and pronounce the message: "star fill". This is the default behavior, and it doesn't make sense in this case. Fortunately, SwiftUI provides us a few modifiers to customize the user experience here.

```swift
struct RatingView: View {
    @Binding var rating: Int

    var body: some View {
        HStack {
            ForEach(1..<6) { index in
                Button(action: { rating = index }) {
                    Image(systemName: index <= rating ? "star.fill" : "star")
                }
            }
        }
        .accessibilityElement()
        .accessibilityLabel(Text("rating"))
        .accessibilityValue(Text(String(rating)))
        .accessibilityAdjustableAction { direction in
            switch direction {
            case .increment:
                guard rating < 5 else { break }
                rating += 1
            case .decrement:
                guard rating > 1 else { break }
                rating -= 1
            @unknown default:
                break
            }
        }
    }
}
```

By default, the *HStack* works as a transparent accessibility container for its children and exposes children's information. We prevent the default behavior using *accessibilityElement* modifier that enables accessibility for *HStack* and ignores the children. We also provide the accessibility label and value.

We use the *accessibilityAdjustableAction* modifier, which automatically adds the adjustable trait. We also have to provide a closure that handles the action. The only parameter of the closure is the instance of the *AccessibilityAdjustmentDirection* enum.

Whenever the user navigates to the *RatingView*, VoiceOver focuses on the *HStack* itself and pronounces the message: "rating, 3. Adjustable". Users can use swipe up and down gestures to change the value of the rating.

> To learn about the adjustable views in UIKit, take a look at my ["Make your app accessible for everyone"](/2018/07/09/make-your-app-accessible-for-everyone/) post.

#### Additional actions
VoiceOver supports additional actions which we can handle in our views. For example, there is a two-finger double-tap action called magic tap. Apple suggests us to use the magic tap for the main action in our view.

There is two-finger scrub (move two fingers back and forth three times quickly, making a "z") action that users do to go back in the navigation or dismiss the alert. You can handle this action in your view if it utilizes the custom navigation behavior.

```swift
struct PlayerView: View {
    @ObservedObject var viewModel: ViewModel
    @Environment(\.presentationMode) var presentation

    var body: some View {
        HStack {
            // Player content
        }
        .accessibilityAction(.magicTap) {
            if viewModel.isPlaying {
                viewModel.pause()
            } else {
                viewModel.play()
            }
        }
        .accessibilityAction(.escape) {
            viewModel.pause()
            presentation.wrappedValue.dismiss()
        }
    }
}
```

You can also provide named actions in addition to the predefined actions.

```swift
struct PlayerView: View {
    @ObservedObject var viewModel: ViewModel

    var body: some View {
        HStack {
            // Player content
        }
        .accessibilityAction(named: Text("skip")) {
            viewModel.skip()
        }
        .accessibilityAction(named: Text("repeat")) {
            viewModel.repeat()
        }
    }
}
```

#### Conclusion
This week we learned how to make our apps more accessible by adding VoiceOver-friendly actions. Remember that accessibility isn't a feature or a "nice to have". It's a necessity. So let's make your app accessible for everyone. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week!