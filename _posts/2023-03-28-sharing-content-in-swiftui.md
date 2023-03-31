---
title: Sharing content in SwiftUI 
layout: post
category: Mastering SwiftUI views
image: /public/wwdc22.jpg
---

Apple introduced a brand new CoreTransferable framework and *ShareLink* view in SwiftUI, allowing us to share and export content from our apps very declaratively. This week we will learn how to make data transferable and use the new *ShareLink* view in SwiftUI.

{% include friends.html %}

#### ShareLink
The new *ShareLink* looks like a plain button but integrates the share sheet and exports data in the provided format. Let's look at the simple example of using the *ShareLink* view in SwiftUI.

```swift
struct Food: Codable {
    var title: String
    var ingredients: [String]
}

struct ContentView: View {
    @State private var food = Food(title: "", ingredients: [])
    
    var body: some View {
        Form {
            Section {
                TextField("title", text: $food.title)
            }
            
            Section {
                ForEach($food.ingredients, id: \.self) { $item in
                    TextField("item", text: $item)
                }
                
                Button("Add") {
                    food.ingredients.append("")
                }
                
                ShareLink(
                    "Export",
                    item: food.ingredients.joined(separator: ","),
                    preview: SharePreview("Export \(food.title)")
                )
            }
        }
    }
}
```

As you can see in the example above, we have the *Food* struct containing a title and an array of ingredients. There is a form allowing us to populate the instance of the *Food* type with data. At the bottom of the form, we have an instance of the *ShareLink* view that exports the content of the food to a plain string by joining ingredients. The code above is simple but handles a share sheet presentation and data export.

But what about other data types like images, binary data, or any other custom format? All the magic here is hidden behind *ShareLink's* **item** parameter. It works with any type conforming to the *Transferable* protocol. *String*, *Data*, and many other types conform to the *Transferable* protocol out of the box, and you don't need to do anything to share them.

#### Transferable
Now we know how the *ShareLink* view works in SwiftUI. It relies on the *Transferable* protocol from the CoreTransferable framework. But what if we want to share our custom type? In this case, we must conform our type to the *Transferable* protocol.

```swift
import CoreTransferable

extension Food: Transferable {
    static var transferRepresentation: some TransferRepresentation {
        CodableRepresentation(contentType: .text)
    }
}
```

The *Transferable* protocol is simple and has the only requirement. We must implement the *transferRepresentation* property and return some instance of the *TransferRepresentation* type. The framework provides ready-to-use representation types: *CodableRepresentation*, *DataRepresentation*, *FileRepresentation*, and *ProxyRepresentation*.

In the example above, we use the *CodableRepresentation* because our *Food* type conforms to the *Codable* protocol. This conformance allows us to automatically export any instance of the Food type as a JSON string.

We also can use *DataRepresentation* if you can convert your value type into an instance of the *Data* type, or we can use *FileRepresentation* whenever the value represents a file on the disk.

```swift
import CoreTransferable
import UniformTypeIdentifiers

extension UTType {
    static var food: UTType {
        UTType(exportedAs: "myapp.food.type")
    }
}

extension Food: Transferable {
    static var transferRepresentation: some TransferRepresentation {
        CodableRepresentation(contentType: .food)
        CodableRepresentation(contentType: .text)
    }
}
```

The *transferRepresentation* property is marked with the *TransferRepresentationBuilder* and allows us to combine different representations. For example, you can define different representations for different content types. Remember, the order makes sense; you should keep the most important representations above others.

Today we learned about the new CoreTransferable framework and how to use the *ShareLink* view in SwiftUI. The CoreTransferable framework plays a massive role in exporting your data from the app. But it works also behind the drag and drop, and I will cover this part next week.

I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week!
