---
title: Mastering NavigationSplitView in SwiftUI
layout: post
category: Mastering SwiftUI views
image: /public/navigation.png
---

My final post in the new navigation APIs series in SwiftUI is about building two-three column apps. I have been waiting for all the betas to solve the critical issues with the brand-new *NavigationSplitView*, and it looks like it is almost ready to use. This week we will learn how to use and customize *NavigationSplitView* to build multi-column apps in SwiftUI.

{% include friends.html %}

#### Basics
The new iteration of the SwiftUI framework provides a brand new *NavigationSplitView* type, allowing us to build multi-column apps in SwiftUI quickly. The usage of the new type is straightforward.

```swift
// For two-column navigation
struct RootView: View {
    var body: some View {
        NavigationSplitView {
            // sidebar
        } detail: {
            // item details
        }
    }
}

// For three-column navigation
struct RootView: View {
    var body: some View {
        NavigationSplitView {
            // sidebar
        } content: {
            // content list
        } detail: {
            // item details
        }
    }
}
```

Usually, we should place the *NavigationSplitView* at the root of our scene. Two different options allow us to display two or three-column navigation. *NavigationSplitView* automatically wraps root views inside the sidebar, content, and detail columns into the *NavigationStack* view. You don't need to do it manually unless you need further navigation outside the content pane.

```swift
struct ContentView: View {
    @State private var selectedFolder: String?
    @State private var selectedItem: String?
    
    @State private var folders = [
        "All": [
            "Item1",
            "Item2"
        ],
        "Favorites": [
            "Item2"
        ]
    ]
    
    var body: some View {
        NavigationSplitView {
            List(selection: $selectedFolder) {
                ForEach(Array(folders.keys.sorted()), id: \.self) { folder in
                    NavigationLink(value: folder) {
                        Text(verbatim: folder)
                    }
                }
            }
            .navigationTitle("Sidebar")
        } content: {
            if let selectedFolder {
                List(selection: $selectedItem) {
                    ForEach(folders[selectedFolder, default: []], id: \.self) { item in
                        NavigationLink(value: item) {
                            Text(verbatim: item)
                        }
                    }
                }
                .navigationTitle(selectedFolder)
            } else {
                Text("Choose a folder from the sidebar")
            }
        } detail: {
            if let selectedItem {
                NavigationLink(value: selectedItem) {
                    Text(verbatim: selectedItem)
                        .navigationTitle(selectedItem)
                }
            } else {
                Text("Choose an item from the content")
            }
        }
    }
}
```

As you can see in the example above, we display the hierarchy of folders and items using the three-column navigation. We use value-based navigation links in conjunction with selection-based lists. SwiftUI automatically assigns the list selection to the value of a navigation link whenever the user presses it.

> To learn about the basics of the new data-driven Navigation API in SwiftUI, look at my ["Mastering NavigationStack in SwiftUI. Navigator Pattern."](/2022/06/15/mastering-navigationstack-in-swiftui-navigator-pattern/) post.

All the navigation links inside selection-based lists work unless you have a navigation link outside of the list, as we have in the content column. In this case, we should wrap the content view with the *NavigationStack* to provide additional destination points.

```swift
struct ContentView: View {
    @State private var selectedFolder: String?
    @State private var selectedItem: String?
    
    @State private var folders = [
        "All": [
            "Item1",
            "Item2"
        ],
        "Favorites": [
            "Item2"
        ]
    ]
    
    var body: some View {
        NavigationSplitView {
            List(selection: $selectedFolder) {
                ForEach(Array(folders.keys.sorted()), id: \.self) { folder in
                    NavigationLink(value: folder) {
                        Text(verbatim: folder)
                    }
                }
            }
            .navigationTitle("Sidebar")
        } content: {
            if let selectedFolder {
                List(selection: $selectedItem) {
                    ForEach(folders[selectedFolder, default: []], id: \.self) { item in
                        NavigationLink(value: item) {
                            Text(verbatim: item)
                        }
                    }
                }
                .navigationTitle(selectedFolder)
            } else {
                Text("Choose a folder from the sidebar")
            }
        } detail: {
            NavigationStack {
                ZStack {
                    if let selectedItem {
                        NavigationLink(value: selectedItem) {
                            Text(verbatim: selectedItem)
                                .navigationTitle(selectedItem)
                        }
                    } else {
                        Text("Choose an item from the content")
                    }
                }
                .navigationDestination(for: String.self) { text in
                    Text(verbatim: text)
                }
            }
        }
    }
}
```

