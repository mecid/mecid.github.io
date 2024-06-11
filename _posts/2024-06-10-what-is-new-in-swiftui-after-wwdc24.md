---
title: What is new in SwiftUI after WWDC 24
layout: post
image: /public/wwdc24.webp
category: Meta
---

WWDC 24 is here, and we have a lot to cover. Every year, SwiftUI matures by introducing more features to catch up with UIKit. This year is no exception. Let's dive into the new features that the SwiftUI framework introduces.

{% include friends.html %}

The major change that I should mention first is the *@MainActor* isolation for *App*, *Scene* and *View* protocols. It might break your code, so keep it in mind. 

#### View collections
SwiftUI introduced the new overloads for *Group* and *ForEach* views, allowing us to create custom containers like *List* or *TabView*.

```swift
struct AppStoreView<Content: View>: View {
    @ViewBuilder var content: Content
    
    var body: some View {
        VStack {
            Group(subviewsOf: content) { subviews in
                HStack {
                    if !subviews.isEmpty {
                        subviews[0]
                    }
                    
                    if subviews.count > 1 {
                        subviews[1]
                    }
                }
                
                if subviews.count > 2 {
                    VStack {
                        subviews[2...]
                    }
                }
            }
        }
    }
}
```

As you can see in the example above, we use the *Group* view with the new initializer, which allows us to access the views of the content view passed via *@ViewBuilder* closure. SwiftUI introduces new *Subview* and *SubviewsCollection* types, providing proxy access to real views.

#### New tab bar experience
Using the new *Tab* type, the new customizable tab bar experience with fluid transition into a sidebar is available in SwiftUI.

```swift
enum Destination: Hashable {
    case home
    case search
    case settings
    case trends
}

struct RootView: View {
    @State private var selection: Destination = .home
    
    var body: some View {
        TabView {
            Tab("home", systemImage: "home", value: .home) {
                HomeView()
            }
            
            Tab("search", systemImage: "search", value: .search) {
                SearchView()
            }
            
            TabSection("Other") {
                Tab("trends", systemImage: "trends", value: .trends) {
                    TrendsView()
                }
                Tab("settings", systemImage: "settings", value: .settings) {
                    SettingsView()
                }
            }
            .tabViewStyle(.sidebarAdaptable)
        }
    }
}
```

As you can see in the example above, we use the new *Tab* type to define our tabs. We also use the *tabViewStyle* view modifier on the instance of the *TabSection* to group and move the particular section of tabs to the sidebar.

#### Hero animations
SwiftUI introduced *matchedTransitionSource* and *navigationTransition*, which we can use in pair on any instance of the *NavigationLink* type.

```swift
struct HeroAnimationView: View {
    @Namespace var hero
    
    var body: some View {
        NavigationStack {
            NavigationLink {
                DetailView()
                    .navigationTransition(.zoom(sourceID: "myId", in: hero))
            } label: {
                ThumbnailView()
            }
            .matchedTransitionSource(id: "myId", in: hero)
        }
    }
}
```

It enables us to create smooth transitions between views using the same identifier and namespace while navigating from one view to another inside *NavigationStack*.

#### Scroll position
The new *ScrollPosition* type, in pair with the *scrollPosition* view modifier, allows us to read the precise position of a *ScrollView* instance. We can also use it to programmatically scroll to the particular point of the scrolling content.

```swift
struct ScrollPositionExample: View {
    @State private var position: ScrollPosition = .init(point: .zero)
    
    var body: some View {
        ScrollView {
            ForEach(1..<1000) { item in
                Text(item.formatted())
            }
            
            Button("jump to top") {
                position = ScrollPosition(point: .zero)
            }
        }
        .scrollPosition($position)
    }
}
```

#### Entry macro
The new *Entry* macro allows us to quickly introduce environment values, focused values, etc, without boilerplate. Let's look at how we define environment values before the *Entry* macro.

```swift
struct ItemsPerPageKey: EnvironmentKey {
    static var defaultValue: Int = 10
}

extension EnvironmentValues {
    var itemsPerPage: Int {
        get { self[ItemsPerPageKey.self] }
        set { self[ItemsPerPageKey.self] = newValue }
    }
}
```

Now, we can minimize our code by using the *Entry* macro.

```swift
extension EnvironmentValues {
    @Entry var itemsPerPage: Int = 10
}
```

#### Previews
The new *Previewable* macro allows us to introduce the state to our previews without wrapping it into additional wrapper-view.

```swift
#Preview("toggle") {
    @Previewable @State var toggled = true
    return Toggle("Loud Noises", isOn: $toggled)
}
```

#### Others
The next iteration of the SwiftUI framework includes many new APIs, such as window pushing, text selection observation in the *TextField* and *TextEditor* views, search focus monitoring, custom text rendering, new *MeshGradient* type, and much more that I can't cover in a single post.

We will cover in details all the new features of the SwiftUI framework during upcoming weeks. So, stay tuned. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week!
