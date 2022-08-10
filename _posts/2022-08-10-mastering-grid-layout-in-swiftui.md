---
title: Mastering grid layout in SwiftUI
layout: post
---

You might be familiar with the *LazyVGrid* and *LazyHGrid* views we have from the second iteration of the SwiftUI framework. They work great for the massive data arrays you want to display as the grid. But it is not always possible to arrange the columns and rows strictly because of their lazy nature. For these cases, SwiftUI introduces another type of grid that immediately lays out all of its children. This week we will learn how to use the new grid layout in SwiftUI and the benefits of the new grid over the lazy version of the grid component.

Let's start with a basic grid layout presenting a list of user data.

```swift
struct ContentView: View {
    @State private var users = mockUsers
    
    var body: some View {
        Grid(alignment: .leading, horizontalSpacing: 10, verticalSpacing: 10) {
            ForEach(users) { user in
                GridRow(alignment: .firstTextBaseline) {
                    Text(user.name)
                    Text(user.familyName)
                    Text(user.age.formatted())
                }
            }
        }
    }
}
```

Here we define a grid layout with leading alignment and particular vertical and horizontal spacing. SwiftUI renders every view inside a row as a column and uses the alignment we pass to align the view inside the available space of the cell. SwiftUI uses spacing parameters to preserve the space between cells. Because of the eager nature of the *Grid* layout, it measures the sizes of all children and lays them into strict columns and rows. As you can see in the example, we also can tune the alignment inside the particular grid row. Now let's add the separator after every grid row.

```swift
struct ContentView: View {
    @State private var users = mockUsers
    
    var body: some View {
        Grid(alignment: .leading, horizontalSpacing: 10, verticalSpacing: 10) {
            ForEach(users) { user in
                GridRow(alignment: .firstTextBaseline) {
                    Text(user.name)
                    Text(user.familyName)
                    Text(user.age.formatted())
                }

                GridRow {
                    Rectangle()
                        .fill(.secondary)
                        .frame(height: 1)
                }
            }
        }
    }
}
```

As I said, the grid layout immediately measures all the children and draws strict columns. It understands that there is three column layout and renders our separator in the first and only column. We can expand our divider to fill all three columns using the *gridCellColumns* view modifier on the view representing the cell.

```swift
struct ContentView: View {
    @State private var users = mockUsers
    
    var body: some View {
        Grid(alignment: .leading, horizontalSpacing: 10, verticalSpacing: 10) {
            ForEach(users) { user in
                GridRow(alignment: .firstTextBaseline) {
                    Text(user.name)
                    Text(user.familyName)
                    Text(user.age.formatted())
                }

                GridRow {
                    Rectangle()
                        .fill(.secondary)
                        .frame(height: 1)
                        .gridCellColumns(3)
                }
            }
        }
    }
}
```

Another solution is to put the separator outside the *GridRow* view. All the views placed directly inside the grid without the *GridRow* view fill the available space.

```swift
struct ContentView: View {
    @State private var users = mockUsers
    
    var body: some View {
        Grid(alignment: .leading, horizontalSpacing: 10, verticalSpacing: 10) {
            ForEach(users) { user in
                GridRow(alignment: .firstTextBaseline) {
                    Text(user.name)
                    Text(user.familyName)
                    Text(user.age.formatted())
                }

                Rectangle()
                    .fill(.secondary)
                    .frame(height: 1)
            }
        }
    }
}
```

We set the grid cell alignment for all cells while defining a grid layout, but we can set a particular alignment only for the cell we need using the *gridColumnAlignment*.

```swift
struct ContentView: View {
    @State private var users = mockUsers
    
    var body: some View {
        Grid(alignment: .leading, horizontalSpacing: 10, verticalSpacing: 10) {
            ForEach(users) { user in
                GridRow(alignment: .firstTextBaseline) {
                    Text(user.name)
                    Text(user.familyName)
                    Text(user.age.formatted())
                        .gridColumnAlignment(.trailing)
                }
            }
        }
    }
}
```

SwiftUI allows us to set not only the alignment of the particular cell but also its anchor point. For example, we might want to shift the cell content to 25% of the horizontally available space. In this case, we can use the *gridCellAnchor* view modifier.

```swift
struct ContentView: View {
    @State private var users = mockUsers
    
    var body: some View {
        Grid(alignment: .leading, horizontalSpacing: 10, verticalSpacing: 10) {
            ForEach(users) { user in
                GridRow(alignment: .firstTextBaseline) {
                    Text(user.name)
                    Text(user.familyName)
                    Text(user.age.formatted())
                        .gridCellAnchor(.init(x: 0.25, y: 0.0))
                }
            }
        }
    }
}
```

As you can see in the previous examples, flexible views like shapes and dividers take the whole available space while placed inside the grid directly. We can prevent this behavior by using the *gridCellUnsizedAxes* view modifier.

```swift
struct ContentView: View {
    @State private var users = mockUsers
    
    var body: some View {
        Grid(alignment: .leading, horizontalSpacing: 10, verticalSpacing: 10) {
            ForEach(users) { user in
                GridRow(alignment: .firstTextBaseline) {
                    Text(user.name)
                    Text(user.familyName)
                    Text(user.age.formatted())
                }

                GridRow {
                    Rectangle()
                        .fill(.secondary)
                        .frame(height: 1)
                        .gridCellColumns(3)
                        .gridCellUnsizedAxes([.horizontal])
                }
            }
        }
    }
}
```

In this case, we ask our separator view to take place by respecting the width of other rows and don't fill all the available width.

Today we learned how to use the new grid layout in SwiftUI. Try to use it by default whenever you need a grid and switch to the lazy version of the grid component whenever performance issues appear.
