---
title: Styling custom SwiftUI views using environment
category: Building custom views
layout: post
image: /public/redacted.jpeg
---

One of my favorite features of SwiftUI is styling. I love the idea of styling protocols provided by every view and sharing them using the environment. I have already covered most of the style protocols for SwiftUI provided views in my previous posts. But what about custom views? This week we will learn how to build style protocols for our custom views.

#### Basics
If you are not familiar with style protocols in SwiftUI, let me show you a rapid example of using them.

=====================================================

In the example above, we attached the listStyle modifier to the root view of the app. This modifier shares the instance of ListStyle protocol with the whole app view hierarchy using the environment. All the app list instances will use InsetGroupedListStyle by default just because of this single line of code. But don't worry. Any particular part of the view hierarchy can override this value with any other style when needed. Let's take a look at another example.

=====================================================

As you can see here, we apply the unified button style for all the buttons in the app. All the app buttons will be filled with the accent color and will use the rounded rectangle's shape.

> To learn more about ButtonStyle protocol, look at my "Mastering buttons in SwiftUI" post.

#### Styling using view parametes
Now we know how to use the style protocols that SwiftUI provides us, but what about our custom views? I maintain a small charting library. As you may know, charts can be highly flexible in terms of configuration. Let's take a look at the usage example of my charting library.

=====================================================

As you can see, multiple parameters allow us to configure the chart presentation. There are actually many more parameters, but all of them have default values, and you don't need to provide them here.

The first thing that I decide to do is extract the chart presentation parameters into a separated struct to keep them apart from chart data.

=====================================================

Let's take a look at the usage example now.

=====================================================

#### Styling using environment 
The previous example looks better than before, but it doesn't give us all the benefits of environment sharing that standard SwiftUI views have. We can easily solve it by inserting an instance of style struct into the environment instead of passing it via initializer. Let's take a look at how we can do that.

=====================================================

First of all, we create an additional environment value that will hold the chart style. Then we create an extension on View protocol that allows us to insert chart styles into a view hierarchy environment.

> To learn more about the possibilities of SwiftUI's environment feature, take a look at my "The power of Environment in SwiftUI" post.

We can use the environment property wrapper inside BarChartView to obtain the style that the environment shares.

=====================================================

#### Conclusion
I really love this approach because it easily allows us to style the whole app hierarchy. For example, in my app, I have a screen that displays a list of charts with different data points. All of these charts have the same styling, which I share using the environment. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this article. Thanks for reading, and see you next week!