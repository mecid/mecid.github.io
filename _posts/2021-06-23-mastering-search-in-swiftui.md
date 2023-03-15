---
title: Mastering search in SwiftUI
layout: post
image: /public/search.png
category: Mastering SwiftUI views
---

SwiftUI Release 3.0 brought tons of expected features that we missed in previous iterations. One of them is the ability to provide the search feature in our apps. Fortunately, we have a new *searchable* view modifier. This week, we will learn about the new *searchable* modifier and how to build a great search experience using it.

{% include friends.html %}

#### Basics
We can mark the content of our view as searchable using the new view modifier. SwiftUI understands the structure of your app and displays the search bar in the appropriate place. Let's take a look at the quick example.

```swift
struct RootView: View {
    @State private var query: String = ""

    var body: some View {
        NavigationView {
            Master()
            Details()
        }
        .searchable(text: $query, prompt: Text("Search"))
    }
}
```

![search](/public/search.png)

We attach the *searchable* view modifier to the *NavigationView* at the root of our app. SwiftUI can put the search bar in different places depending on the environment. For example, it will put a search bar in the *Master* view on iOS and iPadOS. On macOS, SwiftUI places the search bar in the toolbar of the trailing column of the *NavigationView*.

```swift
struct RootView: View {
    @State private var query: String = ""

    var body: some View {
        NavigationView {
            Master()
            Details()
        }
        .searchable(
            text: $query,
            placement: .toolbar,
            prompt: "Type something..."
        )
    }
}
```

You can also suggest placement for the search bar using the *placement* parameter, but keep in mind that SwiftUI can ignore it. A few possible placements for the search bar are *automatic*, which is the default one, *sidebar, toolbar, and navigationBarDrawer.*

```swift
struct ContentView: View {
    @StateObject private var viewModel = SearchViewModel()
    @State private var query = ""

    var body: some View {
        NavigationView {
            List(viewModel.repos) { repo in
                VStack(alignment: .leading) {
                    Text(repo.name)
                        .font(.headline)
                    Text(repo.description ?? "")
                        .foregroundColor(.secondary)
                }
            }
            .navigationTitle("Search")
            .searchable(text: $query)
            .onChange(of: query) { newQuery in
                Task { await viewModel.search(matching: query) }
            }
        }
    }
}
```

Whenever we use the *searchable* modifier, we should provide a binding to a string value. We can observe changes and filter our content depending on the query term using that binding.

#### Environment
SwiftUI provides us *isSearching* environment value that indicated whether the user is currently interacting with the search bar that has been placed by a surrounding *searchable* modifier. We can use this value to understand whether to show quick search results. 

```swift
struct StarredReposList: View {
    @StateObject var viewModel = SearchViewModel()
    @Environment(\.dismissSearch) var dismissSearch
    @Environment(\.isSearching) var isSearching

    let query: String

    var body: some View {
        List(viewModel.repos) { repo in
            RepoView(repo: repo)
        }
        .overlay {
            if isSearching && !query.isEmpty {
                VStack {
                    Button("Dismiss search") {
                        dismissSearch()
                    }
                    SearchResultView(query: query)
                        .environmentObject(viewModel)
                }
            }
        }
    }
}
```

Another search-related environment value we have is *dismissSearch*. *dismissSearch* asks the system to dismiss the current search interaction. Remember that both environment values work only in the views surrounded by the *searchable* modifier.  

> To learn more about environment in SwiftUI, take a look at my ["The power of Environment in SwiftUI"](/2019/08/21/the-power-of-environment-in-swiftui/) post.

#### Suggestions 
Suggestions are a vital part of the excellent search experience, and SwiftUI gives us a very nice API that we can use to provide search suggestions to our users. The *searchSuggestions* view modifier allows us to pass a *@ViewBuilder* closure displaying search suggestions. Let's see how we can use it.

```swift
struct ContentView: View {
    @StateObject private var viewModel = SearchViewModel()
    @State private var query = ""

    let suggestions: [String] = [
        "Swift", "SwiftUI", "Obj-C"
    ]

    var body: some View {
        NavigationView {
            List(viewModel.repos) { repo in
                VStack(alignment: .leading) {
                    Text(repo.name)
                        .font(.headline)
                    Text(repo.description ?? "")
                        .foregroundColor(.secondary)
                }
            }
            .navigationTitle("Search")
            .searchable(text: $query)
            .searchSuggestions {
                ForEach(suggestions, id: \.self) { suggestion in
                    Text(suggestion)
                        .searchCompletion(suggestion)
                }
            }
            .onChange(of: query) { newQuery in
                Task { await viewModel.search(matching: query) }
            }
        }
    }
}
```

![search-suggestions](/public/search1.png)

As you can see in the example above, we use the *searchSuggestions* view modifier and provide a *@ViewBuilder* closure with *ForEach* view that iterates over an array of suggestions. We also use the *searchCompletion* view modifier to wrap every text view in a button that assigns its value to the search query binding.

Keep in mind that *searchCompletion* modifier wraps its content in a *Button*. It means you should apply it to the view that doesn't have user interaction like *Text* or *Label*.

#### Scopes
The SwiftUI framework provides us another view modifier allowing us to improve search experience. We can use the *searchScopes* view modifier to provide search scopes displayed in the segmented control under the search bar.

```swift
struct ContentView: View {
    @State private var query = ""
    @State private var scope: Scope = .local
    
    var body: some View {
        NavigationStack {
            List {
                // ...
            }
            .searchable(text: $query)
            .searchScopes($scope) {
                ForEach(Scope.allCases, id: \.self) { scope in
                    Text(scope.rawValue)
                }
            }
        }
    }
}
```

#### Tokens
Another version of the *searchable* view modifier allows us to display tokens in the search bar. 

```swift
struct Token: Identifiable {
    let id = UUID()
    let value: String
}
    
struct ContentView: View {
    @State private var query = ""
    @State private var tokens: [Token] = []
    @State private var suggestedTokens: [Token] = [
        .init(value: "Token1"),
        .init(value: "Token2"),
        .init(value: "Token3"),
        .init(value: "Token4")
    ]
    
    var body: some View {
        NavigationStack {
            List {
                // ...
            }
            .searchable(
                text: $query,
                tokens: $tokens,
                suggestedTokens: $suggestedTokens
            ) { token in
                Text(verbatim: token.value)
            }
            .navigationTitle("Search")
        }
    }
}
```

In the example above, we use the *searchable* view modifier and pass two bindings for suggested tokens and selected tokens. The SwiftUI displays suggested tokens and move them into the search bar as soon as user taps one of them. It also moves the token from the suggested collection to the selected collection of tokens.

```swift
struct ContentView: View {
    @State private var query = ""
    @State private var tokens: [Token] = []
    
    var body: some View {
        NavigationStack {
            List {
                // ...
            }
            .searchable(
                text: $query,
                tokens: $tokens
            ) { token in
                Text(verbatim: token.value)
            }
            .navigationTitle("Search")
            .onChange(of: query) { newValue in
                if newValue.contains("new") {
                    tokens.append(.init(value: "NEW"))
                }
            }
        }
    }
}
```

Another variation of the *searchable* view modifier allows us to manually filter the query and populate tokens.

#### Conclusion
Today we learned how to build a great search experience using the brand new *searchable* view modifier. It is incredible how easy you can make things like suggestions, the platform adopted placement using the only *searchable* view modifier. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week!
