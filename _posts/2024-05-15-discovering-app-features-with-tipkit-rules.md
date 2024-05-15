---
title: Discovering app features with TipKit. Rules.
layout: post
category: Interactions
image: /public/tip1.png
---

This week, we will continue discussing how to highlight app features using the TipKit framework. TipKit provides a flexible way of customizing the condition under which tips should appear. 

{% include friends.html %}

As I said, we can manage the rules under which tips appear. For example, you can turn off a tip whenever the user has already used the feature.

```swift
struct ContentView: View {
    @State private var store = FeedStore()
    
    var body: some View {
        NavigationStack {
            List {
                ForEach(store.items, id: \.self) { item in
                    Text(verbatim: item)
                }
                
                Button("Add") {
                    store.addItem()
                }
                .popoverTip(FeedTip.add) { action in
                    if action.id == "add" {
                        FeedTip.add.invalidate(reason: .actionPerformed)
                        store.addItem()
                    }
                }
            }
        }
    }
}
```

#### Rules
Let's move forward by discussing another concept of the TipKit framework: rules and parameters. The TipKit framework provides the **#Rule** macro, which allows us to define dynamic rules based on the app state.

```swift
enum FeedTip: Tip {
    @Parameter static var isPro: Bool = false
    
    case add
    case delete
    
    var title: Text {
        switch self {
        case .add:
            Text("Add")
        case .delete:
            Text("delete")
        }
    }
    
    var rules: [Rule] {
        #Rule(Self.$isPro) { isPro in
            isPro == true
        }
    }
}
```

As you can see in the example above, we use the *Parameter* macro to define a variable that we can bind to the app state. Next, we implement another optional property of the *Tip* protocol called *rules*. Here, we use the *Rule* macro to display the tip only for pro users.

You can define many parameters and build complex rules using many of them. The *rules* property of the *Tip* protocol uses the *RuleBuilder* result builder, allowing us to create complex rules using the **if** operator or optional chaining.

```swift
struct ContentView: View {
    @Environment(\.isPro) var isPro
    @State private var store = FeedStore()
    
    var body: some View {
        NavigationStack {
            List {
                ForEach(store.items, id: \.self) { item in
                    Text(verbatim: item)
                }
                
                Button("Add") {
                    store.addItem()
                }
                .popoverTip(FeedTip.add)
                .onAppear {
                    FeedTip.isPro = isPro
                }
            }
        }
    }
}
```

Here is an example showing how to bind a tip parameter to the app state. As you can see, we use the *onAppear* view modifier to sync the *isPro* parameter of the tip.

#### Events
Another thing that the TipKit framework provides us to build custom rules is events. An event is a user action we can donate to. The main difference with the parameters is the persistence. Framework automatically stores the number of events that happened.

```swift
enum FeedTip: Tip {
    static let itemAdded = Event(id: "itemAdded")
    @Parameter static var isPro: Bool = false
    
    case add
    case delete
    
    var title: Text {
        switch self {
        case .add:
            Text("Add")
        case .delete:
            Text("delete")
        }
    }
    
    var rules: [Rule] {
        #Rule(Self.$isPro) { isPro in
            isPro == true
        }
        
        #Rule(Self.itemAdded) {
            $0.donations.count >= 3
        }
    }
}
```

As you can see in the example above, we define an event in the *FeedTip* type using the *Event* type. We only need the unique identifier, which allows the framework to persist the event history. We also combine the rules for the *isPro* parameter, and the new event to display the tip only when both conditions are true.

```swift
struct ContentView: View {
    @Environment(\.isPro) var isPro
    @State private var store = FeedStore()
    
    var body: some View {
        NavigationStack {
            List {
                ForEach(store.items, id: \.self) { item in
                    Text(verbatim: item)
                }
                
                Button("Add") {
                    Task {
                        await FeedTip.itemAdded.donate()
                        store.addItem()
                    }
                }
                .popoverTip(FeedTip.add) { action in
                    if action.id == "add" {
                        FeedTip.add.invalidate(reason: .actionPerformed)
                        store.addItem()
                    }
                }
                .onAppear {
                    FeedTip.isPro = isPro
                }
            }
        }
    }
}
```

To increment the number of times a particular event happened, we can use the *donate* function on the instance of the *Event* type. The TipKit automatically saves the number of events that happened on the disk, and it survives the app restart.

#### Options
The last thing that can affect the visibility of the tip is the *options* parameter you can define while implementing the *Tip* protocol on your custom tip type.

```swift
enum FeedTip: Tip {
    static let itemAdded = Event(id: "itemAdded")
    @Parameter static var isPro: Bool = false
    
    case add
    case delete
    
    var title: Text {
        switch self {
        case .add:
            Text("Add")
        case .delete:
            Text("delete")
        }
    }
    
    var options: [any TipOption] {
        MaxDisplayCount(3)
    }
    
    var rules: [Rule] {
        #Rule(Self.$isPro) { isPro in
            isPro == true
        }
        
        #Rule(Self.itemAdded) {
            $0.donations.count >= 3
        }
    }
}
```

Here, we define the options parameter on the *FeedTip* type. We use the *MaxDisplayCount* type to limit the number of tip displays. Remember that options have priority over rules, and the framework will not display the tip more than three times even if the app meets all the conditions in the rules property.

Today, we learned how to customize the logic that decides whenever to display your tips using the rules, parameters, events, and options.
