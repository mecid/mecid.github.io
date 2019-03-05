---
title: Dependency injection in Swift with Protocols
layout: post
---

There are a lot of third-party libraries which provide Dependency Injection for Swift apps. In my opinion, Swift has a powerful type system which gives us the ability to create type-safe Dependency Injection techniques easily. Today we will talk about creating Dependency Injection in Swift with the power of protocols.

#### Protocol Composition
As I said before Protocols are one of my favorite language features in Swift, especially Protocol Composition, which gives us an opportunity to compose multiple Protocols together in one type. Let's take a look at the implementation of the "Service Locator" pattern in Swift and how we can improve it with the usage of Protocol Composition. 

```swift
protocol HasUserDefaults {
    var userDefaults: UserDefaults { get }
}

protocol HasUrlSession {
    var session: URLSession { get }
}

protocol HasHealthStore {
    var store: HKHealthStore { get }
}

struct Dependencies: HasUserDefaults, HasUrlSession, HasHealthStore {
    let userDefaults: UserDefaults
    let session: URLSession
    let store: HKHealthStore

    init(
        userDefaults: UserDefaults = .standard,
        session: URLSession = .shared,
        store: HKHealthStore = .init()
    ) {
        self.userDefaults = userDefaults
        self.session = session
        self.store = store
    }
}
```

Here we have a bunch of protocols which describes our dependencies. Dependencies struct contains all of our dependencies in the app. Generally, we can create and store our Dependencies struct instance in AppDelegate or root Coordinator/FlowController. Now let's take a look at the usage of our dependency container.

```swift
class ViewController: UIViewController {
    typealias Dependencies = HasUserDefaults & HasUrlSession

    private let userDefaults: UserDefaults
    private let session: URLSession

    init(dependencies: Dependencies) {
        userDefaults = dependencies.userDefaults
        session = dependencies.session
        super.init(nibName: nil, bundle: nil)
    }
}
```

Here we have ViewController which describes its dependencies via Typealias and Protocol Composition. In the Init method, we easily extract our dependencies into field variables. All we need is the passing instance of our Dependencies struct to ViewController, and ViewController will be able to access only defined in Typealias dependencies. Next time when your ViewController will need another dependency all you need to do is add it to Typealias and extract it into the variable. You don't need to change the creation of ViewController, because you already pass all the dependencies.

```swift
extension Dependencies {
    static var mocked: Dependencies {
        return Dependencies(
            userDefaults: UserDefaults(suiteName: #file),
            session: MockedUrlSession(),
            store: MockedHealthStore()
        )
    }
}
```

The example above shows how we can create the mocked version of dependencies to use it for Unit-Testing.

#### Abstract Factory
Another option for Dependency Injection is the "Abstract Factory" pattern. I love to use it to extract the creation of complex objects like ViewControllers and its dependencies. Let's take a look at the Swift version of the "Abstract Factory" pattern by using Protocols and Extensions.

```swift
protocol DependencyFactory {
    func makeHealthService() -> HealthService
    func makeSettingsSevice() -> SettingsService
}

struct Dependencies {
    private let healthStore: HKHealthStore
    private let userDefaults: UserDefaults

    init(
        healthStore: HKHealthStore = .init(),
        userDefaults: UserDefaults = .standard
    ) {
        self.healthStore = healthStore
        self.userDefaults = userDefaults
    }
}

extension Dependencies: DependencyFactory {
    func makeHealthService() -> HealthService {
        return HealthService(store: healthStore)
    }

    func makeSettingsSevice() -> SettingsService {
        return Settings(defaults: userDefaults)
    }
}
```

Here we have DependencyFactory protocol which describes factory methods for every dependency in our app. We also have small Dependencies struct which stores low-level dependencies. By using the extension, we add DependencyFactory protocol conformance to Dependencies struct. Now let's take a look at ViewControllerFactory, which describes ViewController creation process. 

```swift
protocol ViewControllerFactory {
    func makeCalendarViewController() -> CalendarViewController
    func makeDayViewController() -> DayViewController
    func makeSettingsViewController() -> SettingsViewController
}

extension Dependencies: ViewControllerFactory {
    func makeCalendarViewController() -> CalendarViewController {
        return CalendarViewController(
            healthService: makeHealthService()
        )
    }

    func makeDayViewController() -> DayViewController {
        return DayViewController(
            healthService: makeHealthService(),
            settingsService: makeSettingsSevice()
        )
    }

    func makeSettingsViewController() -> SettingsViewController {
        return SettingsViewController(
            settingsService: makeSettingsSevice()
        )
    }
}
```

We use ViewControllerFactory to create every ViewController in our app, for more complex apps we can have more ViewController factories based on the user flow. Here we also use an extension to add protocol conformance to Dependencies struct. It is time to see how we can use these factories while using Coordinators or FlowControllers.

```swift
protocol FlowControllerDelegate {
    func startSettings()
}

class FlowController: UIViewController {
    private let factory: ViewControllerFactory
    init(factory: ViewControllerFactory) {
        self.factory = factory
    }

    override func viewDidLoad() {
        super.viewDidLoad()
        add(factory.makeCalendarViewController())
    }
}

extension FlowController: FlowControllerDelegate {
    func startSettings() {
        show(factory.makeSettingsViewController(), sender: self)
    }
}
```

We can create an instance of Dependencies struct in AppDelegate and pass it to the main FlowController in the app. By extracting creation of ViewControllers into factories, we keep our FlowControllers small and responsible only for controlling user-flow.

#### Conclusion
Today we discussed two Dependency Injection techniques. Both of them use Swift language features without any third-party dependencies. Just take a look at them and choose which will work better for you.

Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading and see you next week!