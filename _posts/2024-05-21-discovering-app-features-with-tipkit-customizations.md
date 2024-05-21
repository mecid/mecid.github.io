---
title: Discovering app features with TipKit. Customizations.
layout: post
---

The final post on the topic of the TipKit framework is customization points. This week, we will learn how to customize a tip look and feel in our apps using the TipKit framework.

The TipKit framework provides a set of view modifiers, allowing us to customize the look and feel of a tip. Let's start with the simple ones.

=====================================================

When we utilize the tipCornerRadius view modifier, we can adjust the corner radius of any tip view displayed in the child views. It's important to note that the tipCornerRadius value propagates through the environment, influencing all the tip views in the view hierarchy.

=====================================================

As you can see in the example above, we can use the tipImageSize view modifier to set the size of the tip image in the case when it is defined in the Tip protocol conformance.

=====================================================

Another view modifier we use to customize the tip is the tipBackground view modifier. We can use the tipBackground view modifier to set any ShapeStyle as the background of the tip view. In the example above, we use the regular material to set the background for tip views.

=====================================================

We talked about the ready-to-use view modifiers, which allow us to customize them roughly. The TipKit framework provides the tipViewStyle view modifier, enabling us to set a fully custom style of the tip view.

The tipViewStyle needs an instance of the type conforming to the TipViewStyle protocol. The TipViewStyle protocol only has a requirement, the function called makeBody accepting the instance of the TipViewStyleConfiguration type and returning the view displaying the tip.

Any instance of the TipViewStyleConfiguration type provides us access to the instance of the Tip type that it will display. Inside the makeBody function, we can define the view in which we want to display the tip.

=====================================================

Here, we define the MyCustomTipViewStyle conforming to the TipViewStyle protocol. In the makeBody function we define multiple stacks to organize the presentation of the tip. As you can see in the example above, we can fully customize the view displaying the tip using SwiftUI @ViewBuilder.

Now, we can use the tipViewStyle view modifier to set the instance of the MyCustomTipViewStyle type. Remember that it also uses the environment to propagate the custom style into the view hierarchy.

As you can see, the TipKit framework provides many tools, allowing full customization of the views displaying our tips.
