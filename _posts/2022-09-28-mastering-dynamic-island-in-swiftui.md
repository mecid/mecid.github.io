---
title: Mastering Dynamic Island in SwiftUI
layout: post
category: Mastering SwiftUI views
image: /public/dynamic-island-expanded-wide.png
---

In the previous post, we talked about live activity widgets displaying your app's ongoing events. Live activity widgets can utilize the dynamic island of the iPhone 14 Pro. In this post, we will discuss possible configurations and customization points of the dynamic island feature using the new API available in the WidgetKit framework.

{% include friends.html %}

The WidgetKit framework provides us with a particular type of configuration called *ActivityConfiguration*, allowing us to define a live activity widget.

```swift
@available(iOSApplicationExtension 16.1, *)
struct FastingActivityWidget: Widget {
    var body: some WidgetConfiguration {
        ActivityConfiguration(for: FastingAttributes.self) { context in
            LiveActivityView(context: context)
                .padding(.horizontal)
        } dynamicIsland: { context in
            DynamicIsland {
                DynamicIslandExpandedRegion(.leading) {
                    LiveActivityView(context: context)
                }
            } compactLeading: {
                Image(systemName: "circle")
                    .foregroundColor(.green)
            } compactTrailing: {
                Text(verbatim: context.state.progress.formatted(.percent.precision(.fractionLength(0))))
                    .foregroundColor(context.state.stage?.color)
            } minimal: {
                Image(systemName: "circle")
                    .foregroundColor(.green)
            }
        }
    }
}
```

We use the *ActivityConfiguration* type to define a SwiftUI view to display on the lock screen. In this case, we will use our custom *LiveActivityView*. We also provide a dynamic island layout configuration to display on iPhone 14 Pro.

> To learn about the basics of live activity widgets, take a look at my dedicated ["Displaying live activities in iOS 16"](/2022/09/21/displaying-live-activities-in-ios16/) post.

WidgetKit has a particular *DynamicIsland* type allowing us to specify how we want to use the dynamic island layout. The *DynamicIsland* type is not a SwiftUI view but requires us to provide views for compact leading and compact trailing, expanded, and minimal cases.

Whenever your app is the only app running live activity widget at the moment, the system displays both compact leading and compact trailing views accordingly. 

![dynamic-island-compact](/public/dynamic-island-compact.png)

The system uses minimal view whenever there is more than one live activity widget at the moment and displays it as detached trailing view.

![dynamic-island-minimal](/public/dynamic-island-minimal.png)

 The expanded view is used when the user uses a long press gesture to expand the dynamic island.

![dynamic-island-expanded](/public/dynamic-island-expanded.png)

The expanded dynamic island divides its space into four areas and allows us to control where we want to place the content. We also can use the *priority* parameter to enable the system to prioritize the views while sizing them.

```swift
@available(iOSApplicationExtension 16.1, *)
struct FastingActivityWidget: Widget {
    var body: some WidgetConfiguration {
        ActivityConfiguration(for: FastingAttributes.self) { context in
            LiveActivityView(context: context)
        } dynamicIsland: { context in
            DynamicIsland {
                DynamicIslandExpandedRegion(.leading, priority: 1) {
                    LiveActivityView(context: context)
                }
                
                DynamicIslandExpandedRegion(.trailing) {
                    LiveActivityView(context: context)
                }
                
                DynamicIslandExpandedRegion(.center) {
                    LiveActivityView(context: context)
                }
                
                DynamicIslandExpandedRegion(.bottom) {
                    LiveActivityView(context: context)
                }
            } compactLeading: {
                Image(systemName: "circle")
                    .foregroundColor(.green)
            } compactTrailing: {
                Text(verbatim: context.state.progress.formatted(.percent.precision(.fractionLength(0))))
                    .foregroundColor(context.state.stage?.color)
            } minimal: {
                Image(systemName: "circle")
                    .foregroundColor(.green)
            }
        }
    }
}
```

Sometimes we might have too wide content in the leading view that doesn't fit into the available space. In this case, we can use the *dynamicIsland* view modifier to move the leading view below the True Depth camera.

![dynamic-island-expanded-wide](/public/dynamic-island-expanded-wide.png)

```swift
@available(iOSApplicationExtension 16.1, *)
struct FastingActivityWidget: Widget {
    var body: some WidgetConfiguration {
        ActivityConfiguration(for: FastingAttributes.self) { context in
            LiveActivityView(context: context)
        } dynamicIsland: { context in
            DynamicIsland {
                DynamicIslandExpandedRegion(.leading, priority: 1) {
                    LiveActivityView(context: context)
                        .dynamicIsland(verticalPlacement: .belowIfTooWide)
                }
            } compactLeading: {
                Image(systemName: "circle")
                    .foregroundColor(.green)
            } compactTrailing: {
                Text(verbatim: context.state.progress.formatted(.percent.precision(.fractionLength(0))))
                    .foregroundColor(context.state.stage?.color)
            } minimal: {
                Image(systemName: "circle")
                    .foregroundColor(.green)
            }
        }
    }
}
```

Another customization point is the background color of compact and minimal views. By default, the system uses black color to fill the compact and minimal views, but we can use the *keylineTint* view modifier to change the color.

```swift
@available(iOSApplicationExtension 16.1, *)
struct FastingActivityWidget: Widget {
    var body: some WidgetConfiguration {
        ActivityConfiguration(for: FastingAttributes.self) { context in
            LiveActivityView(context: context)
        } dynamicIsland: { context in
            DynamicIsland {
                DynamicIslandExpandedRegion(.leading, priority: 1) {
                    LiveActivityView(context: context)
                        .dynamicIsland(verticalPlacement: .belowIfTooWide)
                }
            } compactLeading: {
                Image(systemName: "circle")
                    .foregroundColor(.green)
            } compactTrailing: {
                Text(verbatim: context.state.progress.formatted(.percent.precision(.fractionLength(0))))
                    .foregroundColor(context.state.stage?.color)
            } minimal: {
                Image(systemName: "circle")
                    .foregroundColor(.green)
            }
            .keylineTint(.white)
        }
    }
}
```

Today we learned how to use the dynamic island feature to display live activities from your app on iPhone 14 Pro. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this article. Thanks for reading, and see you next week!

