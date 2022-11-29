---
title: Building custom layout in SwiftUI. Caching.
layout: post
image: /public/flowlayout.png
category: Layout
---

In the previous post, we talked about the basics of the new *Layout* protocol. Today I'm going to continue the series of posts about the new opportunity to build super-custom reusable layouts by covering the idea of caching layout information and tuning its performance.

{% include friends.html %}

SwiftUI calls functions of your custom layout many times across the lifecycle to measure different sizing options during the layout process. It caches a few things automatically, but you can also implement your own caching mechanics if you need to improve your layout performance.

> To learn more about the basics of the *Layout* protocol, take a look at my dedicated ["Building custom layout in SwiftUI. Basics"](/2022/11/16/building-custom-layout-in-swiftui-basics/) post 

The *Layout* protocol has an associated type called *Cache*, which is *Void* by default. But you can define any type you need instead and implement your custom caching behavior. The easiest way is to define a nested type with the name *Cache* inside your custom layout type.

```swift
struct FlowLayout: Layout {
    struct Cache {
        var sizes: [CGSize] = []
        var positions: [CGPoint] = []
    }
}
```

The *Layout* protocol has the *makeCache* function, which we can implement to provide a cache instance and make some initial calculations to store during layout changes.

```swift
struct FlowLayout: Layout {
    struct Cache {
        var sizes: [CGSize]? = nil
        var positions: [CGPoint]? = nil
    }
    
    func makeCache(subviews: Subviews) -> Cache {
        let sizes = subviews.map { $0.sizeThatFits(.unspecified) }
        return Cache(sizes: sizes, positions: nil)
    }
}
```

The *Layout* protocol also provides the *updateCache* function, which we can use to update our cache. The SwiftUI framework calls this function as soon as children of the layout change. You can ignore the updateCache function. In this case, SwiftUI calls the *makeCache* function and builds the cache from scratch.

```swift
struct FlowLayout: Layout {
    struct Cache {
        var sizes: [CGSize]? = nil
        var positions: [CGPoint]? = nil
    }
    
    func makeCache(subviews: Subviews) -> Cache {
        let sizes = subviews.map { $0.sizeThatFits(.unspecified) }
        return Cache(sizes: sizes, positions: nil)
    }
    
    func updateCache(_ cache: inout Cache, subviews: Subviews) {
        cache.sizes = subviews.map { $0.sizeThatFits(.unspecified) }
    }
}
```

As you can see in the example above, the *updateCache* function provides cache instance as the *inout* parameter allowing us to mutate it during the call.

Keep in mind that both *makeCache* and *updateCache* functions provide you only the instance of the *Subviews* type, a proxy on the list of children. It doesn't provide you with the proposed size and bounds rectangle, which means we can't calculate the exact position of the views.

Instead, the *Layout* protocol allows us to mutate our cache in *sizeThatFits* and *placeSubviews* functions where we have the proposed size.

```swift
struct FlowLayout: Layout {
    // ....

    func placeSubviews(
        in bounds: CGRect,
        proposal: ProposedViewSize,
        subviews: Subviews,
        cache: inout Cache
    ) {
        var lineX = bounds.minX
        var lineY = bounds.minY
        var lineHeight: CGFloat = 0
        
        for index in subviews.indices {
            if lineX + cache.sizes[index].width > (proposal.width ?? 0) {
                lineY += lineHeight
                lineHeight = 0
                lineX = bounds.minX
            }
            
            let position = CGPoint(
                x: lineX + cache.sizes[index].width / 2,
                y: lineY + cache.sizes[index].height / 2
            )
            
            cache.positions.append(position)
            
            lineHeight = max(lineHeight, cache.sizes[index].height)
            lineX += cache.sizes[index].width
            
            subviews[index].place(
                at: position,
                anchor: .center,
                proposal: ProposedViewSize(cache.sizes[index])
            )
        }
    }
}
```

As you can see in the example above, we populate our cache with the exact position values during the *placeSubviews* function. Whenever SwiftUI calls this function, we can check our cache and place subviews in the cached position. In the case where our cache is empty, we can populate it with the correct values.

The *Layout* protocol provides us with all the necessary APIs for building performant custom layouts. We will continue learning the massive set of APIs SwiftUI gives us to create reusable and flexible layouts. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week!
