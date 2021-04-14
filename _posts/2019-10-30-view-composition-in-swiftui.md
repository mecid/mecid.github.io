---
title: View composition in SwiftUI
layout: post
category: View Composition
---

SwiftUI is a declarative framework for building User Interfaces on Apple platforms. The keyword here is **declarative**.  Declarative means that you need to declare what you want to achieve, and the framework takes care of it. Framework knows the best way to render the User Interface, which you declare.

{% include friends.html %}

#### View decomposition
Containers in SwiftUI has a nasty limitation for the count of children. It is limited to **ten** items. This restriction can sound ugly, but I think it is awesome. Let's accept it as a rule. Whenever you reach this limitation, decompose your view into several views. Don't be afraid to extract your complex views into small pieces and then compose them into a large view. 

Sometimes to make that possible, we have to embed our views into additional layout containers like stacks. But don't worry about additional containers, it won't affect your rendering performance. SwiftUI is smart enough to understand the final result and optimize your layout to ignore unnecessary layout containers. Let's take a look at the example of complex view decomposition.

```swift
struct SleepDetailsView : View {
    private var chartSection: some View {
        // return some view here
    }

    private var sleepSection: some View {
        // return some view here
    }

    private var phasesSection: some View {
        // return some view here
    }

    private var heartRateSection: some View {
        // return some view here
    }

    var body: some View {
        List {
            chartSection
            sleepSection
            phasesSection
            heartRateSection
        }
    }
}
```

In the example above, we extract our sections into the computed properties. It makes our codebase more clean, easy to follow, and still very fast because SwiftUI knows how to optimize your layout.

#### Groups
*Group* is another way to get around the restriction of ten child views and achieve view composition in SwiftUI. *Group* doesn't have any layout logic like *VStack/HStack/ZStack*, and It is completely transparent. When you put *Group* inside a *VStack*, it behaves like *VStack* and arranges children in a vertical direction. In the case when *Group* embedded into *HStack*, it renders views horizontally.

It looks like SwiftUI has a *Group* component to get rid of that ten element limitation only. But it is not. *Group* component has unusual behavior. Any view modifier added to the *Group* component will apply to every child view **separately**. Let's take a look at the sample to understand how it works.

```swift
import SwiftUI

struct ContentView: View {
    var body: some View {
        VStack {
            Group {
                Text("Hello")
                Text("World")
                Text("!!!")
            }
            .background(Color.yellow)
            .padding()
        }
    }
}
```

In this example, we add padding and background color to a group of text components. These modifiers won't affect *Group* component itself. Instead, *Group* applies them to every text **individually**. It means every text inside *Group* will have a yellow background and padding. In other words, *Group* maps every child view with modifiers, which was applied to the *Group*.

#### ViewModifiers
*ViewModifiers* is a third form of view composition in SwiftUI. All these things which we use to modify our views like foreground color, padding, font, etc. are *ViewModifiers*. When you find yourself repeating the same code to alter your views, just introduce a *ViewModifier* for that and reuse it across your codebase. Here is a small example of extracting few modifiers into a single custom *ViewModifier*.

```swift
import SwiftUI

struct SubheadlineModifier: ViewModifier {
    func body(content: Content) -> some View {
        content
            .foregroundColor(.secondary)
            .font(.subheadline)
    }
}

Text("subhead")
    .modifier(SubheadlineModifier())
```

All the subheads in my apps use the same styling. That's why I decide to extract it into a custom ViewModifier, which I can reuse across my codebase. To learn more about *ViewModifiers*, please check my ["ViewModifiers in SwiftUI"](/2019/08/07/viewmodifiers-in-swiftui/) post.

#### Conclusion
Today we learned three ways of view composition in SwiftUI. Composition allows us to build a highly reusable codebase that we can easily support and maintain. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week! 