---
title: Customizing toolbars in SwiftUI
layout: post
---

Toolbars API is one of my favorite APIs in SwiftUI. It allows you to define the toolbar and its items in a very declarative way behaving differently on separate platforms. The next generation of the SwiftUI framework brings us more ways of customizing toolbars. This week we will learn about the new APIs allowing us to customize toolbars in SwiftUI.

> Look at my dedicated "Mastering toolbars in SwiftUI" post to learn about Toolbar API basics.

#### Toolbar visibility
Let's start with the new view modifier allowing us to control toolbar visibility. 

```swift
struct ContentView: View {
    var body: some View {
        ScrollView {
            Image("beach")
                .resizable()
                .scaledToFit()
        }
        .ignoresSafeArea(.container, edges: .top)
        .navigationTitle("Hello")
        .toolbar(.hidden, for: .navigationBar)
    }
}
```

As you can see in the example above, the new toolbar view modifier allows us to hide or show any toolbar controlled by SwiftUI. We can use it to control the visibility of not only the navigation bar but also the tab and bottom bars.

```swift
struct ContentView: View {
    var body: some View {
        ScrollView {
            Image("beach")
                .resizable()
                .scaledToFit()
        }
        .ignoresSafeArea(.container, edges: .top)
        .navigationTitle("Hello")
        .toolbar(.hidden, for: .tabBar)
    }
}
```

#### Toolbar background visibility
Another new view modifier allows us to control the visibility of any bar owned by SwiftUI.

```swift
struct ContentView: View {
    var body: some View {
        ScrollView {
            Image("beach")
                .resizable()
                .scaledToFit()
        }
        .ignoresSafeArea(.container, edges: .top)
        .navigationTitle("Hello")
        .toolbarBackground(.hidden, for: .navigationBar)
    }
}
```

The toolbar background visibility view modifier allows us to create nice translucent effects whenever we want to show the image under the toolbar.

#### Toolbar color scheme
Another exciting addition to this year's Toolbar API is the opportunity to control the color scheme of the particular toolbar. You can set a preferred color scheme for your toolbar independent of the view hierarchy's color scheme.

```swift
struct ContentView: View {
    var body: some View {
        ScrollView {
            Image("beach")
                .resizable()
                .scaledToFit()
        }
        .navigationTitle("Hello")
        .toolbarColorScheme(.dark, for: .navigationBar)
    }
}
```

#### Toolbar title menu
iOS 16 provides a new user experience, allowing us to display a popup menu in the title of our navigation bar.

```swift
struct TitleMenuExample: View {
    @State private var date = Date.now
    @State private var datePickerShown = false
    
    var body: some View {
        NavigationStack {
            Text(date, style: .date)
                .navigationTitle(Text(date, style: .date))
                .navigationBarTitleDisplayMode(.inline)
                .toolbarTitleMenu {
                    Button("Pick another date") {
                        datePickerShown = true
                    }
                }
                .sheet(isPresented: $datePickerShown) {
                    DatePicker(
                        "Choose date",
                        selection: $date,
                        displayedComponents: .date
                    )
                    .datePickerStyle(.graphical)
                    .presentationDetents([.medium])
                    .presentationDragIndicator(.visible)
                }
        }
    }
}
```

The code above displays a little arrow in the navigation bar's title, allowing the user to press it. SwiftUI shows a popup menu with a button presenting a date picker in the modal sheet.

#### Toolbar role
Another new appearance in iPadOS 16 is the editor toolbar. You can set the role of the toolbar to the editor. In this case, SwiftUI displays toolbar items in the center of the particular toolbar. It also allows the user to customize toolbar items by adding and removing them.

```swift
struct CollapsingToolbarItems: View {
    var body: some View {
        NavigationStack {
            Text("Hello")
                .toolbar {
                    ToolbarItem(placement: .primaryAction) {
                        Button("Primary action") {}
                    }
                    
                    ToolbarItem(
                        id: "copy",
                        placement: .secondaryAction,
                        showsByDefault: true
                    ) {
                        Button("copy") {}
                    }
                    
                    ToolbarItem(
                        id: "delete",
                        placement: .secondaryAction,
                        showsByDefault: false
                    ) {
                        Button("delete") {}
                    }
                }
                .toolbarRole(.editor)
        }
    }
}
```

In the example above, we set the toolbar role to the editor. We also set identifiers for every toolbar item. SwiftUI uses identifiers to store the user configuration of the toolbar setup. Remember that you should provide stable identifiers for your toolbar items to provide a consistent toolbar customization experience.

#### Secondary toolbar items
The current generation of the SwiftUI framework introduces the new placement for secondary toolbar actions. It also automatically collapses them into a single toolbar item displaying the list of collapsed items via a menu.

```swift
struct SecondaryToolbarItemsExample: View {
    var body: some View {
        NavigationStack {
            Text("Hello")
                .toolbar {
                    ToolbarItem(placement: .primaryAction) {
                        Button("Primary action") {}
                    }
                    
                    ToolbarItem(placement: .secondaryAction) {
                        Button("Secondary action 1") {}
                    }
                    
                    ToolbarItem(placement: .secondaryAction) {
                        Button("Secondary action 2") {}
                    }
                }
                
        }
    }
}
```

#### Conclusion
Today we learned a bunch of new APIs allowing us to customize toolbars in SwiftUI.
