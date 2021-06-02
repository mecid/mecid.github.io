---
title: Frames in SwiftUI
layout: post
image: /public/fixed-frame.png
category: Layout
---

SwiftUI provides us a magical frame modifier that you might think is very simple and straightforward to use. But there is a lot of complicated logic under the hood. This week we will talk about fixed and flexible frames and the frame modifier to control them.

{% include friends.html %}

#### Fixed frame
Let's start with the straightforward type of frame called a fixed frame. The fixed frame is a way to create an invisible frame around the view that will propose the size you mention to the view. It doesn't mean that the frame will set the size of the view inside. The frame only suggests the size, and the view can completely ignore it. Let's take a look a two different examples.

=====================================================

As you can see in the example above, we have a Text view that shrinks its content to fit the size proposed by the frame.

=====================================================

In the second example, we use a green rectangle with the size of 100x100. We also add a small fixed frame around the rectangle, but in this case, the green rectangle ignores the size proposed by the frame and draws itself outside of the frame.

#### Flexible frame
There is another version of the frame modifier that accepts seven parameters.

=====================================================

Let's focus on the horizontal axis because the same applies to the vertical. We will talk about ideal width and height in the next section because it doesn't affect flexible frames. As you can see, there are optional parameters for minimal and maximal width. You have three options here:
You provide both minimal and maximal value.
You provide only minimal value.
You provide only maximal value.
	
Let's start with the first one when you provide both minimal and maximal value. In this case, SwiftUI completely ignores the size of the child inside the frame and clamps the final size of the frame between minimal and maximal value.

=====================================================

In the second case, we have only minimal value provided, which leads to the situation where SwiftUI checks the content size of a child view inside a frame and makes a decision based on this value. The final size of the frame will be the minimum value that you pass if the content size of the child view is smaller than the value you provide. Otherwise, it will be equal to the content size of the child view.

=====================================================

In the third case, we have only maximal value provided, and it leads to the condition where SwiftUI checks the content size of a child view inside a frame. It sets the final size of the frame to the maximal value that you pass whenever it is bigger than the content size of the child view. Otherwise, it will be equal to the content size of the child view.

#### Ideal size
Ideal width and heigh parameters allow us to provide an intrinsic. Intrinsic size is usually the size of the content. In the case of the Text view, it is the size of the string presented in the view. In the case of a shape like a Rectangle or Circle, the ideal size is undefined, and the view tries to fill the available space. The frame modifier allows you to provide the ideal size for the views that don't have content. SwiftUI uses ideal size only in conjunction with the fixedSize modifier.

#### Conclusion
The most challenging topic in SwiftUI for me was frame behavior. Fixed-size, flexible frames, ideal size, so many options for a single frame modifier. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week!