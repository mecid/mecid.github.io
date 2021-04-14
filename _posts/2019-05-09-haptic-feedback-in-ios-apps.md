---
title: Haptic feedback in iOS apps
layout: post
category: Interactions
---

Feedback helps people know what an app is doing, discover what they can do next, and understand the results of actions. This week I am going to talk about the Haptic Feedback Engine which provided by Apple in most of the devices.

{% include friends.html %}

On supported devices, haptics provide a way to physically engage users with tactile feedback that gets attention and reinforces actions. Some system-provided interface elements, such as pickers, switches, and sliders, automatically provide haptic feedback as users interact with them. Your app can also ask the system to generate different types of haptic feedback. iOS manages the strength and behavior of this feedback.

Another benefit of the feedback engine is Accessibility. It helps to understand the result of any action without actually watching the screen.

#### UIFeedbackGenerator
Apple provides us with abstract class *UIFeedbackGenerator* for all type haptic feedbacks. We don't need to subclass it in our apps. Instead, we have to use Apple provided ready to use subclasses. There are three classes:
1. [UIImpactFeedbackGenerator](https://developer.apple.com/documentation/uikit/uiimpactfeedbackgenerator)
2. [UISelectionFeedbackGenerator](https://developer.apple.com/documentation/uikit/uiselectionfeedbackgenerator)
3. [UINotificationFeedbackGenerator](https://developer.apple.com/documentation/uikit/uinotificationfeedbackgenerator)

All of them have system predefined haptic feedback patterns, so instead of using custom vibration patterns let's use well-known haptics. All these subclasses share the same logic. First, you need to prepare the haptic engine before using it. The second is the call to the appropriate method to run this haptic. Let's take a look at simple usage of *UINotificationFeedbackGenerator* class.

```swift
class EpisodeViewController: UIViewController {
    private let episodeModelController: EpisodeModelController
    private let feedback = UINotificationFeedbackGenerator()

    init(episodeModelController: EpisodeModelController) {
        self.episodeModelController = episodeModelController
        super.init(nibName: nil, bundle: nil)
    }

    @IBAction func markAsWatched() {
        feedback.prepare()
        episodeModelController.markAsWatched { [weak self] outcome in
            switch outcome {
            case .success:
                self?.feedback.notificationOccurred(.success)
            case .failure:
                self?.feedback.notificationOccurred(.error)
            }
        }
    }
}
```

Here we have a TV show tracking apps screen which maintains an episode. We have a button which should mark the episode as watched on click. As you can see before the request to our API service, we prepare the feedback generator, and when the result comes, we run appropriate feedback type.

Another interesting subclass is *UISelectionFeedbackGenerator*. We can use on the same screen as a result of selection changes. For example, we can have buttons which should fetch next or previous episodes after a click.

```swift
    @IBAction func fetchNext() {
        selectionFeedback.prepare()
        episodeModelController.fetchNext { [weak self] outcome in
            switch outcome {
            case .success:
                self?.selectionFeedback.selectionChanged()
            case .failure:
                self?.feedback.notificationOccurred(.error)
            }
        }
    }
```

As you can see on the code sample above usage of *UISelectionFeedbackGenerator* is very similar to *UINotificationFeedbackGenerator*. We prepare and run the haptic engine.

I want to mention that these classes have UI prefix and we should run them on the main queue. To keep it crash-free I prepared a simple extension which runs methods on the main queue.

```swift
import UIKit

extension UINotificationFeedbackGenerator {
    func safePrepare() {
        DispatchQueue.main.async {
            self.prepare()
        }
    }

    func safeNotificationOccurred(_ type: UINotificationFeedbackGenerator.FeedbackType) {
        DispatchQueue.main.async {
            self.notificationOccurred(type)
        }
    }
}

```

#### Conclusion

Today we discussed a straightforward and accessible feature of iOS SDK. Haptic Feedback can be very useful both for accessible part of your user base and for anyone who engaged in your app. Feel free to follow me on [Twitter](https://twitter.com/mecid) and ask your questions related to this post. Thanks for reading and see you next week!