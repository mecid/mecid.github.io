---
title: ImageRenderer in SwiftUI 
layout: post
---

I work on a medical app where the user needs to export health data rendered by using the Swift Charts framework. It was straightforward to achieve by leveraging the power of the new ImageRenderer type. This week we will learn how to use the ImageRenderer type to export SwiftUI view as image or PDF.

The ImageRenderer type provides an easy-to-use API, allowing us to export the SwiftUI view hierarchy as an image. Let's take a look at a quick example.

=====================================================

As you can see in the example above, we use the StateObject property wrapper to define an instance of the ImageRenderer type. The ImageRenderer type conforms to the ObservableObject protocol, and SwiftUI automatically updates the view hierarchy whenever an instance of the ImageRenderer marked with the StateObject property wrapper changes.

The only parameter we need to create an instance of the ImageRenderer type is the content, and it has to be a SwiftUI view. The ImageRenderer type provides us uiImage and cgImage properties allowing us to access the rendered image of the content view.

In the previous example, we create an instance of the ImageRenderer type by passing an instance of the MyChartView type, which uses the Swift Charts framework to display its content. In the body of the ContentView, we place Image view to show rendered version of the MyChartView. As soon as the renderer is ready, SwiftUI updates the body of the ContentView.

It was a static example of using the ImageRenderer type. Usually, we need to export dynamic SwiftUI views with the data available only during app runtime.

=====================================================

As you can see in the example above, we render the part of the particular SwiftUI view as an image displaying some data summary. The ImageRenderer type allows us to tune a few parameters affecting the final result of the exported image. For example, we can change the scale, size, and color mode using the scale, proposedSize, and colorMode properties.

=====================================================

Another exciting feature of the ImageRenderer type is the ability to draw the rendered image in any instance of the CGContext type. It means you can use the ImageRenderer to make an image of SwiftUI view a part of your PDF context.

=====================================================

As you can see in the example above, we create an instance of the CGContext to generate a PDF file. We use the render function of the ImageRenderer type allowing us to access the size of the image displaying a SwiftUI view, and the renderer function that we can use to inject the image into an instance of the CGContext type.
