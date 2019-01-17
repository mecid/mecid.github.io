---
title: Using protocols as composable extensions
layout: post
---

Today we will talk about using Protocols as composable pieces for our ViewControllers. Protocols and Protocol Extensions are my second favorite Swift feature after Optionals. It helps us to create highly composable and reusable codebase without inheritance. For years we were using inheritance as a gold programming standard. But is it so good? Let's take a look for simple BaseViewController which we used to have in every project.

```swift
import UIKit

class BaseViewController: UIViewController {
    private let activityIndicator = UIActivityIndicatorView(style: .whiteLarge)

    override func viewDidLoad() {
        super.viewDidLoad()
        view.addSubview(activityIndicator)

        activityIndicator.translatesAutoresizingMaskIntoConstraints = false
        NSLayoutConstraint.activate([
            activityIndicator.centerXAnchor.constraint(equalTo: view.safeAreaLayoutGuide.centerXAnchor),
            activityIndicator.centerYAnchor.constraint(equalTo: view.safeAreaLayoutGuide.centerYAnchor)
            ])
    }

    func presenActivity() {
        activityIndicator.startAnimating()
    }

    func dismissActivity() {
        activityIndicator.stopAnimating()
    }

    func present(_ error: Error) {
        let alert = UIAlertController(title: error.localizedDescription, message: nil, preferredStyle: .alert)
        alert.addAction(.init(title: "Cancel", style: .cancel))
        present(alert, animated: true)
    }
}
```

It looks straightforward and usable because most of our ViewControllers need activity indicator while downloading data from the internet and error handling in case of something goes wrong during the data download. But we don't stop with this, and we add more features to BaseViewController over a time. It starts bloating with a lot of general-purpose functions. Here we have at least two main problems:

1. Our BaseViewController breaks the Single Responsibility Principle by implementing all these features in one place. Over time it will turn into Massive-View-Controller, which hard to understand and cover with tests.
2. Every ViewController in our app inherit from BaseViewController to use these all features, and in case of a bug in BaseViewController, we will have this bug in all ViewControllers in our app even if ViewController not using buggy functionality from BaseViewController.

#### Protocols for the rescue.
Protocol Extensions feature was released with Swift 2.0 and bring real power to protocol types which announce new paradigm of programming: Protocol Oriented Programming. I recommend you to watch the [talk](https://developer.apple.com/videos/play/wwdc2015/408/) from WWDC about Protocols and Protocol extensions.

Let's go back to our topic. How can Protocols help us? Let's start by declaring ActivityPresentable Protocol for presenting and dismissing an activity indicator.

```swift
protocol ActivityPresentable {
    func presentActivity()
    func dismissActivity()
}

extension ActivityPresentable where Self: UIViewController {
    func presentActivity() {
        if let activityIndicator = findActivity() {
            activityIndicator.startAnimating()
        } else {
            let activityIndicator = UIActivityIndicatorView(style: .whiteLarge)
            activityIndicator.startAnimating()
            view.addSubview(activityIndicator)

            activityIndicator.translatesAutoresizingMaskIntoConstraints = false
            NSLayoutConstraint.activate([
                activityIndicator.centerXAnchor.constraint(equalTo: view.safeAreaLayoutGuide.centerXAnchor),
                activityIndicator.centerYAnchor.constraint(equalTo: view.safeAreaLayoutGuide.centerYAnchor)
                ])
        }
    }

    func dismissActivity() {
        findActivity()?.stopAnimating()
    }

    func findActivity() -> UIActivityIndicatorView? {
        return view.subviews.compactMap { $0 as? UIActivityIndicatorView }.first
    }
}
```

We extracted presentActivity and dismissActivity method into the particular protocol type. We add default implementation via protocol extension for cases where Type which adopt this protocol is ViewController. It gives us the opportunity of using ViewController methods and properties in our protocol extension.

Let's do the same for error presenting logic.

```swift
protocol ErrorPresentable {
    func present(_ error: Error)
}

extension ErrorPresentable where Self: UIViewController {
    func present(_ error: Error) {
        let alert = UIAlertController(title: error.localizedDescription, message: nil, preferredStyle: .alert)
        alert.addAction(.init(title: "Cancel", style: .cancel))
        present(alert, animated: true)
    }
}
```

Now we have two reusable protocol types which respect the Single Responsibility Principle. We can add them as the extension to any ViewController which need this functionality. The nice thing is that we are adding the only extension which needed in concrete ViewController and not inherits all the stuff from the base ViewController. Here is the usage example of these protocols.

```swift
class ViewController: UIViewController {
    override func viewDidLoad() {
        super.viewDidLoad()
        presentActivity()
    }
}

extension ViewController: ActivityPresentable, ErrorPresentable {}
```

Another opportunity here is that we can easily ignore default implementation of the protocol to implement our customized ActivityIndicator for some of ViewControllers. Let's take a look at the example.

```swift
class CustomViewController: UIViewController {
    override func viewDidLoad() {
        super.viewDidLoad()
        presentActivity()
    }
}

extension CustomViewController: ActivityPresentable {
    func presentActivity() {
        // Custom activity presenting logic
    }

    func dismissActivity() {

    }
}
```

While adopting CustomViewController to ActivityPresentable protocol, we specify the custom implementation of presentActivity and dismissActivity methods.

#### Conclusion
As you can see, we can use protocols as a simple extension for our ViewController type. In the future posts, we will continue using protocols to build reusable parts of ViewController. We will touch associated type, and conditional conformance features to develop more generic data based extensions for ViewControllers. 

Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for the reading and see you next week.