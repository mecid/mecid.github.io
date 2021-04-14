---
title: Building Pager view in SwiftUI
layout: post
image: /public/pager.png
category: Building custom views
---

This week I want to continue the series of posts about building custom interactive views in SwiftUI. Today we will create a pager view. *ScrollView* in SwiftUI support only scrolling content and doesn't have paging behavior. That's why we will build a pager view that supports paging mode.

{% include friends.html %}

Let's start by describing the *PagerView* and all the needed properties. First of all, we need the page count, binding to the currently visible page, and the content of pages.

```swift
import SwiftUI

struct PagerView<Content: View>: View {
    let pageCount: Int
    @Binding var currentIndex: Int
    let content: Content

    init(pageCount: Int, currentIndex: Binding<Int>, @ViewBuilder content: () -> Content) {
        self.pageCount = pageCount
        self._currentIndex = currentIndex
        self.content = content()
    }
}
```

Again, we use binding to extract the state of the view. It allows us to store the state in the parent view and react to page changes. We also use *@ViewBuilder* for the content closure. *@ViewBuilder* enables encapsulation of the presentation logic by keeping content descriptions outside the view. It is a pretty popular technique for any container view in SwiftUI.

> To learn more about the benefits of @ViewBuilder while building custom SwiftUI view take a look at [my dedicated post](/2019/12/18/the-power-of-viewbuilder-in-swiftui/).

Now we can build our presentation logic and render the current page. All we need to do is to place our content inside an *HStack* and set proper offset. Let's see how easily we can achieve that.

```swift
var body: some View {
    GeometryReader { geometry in
        HStack(spacing: 0) {
            self.content.frame(width: geometry.size.width)
        }
        .frame(width: geometry.size.width, alignment: .leading)
        .offset(x: -CGFloat(self.currentIndex) * geometry.size.width)
    }
}
```

In the example above, we use *GeometryReader* to get information about the parent view's size and fill the entire place. We also need to set alignment to leading because, by default, frame place the child view in the center. We also use *offset* modifier to place an active page in the visible area.

> To learn more about GeometryReader, take a look at ["Building BarChart with Shape API in SwiftUI" post](/2019/08/14/building-barchart-with-shape-api-in-swiftui/).

Let's move to the final step and attach the drag gesture to *HStack* to swipe pages interactively. We need to declare a new view-local state for the gesture to store translation while the gesture is active. We can use translation value to add offset to the *HStack* to achieve interactive page dragging.

```swift
@GestureState private var translation: CGFloat = 0

var body: some View {
    GeometryReader { geometry in
        HStack(spacing: 0) {
            self.content.frame(width: geometry.size.width)
        }
        .frame(width: geometry.size.width, alignment: .leading)
        .offset(x: -CGFloat(self.currentIndex) * geometry.size.width)
        .offset(x: self.translation)
        .animation(.interactiveSpring(), value: currentIndex)
        .animation(.interactiveSpring(), value: translation)
        .gesture(
            DragGesture().updating(self.$translation) { value, state, _ in
                state = value.translation.width
            }.onEnded { value in
                let offset = value.translation.width / geometry.size.width
                let newIndex = (CGFloat(self.currentIndex) - offset).rounded()
                self.currentIndex = min(max(Int(newIndex), 0), self.pageCount - 1)
            }
        )
    }
}
```

Whenever drag gesture ends, we need to calculate how big was the translation and if it is enough to move to the next or previous pages. We can estimate it straightforwardly by using geometry and some math functions like min, max, and rounding. Let's finally see how we can use our *PagerView*.

```swift
struct ContentView: View {
    @State private var currentPage = 0

    var body: some View {
        PagerView(pageCount: 3, currentIndex: $currentPage) {
            Color.blue
            Color.red
            Color.green
        }
    }
}
```

> Full source code available on [GitHub](https://gist.github.com/mecid/e0d4d6652ccc8b5737449a01ee8cbc6f).

I enjoy the snapping effect which we have in our view, and it is all possible by using interactive spring. Interactive spring is an animation with a lower response value, intended for driving interactive animations.

> To learn more about animations and transitions in SwiftUI, take a look at my ["Animations in SwiftUI" post](/2019/06/26/animations-in-swiftui/).

#### Conclusion
This week we built another interactive view component in SwiftUI. I feel like it is so easy to build interactive views, all you need to do is declare your state, statically render the state and then attach a gesture which modifies that state. SwiftUI handles all the transitions between states automatically by adding very fluid animations. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week!