---
title: Customizing the shape of views in SwiftUI
layout: post
image: /public/mask.png
category: Mastering SwiftUI views
---

SwiftUI provides us several exciting ways to change the shape of our views. It allows clipping, masking, and a few other operations on the shape of views. This week I want to talk to you about altering view's shape in SwiftUI.

{% include friends.html %}

#### Clipping
Sometimes we use the frame modifier to limit the size of our view. It might be useful when you want to set the size to the image that should be scaled. In case when the scaled image bigger than the provided frame, it can exceed it and overlap other views. To fix this issue, SwiftUI allows us to clip the content of the view to its frame. Let's take a look at a quick example.

```swift
KFImage(recipe.image)
    .resizable()
    .aspectRatio(contentMode: .fill)
    .frame(height: 300)
    .clipped(antialiased: true)
```

As you can see in the example above, we use the *clipped* modifier. This modifier lets us cut the content of the view inside its bounds. It also takes the argument that indicates whenever to apply smoothing to the edges of the frame.

> *KFImage* is a part of [KingFisher](https://github.com/onevcat/Kingfisher) library for loading remote images

#### Clipping the shape
Assume that you are working on the app that presents the avatars. Usually, our designers want to make avatars in a rounded form. Fortunately, SwiftUI allows us to clip the view into any shape we can imagine. SwiftUI provides an exceptional modifier for that called *clipShape*. Let's take a look at how we can use it.

```swift
KFImage(user.avatar)
    .resizable()
    .aspectRatio(contentMode: .fill)
    .frame(width: 100, height: 100)
    .clipShape(Circle())
```

We can easily apply *clipShape* to any view by providing the shape we want. There are plenty of ready to use shapes like *Rectangle, Capsule, Circle, etc*. provided by SwiftUI. But anytime you want something unique, for example, a star form, you can quickly implement it by creating a struct conforming *Shape* protocol. You can apply any shape you want by doing your math in *path* function.

> To learn more about *Shape* protocol, take a look at my ["Building BarChart with Shape API in SwiftUI" post](/2019/08/14/building-barchart-with-shape-api-in-swiftui/).

### Masking
Let's take a look at more complex examples. Assume that you have a text component that should use a gradient as the text color. You can't implement it by using the *foregroundColor* modifier because it accepts only colors. It is the exact case where we might use masking. Let's take a look at how we can apply masking to use a gradient as the text color.

```swift
struct MaskSample: View {
    var body: some View {
        LinearGradient(
            gradient: Gradient(colors: [.blue, .green, .yellow, .orange, .red]),
            startPoint: .leading,
            endPoint: .trailing
        )
            .mask(Text("I love SwiftUI !!!").font(.largeTitle))
            .background(Color.black)
    }
}
```

As you can see in the example above, we have a gradient that masked with the text. SwiftUI allows us to cut our gradient into the shape of our text. Remember that you can mask any view not only gradient. It means you can mask any image with the text to provide a beautiful look and feel to your users.

![mask](/public/mask.png)

#### Conclusion
This week we learned about clipping and masking views in SwiftUI. Both clipping and masking allow us to provide nice effects to our users by adding a few lines of code. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week!