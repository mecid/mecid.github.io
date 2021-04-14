---
title: Building widgets in SwiftUI
layout: post
image: /public/widget.jpg
category: Mastering SwiftUI views
---

This week I want to talk to you about home-screen widgets in iOS 14. I've built several widget collections for my apps, and it is a perfect time to share with you that experience. Today we will learn all about building and updating widgets with SwiftUI.

{% include friends.html %}

#### Basics
Widgets display relevant, glanceable content, letting users quickly get to your app for more details. Your app can provide multiple kinds of widgets, allowing users to focus on essential information. They can add multiple copies of the same widget, tailoring each one to their unique needs and layout. I think it is an excellent definition for widgets, thanks to Apple.

Let's start by adding a widget extension to your app using the Xcode menu: *File -> New -> Target -> Widget Extension*. Xcode creates a widget from the template. It might look like this.

```swift
@main
struct MyWidget: Widget {
    let kind: String = "Widget"

    var body: some WidgetConfiguration {
        StaticConfiguration(kind: kind, provider: Provider()) { entry in
            WidgetEntryView(entry: entry)
        }
        .supportedFamilies([.systemSmall, .systemMedium, .systemLarge])
        .configurationDisplayName("My Widget")
        .description("This is an example widget.")
    }
}
```

As you can see, we develop a widget by creating a struct that conforms to the *Widget* protocol. The only requirement of the *Widget* protocol is body property that should return an instance of *WidgetConfiguration*. SwiftUI provides us two structs that conform to *WidgetConfiguration*: *StaticConfiguration* and *IntentConfiguration*. 

We can use *IntentConfiguration* to provide user-configurable options for our widget. In this post, we will talk about *StaticConfiguration*. *StaticConfiguration* allows us to register a widget configured by the developer that time-to-time updates its data.

*StaticConfiguration* needs three parameters. Let's take a look at them one-by-one.

1. Kind is a string that identifies the type of widget. Your app might have multiple widgets. In this case, a kind identifier allows you to update widgets of a particular kind.
2. The provider is a type conforming to *Provider* protocol and used by the system to fetch widget data.
3. View builder closure that describes the SwiftUI view for displaying widget data.

You can also attach a few modifiers to your widget configuration for setting supported widget families or providing description and name.

> To learn more about *@main* and new App lifecycle in SwiftUI, take a look at my ["Managing app in SwiftUI"](https://swiftwithmajid.com/2020/08/19/managing-app-in-swiftui/) post.

#### Provider
The system will call your provider to fetch new data. Let's look at the provider example that I use in my [CardioBot](https://cardiobot.swiftwithmajid.com) app to display daily heart points.

```swift
final class HeartPointsProvider: TimelineProvider {
    public typealias Entry = HeartPointsViewModel

    let service = HealthService()
    var snapshotCancellable: AnyCancellable?
    var timelineCancellable: AnyCancellable?

    private var entryPublisher: AnyPublisher<Entry, Never> { /*.... */ }

    func placeholder(in with: Context) -> HeartPointsViewModel {
        HeartPointsViewModel(
            heartRates: [HeartRate(bpm: 130)],
            age: 29,
            goal: 20
        )
    }

    func getSnapshot(in context: Context, completion: @escaping (HeartPointsViewModel) -> Void) {
        snapshotCancellable = entryPublisher
            .receive(on: DispatchQueue.main)
            .sink(receiveValue: completion)
    }

    func getTimeline(in context: Context, completion: @escaping (Timeline<HeartPointsViewModel>) -> Void) {
        timelineCancellable = entryPublisher
            .map { Timeline(entries: [$0], policy: .atEnd) }
            .receive(on: DispatchQueue.main)
            .sink(receiveValue: completion)
    }
}
```

We start by defining type alias for the associated type of *Provider* protocol. *Provider* protocol uses the *Entry* type to describe an item that should be displayed in the widget. Entry must conform to the *TimelineEntry* protocol that requires only date property.

```swift
extension HeartPointsViewModel: TimelineEntry {
    var date: Date {
        heartRates.last?.interval.end ?? Date()
    }
}
```

There are three required functions in the protocol:
1. *placeholder* function is used by the system to generate a template view in the widget gallery. You can provide mock data here that will look nice in the gallery.
2. *getSnapshot* function is used when the widget is in a transient state. Call the completion handler as soon as possible to provide data for your widget.
3. *getTimeline* function is used by the system to fetch current and future entries for your widget.

*getTimeline* is the most exciting function here. It allows us to return a timeline of entries for our widget and define a date for the next update.

#### Widget view
Usually, we create a SwiftUI view that accepts an entry and displays it. Then we can easily use it inside a view builder closure in widget configuration. We can use *widgetFamily* environment value to understand the current widget configuration's size and present different views.

```swift
struct HeartPointsEntryView: View {
    @Environment(\.widgetFamily) var size
    let entry: HeartPointsViewModel

    var body: some View {
        switch size {
        case .systemSmall:
            SmallWidgetEntryView(viewModel: entry).padding()
        case .systemMedium:
            HStack(alignment: .bottom) {
                SmallWidgetEntryView(viewModel: entry)
                HeartPointsChartView(viewModel: entry, labelCount: 6)
            }.padding()
        case .systemLarge:
            VStack(alignment: .leading) {
                HeartPointsView(viewModel: entry, showLabel: false)
                HeartPointsChartView(viewModel: entry)
            }.padding()
        @unknown default:
            SmallWidgetEntryView(viewModel: entry).padding()
        }
    }
}
```

#### Widget updates
The system will try to predict the best time to perform widget updates using providers. But you can always ask to update your widgets using *WidgetCenter*.

```swift
WidgetCenter.shared.reloadAllTimelines()
WidgetCenter.shared.reloadTimelines(ofKind: "heartRateWidget")
```

As you can see, there are two options for updating your widgets. You can update all of your widgets or update only the particular kind.

#### Widget bundle
As we know, your app might have more than one widget. In this case, we have to create a widget bundle.

```swift
@main
struct CardioBotWidgets: WidgetBundle {
    var body: some Widget {
        HeartPointsWidget()
        HeartRateSummaryWidget()
    }
}
```

Remember to remove *@main* from your widget definitions and add it only to your widget bundle.

#### Conclusion
Today we learned about another feature that is released during WWDC20. Remember that widgets are available on macOS in the notification center also. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this article. Thanks for reading, and see you next week!