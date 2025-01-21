---
title: Color mixing in SwiftUI
layout: post
---

With the latest release of SwiftUI, Apple has introduced a new feature called color mixing. It’s a single function that enables you to perform various creative tasks. In this week’s discussion, we’ll dive into color mixing in SwiftUI and explore its potential adoption.

{% include friends.html %}

Let’s start with a simple example demonstrating the usage of the color mixing function.

```swift
struct ContentView: View {
    var body: some View {
        Color.blue.mix(with: .red, by: 0.5)
    }
}
```

![color-mixing](/public/color-mixing.png)

As you can see in the provided example, the *Color* struct introduces a new function called *mix*. This function accepts three parameters. The first parameter specifies the color you intend to blend with. The second parameter represents the proportion of the primary color in the blend. 

The third parameter specifies the color space for blending. By default, the *perceptual* color space is used, which closely resembles human eye color recognition. Alternatively, you can select the *device* color space, which slightly alters the final result and interpolates colors in the device’s color space.

```swift
struct ContentView: View {
    @State private var fraction = 0.5
    
    var body: some View {
        Color.blue.mix(with: .red, by: fraction, in: .device)
            .overlay {
                Slider(value: $fraction, in: 0...1)
            }
            .animation(.default, value: fraction)
    }
}
```

Remember that the mix function is fully animatable, and you can use the *animation* view modifier to animate it.

Now that we understand how to use color mixing in SwiftUI, let’s explore its practical usage in our applications. As a developer of a heart rate app, I immediately thought of utilizing color mixing to fill the heart rate status view with a dynamically changing color.

```swift
struct HeartRateStatusView: View {
    let recentHeartRate: Double
    let maxHeartRate: Double
    
    var body: some View {
        RoundedRectangle(cornerRadius: 2)
            .fill(Color.blue.mix(with: .red, by: recentHeartRate / maxHeartRate))
            .animation(.bouncy, value: recentHeartRate / maxHeartRate)
            .frame(height: 8)
    }
}
```

As you can see, I combine the user’s recent heart rate with the maximal possible heart rate value to determine the ratio of mixing blue and red colors representing the normal and elevated heart rate. 

Another intriguing adoption of color mixing is dynamic theming opportunities. For instance, we can utilize color mixing to create a seamless transition between light and dark colors within the app as the day progresses.

```swift
struct Theme {
    let light: Color
    let dark: Color
    
    func resolve() -> Color {
        let hour = Calendar.current.component(.hour, from: .now)
        return light.mix(with: dark, by: Double(hour) / 24.0)
    }
}
```

We use the current hour of the day retrieved from the calendar to calculate the fraction of the day that has passed. This fraction value is then used as a parameter to mix colors.

SwiftUI’s color mixing function offers huge possibilities to enhance the visual appeal of your applications. Any view that utilizes colors as a status indicator can greatly benefit from color mixing. For instance, the priority indicator can represent the priority of a task, calendar event, or any other entity that facilitates calculating the mixing fraction.

```swift
let eventPriority: Double
let maximalPriority: Double

Color.gray.mix(with: .red, by: eventPriority / maximalPriority)
```

We can begin by utilizing the system colors provided by SwiftUI and then apply color mixing based on user input to create visually appealing and dynamic color schemes in our applications.

This week, we delved into the color mixing functionality in SwiftUI. I hope this post inspires you to create more creative apps by harnessing the power of color mixing. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week!
