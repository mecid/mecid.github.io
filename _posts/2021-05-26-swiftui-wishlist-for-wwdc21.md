---
title: SwiftUI wishlist for WWDC21
layout: post
category: Meta
---
WWDC21 is coming pretty soon, and it is a great chance to think about the new features that I want to see in SwiftUI. This wishlist contains not only the list of the features I want to use but also possible APIs. Remember that this post is the result of my imagination, and most of the code examples don't exist at the moment. 

#### List and ScrollView
SwiftUI provides you both List and ScrollView, but under the hood, these views still use the UIKit implementation of UITableView and UIScrollView. I love how UITableView works and the API it provides us. But SwiftUI's List and ScrollView don't expose all the powerful features of UITableView and UIScrollView.

Almost all the screens in my apps use the List view in different styles, and I hope to see more APIs from UITableView, which allows styling separators, cell backgrounds using the ListStyle protocol.

=====================================================

ScrollView is another crucial component for many screens. I usually build my apps with accessibility in mind and support Dynamic Type out of the box. Scroll View is must have root view for every screen where you want Dynamic Type. ScrollView in SwiftUI is still missing paging and content offset features that we used to see in UIScrollView.

=====================================================

> We can read the content offset of ScrollView using preferences API in SwiftUI. To learn more, take a look at my dedicated "Mastering ScrollView in SwiftUI" post.

#### CompositionalLayout
During the last year, Apple gave us LazyHGrid and LazyVGrid views, which we can use to build views like calendars or photo grids. Grids work great, and I love them, but we want to use the power of CompositionalLayout that we have in UIKit. I don't think that Apple should get rid of LazyHGrid and LazyVGrid views, but they can introduce a new CompositionaView that supports all the features of CompositionalLayout.

=====================================================

#### Navigation
Navigation is another point of pain in SwiftUI. It works great for simple use cases but doesn't provide enough flexibility for complex solutions. What I want to use is some sort of RouterView that maps destinations to concrete views.

=====================================================

We also can have a router in the environment and mutate the state of navigation using the environment.

=====================================================

#### Focus management and text fields
Sign-up form with multiple text fields is what I usually implement using UIKit and then wrap with UIViewControllerRepresentable. It is literally impossible to handle the first responder in SwiftUI and move the focus from one view to another.

=====================================================

You might wonder, but this is the real API that we have in SwiftUI at the moment. Unfortunately, it is available only for tvOS and watchOS, but I wish to see it for iOS and macOS.

#### Conclusion
SwiftUI has its own set of pros and cons. But for me, it is the way to go with my projects. I'm so excited about the upcoming WWDC and hope to see at least a part of the features I mentioned in this post.