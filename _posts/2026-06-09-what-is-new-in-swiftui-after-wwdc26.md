---
title: What is new in SwiftUI after WWDC26
layout: post
category: Meta
image: /public/wwdc26.png
---

Platforms State of the Union has just been published, and we have a lot of new APIs to learn, explore, and use to build new features and apps. Let’s start with the most important framework for our apps. This week, we will look at what WWDC26 brings to the new iteration of SwiftUI.

{% include friends.html %}

Liquid Glass design gained a few changes here and there, and fortunately, SwiftUI adopts many of them automatically. You don’t need to write additional code for things like glass tint, as they are automatically applied to your app’s user interface.

SwiftUI also introduces a new *prominent* tab role. You can use the *prominent* role for trailing-separated tabs, similar to search.

```swift
TabView {
    Tab("insights", systemImage: "chart.xyaxis.line") {
        NavigationStack {
            InsightsFeatureView()
                .navigationTitle("insights")
        }
    }
    
    Tab("awareness", systemImage: "text.book.closed") {
        NavigationStack {
            AwarenessView()
                .navigationTitle("awareness")
        }
    }
    
    Tab("create", systemImage: "pencil", role: .prominent) {
        NavigationStack {
            CreateFeatureView()
                .navigationTitle("create")
        }
    }
}
```

You can finally use the *swipeActions* view modifier with any view container, including *List*, *ScrollView*, lazy stacks, and even custom layouts. There is no need to use *List* only to support swipe actions anymore. All you need is the new *swipeActionsContainer* view modifier.

```swift
ScrollView {
    LazyVStack {
        ForEach(items) { item in
            ItemRow(item)
                .swipeActions {
                    Button("Delete", role: .destructive) {
                        delete(item)
                    }
                }
        }
    }
}
.swipeActionsContainer()
```

Reordering via drag and drop is easier than before with the new *reorderContainer* view modifier. It also works with *List*, *ScrollView*, lazy stacks, and custom layouts. Just apply the *reorderable* view modifier and handle the reordering action.

```swift
struct ContentView: View {
    @State private var landmarks: [Landmark] = []


    var body: some View {
        VStack {
            ForEach(landmarks) { landmark in
                LandmarkView(landmark)
            }
            .reorderable()
        }
        .reorderContainer(for: Landmark.self) { difference in
            difference.apply(to: &landmarks)
        }
    }
}
```

Navigation gains a new cross-fade transition that we can use alongside zoom. We still can’t control navigation transitions manually, but now we have more built-in options: automatic, zoom, and cross-fade.

```swift
struct ContentView: View {
    @State private var showSheet = false


    var body: some View {
        VStack {
            Button("Show Sheet") {
                showSheet = true
            }
            .sheet(isPresented: $showSheet) {
                Text("Sheet Content")
                    .presentationDetents([.medium])
                    .navigationTransition(.crossFade)
            }
        }
    }
}
```

*AsyncImage* also receives a performance improvement by introducing caching. You can even control the cache by configuring a custom *URLSession* with a specific cache size.

```swift
 let customCache = URLCache(
    memoryCapacity: 20 * 1024 * 1024,
    diskCapacity: 100 * 1024 * 1024,
    directory: nil // Uses default system cache directory container
)

let configuration = URLSessionConfiguration.default
configuration.urlCache = customCache
let session = URLSession(configuration: configuration)

AsyncImage(
    request: URLRequest(
        url: imageURL,
        cachePolicy: .reloadIgnoringLocalCacheData
    )
)
.asyncImageURLSession(session)
```

Apple introduced a collection of new toolbar view modifiers that allow us to control toolbar visibility more precisely. You can choose which toolbar items should have higher visibility priority on smaller devices, hide secondary items in the overflow menu, and pin an item to the trailing position of the top bar.

```swift
struct RootView: View {
    var body: some View {
        ContentView()
            .toolbar {
                ToolbarItem {
                    SecondaryControl()
                }
                
                ToolbarItem {
                    PrimaryControl()
                }
                .visibilityPriority(.high)
                
                ToolbarOverflowMenu {
                    Button("Action 1") { }
                    Button("Action 2") { }
                }
            }
    }
}
```

Document-based apps get a refreshed look and feel, along with a performance boost. It looks like Xcode has started using these improvements, which might explain why we see so much work around document-based apps this year. And finally, Xcode introduces SwiftUI Specialist and What’s New in SwiftUI skills for agentic coding in Xcode.

SwiftUI continues to move toward a more flexible and system-integrated framework. This year’s updates may not completely change how we build apps, but they remove many small limitations around containers, navigation, toolbars, and document-based workflows.

 These improvements make SwiftUI feel more mature and give us more reasons to rely on it for production apps. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week!
