---
title: Mastering container views in SwiftUI. Values.
layout: post
---

In the series' final post about container views in SwiftUI, I will discuss container values and how SwiftUI allows us to propagate data through the container view logic. This week, we will learn how to declaratively define and pass container values.

Container values are similar to environment values, allowing us to pass data inside through the container view instead of the environment and access it later inside the container view.

```swift
struct Magazine<Content: View>: View {
    @ViewBuilder var content: Content
    
    var body: some View {
        ScrollView {
            LazyVStack {
                Group(sections: content) { sections in
                    if !sections.isEmpty {
                        sections[0].content.frame(minHeight: 100)
                    }
                    
                    if sections.count > 1 {
                        ForEach(sections[1...]) { section in
                            section.header
                            
                            ScrollView(.horizontal) {
                                LazyHStack {
                                    section.content
                                        .containerRelativeFrame(.horizontal)
                                        .frame(minHeight: 100)
                                }
                            }
                            .contentMargins(16)
                            
                            section.footer
                        }
                    }
                }
            }
        }
    }
}
```

As you can see in the example above, we decompose the first section from the content view and place it outside to make it prominent. In this case, we hard-code the section by its index, which must always be the first section.

```swift
struct ContentView: View {
    var body: some View {
        Magazine {
            Section(header: Text("Favorites")) {
                Color.red
                Color.orange
            }
            
            Section {
                Color.green
                Color.yellow
            }
            .featured()
            
            Section(footer: Text("Footer")) {
                Color.blue
                Color.purple
            }
        }
    }
}
```

We can make it more dynamic using container values. What if we could mark a particular section featured, and the container view would find that section and place it at the top.

```swift
extension ContainerValues {
    @Entry var isFeatured = false
}
```

As you can see, we use the @Entry macro to define a container value with the default value. All we need to do is to create an extension for the ContainerValues type and specify a property with the @Entry macro. Now, we can use it to mark any section or view as featured.

```swift
struct ContentView: View {
    var body: some View {
        Magazine {
            Section(header: Text("Favorites")) {
                Color.red
                Color.orange
            }
            
            Section {
                Color.green
                Color.yellow
            }
            .containerValue(\.isFeatured, true)
            
            Section(footer: Text("Footer")) {
                Color.blue
                Color.purple
            }
        }
    }
}
```

We use the containerValue view modifier to set the container value using the keypath. We can also write an extension to simplify the usage.

```swift
extension ContainerValues {
    @Entry var isFeatured = false
}

extension View {
    func featured(_ isFeatured: Bool = true) -> some View {
        containerValue(\.isFeatured, isFeatured)
    }
}

struct ContentView: View {
    var body: some View {
        Magazine {
            Section(header: Text("Favorites")) {
                Color.red
                Color.orange
            }
            
            Section {
                Color.green
                Color.yellow
            }
            .featured()
            
            Section(footer: Text("Footer")) {
                Color.blue
                Color.purple
            }
        }
    }
}
```

We know how to define a container value with a default state and change it using the containerValue view modifier. The next step is to read the value from the container view and decide on view recomposition.

```swift
struct Magazine<Content: View>: View {
    @ViewBuilder var content: Content
    
    var body: some View {
        ScrollView {
            LazyVStack {
                Group(sections: content) { sections in
                    let featured = sections.filter(\.containerValues.isFeatured)
                    
                    if !sections.isEmpty {
                        ForEach(featured) { section in
                            section.content.frame(minHeight: 100)
                        }
                    }
                    
                    let notFeatured = sections.filter { !$0.containerValues.isFeatured }
                    
                    ForEach(notFeatured) { section in
                        section.header.padding(.top)
                        
                        ScrollView(.horizontal) {
                            LazyHStack {
                                section.content
                                    .containerRelativeFrame(.horizontal)
                                    .frame(minHeight: 100)
                            }
                        }
                        .contentMargins(16)
                        
                        section.footer
                    }
                }
            }
        }
    }
}
```

Both the Subview and SectionConfiguration types provide the containerValues property, which allows us to read any defined container value.

As you can see in the example above, we use the containerValues property on instances of the SectionConfiguration type to filter featured and non-featured sections.

We use the isFeatured container value to recompose the view hierarchy as needed. Container values propagate through the view hierarchy similar to environment values. Any view inside the featured section will have the isFeatured value of true.

Today, we learned how to use container values to propagate data through container view and recompose view hierarchies using that information.
