---
title: Mastering GroupBox in SwiftUI
layout: post
image: /public/groupbox2.png
category: Mastering SwiftUI views
---

Styleable views is the thing I love in SwiftUI. You can separate your view logic and its style. You can easily apply different styles in different conditions whenever you need to change appearance depending on the platform or other environmental requirements. This week we will talk about GroupBox, another view container that SwiftUI provides, and allows us easily change its look and feel using a style protocol. 

#### Basics
GroupBox is a stylized view with an optional label that is associated with a logical grouping of content. Default styling on iOS is a simple card with the title and content. Let's take a look at it.

=====================================================

As you can see in the example above, the default styling of GroupBox is a card with the system grouped background and continuous corner radius. It looks similar to cards that we used to see in the Apple Health app or Apple Fitness app. I should mention that it can look differently on different Apple platforms. It is available for macOS and iOS now, but I hope to see it on watchOS and tvOS during the next iteration of SwiftUI.

Cards are great, and I use them a lot in my apps. You can use GroupBox as the root container for every item in your grid or stack, and you will gain a modest card-based look and feel.

=====================================================

As you can see, SwiftUI provides nice card-based styling for GroupBox view by default. Sometimes it is not what we need. For example, when you place GroupBox inside a grouped List, it looks like a card inside a card. Fortunately, we can tune the look and feel using the GroupBoxStyle protocol.

#### Styling
SwiftUI provides us the GroupBoxStyle protocol that allows us to change the look and feel of any GroupBox instance completely. We can create different styles for any use-cases we need. For example, I don't need a background and corner radius when I use GroupBox inside a List.

=====================================================

All you need to do to create your own GroupBox style is create a struct that conforms to GroupBoxStyle protocol. GroupBoxStyle has the only requirement. You have to create a makeBody function that accepts an instance of GroupBoxStyleConfiguration type and returns a new view. GroupBoxStyleConfiguration provides us both the label and content of our GroupBox. You can use them inside the makeBody function as you need. Our PlainGroupBoxStyle example put the label and content view inside a VStack with leading alignment and returns the stack.

=====================================================

You can create super custom GroupBox styles. To show more complex styling options, let's replicate the default card-based layout.

=====================================================

As you can see, we are able to create any GroupBox style we need. Another option might be a style that hides the label or something else. 

#### Conclusion
GroupBox provides us a simple card-based look and feel that we used to see in many apps on iOS. The great thing about GroupBox that it allows us to customize its appearance as we need. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this article. Thanks for reading, and see you next week!