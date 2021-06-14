---
title: What is new in SwiftUI after WWDC21
layout: post
image: /public/wwdc21-v3.jpg
category: Meta
---

WWDC21 is finally here, and there are many new things in the updated version of SwiftUI. I'm happy to share with you that many items on my wishlist have finally arrived. In this post, I will try to give you a summary of the significant SwiftUI additions of this year.

{% include friends.html %}

#### List
The *List* view is the most common view in our apps. The *List* view managed *UITableView* under the hood but didn't expose all the great features of *UITableView* till today. Now we have view modifiers that expose the styling options for separators and tint colors in sections and cells. Let's see how we can use them.

```swift
struct ContentView: View {
    @State private var messages: [String] = [
        "Hello", "World"
    ]

    var body: some View {
        NavigationView {
            List {
                ForEach(messages, id: \.self) { message in
                    Text(message)
                        .listRowSeparatorTint(Color.red)
                        .listRowSeparator(.visible, edges: .all)
                }
            }
            .listSectionSeparator(.visible, edges: .all)
            .listSectionSeparatorTint(Color.purple)
            .navigationTitle("Messages")
        }
    }
}
```

SwiftUI also provides the new *swipeActions* view modifier that we can use to attach the swipeable action to views inside a list.

```swift
struct ContentView: View {
    @State private var messages: [String] = [
        "Hello", "World"
    ]

    var body: some View {
        NavigationView {
            List {
                ForEach(messages, id: \.self) { message in
                    Text(message)
                        .swipeActions(edge: .trailing, allowsFullSwipe: false) {
                            Button("Remove", role: .destructive) {
                                messages.removeAll { $0 == message }
                            }
                        }
                        .swipeActions(edge: .leading, allowsFullSwipe: true) {
                            Button("Copy") {
                                messages.append(message)
                            }
                        }
                }
            }.navigationTitle("Messages")
        }
    }
}
```

As you can see in the example above, we also have new *ButtonRole* enum that allows specifying a role for the button, which can be either *destructive* or *cancel*.

#### Pull-to-Refresh
I used to have the pull-to-refresh gesture in the screens where the content can be updated or refetched. SwiftUI provides the new *refreshable* view modifier that we can use to attach the pull-to-refresh gesture and a callback that SwiftUI runs as soon as the user enables the gesture.

```swift
struct ContentView: View {
    @State private var messages: [String] = [
        "Hello", "World"
    ]

    var body: some View {
        NavigationView {
            List {
                ForEach(messages, id: \.self) { message in
                    Text(message)
                }
            }.refreshable {
                messages.append("!!!")
            }
            .navigationTitle("Messages")
        }
    }
}
```

#### Search
Another thing that I really missed in SwiftUI is *SearchBar* and *SearchController*'s functionality. Fortunately, this iteration of SwiftUI is packed with a brand new collection of view modifiers that enables powerful search capabilities.

```swift
struct ContentView: View {
    @State private var query: String = ""
    @State private var messages: [String] = [
        "Hello", "World"
    ]

    var body: some View {
        NavigationView {
            List {
                ForEach(messages, id: \.self) { message in
                    Text(message)
                }
            }
            .searchable("Search term", text: $query, placement: .automatic)
            .onChange(of: query) { print($0) }
            .navigationTitle("Messages")
        }
    }
}
```

#### AsyncImage
The thing I didn't expect was the new *AsyncImage* view. *AsyncImage* view allows you to download and present remote images using *URLSession*. It provides a very lovely API which very easy to use. Let's take a look at it.

```swift
struct Post: Hashable {
    let image: URL
    let title: String
    let content: String
}

struct PostView: View {
    let post: Post

    var body: some View {
        HStack {
            AsyncImage(url: post.image)
            VStack {
                Text(post.title)
                Text(post.content)
                    .foregroundColor(.secondary)
            }
        }
    }
}
```

#### Text and AttributedString
Swift Foundation has a new *AttributedString* type and SwiftUI supports it out of the box. You can now pass the instances of *AttributedString* to *Text* view to display it. An interesting detail here is the ability to display markdown content using the new *AttributedString*.

```swift
struct PostView: View {
    let post: Post

    var markdown: AttributedString {
        try! AttributedString(markdown: post.content)
    }

    var body: some View {
        HStack {
            AsyncImage(url: post.image)
            VStack {
                Text(post.title)
                Text(markdown)
            }
        }
    }
}
```

As the bonus we gain markdown support everywhere with *Text* view.

```swift
struct ContentView: View {
    var body: some View {
        Text("**Happy WWDC21**")
    }
}
```

#### Focus management 
Finally, SwiftUI provides us a way to manage the focus in our views. There are brand new @*FocusState* property wrappers and a *focused* view modifier that we can use to toggle first responders.

```swift
struct LoginForm: View {
    enum Field: Hashable {
        case usernameField
        case passwordField
    }

    @State private var username = ""
    @State private var password = ""
    @FocusState private var focusedField: Field?

    var body: some View {
        VStack {
            TextField("Username", text: $username)
                .focused($focusedField, equals: .usernameField)

            SecureField("Password", text: $password)
                .focused($focusedField, equals: .passwordField)

            Button("Sign In") {
                if username.isEmpty {
                    focusedField = .usernameField
                } else if password.isEmpty {
                    focusedField = .passwordField
                } else {
                    handleLogin(username, password)
                }
            }
        }
    }
}
```

#### Canvas
SwiftUI *Canvas* is a new way to gain more control over lower-level drawing primitives. It is a modern, GPU-accelerated equivalent of *drawRect*. We can use *Canvas* to draw shapes using *Path*. We can also draw images, texts, and other SwiftUI views.

```swift
struct ContentView: View {
    var body: some View {
        Canvas { context, size in
            context.fill(
                Circle().path(in: .init(origin: .zero, size: size)),
                with: .color(.green)
            )
        }
    }
}
```

#### Materials
iOS provides materials or blur effects that create a translucent effect you can use to evoke a sense of depth. The effect of material lets views and controls hint at background content without distracting from foreground content. We didn't have access to materials in previous versions of SwiftUI. Fortunately, the new version of SwiftUI provides us a whole range of physical materials from ultra-thin to ultra-thick and semantic bars material.

```swift
ZStack {
    Color.teal
    Label("Flag", systemImage: "flag.fill")
        .padding()
        .background(.regularMaterial)
}
```

#### TimelineView
*TimelineView* is another brand new SwiftUI view. Usually, SwiftUI updates views only during environment or state changes. In case of *TimelineView*, SwiftUI updates it according to a schedule that you provide. It might be very useful while building clock or workout apps.

```swift
TimelineView(PeriodicTimelineSchedule(from: startDate, by: 1)) { context in
    AnalogTimerView(date: context.date)
}
```

#### Conclusion
There are many other additions worth mentioning, like a brand new Accessibility Rotors API, the new *SectionedFetchRequest* property wrapper that allows you to make sectioned requests to Core Data, and much more.

I hope to cover all these new features of the SwiftUI framework in the upcoming weeks. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week!