---
title: Customizing view content shape in SwiftUI
layout: post
---

Usually, SwiftUI uses rectangles to render views, but we can control the shape of the view by using the clipShape view modifier. This week we will learn how to modify the interactable shape of the view during hit-testing or previewing drag and drop.

Let's start with a simple view that displays text and an image in a vertical stack.

=====================================================

Now, we want to make this view tappable and provide the hover effect for the iPadOS pointer.

> To learn more about implementing the hover effect on iPadOS, take a look at my dedicated "Hover effect in SwiftUI" post.

=====================================================

Finally, we can customize the shape of the hover effect by using the contentShape view modifier.

=====================================================

The contentShape view modifier changes the shape of the view used for a particular interaction. We change the content shape only for the hover effect in our example. But we can do it for different types of interaction, like drag and drop previews, hit testing, context menu previews, and hover effect.

The contentShape view modifier accepts the instance of the ContentShapeKinds option set that defines interactions on a view. SwiftUI provides us these options to use:

interaction - this one offers a shape for hit testing
dragPreview - this type provides an outline for drag and drop previews
contextMenuPreview - this type provides shape for rendering context menu previews.
hoverEffect - this type provides shape for a hover effect on iPadOS

ContentShapeKinds struct conforms to OptionSet protocol which means you can combine different options together.

=====================================================

> If you are not familiar with OptionSet protocol, take a look at my "Inclusive enums with OptionSet" post.

Remember that the contentShape view modifier affects only the interactable shape of the view. If you need to change the shape while rendering, you should use the clipShape view modifier.

> To learn more about clipping and masking in SwiftUI, take a look at my "Customizing the shape of views in SwiftUI" post.

=====================================================

Keep in mind that you can use any shape you need. As you can see in the example above, we use a custom shape that clips the view content to a triangle.

This week we learned how to customize the shape of the view during interactions via the brand new contentShape view modifier.
