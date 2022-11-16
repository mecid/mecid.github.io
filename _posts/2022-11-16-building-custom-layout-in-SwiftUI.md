---
title: Building custom layout in SwiftUI
layout: post
---

Nowadays, SwiftUI provides the Layout protocol allowing us to build super-custom layouts by digging into the layout system without using GeometryReader. Layout protocol brings us the incredible power of building and reusing any layout you can imagine. This week we will learn how to use the new Layout protocol to build a FlowLayout in SwiftUI.

{% include friends.html %}

Any layout you want to build must conform to the new Layout protocol. It has two required functions to implement.

=====================================================

sizeThatFits function must calculate and return the final size of your layout.
placeSubviews function should place the subviews according to your layout rules.

Today we will start learning the Layout protocol by implementing flow layout in SwiftUI. Flow layout usually behaves like HStack, but it drops the line as soon as the views fill the horizontally available space.

Let's start with calculating the final size of our flow layout. It should iterate over all the subviews and sum the width of the views until we have space horizontally available. As soon as we reach the end of the available space, we should drop the placement line below and continue to place views from the leading edge.

=====================================================

In the example above, we have the implementation of the sizeThatFits function. This function provides a few interesting parameters that we will use to calculate the final size of our layout. The first one is the ProposedViewSize providing us with the size the parent proposes. The second one is the instance of type Subviews which is a proxy instance for the collection of subviews allowing us to place and measure the size of the views.

We use the instance of the Subviews proxy to iterate over all the children and calculate their ideal sizes using the sizeThatFits function. It accepts a parameter allowing us to get its minimal, maximal, and ideal dimensions. In the example above, we use an unspecified parameter which means ideal size. But you can also use zero and infinity instances to get their minimal and maximal size accordingly.

After calculating the ideal sizes for all the views, we go over them to calculate the total width and height of the final layout. We use the ProposedViewSize parameter to understand the width that the parent view provides us to know when we need to drop the line of the flow layout.

Finally, we construct and return the final size of the flow layout using all the knowledge that SwiftUI's layout system provides us. Now we can move to the implementation details of the placeSubviews function.

=====================================================

As you can see in the example above, the placeSubviews function has the same set of parameters as we had in the sizeThatFits but also provides us with the bounds rectangle. The bounds rectangle is the place in the view hierarchy we will fill with our subviews. Please don't assume that it has zero origin because it can be placed anywhere on the screen, and you should use the minX, minY, maxX, maxY, midX, midY properties to put your views ideally.

Subviews type allows us to access the view proxy by index and use its place function to put the view into a specific position. It also allows us to move the view's anchor point according to the position we pass.

=====================================================

Today we learned the Layout protocol's basics and built an elementary version of the flow layout. We will continue digging into the Layout protocol in the following posts to create more flexible configurations.
