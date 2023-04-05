---
title: Drag and drop transferable data in SwiftUI
layout: post
---

Last we discussed the new ShareLink view and the Transferable protocol powering it. The new Transferable protocol is useful for sharing your data from the app, but it also powers drag and drop in your app. This week we will learn how we can drag and drop data conforming to the Transferable protocol in SwiftUI.

#### Dragging
The new Transferable protocol allows us to implement the drag-and-drop experience in our apps much more easily. All you need to do is to attach the draggable view modifier to any view in the view hierarchy and pass the data you want to drag. The data you pass have to conform to the new Transferrable protocol.

=====================================================

As you can see in the example above, we use the new draggable view modifier to enable dragging URLs from the list. The URL type conforms to the new Transferable protocol out of the box, and you don't need to do anything here to implement the conversion of URL data to the transferable format. We also provide a preview for the draggable content by creating a Text view with the URL string.

The example above shows how you can enable the drag and drop feature for the URL item, but you can do it for any type conforming to the Transferable protocol.

#### Dropping
The next step is allowing your app to accept transferable content via drops. All you need to do is to attach the new dropDestination view modifier to any view you want to take drops.

=====================================================

As you can see in the example above, we use the dropDestination view modifier to register an area that takes drops of the URL type. We also use the dropDestination view modifier to register closure handling drops. It takes two parameters: items and location. The items parameter defines an array of transferable data, and the location parameter indicates the drop position on the screen.

#### Conclusion
As you can see, it is straightforward to support the drag and drop feature in your apps with the new draggable and dropDestination view modifiers. Both of them are powered by the brand new CoreTransferable framework.
