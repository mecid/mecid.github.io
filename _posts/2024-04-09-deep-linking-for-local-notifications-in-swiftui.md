---
title: Deep linking for local notifications in SwiftUI
layout: post
image: /public/swiftui.png
---

Notifications are crucial for keeping users engaged with your app. Almost all of my apps provide notifications that not only launch the app but also navigate to different parts of the app. Today, I want to share how I build deep links for local notifications in my apps.

{% include friends.html %}

SwiftUI introduced the *onOpenURL* view modifier to handle universal links in our apps. Here is a quick example showing how to use the *onOpenURL* view modifier.

```swift
@main
struct MyApp: App {
    @State private var offerShown = false
    
    var body: some Scene {
        WindowGroup {
            ContentView()
                .onOpenURL { url in
                    guard let host = url.host(), host == "offer" else {
                        return
                    }
                    offerShown = true
                }
                .sheet(isPresented: $offerShown) {
                    OfferView()
                }
        }
    }
}
```

As you can see in the example above, we use the *onOpenURL* view modifier to parse the opened URL and provide navigation inside the app by displaying the offer sheet.

Before using the *onOpenURL* view modifier, you must register the URL scheme you want to open in the app. Go to project settings, choose your target, and register a URL scheme in the URL types section.

Now, you are ready to handle links in your app. You can try to test how the app handles links by opening a link in Safari, for example, *myapp://offer*.

I want to schedule a notification that displays a special offer to the user after the first launch in 30 minutes. I wrote an extension for *UNUserNotificationCenter* to make it easier.

```swift
extension UNUserNotificationCenter {
    func addOfferNotification() {
        let content = UNMutableNotificationContent()
        content.title = String(localized: "offerTitle")
        content.body = String(localized: "offerBody")
        content.userInfo = ["url": "myapp://offer"]
        
        let request = UNNotificationRequest(
            identifier: "offer", 
            content: content,
            trigger: UNTimeIntervalNotificationTrigger(
                timeInterval: 1800,
                repeats: false
            )
        )
        
        add(request)
    }
}
```

As you can see in the example above, I've created the *addOfferNotification* function to schedule a notification in 30 minutes. I provide the title and body of the notification and include the URL field in the *userInfo* dictionary.

The idea is to intercept notifications with the URL field in the *userInfo* dictionary and open the URL instead of launching the app. The *UNUserNotificationCenter* type allows us to provide a delegate to handle the moment the user taps the notification.

The only place where you should set the delegate for the *UNUserNotificationCenter* type is the *AppDelegate*'s *willFinishLaunchingWithOptions* method. So, we need to define *AppDelegate* for our SwiftUI app.

```swift
final class AppDelegate: NSObject, UIApplicationDelegate {
    func application(_ application: UIApplication, willFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey : Any]? = nil) -> Bool {
        UNUserNotificationCenter.current().delegate = self
        return true
    }
}

extension AppDelegate: UNUserNotificationCenterDelegate {
    func userNotificationCenter(_ center: UNUserNotificationCenter, didReceive response: UNNotificationResponse) async {
        guard
            let urlString = response.notification.request.content.userInfo["url"] as? String,
            let url = URL(string: urlString)
        else { return }
        await UIApplication.shared.open(url)
    }
}
```

As you can see, we define the *AppDelegate* type conforming to the *UIApplicationDelegate* protocol. We also conform to the *UNUserNotificationCenterDelegate* protocol and implement the *didReceive* function. Our app will call this function whenever the user taps a notification while the app is in the background.

It is the best place to verify whenever our notification contains the URL to launch and open the URL using a shared instance of the *UIApplication* type. We can ignore other notifications that don't include the URL field.

```swift
@main
struct MyApp: App {
    @UIApplicationDelegateAdaptor(AppDelegate.self) private var delegate
    @AppStorage("launches") private var launches = 0
    @Environment(\.scenePhase) private var scenePhase
    @State private var offerShown = false
    
    var body: some Scene {
        WindowGroup {
            ContentView()
                .onOpenURL { url in
                    guard let host = url.host(), host == "offer" else {
                        return
                    }
                    offerShown = true
                }
                .sheet(isPresented: $offerShown) {
                    OfferView()
                }
        }
        .onChange(of: scenePhase) {
            switch scenePhase {
            case .background where launches == 1:
                UNUserNotificationCenter.current().addOfferNotification()
            case .active:
                launches += 1
            default:
                break
            }
        }
    }
}
```

In the example above, we use the *UIApplicationDelegateAdaptor* property wrapper to define an *AppDelegate* for the SwiftUI app. We also observe the scene phase to schedule the offer notification after the first launch.

Today, we learned how to implement deep linking functionality in pair with local notifications in a SwiftUI-based app. I hope you enjoy the post. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading, and see you next week!
