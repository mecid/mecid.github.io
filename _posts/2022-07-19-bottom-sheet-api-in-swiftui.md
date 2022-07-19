---
title: Bottom sheet API in SwiftUI
layout: post
---

Two years ago, I wrote a post about building a custom bottom sheet in SwiftUI. Nowadays, there is no need to make it manually, at least if you don't need a super custom behavior. SwiftUI introduces a new API to display a bottom sheet in a few lines of code. This week we learn the new API allowing us to present bottom sheets in different appearances.

The new bottom sheet API in SwiftUI is straightforward to use. All you need to do is to attach the *presentationDetents* view modifier to the content of the *sheet* view modifier.

```swift
struct ContentView: View {
    @State private var sheetShown = false
    @State private var query = ""

    var body: some View {
        Button("Display bottom sheet") {
            sheetShown = true
        }
        .sheet(isPresented: $sheetShown) {
            NavigationStack {
                Text("You query: \(query)")
                    .searchable(text: $query)
                    .navigationTitle("Search")
            }
            .presentationDetents([.medium])
        }
    }
}
```

As you can see in the example above, we use the *presentationDetents* view modifier to display a sheet as the bottom sheet. We also set the size of the bottom sheet to *medium*. In this case, it will take half of the screen.
You can use the *presentationDetents* view modifier to pass an array of available sizes, allowing the user to resize the sheet by dragging it.

```swift
struct ContentView: View {
    @State private var sheetShown = false
    @State private var query = ""

    var body: some View {
        Button("Display bottom sheet") {
            sheetShown = true
        }
        .sheet(isPresented: $sheetShown) {
            NavigationStack {
                Text("You query: \(query)")
                    .searchable(text: $query)
                    .navigationTitle("Search")
            }
            .presentationDetents([.medium, .large])
        }
    }
}
```

The new bottom sheet API is very flexible and allows you to tune the height of the sheet as you need by using the following options.

```swift
struct ContentView: View {
    @State private var sheetShown = false
    @State private var query = ""

    var body: some View {
        Button("Display bottom sheet") {
            sheetShown = true
        }
        .sheet(isPresented: $sheetShown) {
            NavigationStack {
                Text("You query: \(query)")
                    .searchable(text: $query)
                    .navigationTitle("Search")
            }
            .presentationDetents(
                [
                    .fraction(0.2),
                    .height(300),
                    .fraction(0.5),
                    .height(600)
                ]
            )
        }
    }
}
```

As you can see in the example above, we use the *fraction* function to set the height of the sheet to a particular fraction of the available space. You can also use the *height* function to set the height to a fixed size. Remember that you can mix different fractions to achieve the appearance you need.

Depending on the current environment, we might need to build a super custom sizing logic. For example, I would like to display a bottom sheet by taking 80% of the available space on the iPad and full height on the iPhone. We can implement this custom presentation logic and reuse it across multiple screens.

```swift
struct MyCustomDetent: CustomPresentationDetent {
    static func height(in context: Context) -> CGFloat? {
        if context.verticalSizeClass == .regular {
            return context.maxDetentValue * 0.8
        } else {
            return context.maxDetentValue
        }
    }
}
```

First, we should create a type conforming to the *CustomPresentationDetent* protocol. Second, we should implement the only required function called *height* which takes the *context* as the parameter. Parameter *context* holds all the SwiftUI's environment values, and you can use it to decide what height you need in a particular environment. I use vertical size classes in the example above to decide on the sheet's height.

```swift
struct ContentView: View {
    @State private var sheetShown = false
    @State private var query = ""

    var body: some View {
        Button("Display bottom sheet") {
            sheetShown = true
        }
        .sheet(isPresented: $sheetShown) {
            NavigationStack {
                Text("You query: \(query)")
                    .searchable(text: $query)
                    .navigationTitle("Search")
            }
            .presentationDetents([.medium, .custom(MyCustomDetent.self)])
        }
    }
}
```

The last thing we can customize is the drag indicator of the bottom sheet. We can make it always visible, hidden, or allow SwiftUI to decide whether it should be visible.

```swift
struct ContentView: View {
    @State private var sheetShown = false
    @State private var query = ""

    var body: some View {
        Button("Display bottom sheet") {
            sheetShown = true
        }
        .sheet(isPresented: $sheetShown) {
            NavigationStack {
                Text("You query: \(query)")
                    .searchable(text: $query)
                    .navigationTitle("Search")
            }
            .presentationDetents([.medium, .large])
            .presentationDragIndicator(.automatic)
        }
    }
}
```

Today we learned how to use the new bottom sheet API in SwiftUI. I love the level of customization it provides.
