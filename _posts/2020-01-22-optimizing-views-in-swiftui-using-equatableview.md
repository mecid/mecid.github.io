---
title: Optimizing views in SwiftUI using EquatableView
layout: post
image: /public/equatable.png
category: View Composition
---

SwiftUI provides us a very fast and easy to use diffing algorithm, but as you might know, diffing is a linear operation. It means that diffing will be very fast for simple layouts and can take some time for a complicated layout. 

{% include friends.html %}

I have good news for you. SwiftUI allows us to replace the standalone diffing algorithm with our custom logic. This week we will talk about optimizing our SwiftUI layouts using the equatable modifier.

#### Diffing in SwiftUI
As you remember, we already talked about diffing in SwiftUI, but let me remind how it works. Whenever you change the source of truth for your views like *@State* or *@ObservableObject*, SwiftUI runs *body* property of your view to generate a new one. As the last step, SwiftUI renders a new view if something changed. The process of calculating a new *body* depends on how deep is your view hierarchy. Happily, we can replace SwiftUI diffing with our simplified version whenever we know the better way to determine changes.

>To learn more about diffing, take a look at ["You have to change mindset to use SwiftUI" post](/2019/11/19/you-have-to-change-mindset-to-use-swiftui/).

#### EquatableView
Sometimes we don't need the true diffing of SwiftUI, or we want to ignore some changes in data, and this is the exact place where we can use the *EquatableView* struct. *EquatableView* struct is a wrapper for a *View*, and it also conforms to *View* protocol. All you need to do to use *EquatableView* is conforming your view to *Equatable* protocol. Let's take a simple look at a good example.

```swift
struct CalendarView: View, Equatable {
    let sleeps: [Date: [Sleep]]
    let dates: [Date]

    var body: some View {
        List {
            ForEach(dates, id: \.self) { date in
                Section(header: Text("\(date, formatter: DateFormatter.mediumDate)")) {
                    ForEach(self.sleeps[date, default: []], id: \.id) { sleep in
                        CalendarRow(sleep: sleep)
                    }
                }
            }
        }.listStyle(GroupedListStyle())
    }
}
```

In the example above, you see the code from my NapBot app. It is a calendar view that represents your sleep per day. I decide to replace SwiftUI diffing with my own by adding *Equatable* conformance. As you can see, it is a straightforward process. You can go further by overriding *== function* and adding your custom logic there. 

```swift
struct CalendarView: View, Equatable {
    let sleeps: [Date: [Sleep]]
    let dates: [Date]

    var body: some View {
        List {
            ForEach(dates, id: \.self) { date in
                Section(header: Text("\(date, formatter: DateFormatter.mediumDate)")) {
                    ForEach(self.sleeps[date, default: []], id: \.id) { sleep in
                        CalendarRow(sleep: sleep)
                    }
                }
            }
        }.listStyle(GroupedListStyle())
    }

    static func == (lhs: Self, rhs: Self) -> Bool {
        lhs.sleeps.count == rhs.sleeps.count
    }
}
```

Now we can wrap our *CalendarView* with *EquatableView*.

**Remember, you have to wrap your view with EquatableView to replace standalone diffing with yours.**

```swift
struct CalendarContainerView: View {
    @EnvironmentObject var store: CalendarStore

    var body: some View {
        EquatableView(
            CalendarView(
                sleeps: store.sleeps,
                dates: store.dates
            )
        ).onAppear(perform: store.fetch)
    }
}
```

#### Equatable modifier 
SwiftUI allows us to avoid wrapping with *EquatableView* by using an equatable modifier. Basically, it does the same thing but in a short way.

```swift
struct CalendarContainerView: View {
    @EnvironmentObject var store: CalendarStore

    var body: some View {
        CalendarView(sleeps: store.sleeps, dates: store.dates)
            .equatable()
            .onAppear(perform: store.fetch)
    }
}
```

#### Container views and equatable rendering views
It is so easy to add *Equatable* conformance to your view when it only renders some data. You even don't need to override *== function*. You can quickly achieve this behavior by extracting your views into *Container* and *Rendering* views. We already talked multiple times on my blog about *Container* and *Rendering* views. *Rendering* views simply take some data and render it. That's it.

*Rendering* views should not contain any logic or state manipulations, and it should delegate them to *Container* views. This separation allows you to make your *Rendering* views conforming *Equatable* in an effortless way.

> To learn more about *Container and Rendering views*, take a look at my ["Introducing Container views in SwiftUI" post](/2019/07/31/introducing-container-views-in-swiftui/).

#### Conclusion
SwiftUI allows us to build our apps in a very new way, where the framework itself applies a lot of magic behind the scene. But I'm delighted that SwiftUI provides so many capabilities to customize default behavior. *EquatableView* can boost performance when *body* computation is longer than your equality check.
I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week!
