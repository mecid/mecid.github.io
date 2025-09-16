---
title: Feature flags in Swift
layout: post
image: /public/xcode-cloud.png
category: Architecture
---

Almost every project I work on has at least three build configurations: Debug, TestFlight, and App Store. These configurations differ not only in build settings but also in functionality. This week, we’ll learn how to implement feature flags in Swift, which allow us to toggle on and off specific functionalities under certain conditions.

{% include friends.html %}

As a big fan of trunk-based development, feature flags play a crucial role in my development approach. Almost every feature I’m working on recently has a feature flag enabling it in debug and TestFlight builds. While applying the trunk-based approach, I merge my branches even when the feature is not fully implemented, and that’s why I use feature flags to temporarily disable them.

By default, any Xcode project has two configurations: Debug and Release. You can create as many as you need of them, and I always make duplicates for Release configuration named AppStore and TestFlight. This allows you to create custom Xcode schemes running one of the available configurations. Then we can use compilation conditions in code to understand which scheme is active now.

Let’s start first by creating the *Distribution* enum defining our Xcode schemes.

```swift
public enum Distribution: Sendable {
    case debug
    case appstore
    case testflight
}

extension Distribution {
    static var current: Self {
        #if APPSTORE
        return .appstore
        #elseif TESTFLIGHT
        return .testflight
        #else
        return .debug
        #endif
    }
}
```

As you can see in the example above, the *Distribution* type is pretty simple. We define the static *current* property that switches over compilation conditions to find the active one. Now, we can move on to the *FeatureFlags* type that should define features I’m working on at the moment or some configuration flags.

```swift
public struct FeatureFlags: Sendable, Decodable {
    public let requirePaywall: Bool
    public let requireOnboarding: Bool
    public let featureX: Bool

    public init(distribution: Distribution) {
        switch distribution {
        case .debug:
            self.requirePaywall = true
            self.requireOnboarding = true
            self.featureX = true
        case .appstore:
            self.requirePaywall = true
            self.requireOnboarding = true
            self.featureX = false
        case .testflight:
            self.requirePaywall = false
            self.requireOnboarding = true
            self.featureX = true
        }
    }
}
```

The *FeatureFlags* type defines a bunch of properties that I switch on and off for different build types. As you can see, the *init* function takes an instance of the *Distribution* type, allowing me to pass the active distribution. 

While working in Xcode, I choose the debug configuration because it doesn’t contain too many compiler optimizations, builds fast and allows me to see what I’m working on. That’s why I turn on all the flags for debug condition.

I would like to give access to all features, even paid features to my TestFlight users, so I can be sure that everything working well. That’s why I disable paywall for TestFlight users.

```swift
extension EnvironmentValues {
    @Entry public var featureFlags = FeatureFlags(distribution: .debug)
}
```

The final step is to put an instance of the *FeatureFlags* type into the SwiftUI environment to share in the view hierarchy so my views cant disable or enable particular functionality.

```swift
@main
struct CardioBotApp: App {
    var body: some Scene {
        WindowGroup {
            RootView()
                .environment(\.featureFlags, FeatureFlags(distribution: .current))
        }
    }
}
```

Feature flags aren’t permanent. Treat them as temporary helpers - remove them when the feature is ready and proven. And if your project grows, consider making them remotely configurable to roll out or roll back features instantly. With this approach, you’ll keep your development process fast, safe, and ready for continuous delivery.

When paired with trunk-based development, they let you merge work early and often without worrying about breaking production. You can hide incomplete features behind flags, test them safely in Debug and TestFlight builds, and enable them only when you’re confident they’re ready. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week!
