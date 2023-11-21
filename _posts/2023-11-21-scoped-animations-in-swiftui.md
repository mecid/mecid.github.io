---
title: Scoped animations in SwiftUI
layout: post
---

Animations were the most powerful feature of SwiftUI from day one. You can quickly build fluid animations in SwiftUI. The only downside was how we control animations whenever we need to run multi-step animation or scope the animation to a particular part of the view hierarchy.

Let's start with a simple example showing a few downsides of our old approaches to drive animations in SwiftUI.

=====================================================

As you can see in the example above, we have a view hierarchy with a button and two views placed in the vertical stack. We attach the animation view modifier to the whole stack to animate any change inside. 

When we press the button, the stack animates any changes inside. Still, the animation view modifier doesn't connect to the animating property, which means it will animate any change that can happen. Some of these changes can be unexpected, like environmental value change.

We can eliminate unexpected animations by using another version of the animation view modifier where we can bind to a particular value and animate only when the value changes.

=====================================================

In the example above, we use the animation view modifier with the value parameter. It allows us to scope the animation to a single value and animate changes only correlated with the particular value. In this case, we don't have any unexpected animations.

What if we have more than one animatable property? We must attach an animation modifier for every animatable property in this case. This solution works very well but has a few issues on the ergonomic side.

=====================================================

Fortunately, SwiftUI introduced a new variant of the animation view modifier, allowing us to scope animations using a ViewBuilder closure.

=====================================================

As you can see in the example above, we use the animation view modifier by providing a type of animation we need and a ViewBuilder closure where this animation applies. The animation works only in the context of the provided ViewBuilder closure and doesn't spread anywhere else.

As a starting point, the ViewBuilder closure provides a single parameter, the placeholder for the view where you have applied the animation view modifier. It is safe to apply any view modifiers to your view inside the ViewBuilder closure and expect animation only for this code block.

=====================================================

As you can see, SwiftUI provides a similar way to maintain scoped transactions in the view hierarchy.

This week, we have learned about a new approach for building precise and scoped animations in SwiftUI. Remember that it is available only on the latest platforms and is not backward compatible.
