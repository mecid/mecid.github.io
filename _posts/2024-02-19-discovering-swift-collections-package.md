---
title: Discovering Swift Collections package
layout: post
---

I want to continue the topic of the valuable Swift Package that I use in my apps. This we will talk about the Swift Collections package, providing us with a bunch of helpful collection types that Swift language doesn't include out of the box.

The Swift Collections package contains a few collection types that may help you improve the performance of your apps if you apply them whenever needed instead of using generic Array, Dictionary, and Set types. The Swift Collections package lives on Github, where you can find it and your project.

#### Tree-based dictionary and set
Dictionary and Set types that Swift language provides us store values in a single flat hash table that you copy on every write or mutation. The Swift Collection package introduces TreeDictionary and TreeSet types implementing Compressed Hash-Array Mapped Prefix Trees. In other words, TreeDictionary and TreeSet types hold values in the tree-based structure, allowing the efficient updating of only the needed branches.

Imagine a calendar app where you store an event array per date and use the standard Dictionary type. You might need to implement paging and load events per visible month and store them in an instance of the Dictionary type. While the user scrolls through months, your app loads a bunch of events and copies the whole dictionary on every load, even when previously loaded events didn't change.

=====================================================

For this case, the Swift Collections package introduces the TreeDictionary and TreeSet types that link the unchanged parts with the changed branches under the hood without copying the whole dictionary into the memory. The TreeDictionary type provides us with the very same APIs that the Dictionary type has and optimizes memory for us under the hood.

The TreeDictionary is still a struct, but the implementation uses the UnsafeMutablePointer type to access memory and mutate it directly without copying on write. Another benefit of the TreeDictionary and TreeSet types is the optimized way to compare because of their tree-based nature. Usually, they handle this operation in a constant time.

=====================================================

#### Min-max heap
Another tree-based structure that the Swift Collections package provides us is the Heap type. The Heap type stores comparable elements and allows you to query for the minimal or maximal element quickly.

=====================================================

As you can see in the example above, we fetch the calendar events and populate the heap with them. The Event type conforms to the Comparable protocol and allows us to get the minimal and maximal elements depending on the event priority.

=====================================================

You can access the unordered read-only array of elements stored in the Heap type whenever needed. Remember that you can't access the sorted collection of items from the heap. It is, after all, a heap.

#### Ordered dictionary and set
How often do you need to access values in a set or dictionary in the order you have added them? Unfortunately, the flat hash table that Dictionary and Set types use doesn't allow to keep the adding order of elements. The Swift Collection package introduces the OrderedSet and OrderedDictionary types to solve the issue.

=====================================================

The OrderedSet type allows us to access the element by index like the Array type but keeps elements unique.

=====================================================

Whenever you need to pass the elements of the OrderedSet as an Array, you can use the elements property, or you can use the unordered property whenever you want to extract the plain Set of the elements. Remember, the OrderedSet type implements most of the functions from the SetAlgebra protocol but doesn't conform to it.

=====================================================

The OrderedDictionary behaves very similarly to the OrderedSet type and allows you to access the dictionary both by key and index.

#### Deque
Deque is another collection type that the Swift Collections package provides us. Deque is almost identical to the Array type, except it offers efficient insert and removal to both ends of the collection.

=====================================================

The Deque type implements a double-ended queue, allowing us to insert and remove elements from the ends of the collection at O(1) complexity, which may become very handy when you build any queue functionality in your app.

#### Conclusion
Today, we discovered another great Swift package provided by Apple. The community constantly works on the package and adds more value to it. So, check the documentation and find the valuable collection types that may improve your apps.
