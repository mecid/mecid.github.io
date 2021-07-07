---
title: Mastering AsyncImage in SwiftUI
layout: post
---

During our careers, we primarily build apps that work with web services to retrieve and upload data. Remote image is one type of that data that we need to download and display. SwiftUI provides us the AsyncImage type, which is a view that downloads and shows an image via URL. This week we learn how to use and customize AsyncImage in SwiftUI.

{% include friends.html %}

#### Basics
Let's start with a quick example that shows how to use AsyncImage.

```swift
struct AvatarView: View {
    let url: URL

    var body: some View {
        AsyncImage(url: url)
    }
}
```

As you can see in the example above, AsyncImage is a SwiftUI view that takes a URL for an image, downloads it, and displays it. There is no need for additional configuration of the network layer. AsyncImage uses the shared URLSession to download and cache images. Unfortunately, we can't provide a custom URLCache or somehow customize a URLRequest that downloads the image.

AsyncImage is another SwiftUI view which means you can attach any set of view modifiers to have the desired look and feel. For example, we can clip the shape of our image and set the size.

```swift
struct AvatarView: View {
    let url: URL

    var body: some View {
        AsyncImage(url: url)
            .frame(width: 44, height: 44)
            .background(Color.gray)
            .clipShape(Circle())
```

#### Customisation
Usually, we need to customize the downloaded image size, scaling options, rendering mode, etc. We can access the instance of underlining Image using another overload of AsyncImage initializer.

```swift
    AsyncImage(url: url) { image in
        image
            .resizable()
    } placeholder: {
        ProgressView()
    }
    .frame(width: 44, height: 44)
    .background(Color.gray)
    .clipShape(Circle())
```

Here we use another AsyncImage initializer to access the underlying image to make it resizable and apply the filling content mode. This initializer also allows us to provide a placeholder view that SwiftUI displays while downloading the image.

Keep in mind that this initializer uses ViewBuilder closure which means you can take advantage of any SwiftUI you need and build a super custom presentation. 

```swift
    AsyncImage(url: url) { image in
        image
            .resizable()
            .scaledToFill()
            .overlay(Material.ultraThin)
    } placeholder: {
        ProgressView()
    }
    .frame(width: 44, height: 44)
    .background(Color.gray)
    .clipShape(Circle())
```

In the example above, we use the overlay view modifier to cover our image with the ultra-thin material that creates a light blur effect.

AsyncImage allows us to take complete control of all the steps of image presentation using another initializer.

```swift
struct AvatarView: View {
    let url: URL

    var body: some View {
        AsyncImage(
            url: url,
            transaction: Transaction(animation: .easeInOut)
        ) { phase in
            switch phase {
            case .empty:
                ProgressView()
            case .success(let image):
                image
                    .resizable()
                    .transition(.scale(scale: 0.1, anchor: .center))
            case .failure:
                Image(systemName: "wifi.slash")
            @unknown default:
                EmptyView()
            }
        }
        .frame(width: 44, height: 44)
        .background(Color.gray)
        .clipShape(Circle())
    }
}
```

As you can see, we use another initializer that accepts a ViewBuilder closure. This closure has only one parameter that is an instance of AsyncImagePhase enum. AsyncImagePhase enum defines a bunch of image loading states like empty, success, and failed. You can handle all these cases to provide super-custom image presentations.	

Another parameter of the currently used initializer is the SwiftUI transaction. By default, AsyncImage creates a new transaction with the default configuration. In our example, I create a custom transaction with a particular animation that AsyncImage will use whenever phase changes.

#### Conclusion
Today we learned how to download and display remote images using AsyncImage view. I'm happy to see that SwiftUI provides us this feature because it is fundamental for most of the apps we build. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week!