---
title: Using UIKit views in SwiftUI
layout: post
image: /public/map.png
category: View Composition
---

A few weeks ago, we talked about building views like [*PagerView*](/2019/12/25/building-pager-view-in-swiftui/) and [*BottomSheetView*](/2019/12/11/building-bottom-sheet-in-swiftui/) from scratch in SwiftUI. SwiftUI is pretty young and misses some components that we expect to have out of the box. But it provides all the needed APIs to build whatever we want. However, sometimes we need to reuse *UIKit* views instead of making the SwiftUI versions. This week I want to talk to you about using *UIKit* views in SwiftUI.

{% include friends.html %}

#### UIViewRepresentable
One of the good examples of *UIKit* views that we don't want to recreate in SwiftUI is *MKMapView*. Do you have any ideas on how to implement it in SwiftUI from scratch? Happily, we don't need to do that. We can easily use *MKMapView* in SwiftUI by simply creating a wrapper view.

SwiftUI provides us *UIViewRepresentable* protocol that allows us to wrap *UIKit* views and use them from SwiftUI views. Let's take a look at how we can cover *MKMapView* to use it in SwiftUI.

```swift
struct MapView: UIViewRepresentable {
    typealias UIViewType = MKMapView

    func makeUIView(context: UIViewRepresentableContext<MapView>) -> MKMapView {
        MKMapView()
    }

    func updateUIView(_ uiView: MKMapView, context: UIViewRepresentableContext<MapView>) {
    }
}
```

As you can see in the example above, we create a *MapView* struct that conforms to the *UIViewRepresentable* protocol. *UIViewRepresentable* protocol has two requirements: *makeUIView* and *updateUIView* methods.

SwiftUI calls *makeUIView* only one time when it creates a view hierarchy. Whenever you change the state of the view, SwiftUI calls *updateUIView* method to update the view according to state changes.

Let's ignore state changes for now and just create an empty *MKMapView*. Finally, we can use our *MapView* in SwiftUI.

```swift
struct RootView: View {
    var body: some View {
        MapView()
            .edgesIgnoringSafeArea(.all)
    }
}
```

#### State and Binding
*UIViewRepresentable* view can store a state or has a binding like any other SwiftUI components. Let's refactor our *MapView* to accept a binding for a center coordinate via init method.

```swift
struct MapView: UIViewRepresentable {
    typealias UIViewType = MKMapView

    @Binding var center: CLLocationCoordinate2D

    func makeUIView(context: UIViewRepresentableContext<MapView>) -> MKMapView {
        MKMapView()
    }

    func updateUIView(_ uiView: MKMapView, context: UIViewRepresentableContext<MapView>) {
        uiView.setCenter(center, animated: true)
    }
}
```

As you can see in the example above, we finally implemented *updateUIView*. We want to center our map view whenever the binding of the center location changed.

#### Coordinator
*UIKit* views usually have delegates that allow us to handle some user interactions like cell selection in *UITableView* or visible region changes in *MKMapView*. What if we want to handle these actions in SwiftUI? Fortunately, *UIViewRepresentable* provides a **Coordinator** object that allows us to deal with delegates, data sources, and other *UIKit* stuff. The coordinator is a bridge between your *UIKit* and SwiftUI views. Let's see how we can use it.

```swift
struct MapView: UIViewRepresentable {
    typealias UIViewType = MKMapView

    @Binding var center: CLLocationCoordinate2D

    func makeUIView(context: UIViewRepresentableContext<MapView>) -> MKMapView {
        let mapView = MKMapView()
        mapView.delegate = context.coordinator
        return mapView
    }

    func updateUIView(_ uiView: MKMapView, context: UIViewRepresentableContext<MapView>) {
        uiView.setCenter(center, animated: true)
    }

    func makeCoordinator() -> MapView.Coordinator {
        Coordinator(self)
    }

    final class Coordinator: NSObject, MKMapViewDelegate {
        private let mapView: MapView

        init(_ mapView: MapView) {
            self.mapView = mapView
        }

        func mapView(_ mapView: MKMapView, regionDidChangeAnimated animated: Bool) {
            self.mapView.center = mapView.centerCoordinate
        }
    }
}
```

*UIViewRepresentable* has an optional requirement to implement *makeCoordinator* method. This method should create and return a coordinator object. We can fulfill all the needed delegate methods using this object. As you can see in the example above, we want to update our binding whenever the user moves the map. We might need to add some pins based on location changes.

#### Conclusion
This week we learned how to use *UIKit* views in SwiftUI. SwiftUI is a great framework, but sometimes we need to reuse *UIKit* classes that we had for years in *iOS SDK*. Fortunately, it is very effortless to do with help of *UIViewRepresentable* protocol. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week!
