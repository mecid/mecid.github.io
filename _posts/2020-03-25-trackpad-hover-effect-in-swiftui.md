---
title: Trackpad hover effect in SwiftUI
layout: post
---
Last week Apple updated iPad Pro and added trackpad support to iPadOS. We finally have Xcode 11.4, which introduces the onHover and hoverEffect modifiers to help us utilize trackpad and mouse support. This week we will learn how to be a good iOS citizen and add support for the trackpad to our SwiftUI views.

#### hoverEffect modifier
SwiftUI provides us the hoverEffect modifier that we can attach to any view. This modifier enables the transformation of trackpad or mouse pointer into the hovering view shape. It is tough to explain this transformation and better to see. Let's run the example on an iPad simulator.

================================================

Fortunately, iPad simulator supports trackpad emulation. You have to enable it by using I/O -> Input -> Send cursor to Device menu. Now we can see the pointer on the screen. Let's hover the text view with the pointer.

SwiftUI provides us a HoverEffect struct that describes three types of transformation of the pointer into a view shape. By default, hoverEffect modifier uses the first one, which is called automatic. Besides that, we have highlight and lift transformations. You can use them by only passing it as a parameter to hoverEffect modifier.

================================================

Highlight transformation describes an effect that morphs the pointer into a platter behind the view and shows a light source indicating position. On the other hand, lift transformation defines an effect that slides the pointer under the view and disappears as the view scales up and gains a shadow. Usually, we need the automatic transformation that attempts to determine the effect automatically.

#### onHover modifier
Now we are familiar with the standard types of hover effect that SwiftUI provides us. But what about custom effects? Happily, SwiftUI enables us to create super custom hovering effects by using onHover modifier. This modifier allows us to register a closure that will be called whenever the pointer of the trackpad or mouse covers a view. This modifier enables all the power of animations in SwiftUI that we can use to highlight changes.

================================================

As you can see in the example above, we use onHover modifier to register a closure that delivers us a boolean value. This boolean value is true whenever the pointer hovers the view. We save the value into a state variable and scale our view using default animation.

> To learn more about the power of animation modifier in SwiftUI, take a look at my "" post.

#### Conclusion
I am pleased to see that iPadOS gains the support of trackpad. I hope we will see Xcode running on iPadOS pretty soon, and iPad will replace Macbook. For now, let's support trackpad and mouse in our apps. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week!
