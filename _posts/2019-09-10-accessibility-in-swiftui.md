---
title: Accessibility in SwiftUI
layout: post
category: Accessibility
image: /public/accessibility.jpeg
---

This week I want to talk to you about *Accessibility in SwiftUI*. SwiftUI provides a ready to use accessibility implementation for standard User Interface elements like *Text, Button, Toggle, etc*. In most of the cases, you don't need to do something additional to make it work. But I will show how you can modify the accessibility tree by using accessibility modifiers to improve your User Experience. 

{% include friends.html %}

#### Accessibility modifiers
SwiftUI provides a bunch of accessibility modifiers for any view. By using accessibility modifiers, you can easily set label and value for a view, add accessibility traits and actions.

Assume that we are working on *BarChart* component and we want to make every *Bar* accessible to VoiceOver and VoiceControl users. Let's take a look at how we can do that by adding accessibility modifiers to custom shape view.

```swift
struct BarsView: View {
    let bars: [Bar]

    private var max: Double {
        bars.map { $0.value }.max() ?? 0
    }

    var body: some View {
        GeometryReader { container in
            HStack(alignment: .bottom, spacing: 1) {
                ForEach(self.bars, id: \.self) { bar in
                    Capsule()
                        .fill(bar.legend.color)
                        .accessibility(label: Text(bar.label))
                        .accessibility(value: Text(bar.legend.label))
                        .frame(height: CGFloat(bar.value) / CGFloat(self.max) * container.size.height * 0.8)
                }
            }
        }
    }
}
```

In the example above, we use multiple overloaded versions of accessibility modifier to set label and value on a capsule shape.

#### Accessibility tree
Let's talk about accessibility path of every view. SwiftUI builds accessibility tree for the entire screen and provides it to VoiceOver or any other assistive technology. VoiceOver reads the accessibility tree from top to bottom and allows to user navigate through it via short swipes.

SwiftUI allows us to modify the accessibility tree in different ways. For example, we can change the order used by VoiceOver to navigate through the tree. To do that we have to use accessibility modifier with sort priority parameter.

We also can hide some elements in the accessibility tree by using another overloaded version of accessibility modifier. It accepts a boolean parameter indicating whenever element should be hidden or not.

```swift
VStack {
    BarsView(bars: bars)
    LabelsView(bars: bars, labelsCount: labelsCount)
        .accessibility(hidden: true)
    LegendView(bars: bars)
        .padding()
        .accessibility(sortPriority: 1)
}
```

In the recent example, we use accessibility with sort priority parameter to reorder accessibility tree of the view. By default, every view has a priority equals to zero. **Remember that higher numbers appear first.**

#### AccessibilityChildBehavior
Sometimes we need to combine accessibility elements inside any view and assign it to a parent view as a single accessibility element. It becomes super handy when you want to make your cells easily navigable with VoiceOver. You can combine accessibility elements inside it into a single element attached to the cell itself. We can do that by using *accessibilityElement* modifier. Let's take a look at a quick example.

```swift
struct SleepDetailsRow: View {
    let value: String
    let symbol: String
    let description: String

    var body: some View {
        HStack {
            Text(description)
                .foregroundColor(.secondary)
                .font(.body)

            Spacer()

            Text(value)
                .font(.headline)
                +
                Text(symbol)
                    .font(.subheadline)
        }.accessibilityElement(children: .combine)
    }
}
```

Here we use *accessibilityElement* modifier with children combine option which merges all the accessibility information provided by the children into a single element and assign it to a parent view. *accessibilityElement* modifier also allows us to ignore or contain children accessibility information by using other options provided by *AccessibilityChildBehavior*.

#### Conclusion
Today we learned how to customize accessibility experience in our apps by using simple accessibility modifier in SwiftUI. I hope you will use that information to improve your User Experience. Remember that accessibility isn't a feature or a "nice to have." **It's a necessity**. So let's make your app accessible for everyone. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading and see you next week! 