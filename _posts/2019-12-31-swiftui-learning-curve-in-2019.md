---
title: SwiftUI learning curve in 2019
layout: post
image: /public/swiftui.png
category: Meta
---

This year we had a massive change in iOS development world. We got a SwiftUI framework. SwiftUI is a brand new declarative way of building apps for Apple ecosystem. Let's build our learning curve. I want to share with you the list of all the needed posts to learn SwiftUI.

{% include friends.html %}

#### Basics
Apple did an excellent job this year by creating the SwiftUI [tutorials page](https://developer.apple.com/tutorials/swiftui/). I suggest starting with those tutorials to learn the basics of SwiftUI. 

One of the new features of the Swift language powering SwiftUI framework is property wrappers. To learn about the most important property wrappers like *@State, @Binding, @ObservedObject, @EnvironmentObject, and @Environment* read my ["Understanding Property Wrappers in SwiftUI"](/2019/06/12/understanding-property-wrappers-in-swiftui/) post.

Another post that I wrote about the mechanics of SwiftUI is ["You have to change mindset to use SwiftUI"](/2019/11/19/you-have-to-change-mindset-to-use-swiftui/). It should help you to understand how SwiftUI works under the hood.

#### Layout system
SwiftUI also has a brand new layout system, which I enjoy much more than AutoLayout. The new layout system is very powerful and straightforward. I highly suggest you read these posts to understand the new layout system and its benefits.

1. [SwiftUI Layout System](https://kean.github.io/post/swiftui-layout-system)
2. [Alignment guides in SwiftUI](/2020/03/11/alignment-guides-in-swiftui/)
3. [The magic of view preferences in SwiftUI](/2020/01/15/the-magic-of-view-preferences-in-swiftui/)
4. [Anchor preferences in SwiftUI](/2020/03/18/anchor-preferences-in-swiftui/)
5. [Inspecting the View Tree – Part 3: Nested Views](https://swiftui-lab.com/communicating-with-the-view-tree-part-3/)

#### Architecture
SwiftUI has a lot of similarities with *React* framework, which brings many new concepts to iOS development. I've built a few apps using those ideas and really like how it works. Here is the list of useful posts.

1. [Introducing Container views in SwiftUI](/2019/07/31/introducing-container-views-in-swiftui/)
2. [Modeling app state using Store objects in SwiftUI](/2019/09/04/modeling-app-state-using-store-objects-in-swiftui/)
3. [Redux-like state container in SwiftUI](/2019/09/18/redux-like-state-container-in-swiftui/)

#### Declarative nature of SwiftUI
SwiftUI is a declarative framework. It means you declare what you want to achieve, and the framework takes care of that and decides how to render your views. The very same view can have another look, depending on the context. I wrote a few posts about the declarative approach in SwiftUI.

1. [Building forms with SwiftUI](/2019/06/19/building-forms-with-swiftui/)
2. [View composition in SwiftUI](/2019/10/30/view-composition-in-swiftui/)
3. [Reusing SwiftUI views across Apple platforms](/2019/10/23/reusing-swiftui-views-across-apple-platforms/)
4. [ViewModifiers in SwiftUI](/2019/08/07/viewmodifiers-in-swiftui/)
5. [Composable styling in SwiftUI](/2019/08/28/composable-styling-in-swiftui/)

#### Animations and interactions
SwiftUI handles all the state changes using animations just for free for you. It allows us to build interactive views in a very straightforward and nice way. I covered this topic many times on my blog.

1. [Animations in SwiftUI](/2019/06/26/animations-in-swiftui/)
2. [Gestures in SwiftUI](/2019/07/10/gestures-in-swiftui/)
3. [Building Bottom sheet in SwiftUI](/2019/12/11/building-bottom-sheet-in-swiftui/)
4. [Building Pager view in SwiftUI](/2019/12/25/building-pager-view-in-swiftui/)

I also want to mention the excellent series of posts by [Javier](https://swiftui-lab.com) about advanced animations in SwiftUI.

1. [Advanced SwiftUI Animations – Part 1: Paths](https://swiftui-lab.com/swiftui-animations-part1/)
2. [Advanced SwiftUI Animations – Part 2: GeometryEffect](https://swiftui-lab.com/swiftui-animations-part2/)
3. [Advanced SwiftUI Animations – Part 3: AnimatableModifier](https://swiftui-lab.com/swiftui-animations-part3/)

#### Accessibility
SwiftUI did another step to make our apps accessible by default. It works very well out of the box, but it also provides a very nice API to customize the accessibility tree. To learn more about accessibility in SwiftUI, I suggest reading these articles:

1. [Accessibility in SwiftUI](/2019/09/10/accessibility-in-swiftui/)
2. [Dynamic Type in SwiftUI](/2019/10/09/dynamic-type-in-swiftui/)
3. [Localization in SwiftUI](/2019/10/16/localization-in-swiftui/)

#### Drawing custom views
SwiftUI provides a lovely *Shape API* that allows us to build custom views quickly. I learned it while making charts in one of my apps.

1. [Building BarChart with Shape API in SwiftUI](/2019/08/14/building-barchart-with-shape-api-in-swiftui/)
2. [Gradient in SwiftUI](/2019/11/13/gradient-in-swiftui/)
3. [GeometryReader to the Rescue](https://swiftui-lab.com/geometryreader-to-the-rescue/)

#### Unique features 
SwiftUI has unique features like *Environment* and *Preferences*, which we don't have in *UIKit*. To learn more about these features, take a look at the next posts.
1. [The power of Environment in SwiftUI](/2019/08/21/the-power-of-environment-in-swiftui/)
2. [The magic of view preferences in SwiftUI](/2020/01/15/the-magic-of-view-preferences-in-swiftui/)
3. [The power of @ViewBuilder in SwiftUI](/2019/12/18/the-power-of-viewbuilder-in-swiftui/)

#### Conclusion
It was a huge year, and I hope we will get SwiftUI 2.0 with a lot of new features during WWDC 2020. We have many things to learn until the next release of SwiftUI. Let's say thanks to our awesome community which wrote all these articles. Merry Christmas and Happy New Year!