#### Styling
*NavigationSplitView* allows us to control the visibility of columns by providing a binding for the *NavigationSplitViewVisibility* type. You can change it programmatically also. Here is an example of changing column visibility programmatically.

```swift
struct ContentView: View {
    @State private var visibility: NavigationSplitViewVisibility = .all
    @State private var selectedFolder: String?
    @State private var selectedItem: String?
    
    @State private var folders = [
        "All": [
            "Item1",
            "Item2"
        ],
        "Favorites": [
            "Item2"
        ]
    ]
    
    var body: some View {
        NavigationSplitView(columnVisibility: $visibility) {
            List(selection: $selectedFolder) {
                ForEach(Array(folders.keys.sorted()), id: \.self) { folder in
                    NavigationLink(value: folder) {
                        Text(verbatim: folder)
                    }
                }
            }
            .navigationTitle("Sidebar")
        } content: {
            if let selectedFolder {
                List(selection: $selectedItem) {
                    ForEach(folders[selectedFolder, default: []], id: \.self) { item in
                        NavigationLink(value: item) {
                            Text(verbatim: item)
                        }
                    }
                }
                .navigationTitle(selectedFolder)
            } else {
                Text("Choose a folder from the sidebar")
            }
        } detail: {
            NavigationStack {
                ZStack {
                    if let selectedItem {
                        NavigationLink(value: selectedItem) {
                            Text(verbatim: selectedItem)
                                .navigationTitle(selectedItem)
                                .toolbar {
                                    Button("Focus") {
                                        visibility = .detailOnly
                                    }
                                }
                        }
                    } else {
                        Text("Choose an item from the content")
                    }
                }
                .navigationDestination(for: String.self) { text in
                    Text(verbatim: text)
                }
            }
        }
    }
}
```

The *NavigationSplitViewVisibility* type provides a few options: *automatic, all, doubleColumn, detailOnly*. 

* *automatic* - Chooses the one from *all, doubleColumn, and detailOnly* depending on the device environment.
* *all* - Shows all the columns
* *doubleColumn* - Shows two trailing columns
* *detailOnly* - Shows only the details column by hiding the sidebar and content columns.

Another customization point of the *NavigationSplitView* is its style. You can set the style of the *NavigationSplitView* by using the *navigationSplitViewStyle* view modifier. It accepts a few options: *automatic, balanced, and prominentDetail*.

* *automatic* - Chooses the style depending on the device environment.
* *balanced* - Tries to show the first two columns by reducing the detail column size.
* *prominentDetail* - Focuses on presenting the detail column.

```swift
struct RootView: View {
    var body: some View {
        NavigationSplitView {
            // sidebar
        } content: {
            // content list
        } detail: {
            // item details
        }
        .navigationSplitViewStyle(.balanced)
    }
}
```

The last thing you can customize is the width of the particular column in the *NavigationSplitView* by using the *navigationSplitViewColumnWidth* view modifier on the root view inside the column.

```swift
struct RootView: View {
    var body: some View {
        NavigationSplitView {
            // sidebar
        } content: {
            ZStack {
            
            }
            .navigationSplitViewColumnWidth(300)
        } detail: {
            // item details
        }
        .navigationSplitViewStyle(.balanced)
    }
}
```

#### Conclusion
Today we learned how to build two or three-column navigation in SwiftUI using the brand-new *NavigationSplitView*. I think it is much easier than before or at least works better now. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week!

