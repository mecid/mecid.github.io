---
title: Building custom layout in SwiftUI. Spacing.
layout: post
---

Multiple layouts allow us to compose views in different ways. One crucial thing is the spacing between children of the concrete layout. This week we will learn how to build a custom layout allowing us to specify a particular spacing between views and how to respect the platform-oriented predefined spacing rules in SwiftUI.

As many types we create in Swift, the type conforming to the Layout protocol can define its properties and initialize it via the init function. Our FlowLayout type is not an exception here. Let's add the spacing property to the FlowLayout type.

=====================================================

As you can see in the example above, now we can place an instance of the FlowLayout type with the particular spacing between views. But first, we should tune the sizeThatFits function to respect the spacing between views while calculating the final size of the layout.

=====================================================

Second, we must add spacing between views while placing them in the placeSubviews function.

=====================================================

We have a fully working FlowLayout that allows us to set the custom spacing between views.

#### Preferred spacing
As you can see, most of the layouts in SwiftUI allow us to set the spacing to nil, where the layout uses the preferred spacing instead of 0. SwiftUI has spacing preferences between views. For example, it has preferred spacing between Image and Text views, but this value might differ from Text to Text views. And these spacing preferences might be different for iOS, macOS, watchOS, and tvOS.

Fortunately, SwiftUI provides an API to calculate spacing between views by respecting platform-oriented spacing preferences. This API is a part of the Layout protocol and lives in the Subview proxy type.

=====================================================

Let's start by adding the spacing property to our cache. It is a perfect candidate to live in the cache because we want to calculate it once when the list of the subviews changes. Let's tune the sizeThatFits function to respect the spacing between views while calculating the final size of the layout.

=====================================================

The next step is to add spacing between views while placing them. We can apply similar changes to the placeSubviews function to respect the preferred spacing between different views.

=====================================================

Finally, we have a flow layout that respects the preferred view spacing by default. The Layout protocol provides us with the reach API to build reusable layouts across all the platforms. We should carefully learn the API SwiftUI provides to create layouts respecting the platform-oriented rules.
