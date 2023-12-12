---
title: Mastering MapKit in SwiftUI. Camera.
layout: post
---

In this post, we will continue the topic of the new MapKit API in SwiftUI. We will cover one of the most critical cases of displaying a map. This week, we will learn about camera position and map bounds.

{% include friends.html %}

#### Map bounds
The new MapKit API introduces the *MapCameraBounds* type, allowing us to limit the bounds of the map view. The *MapCameraBounds* type has a few initializers that we can use to create camera bounds from the instance of *MKMapRect* or *MKCoordinateRegion*.

```swift
extension CLLocationCoordinate2D {
    static let newYork: Self = .init(
        latitude: 40.730610,
        longitude: -73.935242
    )
}

let rect = MKMapRect(
    origin: MKMapPoint(.newYork),
    size: MKMapSize(width: 1, height: 1)
)

struct ContentView: View {
    var body: some View {
        Map(bounds: MapCameraBounds(centerCoordinateBounds: rect)) {
            Marker("New York", monogram: Text("NY"), coordinate: .newYork)
        }
    }
}
```

As you can see in the example above, we use the *MKMapRect* type to define the visible bounds of the map that user can't leave by using any interaction.

 To create an instance of the *MKMapRect* type, we should call the initializer with *origin* and *size* parameters. We can use any instance of the *CLLocationCoordinate2D* type to define an origin point. The second parameter must be an instance of the *MKMapSize*, representing the width and height in map points.

Now, we can use an instance of the *MKMapRect* type to pass into the initializer of the *MapCameraBounds* type to limit our map to a particular rectangle. We can also allow users to zoom in or out to a limited amount of meters using *maximumDistance* and *minimumDistance* parameters of the *MapCameraBounds* initializer.

```swift
struct ContentView: View {
    var body: some View {
        Map(
            bounds: MapCameraBounds(
                centerCoordinateBounds: rect,
                minimumDistance: 10,
                maximumDistance: 100
            )
        ) {
            Marker("New York", monogram: Text("NY"), coordinate: .newYork)
        }
    }
}
```

You may have a set of coordinates you want to zoom in and limit to the rectangle displaying these markers. In this case, you can create an instance of the *MKMapRect* type per coordinate and use the *union* function on the *MKMapRect* type to create a rectangle including all the coordinates.

```swift
let coordinates: [CLLocationCoordinate2D] = [.newYork, .sanFrancisco, .seattle]
let rect = coordinates
    .map { MKMapRect(origin: .init($0), size: .init(width: 1, height: 1)) }
    .reduce(MKMapRect.null) { $0.union($1) }
```

We discussed how to use the *MKMapRect* in pair with the *MapCameraBounds* type to limit our map to a particular rectangle. The *MKMapRect* uses map points to represent a rectangle. *MKMapPoint* uses the 2D projection of the map on a flat surface to calculate x and y on the map. You can use *x*, *y*, and *coordinate* properties of the *MKMapPoint* type to convert coordinates to map points and back.

Whenever you want to use latitude and longitude deltas instead of map points, you can use the *MKCoordinateRegion* type. It provides functionality similar to *MKMapRect* but operates on other units.

#### Map camera position
The MapKit provides the *MapCameraPosition* type that we can use for two-way binding of the recently visible camera position. We can create an instance of the *MapCameraPosition* type by passing *MKMapRect*, *MKCoordinateRegion*, *MKMapItem*, *CLLocationCoordinate2D*, etc.

```swift
struct ContentView: View {
    @State private var position: MapCameraPosition = .rect(
        MKMapRect(
            origin: MKMapPoint(.newYork),
            size: MKMapSize(width: 1, height: 1)
        )
    )
    
    var body: some View {
        Map(
            position: $position,
            bounds: MapCameraBounds(
                centerCoordinateBounds: rect,
                minimumDistance: 10,
                maximumDistance: 100
            )
        ) {
            Marker("New York", monogram: Text("NY"), coordinate: .newYork)
        }
        .onAppear {
            position = .camera(.init(centerCoordinate: .sf, distance: 0))
        }
    }
}
```

We can also use the *MapCameraPosition* type to ask for a map view to follow the user location.

```swift
struct ContentView: View {
    @State private var position: MapCameraPosition = .automatic
    
    var body: some View {
        Map(
            position: $position,
            bounds: MapCameraBounds(
                centerCoordinateBounds: rect,
                minimumDistance: 10,
                maximumDistance: 100
            )
        ) {
            Marker("New York", monogram: Text("NY"), coordinate: .newYork)
        }
        .onChange(of: position) {
            print(position.positionedByUser)
            print(position.camera)
            print(position.region)
            print(position.rect)
        }
        .onAppear {
            position = .camera(.init(centerCoordinate: .newYork, distance: 0))
        }
    }
}
```

As I said before, we can use *MapCameraPosition* for two-way binding, which means we can query an instance of the *MapCameraPosition* type to read some data.

```swift
struct ContentView: View {
    @State private var position: MapCameraPosition = .automatic
    
    var body: some View {
        Map(
            position: $position,
            bounds: MapCameraBounds(
                centerCoordinateBounds: rect,
                minimumDistance: 10,
                maximumDistance: 100
            )
        ) {
            Marker("New York", monogram: Text("NY"), coordinate: .newYork)
        }
        .onChange(of: position) {
            print(position.positionedByUser)
            print(position.camera)
            print(position.region)
            print(position.rect)
        }
        .onAppear {
            position = .camera(.init(centerCoordinate: .newYork, distance: 0))
        }
    }
}
```

As you can see in the example above, we use an instance of the *MapCameraPosition* to access the recent camera, region, rectangle, etc, of the map. All of the mentioned fields are optional and will be non-nil values if the particular instance of the *MapCameraPosition* type is used.

There is also the *positionedByUser* property. It is a boolean value defining whenever the camera is positioned by the user or positioned by the developer programmatically.

Today, we learned how to manage the map camera position using the new *MapCameraPosition* type, part of the new rich MapKit API.
