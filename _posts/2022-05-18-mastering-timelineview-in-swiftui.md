---
title: Mastering TimelineView in SwiftUI
layout: post
category: Mastering SwiftUI views
image: /public/timeline.png
---

TimelineView is a SwiftUI view type that updates its body according to a provided schedule. We used to see SwiftUI views updating its body whenever the data it presents changes. *TimelineView* doesn't follow this rule and allows us to build a super-custom schedule to update its content in a precise way. We will learn how to use *TimelineView* to create time-based views this week.

> To learn more about how SwiftUI updates views, take a look at my ["You have to change mindset to use SwiftUI"](https://swiftwithmajid.com/2019/11/19/you-have-to-change-mindset-to-use-swiftui/) post.

#### Basics
*TimelineView* reevaluates its body on the schedule we provide. Let's look at the quick example where we draw an animated circle for a minute.

```swift
struct ContentView: View {
    var body: some View {
        TimelineView(.animation) { context in
            let date = context.date
            let value = secondsValue(for: context.date)

            Circle()
                .trim(from: 0, to: value)
                .stroke()
        }
    }

    private func secondsValue(for date: Date) -> Double {
        let seconds = Calendar.current.component(.second, from: date)
        return Double(seconds) / 60
    }
}
```

In the example above, we use *TimelineView* with the *animation* schedule. The *animation* schedule is the system-provided scheduler that uses animation duration on the current platform and reevaluates its body very often to provide a nice transition. The second parameter is the *ViewBuilder* closure defining a view that *TimelineView* should draw. It also takes the single parameter called *context*. The *context* contains the date from the scheduler that triggers the update. In our example, we use the *date* field to draw the circle.

#### Cadence
The second field of the *Context* type is the cadence. The cadence represents the rate at which *TimelineView* updates, and it might change many times during the view's lifecycle. For example, running the *TimelineView* on Apple Watch might decrease cadence while the user lowers the wrist. Fortunately, the *Cadence* type conforms to *Comparable* protocol, and we can easily compare them.

```swift
struct ContentView: View {
    var body: some View {
        TimelineView(.animation) { context in
            let date = context.date
            let value = context.cadence <= .live ?
                nanosValue(for: date): secondsValue(for: date)

            Circle()
                .trim(from: 0, to: value)
                .stroke()
        }
    }

    private func secondsValue(for date: Date) -> Double {
        let seconds = Calendar.current.component(.second, from: date)
        return Double(seconds) / 60
    }

    private func nanosValue(for date: Date) -> Double {
        let seconds = Calendar.current.component(.second, from: date)
        let nanos = Calendar.current.component(.nanosecond, from: date)
        return Double(seconds * 1_000_000_000 + nanos) / 60_000_000_000
    }
}
```

Here we use the cadence parameter to understand how fluid our animation should be. The *Cadence* enum provides three cases: *live*, *seconds*, and *minutes*.

#### Schedulers
We touched on the basics of *TimelineView*. Let's move forward and learn about schedulers provided by SwiftUI and how we can build a custom scheduler. SwiftUI provides us with another two schedulers: *everyMinute* and *periodic* scheduler. The *everyMinute* scheduler updates the timeline every minute. The *periodic* scheduler allows us to give a start date and interval, after which another update event should be fired.

```swift
struct ContentView: View {
    var body: some View {
        TimelineView(.periodic(from: .now, by: 5)) { context in
            let date = context.date
            let value = secondsValue(for: context.date)

            Circle()
                .trim(from: 0, to: value)
                .stroke()
        }
    }

    private func secondsValue(for date: Date) -> Double {
        let seconds = Calendar.current.component(.second, from: date)
        return Double(seconds) / 60
    }
}
```

The periodic schedule is good enough to cover almost all the needed cases, but we can also build a custom schedule. We need to create a type conforming to the *TimelineSchedule* protocol and implement a single requirement.

```swift
final class DailySchedule: TimelineSchedule {
    typealias Entries = [Date]

    func entries(from startDate: Date, mode: Mode) -> Entries {
        (1...30).map { startDate.addingTimeInterval(Double($0 * 24 * 3600)) }
    }
}

extension TimelineSchedule where Self == DailySchedule {
    static var daily: Self { .init() }
}

struct ContentView: View {
    var body: some View {
        TimelineView(.daily) { context in
            let value = dayValue(for: context.date)

            Circle()
                .trim(from: 0, to: value)
                .stroke()
        }
    }

    private func dayValue(for date: Date) -> Double {
        let day = Calendar.current.component(.day, from: date)
        return Double(day) / 30
    }
}
```

As you can see in the example above, we have a custom schedule that generates a timeline from the starting point on hourly basis.

#### Conclusion
Today we learned how to use the TimelineView to build views updating with a specific period. It might be helpful while creating timer apps or custom animations. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week!
