---
title: Glassifying toolbars in SwiftUI
layout: post
---

Liquid Glass is the new design language Apple is using across all of its platforms. The look and feel of tabs were the major change that we covered last week. This week we will focus on another major change related to toolbars.

Toolbars changed significantly in Liquid Glass. They have a new glassy background, they can be split into groups, etc. Fortunately, similar to the tabs API, the toolbar API doesn’t change a lot and takes the Liquid Glass look and feel without any code change. But you still can use the new APIs to fine-tune behaviour. 

======================================================

As you can see in the example above, we use the same toolbar view modifier in pair with the ToolbarItem type. The new design language moves away from text-based toolbar items to symbol-based. So, remember to use buttons with both images and text labels.

Whenever you support platforms for Liquid Glass, you may need to keep the old text-based toolbar items. You can easily achieve that by using the labelStyle view modifier.

======================================================

As you can see, we use the labelStyle view modifier with toolbarStyle. The toolbarStyle doesn’t come as a part of SwiftUI framework; instead, we create it manually.

======================================================

It is pretty easy to define a custom label style. We check the availability of the Liquid Glass and apply necessary styling. We also mark the style as obsolete in iOS 26, which will be an error when you change the target of your project to iOS 26, so it becomes unnecessary and you can remove it.

ToolbarItemPlacement become really important while adopting Liquid Glass, because it controls not only placement but also the way it looks. For example, we use the confirmationAction placement, and it applies the glassProminent button style.

======================================================

Liquid Glass allows us to tint toolbar items using the tint view modifier and badge them using the badge view modifier. But use it wisely, it is not something you are going to use often, instead rely on toolbar placement API.

The new Liquid Glass provides us the new toolbar grouping functionality. For example, you can group a bunch of secondary actions together by splitting them from primary action.

======================================================

SwiftUI introduced the new ToolbarSpacer type allowing us to split toolbar items and place the space between them. You can apply fixed or flexible spacing between toolbar items.
