---
title: What is new in SwiftUI after WWDC 23.
layout: post
---

WWDC 23 is here, so many things have changed and been added to the SwiftUI framework. In this post, you can find the summary of the most significant SwiftUI features available in the 5th iteration of the framework.

#### Data flow
Swift 5.9 introduced the macros feature, which became the heart of the SwiftUI data flow. SwiftUI became Combine-free and uses the new Observation framework now. The Observation framework provides us with the Observable protocol that we have to use to allow SwiftUI to subscribe to changes and update views.

=====================================================

You don't need to conform to the Observable protocol in your code. Instead, you mark your type with the @Observable macro, which makes that conformance to the Observable protocol for you. You also don't need the @Published property wrapper now because SwiftUI views automatically track the changes in the available properties of any observable type.

=====================================================

Previously, there were a bunch of property wrappers like State, StateObject, ObservedObject, and EnvironmentObject, which you should understand when and why to use. Nowadays, state management has become much more manageable. You will only use the State property wrappers both for value types like Strings and Integers and reference types conforming to the Observable protocol.

=====================================================

In the example above, we have a view that accepts the Store type. In the previous iterations of the SwiftUI framework, we should mark it with the @ObservedObject property wrapper to subscribe changes. We don't need it now because SwiftUI views automatically track changes in the types conforming to the Observable protocol.

=====================================================

You can also use the Environment property wrapper in pair with the environment view modifier to put the observable type into the SwiftUI environment. There is no need to use the @EnvironmentObject property wrapper or the environmentObject view modifier. The same Environment property wrapper works with the observable types now.

#### Animations
Animations always was the most vital part of the SwiftUI framework. It is effortless to animate anything in SwiftUI, but previous framework versions lack some features that we have now.  

=====================================================

As you can see in the example above, we have the new version of the withAnimation function allowing us to provide an animation completion handler. It is a great addition, and you can build phased animations now. 

=====================================================

The SwiftUI framework introduces the new PhaseAnimator view that iterates over the sequence of phases, allows you to provide different animations for every phase, and updates the content whenever phase changes. There is also the KeyframeAnimator view allowing us to animate changes with keyframes.

#### ScrollView
ScrollView has excellent additions this year. First, we can observe content offset using the scrollPosition view modifier.

=====================================================

As you can see, we use the scrollPosition view modifier to bind the content offset to a state property. Whenever the user scrolls the view, it updates the binding by setting the identity of the first visible view. You can also scroll to any view by updating the binding programmatically. But remember that you should use the scrollTargetLayout view modifier to tell the SwiftUI framework where to find identities to update the binding.

=====================================================

You can now change the scroll behavior by using the scrollTargetBehavior view modifier. It allows you to enable paging in a scroll view.

#### New gestures
New RotateGesture and MagnifyGesture allow us to track the view's rotation and magnification.

====================================================

#### New views
We have the brand new ContentUnavailableView type that we can use whenever we should display an empty view. It is a small but delightful addition.

====================================================

#### Conclusion
There are tons of small additions all over the SwiftUI framework that we will cover during the upcoming months. So stay tuned to my blog, and don't miss anything.

