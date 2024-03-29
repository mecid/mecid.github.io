---
title: Lock screen widgets in SwiftUI
layout: post
category: Mastering SwiftUI views
image: /public/wwdc22.jpg
---

One of the most requested features for iOS was customizable lock screens. We can populate our lock screen with glanceable widgets. Implementing a lock screen widget is straightforward because its API shares the same code with home screen widgets. This week we will learn how to implement lock screen widgets for our apps.

{% include friends.html %}

Let's start with a code you might already have in your app for displaying home screen widgets.

```swift
struct WidgetView: View {
    let entry: Entry
    
    @Environment(\.widgetFamily) private var family
    
    var body: some View {
        switch family {
        case .systemSmall:
            SmallWidgetView(entry: entry)
        case .systemMedium:
            MediumWidgetView(entry: entry)
        case .systemLarge, .systemExtraLarge:
            LargeWidgetView(entry: entry)
        default:
            EmptyView()
        }
    }
}
```

In the example above, we have a typical view defining a widget. We use the environment to understand the widget family and display a properly sized view. All we need to do to support lock screen widgets is to remove the default statement and implement all the new cases defining lock screen widgets.

```swift
struct WidgetView: View {
    let entry: Entry
    
    @Environment(\.widgetFamily) private var family
    
    var body: some View {
        switch family {
        case .systemSmall:
            SmallWidgetView(entry: entry)
        case .systemMedium:
            MediumWidgetView(entry: entry)
        case .systemLarge, .systemExtraLarge:
            LargeWidgetView(entry: entry)
        case .accessoryCircular:
            Gauge(value: entry.goal) {
                Text(verbatim: entry.label)
            }
            .gaugeStyle(.accessoryCircularCapacity)
        case .accessoryInline:
            Text(verbatim: entry.label)
        case .accessoryRectangular:
            VStack(alignment: .leading) {
                Text(verbatim: entry.label)
                Text(entry.date, format: .dateTime)
            }
        default:
            EmptyView()
        }
    }
}
```

It would be best to remember that the system uses different rendering modes for lock screen and home screen widgets. The system provides us with three different rendering modes.

1. Full-color mode for home screen widgets and watchOS complications supporting colors. And yes, you can also use WidgetKit to implement watchOS complications, starting with watchOS 9.
2. Vibrant mode is where the system desaturates text, images, and gauges into monochrome and colors them properly for the Lock Screen background. Vibrant mode is also used in StandBy.
3. The accented mode is used only on watchOS, where the system divides the widget into two groups, default and accented. The system colors the accented part of your widget with the tint color the user chooses in the watch face settings.

> Look at my dedicated ["Building widgets in SwiftUI"](/2020/09/09/building-widgets-in-swiftui/) post to learn more about home screen widgets.

Rendering mode is available via the SwiftUI environment, so you can always check which rendering mode is active and reflect it in your design. For example, you might use different images with different rendering modes.

```swift
struct WidgetView: View {
    let entry: Entry
    
    @Environment(\.widgetRenderingMode) private var renderingMode
    
    var body: some View {
        switch renderingMode {
        case .accented:
            AccentedWidgetView(entry: entry)
        case .fullColor:
            FullColorWidgetView(entry: entry)
        case .vibrant:
            VibrantWidgetView(entry: entry)
        default:
            EmptyView()
        }
    }
}
```

As you can see in the example above, we use the *widgetRenderingMode* environment value to get the actual rendering mode and behave differently. As I said before, in the accented mode, the system divides your widget into two parts and colors them specially. You can mark the part of your view hierarchy with the *widgetAccentable* view modifier. The system will know which views apply the tint color in this case.

```swift
struct AccentedWidgetView: View {
    let entry: Entry
    var body: some View {
        HStack {
            Image(systemName: "moon")
                .widgetAccentable()
            Text(verbatim: entry.label)
        }
    }
}
```

Finally, we need to configure our widget with the supported types.

```swift
@main
struct MyAppWidget: Widget {
    let kind: String = "Widget"
    
    var body: some WidgetConfiguration {
        StaticConfiguration(kind: kind, provider: Provider()) { entry in
            WidgetView(entry: entry)
                .containerBackground(.regularMaterial, for: .widget)
        }
        .configurationDisplayName("My app widget")
        .supportedFamilies(
            [
                .systemSmall,
                .systemMedium,
                .systemLarge,
                .systemExtraLarge,
                .accessoryInline,
                .accessoryCircular,
                .accessoryRectangular
            ]
        )
    }
}
```

As you can see in the example above, we use the *containerBackground* view modifier allowing us to set a removable background for the view. It is required for all widgets to use the removable background from iOS 17. The system removes the background in StandBy and in smart stack on watch.

Today we learned how to implement new lock screen widgets in iOS 16. Remember that we can reuse the same API to build watchOS complications. And you can easily share the widget codebase by looking into environment values to understand which rendering mode is active at the very moment. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this article. Thanks for reading, and see you next week!
