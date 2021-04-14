---
title: Mastering MVVM on iOS
layout: post
category: Architecture
---

There are a plenty of posts on the internet about app architectures in the iOS development world. Today I will show some tips for using MVVM architecture while developing iOS apps. I am not going to show other architectures if you need them there is a great [post](https://medium.com/ios-os-x-development/ios-architecture-patterns-ecba4c38de52).
The main problem of Apple MVC is mixed responsibility, which leads to the appearance of some kinds of problems such as Massive-View-Controller.

{% include friends.html %}

#### Why MVVM?
We should accept that *UIViewController* is the main component in Apple’s iOS SDK and all the actions are started and built across this entity. Despite the name, it is more view than a classic Controller (or Presenter) from MVC(or MVP), because of callbacks like *viewDidLoad*, *viewWillLayoutSubviews*, and other view related methods. That is the reason why we should ignore the **controller** keyword in the name and use it as View, where the role of real controller takes the view model.

View model is the full data representation of the view. Every view should hold only one view model instance. Generally, view model uses a manager to fetch data and transforms it into necessary format. Let’s dive into examples:

```swift
import Foundation

class ItemsViewModel {
    var items: [Item] = []
    var error: Error?
    var refreshing = false
    
    private let dataManager: DataManager
    init(dataManager: DataManager) {
        self.dataManager = dataManager
    }
    
    func fetch(completion: @escaping () -> Void) {
        refreshing = true
        dataManager.fetchItems { [weak self] (items, error) in
            self?.items = items ?? []
            self?.error = error
            self?.refreshing = false
            completion()
        }
    }
}
```

Here we have view model that fetches items via *DataManager* and holds it inside some variable. It also holds an error and refreshing state, which brings opportunity to build UI with all required conditions.

```swift
import UIKit

class ItemsViewController: UIViewController {
    @IBOutlet private weak var tableView: UITableView!
    private var viewModel: ItemsViewModel
    
    init(viewModel: ItemsViewModel) {
        self.viewModel = viewModel
        super.init(nibName: nil, bundle: nil)
    }
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        viewModel.fetch { [weak self] in
            self?.tableView.reloadData()
        }
    }
}

extension ItemsViewController: UITableViewDataSource {
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return viewModel.items.count
    }
}
```

As you can see above we have *ItemsViewController* that represents a list of items in *UITableView*. It holds view model instance and asks it to fetch data in the *viewDidLoad* callback. We are also passing closure, which reloads *UITableView* as soon as the data is fetched. *UITableViewDataSource* methods also use the view model to get the count of cells.

#### Reactive Bindings
Bindings between view and view model is the main idea of MVVM architecture, where developers can work on view model code and designers can work on view in Interface Designer. In the first example, we used closures, because there is no binding ability in iOS SDK out of the box. In real life applications, you can use some of popular FRP extensions, like ReactiveCocoa, RxSwift or Bond. I prefer Bond library for its simplicity.

```swift
import Bond

class ItemsViewModel {
    let items = Observable<[Item]>([])
    let error = Observable<Error?>(nil)
    let refreshing = Observable<Bool>(false)
    
    private let dataManager: DataManager
    init(dataManager: DataManager) {
        self.dataManager = dataManager
    }
    
    func fetch() {
        refreshing.value = false
        dataManager.fetchItems { [weak self] (items, error) in
            self?.items.value = items ?? []
            self?.error.value = error
            self?.refreshing.value = false
        }
    }
}
```

This is the same *ItemsViewModel*, but now we use reactive programming to observe changes.

```swift
class ItemsViewController: UIViewController {
    @IBOutlet private weak var tableView: UITableView!
    private let activityIndicator = ActivityIndicatorView()
    private var viewModel: ItemsViewModel
    
    init(viewModel: ItemsViewModel) {
        self.viewModel = viewModel
        super.init(nibName: nil, bundle: nil)
    }
    
    override func viewDidLoad() {
        super.viewDidLoad()
        setupUI()
        bindViewModel()
        viewModel.fetch()
    }
    
    func bindViewModel() {
        viewModel.refreshing.bind(to: activityIndicator.reactive.isAnimating)
        viewModel.items.bind(to: self) { strongSelf, _ in
            strongSelf.tableView.reloadData()
        }
    }
    
    func setupUI() {
        view.addSubview(activityIndicator)
    }
}

extension ItemsViewController: UITableViewDataSource {
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return viewModel.items.value.count
    }
}
```

Let’s look at *bindViewModel* method, here we bind view model to view. Whenever refreshing value changes it sets activityIndicator’s *isAnimating* property. Or when items are changed, *UITableView* reloads. As you can see, reactive bindings simplify code in most cases.

#### ViewModel Composition
Sometimes we have complex views with multiple data sources. For example, Instagram user profile where we have some information about the user and all photos related to this user. Good job here is separating this logic into two or more view models. But we have a rule: every view should have only one view model. In this case, the best option is using view model composition. Let’s look at examples:

```swift
import Bond
import ReactiveKit

class UserProfileViewModel {
    let refreshing = Observable<Bool>(false)
    let username = Observable<String>("")
    let photos = Observable<[Photos]>([])
    
    private let userViewModel: UserViewModel
    private let photosViewModel: PhotosViewModel
    
    init(userManager: UserManager, photoManager: PhotoManager) {
        userViewModel = UserViewModel(manager: userManager)
        photosViewModel = PhotosViewModel(manager: photoManager)
        
        userViewModel.username.bind(to: username)
        photosViewModel.photos.bind(to: photos)
        combineLatest(userViewModel.refreshing, photosViewModel.refreshing)
            .map { $0 || $1 }
            .bind(to: refreshing)
    }
    
    func fetch() {
        userViewModel.fetch()
        photosViewModel.fetch()
    }
}

class UserViewModel {
    let refreshing = Observable<Bool>(false)
    let username = Observable<String>("")
    
    func fetch() {
        refreshing.value = true
        manager.fetch(user: id) { [weak self] (user, error) in
            self?.username.value = "@" + user.username
            self?.refreshing.value = false
        }
    }
}

class PhotosViewModel {
    let refreshing = Observable<Bool>(false)
    let photos = Observable<[Photo]>([])
    
    func fetch() {
        refreshing.value = true
        manager.fetch(for user: id) { [weak self] (photos, error) in
            self?.photos.value = photos ?? []
            self?.refreshing.value = false
        }
    }
}
```

As you can see, we have *UserProfileViewModel* which holds two another view models and collects data from them. We also have the refreshing state which combines two refreshing states of internal view models. The second important point on line 36, where view model formats data into the required form. View just has to bind components to view model and show data as is.

#### Conclusion
View model is really nice and simple way of separating presentation logic into another entity, which helps us to avoid Massive-View-Controller and keep things easier to control and covered with Unit Tests. That is where we go, simple and testable architecture.