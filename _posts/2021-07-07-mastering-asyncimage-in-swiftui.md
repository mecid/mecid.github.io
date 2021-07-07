---
title: Mastering AsyncImage in SwiftUI
layout: post
---

During our careers, we primarily build apps that work with web services to retrieve and upload data. Remote image is one type of that data that we need to download and display. SwiftUI provides us the AsyncImage type, which is a view that downloads and shows an image via URL. This week we learn how to use and customize AsyncImage in SwiftUI.

#### Basics
Let's start with a quick example that shows how to use AsyncImage.

=====================================================

As you can see in the example above, AsyncImage is a SwiftUI view that takes a URL for an image, downloads it, and displays it. There is no need for additional configuration of the network layer. AsyncImage uses the shared URLSession to download and cache images. Unfortunately, we can't provide a custom URLCache or somehow customize a URLRequest that downloads the image.

AsyncImage is another SwiftUI view which means you can attach any set of view modifiers to have the desired look and feel. For example, we can clip the shape of our image and set the size.

=====================================================

#### Customisation
Usually, we need to customize the downloaded image size, scaling options, rendering mode, etc. We can access the instance of underlining Image using another overload of AsyncImage initializer.

=====================================================

Here we use another AsyncImage initializer to access the underlying image to make it resizable and apply the filling content mode. This initializer also allows us to provide a placeholder view that SwiftUI displays while downloading the image.

Keep in mind that this initializer uses ViewBuilder closure which means you can take advantage of any SwiftUI you need and build a super custom presentation. 

=====================================================

In the example above, we use the overlay view modifier to cover our image with the ultra-thin material that creates a light blur effect.

AsyncImage allows us to take complete control of all the steps of image presentation using another initializer.

=====================================================

As you can see, we use another initializer that accepts a ViewBuilder closure. This closure has only one parameter that is an instance of AsyncImagePhase enum. AsyncImagePhase enum defines a bunch of image loading states like empty, success, and failed. You can handle all these cases to provide super-custom image presentations.	

Another parameter of the currently used initializer is the SwiftUI transaction. By default, AsyncImage creates a new transaction with the default configuration. In our example, I create a custom transaction with a particular animation that AsyncImage will use whenever phase changes.

#### Conclusion
Today we learned how to download and display remote images using AsyncImage view. I'm happy to see that SwiftUI provides us this feature because it is fundamental for most of the apps we build.