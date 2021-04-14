---
title: Template-view pattern in SwiftUI
layout: post
image: /public/template2.png
category: Layout
---

Today I want to share with you a technique that I use a lot in SwiftUI. It helps me to solve the problem when I need to place a vertical or horizontal stack with equal-sized views that support Dynamic Type. I didn't find the right name for this approach and call it template-view.

{% include friends.html %}

Assume that you're working on a view that should represent an average heart rate for every day of the week. I would implement that view using a horizontal stack with text components.

```swift
struct WeekView: View {
    let heartRates: [Int]

    var body: some View {
        HStack {
            ForEach(self.heartRates, id: \.self) { hr in
                Text(String(hr))
                    .padding(4)
                    .background(Color.purple)
                    .cornerRadius(4)
            }
        }
    }
}
```

![template-view](/public/template1.png)

Pretty easy, right? But this code has one issue. Every view inside the horizontal stack has a different size based on its text content. The fundamental layout system rule says that every view in SwiftUI calculates its size and sends it back to the parent.

> To learn more about the layout system in SwiftUI, take a look at my ["Layout priorities in SwiftUI"](/2020/04/15/layout-priorities-in-swiftui/) post.

I want to make all views in the horizontal stack the same size. We can easily solve the issue by adding the *frame* modifier to fix the size for every view.

```swift
struct WeekView: View {
    let heartRates: [Int]

    var body: some View {
        HStack {
            ForEach(self.heartRates, id: \.self) { hr in
                Text(String(hr))
                    .frame(width: 40, height: 40, alignment: .center)
                    .background(Color.purple)
                    .cornerRadius(4)
            }
        }
    }
}
```

![template-view](/public/template2.png)

Looks nice, but what about Dynamic Type? My users can change the font size in system settings, and I want respect that font size configuration. It is impossible with *frame* modifier because it just sets the size of the view and doesn't support Dynamic Type. Let's see what happens when I increase the font size in system settings.

![template-view](/public/template3.png)

> To learn more about the benefits of Dynamic Type, take a look at my ["Dynamic Type in SwiftUI"](/2019/10/09/dynamic-type-in-swiftui/) post.

As you can see, the *frame* modifier creates additional problems for us. It is a perfect time to introduce a template-view here.

```swift
struct WeekView: View {
    let heartRates: [Int]

    var body: some View {
        HStack {
            ForEach(self.heartRates, id: \.self) { hr in
                Text("190")
                    .hidden()
                    .padding(4)
                    .background(Color.purple)
                    .cornerRadius(4)
                    .overlay(Text(String(hr)))
            }
        }
    }
}
```

It might look strange, but let me describe what happens here. First of all, I create a hidden text component with some stub value that I think should have the largest width. SwiftUI doesn't show a hidden view but still calculates its size, and this hidden view supports Dynamic Type.

We use an overlay to display content on top of the background that we have configured using the template-view. The benefit of using overlay is the ability to place the view in the center of template-view and limit its size with the size of template-view.

![template-view](/public/template4.png)

Remember that we have to use the same font size both for template-view and the overlay to support Dynamic Type and to keep layout correct. As soon as the user changes the font size in system settings, template-view reacts by recalculating its size and providing more or less room for its overlay depending on user preferred font size.

#### Dynamic Type for images
Dynamic Type is not something connected only to text. Assume that you have a button with an image and text label.

```swift
Button(action: { print("Hello!")}) {
    HStack {
        Image("icon")
            .resizable()
            .frame(width: 24, height: 24, alignment: .center)
        Text("Press me")
    }
}
```

What happens when the user increases the font size? SwiftUI increases the size of the text component, but image size stays the same. It might look really weird, but how can I change the size of the image according to the text?

```swift
Button(action: { print("Hello!")}) {
    HStack {
        Text("00")
            .font(.title)
            .hidden()
            .overlay(
                Image("icon")
                    .resizable()
                    .scaledToFit()
            )
        Text("Press me")
    }
}
```

As you can see, we use the very same approach by creating a template-view. We display a resizable image in the overlay of the hidden text. SwiftUI resizes the image in the overlay as soon as the template-view changes according to system font settings.

#### Conclusion
Dynamic Type is essential, and I believe that every app should support and respect user-defined font size. Template-view is a great way to limit the size of your view but also appreciate the Dynamic Type. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week!
