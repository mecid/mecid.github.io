---
title: Introducing Container views in SwiftUI
layout: post
---

During app development using *SwiftUI*, you can see that your views are very coupled with the data flow. Views fetch and render the data, handle user input and actions, etc. By doing so many things views become very fat and we can't reuse them across the app. Let's take a look at a different way of decomposing views by using *Container* views.

In my first post about *SwiftUI*, we build a Github app.

```swift
import SwiftUI
import Combine

struct FavoritesView : View {
    @EnvironmentObject var store: ReposStore

    var body: some View {
        NavigationView {
            List {
                ForEach(store.repos) { repo in
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
            .navigationBarTitle(Text("Favorites"))
            .onAppear(perform: fetch)
        }
    }

    private func fetch() {
        store.fetchFavorites()
    }
}
```

Here we have a simple view which fetches and renders my starred repositories. It looks very straightforward, but there is a massive problem. FavoritesView mixes data fetching plus rendering, and because of that, we can't reuse this view. For example, I want to use it to render a user's repositories or repos search result. To make it possible, let's start our refactoring process.

#### Composition
As you can see, *SwiftUI* uses mainly value types instead of classes and built on top *Composition over Inheritance* principle. Let's follow this way by decomposing our *FavoritesView* into few small composable views.

```swift
import SwiftUI

struct ReposView : View {
    let repos: [Repo]

    var body: some View {
        List {
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
}
```

Now we have a simple *ReposView*, which accepts an array of repos and render them. That's it. We can use it anywhere across the app where we need to display a repos list.

#### Introducing Container views
But now we have another question, where we can do data-flow stuff like fetching data and handling user actions. Let's introduce *Container* view concept. *Container* view fetches data and passes it to a simple *Rendering* view. *Container* view didn't render any User Interface itself. It just passes the data to the *Rendering* view.

```swift
import SwiftUI

struct FavoritesContainerView: View {
    @EnvironmentObject var store: ReposStore

    var body: some View {
        ReposView(repos: store.repos)
            .onAppear(perform: fetch)
    }

    private func fetch() {
        store.fetchFavorites()
    }
}
```

In the example above, we have a FavoritesContainerView which handles the data fetching and passes repos array to ReposView. By doing this, we have a clear separation between our data-flow and data rendering. Let's take a look at a more complicated example.

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

struct SearchView : View {
    @Binding var query: String
    
    let repos: [Repo]
    let onCommit: () -> Void

    var body: some View {
        List {
            TextField("Type something", text: $query, onCommit: onCommit)
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
}
```

Here we have a more complex example, where Container view provides an acton handling closure and state binding to Rendering view. Let's summarize our thoughts about Container and Rendering views in SwiftUI.

Container views should do things only related to data-flow:
1. Store the state of the view
2. Handle life cycle (onAppear/onDisappear)
3. Fetch data using ObservableObject
4. Provide action handlers to the Rendering view

Rendering views should do things only related to rendering:
1. Build User Interface using primitive components provided by SwiftUI.
2. Compose User Interface by using other Rendering views.
3. Use data as input to render User Interface and don't store any state.

#### Conclusion
Today we discussed a way of decomposing your complex view into small and reusable pieces. I try to follow this approach as much as possible to make a clean separation between data-flow and displaying data. Try to use this method and share with me your thoughts about it. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading and see you next week!  

