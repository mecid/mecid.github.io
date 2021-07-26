---
title: Redux-like state container in SwiftUI. Container Views.
layout: post
category: Architecture
image: /public/store.png
---

This week I want to continue the topic of using a *Redux-like state container in SwiftUI*. I'm delighted with the new approach and already finished the refactoring of the [NapBot app](https://napbot.swiftwithmajid.com) in this way. That's why today I want to share with you how I use *Container Views* with a state container similar to *Redux*.

{% include friends.html %}

In previous weeks we already discussed the basics and some good practices while using *Redux-like state containers*. If you are not familiar with *Redux*, please take a look at those posts to understand how to build it in SwiftUI and which benefits you get by using it.

1. [Redux-like state container in SwiftUI. Basics](/2019/09/18/redux-like-state-container-in-swiftui/)
2. [Redux-like state container in SwiftUI. Best practices](/2019/09/25/redux-like-state-container-in-swiftui-part2/)

The container which holds the whole app's state as a single source of truth simplifies my codebase and eliminates a bunch of bugs that I had during manual sync between multiple states across the app screens. 

> To learn more about modeling app state using multiple store objects, take a look at my dedicated ["Modeling app state using Store objects in SwiftUI"](/2019/09/04/modeling-app-state-using-store-objects-in-swiftui/) post.

#### Container Views
Today, I want to touch another subject from my previous posts which plays very nice in conjunction with a *Redux-like state container*, and this is *Container Views*. *Container Views* help us to keep our SwiftUI views simple and responsible for only one job.

The main idea is dividing your views into two types: *Container Views and Rendering Views*. The *Rendering View* is responsible for drawing the content, and that's all. So basically it should not store the state or handle any lifecycle event. It usually renders the data which you pass via the init method.

*Container View*, on another hand, is responsible for handling data-flow and lifecycle events by providing the functions/closures to a *Rendering View*. Let's take a look at a simple example.

```swift
import SwiftUI

struct SearchContainerView: View {
    @EnvironmentObject var store: ReposStore
    @State private var query: String = "Swift"

    var body: some View {
        SearchView(query: $query, repos: store.repos, onCommit: fetch)
            .onAppear(perform: fetch)
    }

    private func fetch() {
        store.fetch(matching: query)
    }
}

struct SearchView: View {
    @Binding var query: String

    let repos: [Repo]
    let onCommit: () -> Void

    var body: some View {
        List {
            TextField("Type something", text: $query, onCommit: onCommit)
            ReposView(repos: repos)
        }
    }
}

struct ReposView: View {
    let repos: [Repo]

    var body: some View {
        ForEach(repos) { repo in
            HStack(alignment: .top) {
                VStack(alignment: .leading) {
                    Text(repo.name)
                        .font(.headline)
                    Text(repo.description ?? "")
                        .font(.subheadline)
                }
            }
        }
    }
}
```

In the example above, you can see how we build a connection between *Container and Rendering views*. *Container View* provides the data to *Rendering Views*. By doing this, we can easily reuse our *ReposView* anywhere across the app. *ReposView* doesn't have any dependency on some state or datastore and gets all the needed data via the init method.

> To learn more about *Container Views*, take a look at ["Introducing Container views in SwiftUI" post](/2019/07/31/introducing-container-views-in-swiftui/).

#### Using Container Views with Redux-like state container
During my transition from multiple stores to a single source of truth, I realize that *Container Views* play a significant role in this approach. I mainly use them for sending actions to the store and mapping the global app state to *Rendering View* properties. *Container Views* perfectly fit into my current app architecture. Let's take a look at the example.

```swift
import SwiftUI

struct SearchContainerView: View {
    @EnvironmentObject var store: AppStore
    @State private var query: String = "Swift"

    var body: some View {
        SearchView(
            query: $query,
            repos: store.state.search.result,
            onCommit: fetch
        ).onAppear(perform: fetch)
    }

    private func fetch() {
        store.send(SideEffect.search(query))
    }
}

struct SearchView: View {
    @Binding var query: String

    let repos: [Repo]
    let onCommit: () -> Void

    var body: some View {
        List {
            TextField("Type something", text: $query, onCommit: onCommit)
            ReposView(repos: repos)
        }
    }
}
```

As you can see in the example above, *Container View* helps us to keep *Rendering Views* small and independent.

#### Conclusion
Today we talked about the benefits of using *Container Views with Redux-like state containers*. I love this approach and use it as my main way to go with SwiftUI. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading and see you next week! 

1. [Redux-like state container in SwiftUI. Basics](/2019/09/18/redux-like-state-container-in-swiftui/)
2. [Redux-like state container in SwiftUI. Best practices](/2019/09/25/redux-like-state-container-in-swiftui-part2/)
3. Redux-like state container in SwiftUI. Container Views.
4. [Redux-like state container in SwiftUI. Connectors.](/2021/02/03/redux-like-state-container-in-swiftui-part4/)

#### References
The series of posts have built on a foundation of ideas started by other libraries, particularly Redux, Elm, and TCA.
0. [WWDC20 - Data Essentials in SwiftUI](https://developer.apple.com/videos/play/wwdc2020/10040/)
1. [Redux](https://redux.js.org) - The JavaScript library that popularized unidirectional data flow.
2. [The Elm Architecture](https://guide.elm-lang.org/architecture/) - A purely functional language and runtime that inspired the creation of Redux.
3. [The Composable Architecture](https://github.com/pointfreeco/swift-composable-architecture) - A library that bridges concepts from the Elm Architecture and Redux to Swift. It introduced the “environment” and “effect” patterns that this series covers.
