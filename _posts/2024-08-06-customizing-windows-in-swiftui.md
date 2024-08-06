---
title: Customizing windows in SwiftUI
layout: post
image: /public/visionOS.webp
category: View Composition
---

SwiftUI has become the leading framework for building apps on all Apple platforms. Almost half of these platforms support multiple windows, so we see more APIs allowing us to manipulate windows. This week, we will learn how to customize windows in SwiftUI using new APIs.

{% include friends.html %}

#### Size
The SwiftUI framework provides the *defaultSize* view modifier, which allows us to set the default size of the window.

```swift
struct SugarBotApp: App {
    var body: some Scene {
        WindowGroup {
            RootView()
        }
        
        WindowGroup(id: "search") {
            SearchFeatureView()
        }
        .defaultSize(width: 500, height: 500)
    }
}
```

> To learn more about the basics of window management in SwiftUI, take a look at my ["Customizing windows in SwiftUI"](/2024/08/06/customizing-windows-in-swiftui/) post.

As you can see in the example above, we use the *defaultSize* view modifier to set the default window size to 300x500 points. Most of the Apple platforms support the window resizability feature. The SwiftUI framework introduces the *windowResizability* view modifier to control how users can resize the window.

```swift
struct SugarBotApp: App {
    var body: some Scene {
        WindowGroup {
            RootView()
        }
        
        WindowGroup(id: "search") {
            SearchFeatureView()
        }
        .defaultSize(width: 500, height: 800)
        .windowResizability(.contentSize)
    }
}
```

In the example above, we set the resizability option to the content size, which prevents the user from changing the window's size to less than the minimal content size and more than the maximal content size. 

```swift
struct SugarBotApp: App {
    var body: some Scene {
        WindowGroup {
            RootView()
        }
        
        WindowGroup(id: "search") {
            SearchFeatureView()
        }
        .defaultSize(width: 500, height: 800)
        .windowResizability(.contentMinSize)
    }
}
```

Another option is the *contentMinSize* case, which prevents the user from changing the window size to less than its content but doesn't control the maximal size of the window.

#### Placement
Whenever you use the *openWindow* environment value to open a window, it appears in front of the last opened window. However, you can control the window placement using the *defaultWindowPlacement* view modifier. 

```swift
struct SugarBotApp: App {
    var body: some Scene {
        WindowGroup {
            RootView()
        }
        
        WindowGroup(id: "search") {
            SearchFeatureView()
        }
        .defaultWindowPlacement { content, context in
            #if os(visionOS)
            if context.windows.last?.id == "search" {
                return WindowPlacement(.trailing(context.windows.last!))
            } else {
                return WindowPlacement(.utilityPanel)
            }
            #else
            // ...
            #endif
        }
    }
}
```

As you can see in the example above, we use the *defaultWindowPlacement* view modifier to tune the placement. The *defaultWindowPlacement* view modifier takes the closure and returns an instance of the *WindowPlacement* type. The *WindowPlacement* type defines the *Position* type, allowing us to control which edge to place a window on.

> To learn more about windows on visionOS, take a look at my ["Introducing SwiftUI on visionOS"](/2024/01/23/introducing-swiftui-on-visionOS/) post.

The SwiftUI framework defines the *utilityPanel* position on the visionOS, which displays the window slightly below the presenting window. We also got the identifier of the last presented window to display it on the trailing edge whenever it is a search window.

```swift
struct SugarBotApp: App {
    var body: some Scene {
        WindowGroup {
            RootView()
        }
        
        WindowGroup(id: "search") {
            SearchFeatureView()
        }
        .defaultSize(width: 500, height: 800)
        .windowResizability(.contentSize)
        .defaultWindowPlacement { content, context in
            #if os(visionOS)
            // ...
            #else
            let size = content.sizeThatFits(.unspecified)
            let positionX = context.defaultDisplay.bounds.midX - (size.width / 2)
            let positionY = context.defaultDisplay.bounds.maxY - size.height
            let position = CGPoint(x: positionX, y: positionY)
            return WindowPlacement(position, size: size)
            #endif
        }
    }
}
```

On macOS, we don't have access to the *utilityPanel* position or other factory methods like the ones above and below. However, we can access the display property on the *context* parameter of the closure and retrieve information about the current display. We also use the *content* parameter to measure the window's content size and calculate the precise position of the window on the screen.

#### Bonus tip
The last but not least minor thing I want to talk about is the *WindowDragGesture* type. This is the gesture type that allows us to react to a window dragging event.

```swift
struct SomeView: View {
    @GestureState private var isWindowDragging = false
    
    var body: some View {
        Text("Some text here")
            .redacted(reason: isWindowDragging ? .placeholder : [])
            .gesture(
                WindowDragGesture()
                    .updating($isWindowDragging) { _, state, _ in
                        state = true
                    }
            )
    }
}
```

As you can see in the example above, we use the instance of the *WindowDragGesture* type to handle the window dragging event and redact the text while the user moves the window. 

Today we learned how to customize windows on iPadOS, macOS and visionOS. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week!
