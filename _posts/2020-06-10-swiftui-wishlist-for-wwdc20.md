---
title: SwiftUI wishlist for WWDC20
layout: post
image: /public/swiftui.png
category: Meta
---

We already started collecting our questions for Apple engineers. On the other hand, I decided to share with you my SwiftUI wishlist for WWDC20. This week we will talk about possible additions and changes in SwiftUI. I will show you also *API* that I expect to see during the next release of SwiftUI.

{% include friends.html %}

#### ScrollView
*ScrollView* has a bunch of bugs right now. I believe Apple will fix all of them, but I also hope for a content offset binding option. There is no way to get or set the current content offset of *ScrollView* in SwiftUI. We need a way to both assign and read the offset, and it is a perfect use-case for a *Binding*. Let's take a look at the quick example.

```swift
struct RootView: View {
    @State private var offset: CGPoint = .zero

    var body: some View {
        ScrollView(.vertical, showsIndicators: false, offset: $offset) {
            Text("Very long text")
            Button("Jump to top") {
                self.offset = .zero
            }
        }
    }
}
```

#### CollectionView and CompositionalLayout
Last year Apple released *CompositionLayout* that provides us a declarative way of building complex layouts for *UICollectionView*. The buzzword here is **declarative**. It feels very natural for SwiftUI to have a similar *API*, I'm not sure why Apple didn't release it last year, but I expect to see it soon. 

*CompositionalLayout* introduces a few concepts to manage complex layouts. For example, it has sections, groups, and items. As you know, we already have these concepts in SwiftUI. I think SwiftUI views like *Group* and *Section* can behave in another way depending on the context. We already saw that views can behave differently inside a *Form* or a *List*. Let's take a look at how it might work in SwiftUI.

```swift
struct AppStoreView: View {
    let featured: [App]
    let appsWeLove: [App]

    var body: some View {
        CompositionalLayout {
            Section(.groupPagingCentered) {
                Group(.horizontal, width: .fractionalWidth(0.5), height: .fractionalHeight(0.5)) {
                    ForEach(featured) { app in
                        FeatureAppView(app: app)
                    }
                }
            }

            Section {
                Group(.vertical, width: .fractionalWidth(0.9), height: .estimated(200)) {
                    ForEach(appsWeLove) { app in
                        SmallAppView(app: app)
                    }
                }
            }
        }
    }
}
```

#### Navigation
As I already said multiple times, *Navigation* in SwiftUI is really problematic. The main rule behind the SwiftUI framework is *"view is a function of some state"*. SwiftUI doesn't apply this rule to *Navigation*. *Navigation* in SwiftUI looks like uncontrollable magic. In my opinion, it also should be a function of a state where the state describes a navigation stack. Let's take a look at the example.

```swift
enum Destination {
    case master
    case details(Post)
}

struct RootView: View {
    @EnvironmentObject var router: Router<Destination>

    var body: some View {
        RouterView(router: router) { destination in
            switch destination {
            case .master:
                return PostsView()
            case .details(let post):
                return PostDetails(post: post)
            }
        }
    }
}
```

As you can see, *RouterView* accepts an instance of the *Router* class, which describes a navigation stack and a *ViewBuilder* closure that we use to build a view according to the navigation state.

> To learn more about *@ViewBuilder* in SwiftUI, take a look at my ["The power of @ViewBuilder in SwiftUI"](/2019/12/18/the-power-of-viewbuilder-in-swiftui/) post.

#### Appearance API
SwiftUI provides us styling options by introducing a bunch of protocols. For example, *ButtonStyle*, *ToggleStyle*, etc. We create a struct that conforms to the *ButtonStyle* protocol. After that, we can put it into the environment, and all views inside the environment gain particular styling. I appreciate the way styling works in SwiftUI right now, but we need to expand it to support more views like Form and *NavigationView*.

> To learn more about styling in SwiftUI, take a look at my ["Composable styling in SwiftUI"](/2019/08/28/composable-styling-in-swiftui/) post.

#### Missing views
SwiftUI provides us a set of very basic views that we can use to build our apps. There are a lot of views that we don't have in SwiftUI. I hope to see a bunch of new views coming in the next weeks. For example, *SearchBar*, *TextView*, *RefreshControl*, etc.

#### Conclusion
This week, I share my vision and expectations with you about the future of the SwiftUI framework that we hopefully will see later this month. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week!