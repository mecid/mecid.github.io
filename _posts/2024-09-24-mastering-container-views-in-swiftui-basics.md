---
title: Mastering container views in SwiftUI. Basics.
layout: post
---

Since the very first version of the framework, SwiftUI has had several container views. The most popular ones are HStack, VStack, List, etc. This year, Apple introduced new APIs that allow us to build custom container views in a new way. This week, we will learn about the benefits of SwiftUI's new decomposition APIs.

What is a container view? It is a view holding other views. We can easily define a container view using @ViewBuilder closures. Here is an example.

=====================================================

As you can see in the example above, we create the Card view, which is a container view for any SwiftUI view. It wraps the content you make using the @ViewBuilder closure with a rounded background and adds the shadow. 

=====================================================

Our card type is simple to use. You create a card and provide content using a closure. By embedding different views inside the Card container view, you can reuse it across many screens of your app.

This is the main benefit of using container views, as you can reuse them in different places across the app by encapsulating a shared functionality in the container view.

@ViewBuilder closures allow us to compose multiple views easily and embed one view into another. But what about extracting child views from the @ViewBuilder closure? SwiftUI introduced new APIs, allowing us to recompose views. For example, we can extract child from the content view built with the @ViewBuilder closure and place them as we need.

=====================================================

As you can see in the example above, we use the ForEach view with the subviews parameter, which allows us to extract the content view's subviews and iterate over them. 

SwiftUI uses a particular Subview type to expose the instance of an extracted view. It conforms to the View protocol, so we can still attach additional SwiftUI view modifiers. It also provides us with the id property, which is a unique identifier, and container values associated with the particular view. We will talk more about container values in the upcoming post.

Another new API allows us to access child views by index instead of iterating them using the ForEach view.

=====================================================

In the example above, we use the Group view with the subviews parameter, which allows us to extract the child views into a collection type called SubviewsCollection. The SubviewsCollection type conforms to the RandomAccessCollection protocol and provides us access by index.

As you can see, we use the Group view to decompose the content view and then compose the subviews in another way. We also leverage the power of the ID parameter, which allows us to use the ForEach view with plain data.

The new container APIs that SwiftUI introduced this year were missing the part about creating customizable and reusable views. I really love how easily it allows us to decompose views and compose them in other ways by hiding the implementation details of our container views.

Today, we learned how to create custom container views in SwiftUI using new Group and ForEach APIs.
