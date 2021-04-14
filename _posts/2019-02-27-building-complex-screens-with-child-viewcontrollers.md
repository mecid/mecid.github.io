---
title: Building complex screens with Child ViewControllers
layout: post
category: View Composition
---

Container view controllers are a way to combine the content from multiple ViewControllers into a single user interface. Child ViewControllers are one of the undervalued features of iOS SDK. We use it every day by use of UINavigationController or UITabBarController. Last week we talked about using ViewController containment feature to create [FlowControllers](/2019/02/20/navigation-with-flow-controllers/). But today we are going to discuss how to use this feature to build complex screens.

{% include friends.html %}

#### Complex screens
CardioBot is Heart Rate tracker app on which I was working last two years. It uses HealthKit to read Heart Rate values, make some calculations and present Heart Rate analysis for every day in a nice way. Here is Day screen of CardioBot app.

![CardioBot](/public/cardiobot.jpg)

Every day has several sections like Average, Summary, Sleep, Workout, Weekly sleep, etc. If I build this screen as a single ViewController for sure, it will be a Massive-View-Controller. Instead, let's extract every section in separated ViewController and combine them in parent ViewController by using ViewController containment. It gives me the opportunity to make my ViewControllers small and testable. Another achievement here is reusability. I want to use SummaryViewController as Today extension, so extracting it in another ViewController is what I need.

#### StackViewController
Let's create base ViewController for our day screen. First of all, it should be able to manage the dynamic count of child ViewControllers with the opportunity to scroll the screen if items too many. The second needed feature is the ability to hide/show Child ViewControllers with animation. UIStackView embedded in UIScrollView is the best candidate here. Animating views inside by switching isHidden property is super easy. Another benefit here is support for intrinsicContentSize by UIStackView. We need to add proper constraints to child ViewController's root view to make it self sizing, and UIStackView will take care of the right positioning of this view.

```swift
import UIKit

class StackViewController: UIViewController {
    private let scrollView = UIScrollView()
    private let stackView = UIStackView()

    override func viewDidLoad() {
        super.viewDidLoad()
        view.addSubview(scrollView)
        scrollView.addSubview(stackView)
        setupConstraints()
        stackView.axis = .vertical
    }

    private func setupConstraints() {
        scrollView.translatesAutoresizingMaskIntoConstraints = false
        stackView.translatesAutoresizingMaskIntoConstraints = false

        NSLayoutConstraint.activate([
            scrollView.leadingAnchor.constraint(equalTo: view.safeAreaLayoutGuide.leadingAnchor),
            scrollView.trailingAnchor.constraint(equalTo: view.safeAreaLayoutGuide.trailingAnchor),
            scrollView.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor),
            scrollView.bottomAnchor.constraint(equalTo: view.safeAreaLayoutGuide.bottomAnchor),
            stackView.leadingAnchor.constraint(equalTo: scrollView.leadingAnchor),
            stackView.trailingAnchor.constraint(equalTo: scrollView.trailingAnchor),
            stackView.topAnchor.constraint(equalTo: scrollView.topAnchor),
            stackView.bottomAnchor.constraint(equalTo: scrollView.bottomAnchor),
            stackView.widthAnchor.constraint(equalTo: view.safeAreaLayoutGuide.widthAnchor)
            ])
    }
}

extension StackViewController {
    func add(_ child: UIViewController) {
        addChild(child)
        stackView.addArrangedSubview(child.view)
        child.didMove(toParent: self)
    }

    func remove(_ child: UIViewController) {
        guard child.parent != nil else {
            return
        }

        child.willMove(toParent: nil)
        stackView.removeArrangedSubview(child.view)
        child.view.removeFromSuperview()
        child.removeFromParent()
    }
}
```

In the example above, I create StackViewController which has UIStackView embedded in UIScrollView. I added all needed constraints to make UIScrollView working correctly with UIStackView and understand its real content size. We also have here two public methods which we will use to add or remove child ViewControllers to UIStackView.

#### Day Screen
Now we can implement our Day Screen which extends from StackViewController and populates its view with child ViewControllers. Here is a source code.

```swift
class DayViewController: StackViewController {
    override func viewDidLoad() {
        super.viewDidLoad()
        setupUI()
    }

    private func setupUI() {
        let today = Date()
        let calendarVC = CalendarViewController(date: today)
        calendarVC.delegate = self

        add(calendarVC)
        add(SummaryViewController(date: today))
        add(SleepViewController(date: today))
        add(WorkoutViewController(date: today))
    }
}

extension DayViewController: CalendarViewControllDelegate {
    func dateSelected(_ date: Date) {
        reloadDay(with: date)
    }
}
```

As you can see, DayViewController hosts four child ViewControllers. It also acts as a delegate for CalendarViewController, which presents the current week. As soon as any date selected in the calendar, it passes the date via delegate to parent ViewController, which job is asking child ViewControllers to reload data.

#### Conclusion
We used to create one ViewController per screen, but sometimes this rule leads to buggy Massive-View-Controllers. Today we discussed how we could break this rule and compose complex screens from as many child ViewControllers as we need.

Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading and see you next week!