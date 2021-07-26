---
title: Using MapKit with SwiftUI
layout: post
category: Mastering SwiftUI views
image: /public/map1.png
---

This year we saw that Apple started using SwiftUI across macOS and iOS to build notification center and widgets. Another great addition was a SwiftUI integration for frameworks that Apple provides us like MapKit and AVKit. This week we will talk about *Map* view that SwiftUI provides us as soon as you import both MapKit and SwiftUI.

{% include friends.html %}

#### Basics
As you might remember, I already covered using *MKMapView* in SwiftUI by using the *UIViewRepresentable* protocol. This protocol easily allows us to wrap *UIView* and use it in SwiftUI. You don't need to do it anymore, starting iOS 14, macOS 11, tvOS 14, and watchOS 7 SwiftUI provides us *Map* view. Let's take a look at how easily we can use it.

> To learn more about using [UIKit views in SwiftUI](/2020/01/29/using-uikit-views-in-swiftui/), take a look at my dedicated post.

```swift
import SwiftUI
import MapKit

struct MapExample: View {
    @State private var region = MKCoordinateRegion(
        center: CLLocationCoordinate2D(
            latitude: 25.7617,
            longitude: 80.1918
        ),
        span: MKCoordinateSpan(
            latitudeDelta: 10,
            longitudeDelta: 10
        )
    )

    var body: some View {
        Map(coordinateRegion: $region)
    }
}
```

As you can see in the example above, all you need to do is constructing *Map* view and passing it the binding for an instance of *MKCoordinateRegion*. SwiftUI will update the binding as soon as the user changes the visible region on the map by zoom or pan gestures. You can also update the camera by assigning a new value to the state property that controls the visible region.

```swift
import SwiftUI
import MapKit

struct MapExample: View {
    @State private var region = MKCoordinateRegion(
        center: CLLocationCoordinate2D(
            latitude: 25.7617,
            longitude: 80.1918
        ),
        span: MKCoordinateSpan(
            latitudeDelta: 10,
            longitudeDelta: 10
        )
    )

    var body: some View {
        VStack {
            Map(coordinateRegion: $region)

            Button("zoom") {
                withAnimation {
                    region.span = MKCoordinateSpan(
                        latitudeDelta: 100,
                        longitudeDelta: 100
                    )
                }
            }
        }
    }
}
```

*Map* view supports the animation system of SwiftUI, which allows us to change the visible boundaries of the map with animation. Let's move further and take a look at the customization points that map view provides us.

```swift
import SwiftUI
import MapKit

struct MapExample: View {
    @State private var userTrackingMode: MapUserTrackingMode = .follow
    @State private var region = MKCoordinateRegion(
        center: CLLocationCoordinate2D(
            latitude: 25.7617,
            longitude: 80.1918
        ),
        span: MKCoordinateSpan(
            latitudeDelta: 10,
            longitudeDelta: 10
        )
    )

    var body: some View {
        Map(
            coordinateRegion: $region,
            interactionModes: MapInteractionModes.all,
            showsUserLocation: true,
            userTrackingMode: $userTrackingMode
        )
    }
}
```

As you can see map view provides us quite a few customization points. Let's discuss them one by one.

1. *coordinateRegion* represents the currently visible region of the map view. You can easily change it by updating the value of your binding. Remember that it is an animatable parameter of the view.
2. *interactionModes* allows us to set allowed gestures for our map view. For example, it might be only zoom or pan gestures. In our case, we use all the available gestures to interact with the map.
3. *showsUserLocation* is the boolean parameter that allows us to control whenever we want to show the user's current location on the map or not. Remember that you have to request permission to access the location. 
4. *userTrackingMode* is the way to configure map tracking mode. It describes whenever we want to follow the user as soon as location changes.

#### Annotations 
We usually use map screens to display the points of interest in our apps. SwiftUI allows us to place the annotations on the map by using simple views. To provide annotation items, we have to use another initializer that accepts the random access collection of identifiable elements.

```swift
import SwiftUI
import MapKit

struct City: Identifiable {
    let id = UUID()
    let coordinate: CLLocationCoordinate2D
}

struct MapExample: View {
    @State private var cities: [City] = [
        City(coordinate: .init(latitude: 40.7128, longitude: 74.0060)),
        City(coordinate: .init(latitude: 37.7749, longitude: 122.4194)),
        City(coordinate: .init(latitude: 47.6062, longitude: 122.3321))
    ]

    @State private var userTrackingMode: MapUserTrackingMode = .follow
    @State private var region = MKCoordinateRegion(
        center: CLLocationCoordinate2D(latitude: 25.7617, longitude: 80.1918),
        span: MKCoordinateSpan(latitudeDelta: 10, longitudeDelta: 10)
    )

    var body: some View {
        Map(coordinateRegion: $region, annotationItems: cities) { city in
            MapAnnotation(
                coordinate: city.coordinate,
                anchorPoint: CGPoint(x: 0.5, y: 0.5)
            ) {
                Circle()
                    .stroke(Color.green)
                    .frame(width: 44, height: 44)
            }
        }
    }
}
```

SwiftUI provides us *MapAnnotation* struct that we can use to build our annotation view. It accepts the center coordinate of the annotation, its anchor point, and the @*ViewBuilder* closure to create a custom view.

SwiftUI also provides us two standard views that we can use to display our points of interest. These are *MapMarker* and *MapPin*.

```swift
var body: some View {
    Map(coordinateRegion: $region, annotationItems: cities) { city in
        MapPin(coordinate: city.coordinate, tint: .green)
    }
}
```
#### Fit the map with MKMapRect 
There is another initializer for the *Map* view that accepts *MKMapRect* instead of *MKCoordinateRegion* to control the visible part of the map.

```swift
final class PinsViewModel: ObservableObject {
    @Published var mapRect = MKMapRect()
    let cities: [City]

    init(cities: [City]) {
        self.cities = cities
    }

    func fit() {
        let points = cities.map(\.coordinate).map(MKMapPoint.init)
        mapRect = points.reduce(MKMapRect.null) { rect, point in
            let newRect = MKMapRect(origin: point, size: MKMapSize())
            return rect.union(newRect)
        }
    }
}

struct PinsView: View {
    @ObservedObject var viewModel: PinsViewModel

    var body: some View {
        Map(
            mapRect: $viewModel.mapRect,
            annotationItems: viewModel.cities
        ) { city in
            MapPin(coordinate: city.coordinate, tint: .accentColor)
        }.onAppear(perform: viewModel.fit)
    }
}
```
In the example above, we use *MKMapRect* to calculate the rectangle that is able to show all the pins on the map at once.

#### Conclusion
I am happy to see that Apple provides us more views to use in SwiftUI. I hope to see 100% coverage of UIKit views in SwiftUI anytime in the future. But let's learn the things that we have right now. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this article. Thanks for reading, and see you next week!
