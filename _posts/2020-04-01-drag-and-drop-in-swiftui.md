---
title: Drag and drop in SwiftUI
layout: post
image: /public/draganddrop.png
category: Interactions
---

Another *iPadOS* feature released in SwiftUI with Xcode 11.4 was the drag and drop. Finally, we can use drag and drop API not only with *UIKit* but also with SwiftUI. This week we will learn all about drag and drop interactions in SwiftUI. 

{% include friends.html %}

#### Basics
Drag and drop interactions allow users to send data between two apps or two scenes of the same app. For example, you can run Safari and Notes apps side-by-side and drag the links from Safari to Notes app. As part of the post, we will build a bookmarking app that uses drag and drop to save or open links stored in the app.

#### Drag
SwiftUI provides *onDrag* modifier that allows us to register a closure that will create and return *NSItemProvider*. *NSItemProvider* is the class that describes the content and the type of draggable item. Let's take a look at how we can use *onDrag* modifier.

```swift
struct BookmarksList: View {
    @State private var links: [URL] = [
        URL(string: "https://twitter.com/mecid")!
    ]

    var body: some View {
        List {
            ForEach(links, id: \.self) { url in
                Text(url.absoluteString)
                    .onDrag { NSItemProvider(object: url as NSURL) }
            }
        }
        .navigationBarTitle("Bookmarks")
    }
}
```

As you can see in the example above, all we need to do is to use *onDrag* modifier that returns an instance of *NSItemProvider*. Now you can run the app on the iPad simulator side-by-side with Safari and try to drag and drop the link from our app to Safari.

#### Drop
Unlike drag operation, drop interaction is a little bit complicated. SwiftUI gave us three overloads of *onDrop* modifier. The most interesting one is the overload that accepts *DropDelegate*. *DropDelegate* is the protocol that we can implement to control drop interaction. Let's take a look at the quick example.

```swift
struct BookmarksDropDelegate: DropDelegate {
    @Binding var bookmarks: [URL]

    func performDrop(info: DropInfo) -> Bool {
        guard info.hasItemsConforming(to: ["public.url"]) else {
            return false
        }

        let items = info.itemProviders(for: ["public.url"])
        for item in items {
            _ = item.loadObject(ofClass: URL.self) { url, _ in
                if let url = url {
                    DispatchQueue.main.async {
                        self.bookmarks.insert(url, at: 0)
                    }
                }
            }
        }

        return true
    }
}
```

Here we have a *BookmarksDropDelegate* struct that conforms to *DropDelegate* protocol. *DropDelegate* protocol requires us to implement *performDrop* function. This function should return *true* whenever the drop succeeded or *false* if it failed. *performDrop* function has *DropInfo* parameter, which provides us the information about items that should be dropped.

Using *DropInfo*, we can filter items and accept only links. Apple uses *Uniform Type Identifiers* to identify the type of data. We want to receive only URLs that's why we use **public.link** identifier. *DropInfo* also provides us the location of drop using *CGPoint* instance.

> To learn more about *Uniform Type Identifiers*, take a look at [Apple's documentation](https://developer.apple.com/library/archive/documentation/FileManagement/Conceptual/understanding_utis/understand_utis_intro/understand_utis_intro.html).

*DropDelegate* also has a few not required functions that we can use to understand when dropping started and ended.
Finally, we are ready to register our *BookmarksDropDelegate* on the *List* or *ForEach* to enable drop interaction.

```swift
struct BookmarksList: View {
    @State private var links: [URL] = [
        URL(string: "https://twitter.com/mecid")!
    ]

    var body: some View {
        List {
            ForEach(links, id: \.self) { url in
                Text(url.absoluteString)
                    .onDrag { NSItemProvider(object: url as NSURL) }
            }
            .onDrop(
                of: ["public.url"],
                delegate: BookmarksDropDelegate(bookmarks: $links)
            )
        }
        .navigationBarTitle("Bookmarks")
    }
```

Unfortunately, *onDrop* modifier doesn't affect the *List* or any view that inside a *List*. It works well with *VStack*, *ScrollView*, and other views. But it doesn't work with the *List*.

I hope it is a bug, and Apple will fix it soon. We can replace the *List* with *ScrollView* to make *onDrop* modifier working. But in this case, we will lose all the features of the *List* component.

#### Insert
Gladly, SwiftUI provides us another modifier that we can use on *ForEach* view. *onInsert* modifier looks like a simplified version of *onDrop* modifier, and it works inside *List* component. Let's take a look at the quick usage example of *onInsert* modifier.

```swift
struct BookmarksList: View {
    @State private var links: [URL] = [
        URL(string: "https://twitter.com/mecid")!
    ]

    var body: some View {
        List {
            ForEach(links, id: \.self) { url in
                Text(url.absoluteString)
                    .onDrag { NSItemProvider(object: url as NSURL) }
            }
            .onInsert(of: ["public.url"], perform: drop)
        }
        .navigationBarTitle("Bookmarks")
    }

    private func drop(at index: Int, _ items: [NSItemProvider]) {
        for item in items {
            _ = item.loadObject(ofClass: URL.self) { url, _ in
                DispatchQueue.main.async {
                    url.map { self.links.insert($0, at: index) }
                }
            }
        }
    }
}
```

As you can see, we use *onInsert* modifier to accept the insertion of links. We also pass the drop function that handles the process of loading the URL object and insertion.

#### Conclusion
Today we learned another new feature that released with Xcode 11.4. Please note that all drag and drop related modifiers are available as part of iOS 13.4. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week!
