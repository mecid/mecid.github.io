---
title: Building custom layout in SwiftUI. Caching.
layout: post
---

In the previous post, we talked about the basics of the new Layout protocol. Today I'm going to continue the series of posts about the new opportunity to build super-custom reusable layouts by covering the idea of caching layout information and tuning its performance.

SwiftUI calls functions of your custom layout many times across the lifecycle to measure different sizing options during the layout process. It caches a few things automatically, but you can also implement your own caching mechanics if you need to improve your layout performance.

The Layout protocol has an associated type called Cache, which is Void by default. But you can define any type you need instead and implement your custom caching behavior. The easiest way is to define a nested type with the name Cache inside your custom layout type.

=====================================================

The Layout protocol has the makeCache function, which we can implement to provide a cache instance and make some initial calculations to store during layout changes.

=====================================================

The layout protocol also provides the updateCache function, which we can use to update our cache. The SwiftUI framework calls this function as soon as children of the layout change. You can ignore the updateCache function. In this case, SwiftUI calls the makeCache function and builds the cache from scratch.

=====================================================

As you can see in the example above, the updateCache function provides cache instance as the inout parameter allowing us to mutate it during the call.

Keep in mind that both makeCache and updateCache functions provide you only the instance of the Subviews type, a proxy on the list of children. It doesn't provide you with the proposed size, which means we can't calculate the exact position of the views.

Instead, the Layout protocol allows us to mutate our cache in sizeThatFits and placeSubviews functions where we have the proposed size.

=====================================================

As you can see in the example above, we populate our cache with the exact position values during the placeSubviews function. Whenever SwiftUI calls this function, we check our cache and place subviews in the cached position. In the case where our cache is empty, we populate it with the correct values.

The Layout protocol provides us with all the necessary APIs for building performant custom layouts. We will continue learning the massive set of APIs SwiftUI gives us to create reusable and flexible layouts.
