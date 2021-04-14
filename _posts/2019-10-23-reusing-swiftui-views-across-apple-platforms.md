---
title: Reusing SwiftUI views across Apple platforms
layout: post
image: /public/napbot-watch.jpeg
category: View Composition
---

This week we will talk about reusing SwiftUI views between Apple platforms. We will learn how to run the same views both on iOS, watchOS and macOS without any changes. To make it possible, all we need is an understanding of the view decomposition principle.

{% include friends.html %}

One of the slogans of SwiftUI during *WWDC* this year was "Learn one time, use anywhere". SwiftUI is a *declarative* framework for building *User Interfaces* on Apple platforms. But what does mean the word **declarative**? It means that you describe what you want to achieve, and the framework does all the needed work for you. Depending on the platform, SwiftUI can render content differently.

![napbot-screenshot](/public/napbot.jpeg)

Last month I had released my [NapBot app](https://napbot.swiftwithmajid.com), which I wrote entirely in SwiftUI. What I really love in this app is the how easy I was able to share my views between iOS and watchOS apps. I was able to run my app on watchOS just in a few minutes by adding a couple of small target checks. 

Let's take a look at a calendar day details screen. It uses a list component as the root view. Inside the list, we have a chart and a bunch of sections with text data. We already talked about creating a bar chart in ["Building BarChart with Shape API in SwiftUI"](/2019/08/14/building-barchart-with-shape-api-in-swiftui/) post, check that post if you want to learn more about *Shapes API*.

#### #if macro
This screen also has a picker component with the *SegmentedPicker* style, which allows changing the data represented in the chart. Unfortunately, watchOS doesn't support *SegmentedPicker* style. That's why I decide to remove it from the watchOS app but keep it in the iOS app. We can conditionally include or exclude code by using **#if macro** in SwiftUI. Let's take a look at the source code of my *SleepDetailsView*.

```swift
    @State private var chart: Chart = .sleepPhases

    private var chartSection: some View {
        VStack {
            #if os(iOS)
            Picker(selection: $chart, label: Text("chartType")) {
                Text("phases").tag(Chart.sleepPhases)
                Text("heartRate").tag(Chart.heartRate)
            }
            .pickerStyle(SegmentedPickerStyle())
            .labelsHidden()
            #endif

            BarChartView(bars: bars, labelsCount: 5)
                .frame(height: 280)
        }
    }
```

As you can see in the example above, we keep the picker component in the view hierarchy only when we build it for iOS. Swift compiler will exclude these lines during the watch app build process.

#### Container views
Another problem while adopting the sleep details screen to watchOS was a *NavigationBarItems*. watchOS doesn't have support for navigation bar items at all. Fortunately, I'm a huge fan of the *Container Views* concept, which allows us to get rid of this problem.

*Container View* is a view that handles the data flow and doesn't render any *User Interface* itself. Instead, *Container View* fetches the data and passes it to a *Rendering View*. We already talked about *Container Views* a few times on my blog. Please check the ["Introducing Container views in SwiftUI"](/2019/07/31/introducing-container-views-in-swiftui/) post to learn more.

I have a *SleepDetailsContainer* and *SleepDetailsView*. *SleepDetailsContainer* controls the data flow and passes data to *SleepDetailsView*. *SleepDetailsView* simply renders the data. Here is the source code of these two views.

```swift
struct SleepDetailsContainer: View {
    @EnvironmentObject var store: SleepStore

    var body: some View {
        SleepDetailsView(sleep: store.sleep)
            .onAppear(perform: store.fetch)
            .navigationBarItems(trailing: EditButton())
    }
}

struct SleepDetailsView: View {
    let sleep: Sleep

    var body: some View {
        Form {
            chartSection
            durationSection
            phasesSection
            heartRateSection
            noiseSection
        }
    }
}
```

In this case, we have *SleepDetailsView*, which is fully compatible with watchOS. All we need is creating a new version of *SleepDetailsContainer* for watchOS, which doesn't use navigation items. And here is the result.

![napbot-watch-screenshot](/public/napbot-watch.jpeg)

#### Conclusion
Today we learned how to reuse SwiftUI views on all Apple platforms with the minimum amount of changes. *Container Views* concept is still one of my favorite patterns, which I use a lot while developing SwiftUI apps. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week! 