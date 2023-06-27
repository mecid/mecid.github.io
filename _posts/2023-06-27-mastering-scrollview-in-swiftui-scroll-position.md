---
title: Mastering ScrollView in SwiftUI. Scroll Position
layout: post
category: Mastering SwiftUI views
---

This week we will continue the series of posts dedicated to mastering ScrollView. Today we will talk about a set of new options for controlling the scroll content position of a ScrollView in SwiftUI.

#### Initial position
The SwiftUI framework provides us the brand new scrollPosition view modifier allowing us to set the initial anchor for the content in the ScrollView. Let's take a look at a very quick example.

```swift
struct ContentView: View {
    var body: some View {
        ScrollView {
            LazyVStack {
                ForEach(0..<100) { index in
                    Rectangle()
                        .fill(Color.green.gradient)
                        .frame(height: 300)
                        .id(index)
                }
            }
        }
        .scrollPosition(initialAnchor: .center)
    }
}
```

As you can see in the example above, we use the new scrollPosition view modifier to set the initial anchor to the center of the content. The center option here is the instance of the UnitPoint type, which has a bunch of predefined values like bottom, top, trailing, leading, etc. You can also create your own UnitPoint by providing x and y values which take normalized values from 0 to 1.

```swift
struct ContentView: View {
    var body: some View {
        ScrollView {
            LazyVStack {
                ForEach(0..<100) { index in
                    Rectangle()
                        .fill(Color.green.gradient)
                        .frame(height: 300)
                        .id(index)
                }
            }
        }
        .scrollPosition(initialAnchor: .init(x: 0, y: 0.9)
    }
}
```

#### Scroll position
Another variant of the new scrollPosition view modifier takes a binding to the hashable value allowing us to read and write the scroll content offset.

```swift
struct ContentView: View {
    @State private var position: Int?
    
    var body: some View {
        ScrollView {
            LazyVStack {
                ForEach(0..<100) { index in
                    Rectangle()
                        .fill(Color.green.gradient)
                        .frame(height: 300)
                        .id(index)
                }
            }
            .scrollTargetLayout()
        }
        .scrollPosition(id: $position)
    }
}
```

As you can see in the example above, we use the scrollPosition view modifier with the binding to the integer value. We also use the id view modifier to explicitly set the identifier to the view inside the scroll. You don't need to set id manually if you are using ForEach with identifiable types or provide the KeyPath to id. 

Remember that you should always use the scrollTargetLayout or scrollTarget view modifiers to allow the ScrollView to understand where to find the identifiers for the binding.

As I said before, you can use the scrollPosition view modifier not only for reading but also to change the scroll content offset. In the following example, we will add the button changing the value of the binding, and the scroll view updates content offset accordingly.

```swift
struct ContentView: View {
    @State private var position: Int?
    
    var body: some View {
        ScrollView {
            LazyVStack {
                Button("Jump...") {
                    position = 99
                }
                
                ForEach(0..<100) { index in
                    Rectangle()
                        .fill(Color.green.gradient)
                        .frame(height: 300)
                        .id(index)
                }
            }
            .scrollTargetLayout()
        }
        .scrollPosition(id: $position)
    }
}
```

#### Indicator flash
Another brand new view modifier related to the scrolling content is the scrollIndicatorsFlash view modifier. It allows us to show a scroll indicator whenever needed. For example, you might need to fade in the scroll indicator while appending the content in the ScrollView.

```swift
struct ContentView: View {
    @State private var trigger = false
    
    var body: some View {
        ScrollView {
            LazyVStack {
                Button("Indicator flash") {
                    trigger.toggle()
                }
                
                ForEach(0..<100) { index in
                    Rectangle()
                        .fill(Color.green.gradient)
                        .frame(height: 300)
                        .id(index)
                }
            }
        }
        .scrollIndicatorsFlash(trigger: trigger)
    }
}
```

As you can see, we use the scrollIndicatorsFlash view modifier and pass some equitable value. The ScrollView blinks the indicator as soon as the passed value changes.

