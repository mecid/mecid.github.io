---
title: Animating changes in UITableView and UICollectionView
layout: post
category: Interactions
---

Most of our apps present lists or grids of some data by using UITableView or UICollectionView. Often users can update this list by using Pull-to-Refresh technique or by pressing the update button. Everybody knows how to update UITableView by calling the reloadData method on the tableView instance. But what about animation? ReloadData method is invalidating the current items provided by data source and draws new ones without any animation. Today we will talk about animating data changes in UITableView and UICollectionView.

{% include friends.html %}

#### UICollectionView/UITableView animation API
iOS SDK provides particular methods like insertRows, deleteRows, and moveRow which give us an opportunity to animate changes in our data source. But only a few apps use this technique to update UITableView and here are two reasons for that.

1. It's hard to calculate what kind of changes applied to your data after the update.
2. There is a particular order on animating changes. You should call deletions first and insertions next, and after that make your movement changes.

I think it's a little bit complicated, and that's why not so many apps use this API. But we are going to find an easy way of animating changes.

#### Understanding the changes
There are a lot of third-party libraries which provide fast diffing algorithm implementations. Here is a list of the most famous.

* [DeepDiff](https://github.com/onmyway133/DeepDiff)
* [Differ](https://github.com/tonyarnold/Differ)
* [ListDiff](https://github.com/lxcid/ListDiff)

I prefer the second one. It works super fast with my dataset. As a part of this post, we will use Differ library, but you can choose any library you want because usually, it provides a similar API.

```swift
let old = ["1", "2", "3"]
let new = ["4", "3", "2"]

let diff = old.diff(new)
```

#### Hashable
First of all, we have to implement Hashable protocol on our model types, thanks to Swift it is super easy, all you need to do is adding Hashable protocol to your type declaration and compiler will generate all required code. Here is a sample from my pet project which is a TV show tracking app.

```swift
struct Show: Hashable {
    let title: String
    let year: Int
    let ids: Ids
    let overview: String
    let runtime: Int
    let certification: String
    let network: String
    let country: String
    let trailer: String?
    let homepage: String?
    let status: String
    let rating: Double
    let votes: Int
    let commentCount: Int
    let updatedAt: Date
    let language: String
    let availableTranslations: [String]
    let genres: [String]
    let airedEpisodes: Int
}
```

#### Animation API
Most of the diffing libraries provide UICollectionView and UITableView extensions, which takes oldData, newData, and animate the changes from one state to another. All you need to do is put new data to your data source and instead of calling the reloadData method call animateRowChanges method. This method will handle all the animations based on your changes.

First of all, let's use the technique which we discussed before [to hide our third-party dependencies with extensions](/2019/02/13/hiding-third-party-dependencies-with-protocols-and-extensions).

```swift
import UIKit
import Differ

extension UICollectionView {
    func reloadChanges<T: Collection>(from old: T, to new: T) where T.Element: Equatable {
        animateItemChanges(oldData: old, newData: new)
    }
}

extension UITableView {
    func reloadChanges<T: Collection>(from old: T, to new: T) where T.Element: Equatable {
        animateRowChanges(oldData: old, newData: new)
    }
}
```

We create an extension which adds a reloadChanges method to UICollectionView and UITableView. With the help of this extension, you can easily switch libraries or your implementation by making changes in a single file.

```swift
func render(_ newData: [Show]) {
    let oldData = dataSource.posters
    dataSource.posters = newData
    collectionView.reloadChanges(from: oldData, to: newData)
}
```

![ShowBot-animation](/public/showbot-animation.gif)

#### Conclusion
Today we discussed how it is easy to add delight to our apps by animating changes. This kind of animations makes apps feel natural and fluid. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading and see you next week!