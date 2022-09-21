---
title: Displaying live activities in iOS 16
layout: post
category: Mastering SwiftUI views
image: live-activity.jpg
---

One of the most prominent features of iOS 16 is live activity widgets. iOS 16 allows us to display the live state of ongoing activities from our apps on the lock screen or in the Dynamic Island of the new iPhone 14 Pro. This week we will learn how to build live activity widgets for our apps using the new ActivityKit framework.

#### Basics
First, we should be aware of the limitations of live activities in iOS 16. It will allow us to understand use cases where the live activity widget makes sense. Live activity has eight hours before the system ends it. I hope Apple will increase the lifetime of life activities by at least 24 hours. In this case, we can display things like long flights, etc.

Live activity widgets use WidgetKit to display information on the lock screen or in the Dynamic Island of the iPhone 14 Pro. We can share the code between home screen widgets and live activity widgets. The only difference is the lifecycle. Live activity widgets don't have a timeline provider and should be managed from the app target using the ActivityKit framework.

#### Data flow
Every live activity has two parts of data: static and dynamic. Static is not going to change and is available whenever an activity starts. The dynamic part is going to change during the life activity. In the case of a football match, teams are static data because it is not going to change during the game. The score is a dynamic part of the data and can change during the activity. The dynamic portion of the data should not exceed 4kb. Let's model our football match using the new ActivityKit framework.

```swift
import ActivityKit

struct FootballMatch: ActivityAttributes {
    typealias ContentState = Score

    struct Score: Codable, Hashable {
        let score1: Int
        let score2: Int
    }
    
    let team1: String
    let team2: String
}
```

The ActivityKit framework requires us to create a type conforming to the *ActivityAttributes* protocol. The type conforming to the *ActivityAttributes* should provide static information about the activity and define the *ContentState* type containing the dynamic part of the data. 

Your app must be in the foreground to start a live activity, and don't forget to add the **Supports Live Activities** key with **YES** value to the app target's *Info.plist*. Now you can start a live activity using the following code:

```swift
let activity = try Activity.request(
    attributes: FootballMatch(
        team1: "AJAX",
        team2: "PSV"
    ),
    contentState: .init(
        score1: 0,
        score2: 0
    )
)
```

As you can see in the example above, we request a live activity by providing the initial attributes. Now we can hold the result of the live activity request in the variable and update or end it whenever needed.

```swift
// Update live activity
Task {
    await activity.update(using: .init(score1: 1, score2: 0))
}

// End live activity
Task {
    await activity.end(
        using: .init(score1: 1, score2: 0),
        dismissalPolicy: .immediate
    )
}

```


You can use the BackgroundTasks framework or push notifications to update or end your live activity without running the app in the foreground.

#### Live activity presentation
We have to use the WidgetKit framework to display a live activity. WidgetKit provides us with the particular *ActivityConfiguration* type allowing us to define a live activity widget.

```swift
@available(iOSApplicationExtension 16.1, *)
struct FastingActivityWidget: Widget {
    var body: some WidgetConfiguration {
        ActivityConfiguration(for: FastingAttributes.self) { context in
            LiveActivityView(context: context)
                .padding(.horizontal)
        } dynamicIsland: { context in
            DynamicIsland {
                DynamicIslandExpandedRegion(.center) {
                    LiveActivityView(context: context)
                }
            } compactLeading: {
                Image(systemName: "circle")
                    .foregroundColor(.green)
            } compactTrailing: {
                Text(verbatim: context.state.progress.formatted(.percent)
                    .foregroundColor(context.state.stage?.color)
            } minimal: {
                Image(systemName: "circle")
                    .foregroundColor(.green)
            }
        }
    }
}
```

You should define the widget type using *ActivityConfiguration* and add it to your widget bundle. *ActivityConfiguration* requires us to provide a view that represents the activity on the lock screen and the Dynamic Island configuration to display activity on the iPhone 14 Pro. 

```swift
@main struct FastBotWidgetBundle: WidgetBundle {
    var body: some Widget {
        FastBotWidget()
        
        if #available(iOS 16.1, *) {
            FastingActivityWidget()
        }
    }
}

```

#### Conclusion
Today we learned how to build a live activity widget for iOS 16. In the next post, I will try to cover the Dynamic Island configuration type allowing us to layout our content in a few ways.
