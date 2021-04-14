---
title: Sidebar navigation in SwiftUI
layout: post
image: /public/sidebar.png
category: Mastering SwiftUI views
---

We already covered master-detail navigation in SwiftUI on my blog. But today, I want to talk about the new three-column navigation that landed this year into iOS and macOS worlds. We will learn how to build a sidebar navigation flow by using *NavigationView* in SwiftUI.

{% include friends.html %}

A sidebar provides app-level navigation and quick access to top-level collections of content in your app. Selecting an item in the sidebar allows people to navigate to a specific piece of content. For example, the sidebar in Mail shows a list of all mailboxes. People can select a mailbox to access its list of messages, and select a specific message to display in the content pane.

![sidebar](/public/sidebar.png)

Let's build a prototype of a mail app that uses three-column navigation. SwiftUI provides *NavigationView* that allows us to create a master-detail flow. You can put up to three children inside a *NavigationView*. In this case, SwiftUI will place views side-by-side. But let's start with declaring our data model. 

```swift
import SwiftUI

struct Mail: Identifiable, Hashable {
    let id = UUID()
    let date: Date
    let subject: String
    let body: String
    var isFavorited = false
}

final class MailStore: ObservableObject {
    @Published var allMails: [String: [Mail]] = [
        "Inbox": [ .init(date: Date(), subject: "Subject1", body: "Very long body...") ],
        "Sent": [ .init(date: Date(), subject: "Subject2", body: "Very long body...") ],
    ]
}
```

> To learn about building navigation using *NavigationView* and *NavigationLink*, take a look at my ["Navigation in SwiftUI"](/2019/07/17/navigation-in-swiftui/) post.

In the listing above, we create a simple mail store that we will use as a datastore for our prototype. Now let's move forward by implementing the first column of our navigation flow.

```swift
struct Sidebar: View {
    @ObservedObject var store: MailStore
    @Binding var selectedFolder: String?
    @Binding var selectedMail: Mail?

    var body: some View {
        List {
            ForEach(Array(store.allMails.keys), id: \.self) { folder in
                NavigationLink(
                    destination: FolderView(
                        title: folder,
                        mails: store.allMails[folder, default: []],
                        selectedMail: $selectedMail
                    ),
                    tag: folder,
                    selection: $selectedFolder
                ) {
                    Text(folder).font(.headline)
                }
            }
        }.listStyle(SidebarListStyle())
    }
}
```

As you can see, we create a view called sidebar. It needs an instance of store object to access our emails and two bindings that we will use to bind the current folder and selected email. We also use the new *SidebarListStyle* to apply a brand new styling to our list that available on iOS 14 and macOS Big Sur. 

```swift
struct FolderView: View {
    let title: String
    let mails: [Mail]
    @Binding var selectedMail: Mail?

    var body: some View {
        List {
            ForEach(mails) { mail in
                NavigationLink(
                    destination: MailView(mail: mail),
                    tag: mail,
                    selection: $selectedMail
                ) {
                    VStack(alignment: .leading) {
                        Text(mail.subject)
                        Text(mail.date, style: .date)
                    }
                }
            }
        }.navigationTitle(title)
    }
}
```

Here we have the *FolderView* struct that we use to display a list of emails in the folder. *FolderView* also needs the binding for a selected email. Whenever the user selects an email in the list, SwiftUI sets the value of the binding and updates the view hierarchy. Now let's take a look at *MailView*.

```swift
struct MailView: View {
    let mail: Mail

    var body: some View {
        VStack(alignment: .leading) {
            Text(mail.subject)
                .font(.headline)
            Text(mail.date, style: .date)
            Text(mail.body)
        }
    }
}
```

OK, now we have all the needed pieces to build our three-column navigation flow. As I mentioned earlier, we will put our column views inside a *NavigationView*.

```swift
@main
struct TestProjectApp: App {
    @StateObject var store = MailStore()
    @State private var selectedLabel: String? = "Inbox"
    @State private var selectedMail: Mail?

    var body: some Scene {
        WindowGroup {
            NavigationView {
                Sidebar(
                    store: store,
                    selectedFolder: $selectedLabel,
                    selectedMail: $selectedMail
                )

                Text("Select label...")
                Text("Select mail...")
            }
        }
    }
}
```

As you can see, we have *NavigationView*, which is the root of our app scene. We also define two state properties which describe selected label and email. We pass bindings to these state properties down into the view hierarchy, it allows us to programmatically navigate to the selected folder and mail when needed. You can remove these bindings if you don't need programmatic navigation.

Thanks to SwiftUI's declarative nature, the code above works great both on iPhone, where it uses the single column navigation and iPad where is uses sidebar navigation.

Sidebar navigation plays a huge role in new [Human Interface Guidelines](https://developer.apple.com/design/human-interface-guidelines/ios/bars/sidebars/). It is effortless to implement in SwiftUI using *NavigationView*. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this article. Thanks for reading, and see you next week!