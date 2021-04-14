---
title: Navigation with Flow Controllers
layout: post
category: Architecture
---

Last month I started refactoring navigation flow in my pet project. I've been using Coordinator pattern for a while, but now I decide to switch to a more native and simple approach like Flow Controllers. Today we will talk about Flow Controllers and why it is more native than Coordinators.

{% include friends.html %}

#### Coordinators
Coordinator is a plain object which handles the navigation flow. It owns rootViewController, where it pushes next ViewControllers. ViewControllers and Coordinators talk with each other by delegates. It gives us an opportunity to keep ViewControllers reusable, by extracting navigation knowledge from them. More about Coordinator pattern you can read in the original [post by Soroush Khanlou](http://khanlou.com/2015/01/the-coordinator/).

The one huge problem which I have with Coordinator pattern is keeping it in sync with ViewController hierarchy. Every Coordinator has a childCoordinators field which is used to keep navigation sub-flows. Users can finish sub-flow anytime they want by pressing back button in the navigation bar. Coordinators by default can't handle this situation, and our child coordinator will live in childCoordinators array forever, which leads to huge memory leak. 

To fix this problem, we have to implement a navigation controller delegate in a coordinator to understand when the user finishes flow by pressing back button and remove child coordinator from the array. This solution described very well in the original [post](http://khanlou.com/2017/05/back-buttons-and-coordinators/). I think we can avoid this complexity and boilerplate by using Flow Controllers.

#### Flow Controllers
Flow Controller is a UIViewController subclass which handles navigation flow by using ViewController containment feature. Let's dive into the code example. Assume that we have Master-Details flow, where our app navigates from product list screen to product details.

```swift
import UIKit

class ProductsFlowController: UIViewController {
    private let navigation = UINavigationController()

    override func viewDidLoad() {
        super.viewDidLoad()
        let productsVC = ProductListViewController()
        productsVC.delegate = self
        navigation.show(productsVC, sender: self)
        add(navigation)
    }
}

protocol ProductsFlowControllerDelegate: AnyObject {
    func startDetails(for productId: Int)
}

extension ProductsFlowController: ProductsFlowControllerDelegate {
    func startDetails(for productId: Int) {
        let productVC = ProductDetailsViewController(productId: productId)
        navigation.show(productVC, sender: self)
    }
}
```

As you can see, ProductsFlowController creates UINavigationController, add it as a child, then it pushes ProductListViewController to the NavigationController which it owns. It also sets delegate to ProductListViewController which will be used to ask FlowController to show details of the selected product.

Here is my extension which I use to add child ViewControllers to a parent.
```swift 
extension UIViewController {
    func add(_ child: UIViewController) {
        addChild(child)
        view.addSubview(child.view)
        child.didMove(toParent: self)
    }
}
```

#### Handling sub-flows with Flow Controllers
Let's take a look at another example where we have to start sub-flow which handled by separated Flow Controller.

```swift
extension ProductsFlowController: ProductsFlowControllerDelegate {
    func startDetails(for productId: Int) {
        let detailsFlow = DetailsFlowController(productId: productId)
        navigation.show(detailsFlow, sender: self)
    }
}
```

In this example, we start DetailsFlowController which handles another flow. We don't need to add it to childs array as we do it with Coordinators. It is plain UIViewController, as soon as the user presses the back button in the navigation bar, UINavigationController will remove this Flow Controller both from the screen and the memory. As you can see by using UIViewController as FlowControllers, we don't need to deal with the synchronization between flow and visible ViewController. It is coming out of the box from iOS SDK.

#### AppFlowController
Very often we are using UITabBarController as a rootViewController in our app. Let's extract tab configuration from AppDelegate and wrap inside AppFlowController. Here is a quick example of this idea.

```swift
import UIKit

class AppFlowViewController: UIViewController {
    private let tabController = UITabBarController()

    override func viewDidLoad() {
        super.viewDidLoad()
        tabController.viewControllers = [ProductsFlowController()]
        add(tabController)
    }
}

@UIApplicationMain
class AppDelegate: UIResponder, UIApplicationDelegate {
    var window: UIWindow?

    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
        window = UIWindow(frame: UIScreen.main.bounds)
        window?.rootViewController = AppFlowViewController()
        window?.makeKeyAndVisible()
        return true
    }
}
```

As you can see in the example above, we create AppFlowController which creates UITabBarController and populate every tab with separated navigation flow.

#### Conclusion
Today we talked about navigation flow and how we can extract it into Flow Controllers by using ViewController containment feature. It gives us an opportunity to reuse our ViewControllers and make them more testable. We will continue to cover ViewController containment feature in next posts.

Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading and see you next week!