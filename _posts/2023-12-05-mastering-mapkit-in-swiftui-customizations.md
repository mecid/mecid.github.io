---
title: Mastering MapKit in SwiftUI. Customizations.
layout: post
category: Mastering SwiftUI views
image: /public/map4.png
---

Last week, we started a series of posts about the new MapKit API in SwiftUI. We talked about the basics of the new API, and now we can continue the topic by covering the customization part of new APIs. This week, we will learn the customization points of MapKit API in SwiftUI.

{% include friends.html %}

#### Map style
The new MapKit API introduces the mapStyle view modifier, allowing us to customize the style of the data presented on the map.

```swift
struct ContentView: View {
    var body: some View {
        Map {
            // ...
        }
        .mapStyle(.imagery(elevation: .realistic))
    }
}
```

As you can see in the example above, we use the mapStyle view modifier with the imagery style and realistic elevation. Another option for the elevation parameter of the imagery style is flat.

SwiftUI provides us with a set of predefined and configurable map styles. In the previous example, we used a style called imagery. By default, the SwiftUI framework uses the standard style. The standard style allows us to configure the elevation of the map, points of interest we want to include or exclude from the map, and whenever we need to show traffic.

```swift
struct ContentView: View {
    var body: some View {
        Map {
            // ...
        }
        .mapStyle(
            .standard(
                elevation: .flat,
                pointsOfInterest: .excluding([.store]),
                showsTraffic: false
            )
        )
    }
}
```

Another option is the hybrid style, allowing us to display imagery, roads, and road names on the map. The hybrid style also configures elevation, traffic, and points of interest.

```swift
struct ContentView: View {
    var body: some View {
        Map {
            // ...
        }
        .mapStyle(
            .hybrid(
                elevation: .flat,
                pointsOfInterest: .including([.airport]),
                showsTraffic: true
            )
        )
    }
}
```

#### Map interactions
MapKit supports different types of interactions with the map. You can zoom, pan, pitch, and rotate the content on the map. By default, SwiftUI activates all of the available gestures. But you can easily limit available interactions to the list of the preferred ones.

```swift
struct ContentView: View {
    var body: some View {
        Map(interactionModes: [.pan, .pitch]) {
            // ...
        }
    }
}
```

#### Map controls
Whenever you import MapKit in pair with SwiftUI, you will get access to the particular SwiftUI views you can use as map controls. These views include MapScaleView, MapCompass, MapPitchToggle, MapUserLocationButton, and MapZoomStepper views.

```swift
struct ContentView: View {
    var body: some View {
        Map {
            // ...
        }
        .mapControls {
            MapScaleView()
            MapCompass()
        }
    }
}
```

You can use these views in pair with the mapControls view modifier to specify controls for any map instances sharing the same environment in the SwiftUI view hierarchy.

Whenever you place MapScaleView or MapCompass views inside the mapControls view modifier, you allow SwiftUI to control the placement of the map controls. In this case, SwiftUI decides the placement of the control depending on the platform running the app.

As you may have noticed, MapScaleView and other map controls are simple SwiftUI views, which means you can use them outside of the mapControls view modifier anywhere you want. In this case, to bind a map control to a particular map instance, you should use the mapScope view modifier.

```swift
struct MapScopeExample: View {
    @Namespace private var favoritesMap
    var body: some View {
        VStack {
            Map(scope: favoritesMap) {
                // favorite pins
            }
            
            HStack {
                MapScaleView(scope: favoritesMap)
                MapCompass(scope: favoritesMap)
            }
        }
        .mapScope(favoritesMap)
    }
}
```

You can also use the mapControlVisibility view modifier when you need to change the automatic visibility configuration to always visible or hidden.

```swift
struct MapScopeExample: View {
    @Namespace private var favoritesMap
    var body: some View {
        VStack {
            Map(scope: favoritesMap) {
                // favorite pins
            }
            
            HStack {
                MapScaleView(scope: favoritesMap)
                MapCompass(scope: favoritesMap)
                    .mapControlVisibility(.hidden)
            }
        }
        .mapScope(favoritesMap)
    }
}
```

Today, we learned how to customize map presentation in SwiftUI. SwiftUI provides a flexible and easy-to-use API for configuring map controls, styles, and interaction modes. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week!

