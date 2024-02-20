---
title: Discovering Swift Collections package
category: Swift Language Features
image: /public/spm.jpg
layout: post
---

I want to continue the topic of the valuable Swift packages that I use in my apps. This time, we will talk about the Swift Collections package, providing us with a bunch of helpful collection types that Swift language doesn't include out of the box.

{% include friends.html %}

The Swift Collections package contains a few collection types that may help you improve the performance of your apps if you apply them whenever needed instead of using generic *Array*, *Dictionary*, and *Set* types. The Swift Collections package lives on Github, where you can find it and your project.

#### Tree-based dictionary and set
Dictionary and Set types that Swift language provides us store values in a single flat hash table that you copy on every write or mutation. The Swift Collection package introduces *TreeDictionary* and *TreeSet* types implementing Compressed Hash-Array Mapped Prefix Trees. In other words, *TreeDictionary* and *TreeSet* types hold values in the tree-based structure, allowing the efficient updating of only the needed branches.

Imagine a calendar app where you store an event array per date and use the standard *Dictionary* type. You might need to implement paging and load events per visible month and store them in an instance of the *Dictionary* type. While the user scrolls through months, your app loads a bunch of events and copies the whole dictionary on every load, even when previously loaded events didn't change.

```swift
@Observable final class CalendarStore {
    typealias Fetch = (DateInterval) async -> [Event]
    
    private(set) var events: TreeDictionary<Date, [Event]> = [:]
    private let fetch: Fetch
    
    init(fetch: @escaping Fetch) {
        self.fetch = fetch
    }
    
    func fetchEvents(inside interval: DateInterval) async {
        let newEvents = await fetch(interval)
        let groupedByDate = TreeDictionary(grouping: newEvents, by: \.date)
        events.merge(groupedByDate) { $1 }
    }
}
```

For this case, the Swift Collections package introduces the *TreeDictionary* and *TreeSet* types that link the unchanged parts with the changed branches under the hood without copying the whole dictionary into the memory. The *TreeDictionary* type provides us with the very same APIs that the *Dictionary* type has and optimizes memory for us under the hood.

The *TreeDictionary* is still a struct, but the implementation uses the *UnsafeMutablePointer* type to access memory and mutate it directly without copying on write. Another benefit of the *TreeDictionary* and *TreeSet* types is the optimized way to compare because of their tree-based nature. Usually, they handle this operation in a constant time.

```swift
let oldEvents: TreeDictionary<Date, [Event]> = [:]
let newEvents: TreeDictionary<Date, [Event]> = [:]
    
newEvents.keys.subtracting(oldEvents.keys)
```

#### Min-max heap
Another tree-based structure that the Swift Collections package provides us is the *Heap* type. The *Heap* type stores comparable elements and allows you to query for the minimal or maximal element quickly.

```swift
struct Event: Identifiable, Comparable {
    static func < (lhs: Event, rhs: Event) -> Bool {
        lhs.priority < rhs.priority
    }
    
    let id = UUID()
    let date: Date
    let priority: Int
}

@Observable final class EventStore {
    typealias Fetch = () async -> [Event]
    
    private(set) var events: Heap<Event>
    private let fetch: Fetch
    
    init(fetch: @escaping Fetch) {
        self.fetch = fetch
    }
    
    var nextEvent: Event? { events.max }
    
    func fetchEvents() async {
        let allEvents = await fetch()
        events.insert(contentsOf: allEvents)
    }
}
```

As you can see in the example above, we fetch the calendar events and populate the heap with them. The *Event* type conforms to the *Comparable* protocol and allows us to get the minimal and maximal elements depending on the event priority.

```swift
@Observable final class EventStore {
    private(set) var events: Heap<Event>
    
    func printEvents() {
        for event in events.unordered {
            print(event)
        }
    }
}
```

You can access the unordered read-only array of elements stored in the *Heap* type whenever needed. Remember that you can't access the sorted collection of items from the heap. It is, after all, a heap.

#### Ordered dictionary and set
How often do you need to access values in a set or dictionary in the order you have added them? Unfortunately, the flat hash table that *Dictionary* and *Set* types use doesn't allow to keep the adding order of elements. The Swift Collection package introduces the *OrderedSet* and *OrderedDictionary* types to solve the issue.

```swift
let letters: OrderedSet = ["a", "b", "c"]
    
for element in letters {
    print(element)
}
    
print(letters[0])
print(letters.contains("b"))
print(letters.isSuperset(of: ["a", "b", "c", "d"]))
```

The *OrderedSet* type allows us to access the element by index like the *Array* type but keeps elements unique.

```swift
printArray(letters.elements) // Array
printSet(letters.unordered) // Set
```

Whenever you need to pass the elements of the *OrderedSet* as an *Array*, you can use the *elements* property, or you can use the *unordered* property whenever you want to extract the plain *Set* of the elements. Remember, the *OrderedSet* type implements most of the functions from the *SetAlgebra* protocol but doesn't conform to it.

```swift
let lettersAndNumbers: OrderedDictionary = [
    "a": 1,
    "b": 2,
    "c": 3
]
    
print(lettersAndNumbers["a"])
print(lettersAndNumbers.elements[0])
```

The *OrderedDictionary* behaves very similarly to the *OrderedSet* type and allows you to access the dictionary both by key and index.

#### Deque
*Deque* is another collection type that the Swift Collections package provides us. *Deque* is almost identical to the *Array* type, except it offers efficient insert and removal to both ends of the collection.

```swift
var deque: Deque = [1, 2, 3, 4]

deque.prepend(0)
deque.append(5)
deque.popFirst()
deque.popLast()
    
deque[0]
```

The *Deque* type implements a double-ended queue, allowing us to insert and remove elements from the ends of the collection at O(1) complexity, which may become very handy when you build any queue functionality in your app.

#### Conclusion
Today, we discovered another great Swift package provided by Apple. The community constantly works on the package and adds more value to it. So, check the documentation and find the valuable collection types that may improve your apps. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week!
