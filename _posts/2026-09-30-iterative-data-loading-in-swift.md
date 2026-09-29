---
title: Iterative data loading in Swift
layout: post
image: /public/sequence.png
category: Architecture
---

How to deal with complex features that might need 40–50 async requests to populate the screen? How might we arrange these requests to prevent overwhelming the Cooperative Thread Pool, or how could we refine concurrent tasks? This week, we will talk about the iterative data loading approach that I use in my CardioBot app.

{% include friends.html %}

Let’s talk a bit about today screen of my CardioBot app, it runs in average 40-50 HealthKit requests to build a representation of your recent health data. It fetches and analyze your activity, sleep, workouts, recovery, vitals, etc. 

```swift
struct MetricsSnapshot: Hashable, Sendable {
    var heartPoints: HeartPointsSnapshot
    var cardioFitness: CardioFitnessSnaphot
    var workouts: [WorkoutSnapshot]

    // ...

    var hrv: HRVSnapshot
    var restingHeartRate: RestingHeartRateSnapshot
    var sleeps: [SleepSnapshot]

    // ...
}

struct MetricsService {
    let health: HealthService

    func fetch(inside interval: DateInterval) async throws -> MetricsSnapshot {
        async let heartPoints = fetchHeartPoints(inside: interval)
        async let cardioFitness = fetchCardioFitness(inside: interval)
        async let workouts = fetchWorkouts(inside: interval)
        // ...
        async let hrv = fetchHRV(inside: interval)
        async let restingHeartRate = fetchRestingHeartRate(inside: interval)
        async let sleeps = fetchSleeps(inside: interval)
        // ...

        return try await MetricsSnapshot(
            heartPoints: heartPoints,
            cardioFitness: cardioFitness,
            workouts: workouts,
            // ...
            hrv: hrv,
            restingHeartRate: restingHeartRate,
            sleeps: sleeps
            // ...
        )
    }

    private func fetchHeartPoints(inside interval: DateInterval) async throws -> HeartPointsSnapshot {
        // ...
    }

    //  ...
}
```

Assume that we run fifty HealthKit queries using async let; what does it mean for our app? Will it run all of them concurrently? No, Swift defines a Cooperative Thread Pool where our asynchronous tasks run.

The number of threads in the pool is usually limited to your CPU’s cores, which prevents thread explosion. So, it means Swift will allocate a lot of memory for 50 asynchronous tasks, but pushes them step by step by limiting the concurrent count.

Another issue with this approach is that we should wait for all of these tasks to update the screen, which might need a large amount of time. That’s why I decided to fetch my data iteratively in a controlled number of steps.

Therefore, we can model our data fetching using the AsyncSequence type. The AsyncSequence protocol is designed for sequential and iterative access to its elements, which fits our use case.

```swift
struct MetricsSequence: AsyncSequence {
    let health: HealthService
    let interval: DateInterval

    func makeAsyncIterator() -> Iterator {
        Iterator(health: health, interval: interval)
    }

    struct Iterator: AsyncIterator {
        let health: HealthService
        let interval: DateInterval

        enum Step: CaseIterable {
            case heartPoints
            case workouts
            case cardioFitness
            // ...
            case sleep
            case hrv
            case restingHeartRate
            // ...
        }

        private var snapshot = MetricsSnapshot()
        private var steps = Step.allCases.makeIterator()

        mutating func next() async -> MetricsSnapshot? {
            switch steps.next() {
            case .heartPoints:
                snapshot.heartPoints = await fetchHeartPoints(inside: interval)
            case .workouts:
                snapshot.workouts = await fetchWorkouts(inside: interval)
            case .cardioFitness:
                snapshot.cardioFitness = await fetchCardioFitness(inside: interval)
            case .sleep:
                snapshot.sleeps = await fetchSleeps(inside: interval)
            case .hrv:
                snapshot.hrv = await fetchHRV(inside: interval)
            case .restingHeartRate:
                snapshot.restingHeartRate = await fetchRestingHeartRate(inside: interval)
	    // ...
            default:
                return nil
            }

            return Task.isCancelled ? nil : snapshot
        }
    }
}
```

As you can see in the example above, we introduce the MetricsSequence type conforming to the AsyncSequence protocol. The only requirement of this protocol is the makeAsyncIterator function returning some instance of AsyncIterator. The AsyncIterator is also a protocol with a single requirement, the async next function returning the element.

We also define the Step enum representing our steps of loading, and this is the place where you can group your data loading in sections or keep them ungrouped. We conform Step enum to the CaseIterable protocol, it allows us to get all cases in an array and create an iterator over this array.

The real work happens inside the next function of our Iterator type. We check the current step, run async helper functions for the particular step, mutate our instance of the MetricsSnapshot and return the accumulated result. We also handle Cooperative Cancellation and return nil whenever the task is already cancelled. 

Another point: you should keep in mind that AsyncIterator should return nil when there is nothing more to return; nil means end of the sequence. That’s why we return nil in the default case when there is no remaining step to fetch.

```swift
@Observable final class MetricsViewModel {
    private(set) var metrics = MetricsSnapshot()

    private let health: HealthService
    public init(health: HealthService) {
        self.health = health
    }

    func fetch(inside interval: DateInterval) async {
        let metricsSequence = MetricsSequence(health: health, interval: interval)

        for await metrics in metricsSequence {
            self.metrics = metrics
        }
    }
}
```

As you can see in the example above, we use the for await to iterate over the AsyncSequence and update the state on every iteration. This way, we keep our feature updating on every step of data loading. We can go further and tune our MetricsSequence to start loading data with the particular step. For example, it might be a section visible to the user at the very moment.

Running dozens of asynchronous requests at once might look like the simplest solution, but it doesn’t necessarily provide the best user experience. In data-heavy screens, how and when we deliver the results can be just as important as how quickly we fetch them. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week!
