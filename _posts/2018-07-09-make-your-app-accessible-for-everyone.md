---
layout: post
title: Make your app accessible for everyone
category: Accessibility
image: /public/accessibility.jpeg
---

Last few months I was working on implementing Accessibility support for my app. During this years WWDC I’ve visited all Accessibility related sessions and Labs to improve my knowledge and catch some best practices from Apple Engineers. So now I’m finished my work and finally ready to share with you story of my Accessibility way.

{% include friends.html %}

First of all, I would like to mention that Apple had done a great job with Accessibility framework. Most of the things handled by the system without our action. All the UIKit controls have Accessibility support out of the box. *UIButton, UILabel, UISegmentedControl, UISwitch,* etc. are ready to be used by assistive technologies like VoiceOver, Switch Control, etc. If you’re not familiar with VoiceOver technology, I suggest watching this [talk](https://developer.apple.com/videos/play/wwdc2018/226/) from last WWDC event.

Let’s check out my app called [CardioBot](https://cardiobot.swiftwithmajid.com). It is Heart Rate analyzer for iOS and watchOS. CardioBot has a massive amount of data presented in charts and other visual ways, which are custom views and don’t support Accessibility out of the box. Good news is that it is easy to add Accessibility support to our custom views.

![CardioBot](/public/cardiobot.jpg)

#### Group elements inside cells
As you can see on the first screen, I have a *UICollectionView* which represents the current month with cells for every day. DayCell is a class which uses two labels to describe daily information: date and average heart rate. I am also using label background color to express the heart rate zone.
VoiceOver users are moving through the app by using right and left swipe gestures. These gestures navigate VoiceOver focus on next and previous elements respectively. So VoiceOver focus will switch from label to label and read the text content of every label. However, this is not very easy for users; they have to do many swipes to understand the data. I would like to ask VoiceOver to focus on cell entire, not on every label inside and read the specially formatted string as cell content.

```swift
self.isAccessibilityElement = true
self.accessibilityLabel = "\(date), \(value) - \(status)"
self.accessibilityTraits |= UIAccessibilityTraitButton
```

Setting *isAccessibilityElement* property to true on *UICollectionViewCell* makes it appear as a single Accessibility element. I am also set a formatted string to *accessibilityLabel* property to make it easier to understand data which presented by this cell. To describe the action which user can do on the cell, I am using *UIAccessibilityTraitButton* trait. This trait indicates to VoiceOver that it is a button and can be selected or activated by the user. So when VoiceOver focused on the DayCell, it pronounces the phrase like:
“December 1st, 69 — Resting. Button.” When the user swipes right to next element VoiceOver pronounces: “December 2nd, 78 — High resting. Button.”

#### Charts
On the second screen, you can see that I use a custom view to display a chart of heart rate zones during the day. To make it accessible just set to true *isAccessibilityElement* property, set formatted string as *accessibilityLabel* and add *UIAccessibilityTraitStaticText* trait to *accessibilityTraits* property of the custom view. I wrote small Formatter class which generates a string from heart rate data. So when VoiceOver focuses on chart view, it pronounces phrase like: “Low — 12%, Resting — 64%, High Resting — 16%, Elevated — 7%”.

On the third screen, CardioBot displays a detailed heart rate chart with a 4-minute interval. I would like to make it possible to focus VoiceOver on every bar inside the chart, pronounce the time and value for a selected period and easily move between the bars. As I say before this is a custom view with the overridden *drawInRect* method which handles plotting logic that’s why we don’t have accessibility support out of the box here. Happily, we have a particular API for this kind of situations. We can set an array of Accessibility elements inside a custom view.

```swift
let elements = statistics.enumerated().map {
    let frame = CGRect(
        x: 0, 
        y: CGFloat($0 * Layout.barHeight), 
        width: bounds.width, 
        height: Layout.barHeight
    )
    
    let element = UIAccessibilityElement(accessibilityContainer: self)
    element.accessibilityLabel = $1.time
    element.accessibilityValue = "\(Int($1.value)), \($1.status)"
    element.accessibilityFrameInContainerSpace = frame
    return element
}

self.accessibilityElements = elements
```

Pay attention to *accessibilityFrameInContainerSpace* property which is a frame inside chart view which used to highlight focused area by VoiceOver.

#### Traits
Accessibility trait is a type of accessibility element which used by VoiceOver to describe available actions on it. For example, *UIAccessibilityTraitAdjustable* trait used to describe adjustable elements like *UISlider*. When VoiceOver focused on the adjustable view user can use swipe up and swipe down gestures to increment or decrement the value of the component. That’s why you have to implement *accessibilityIncrement* and *accessibilityDecrement* methods of *UIAccessibilityAction* protocol to handle these gestures.

Another useful trait is *UIAccessibilityTraitHeader*. Marking section header labels with this trait add the opportunity for VoiceOver to recognize sections and navigate between them by using rotor navigation.

Day details screen has various sections like Summary, HRV, Sleep, Weekly sleep, Workout. For example every morning I would like to check out how healthy was my sleep and I don’t need to navigate to the Sleep section through all these elements. To use the rotor, rotate two fingers on your iOS device's screen as if you're turning a dial. VoiceOver will say the first rotor option. Keep rotating your fingers to hear more options. Lift your fingers to choose an option. After selecting the headings option, flick your finger up or down on the screen to use it.

#### Exit gesture
VoiceOver users use two fingers Z-gesture to go back to the previous screen or close modal window. Handling of this gesture implemented by default for *UINavigationController* and *UIStoryboard* segues, and you don’t need to do something to achieve this behavior. However, sometimes we are showing *UIViewController* by calling *present()* method, in this case, we have to override *accessibilityPerformEscape* method on presented *UIViewController* and call *dismiss()* manually to exit.

#### Hints
Another interesting property on *UIAccessibility* protocol is *accessibilityHint*. This property used by VoiceOver to describe the result of action on accessibility element. For example, I use it to indicate to users that activating chart control display the detailed chart. Be careful and don’t put here some valuable data because the user can disable this feature and VoiceOver will ignore hint labels. Follow these guidelines to describe your hints in a clean way.

#### Small things matter
Besides the heart rate analysis CardioBot also shows some activity summary like total step count, walking distance and calories. I use HealthKit to calculate all this stuff. Some of my users have motor disabilities and use a wheelchair to move around. For this users, I apply another query to calculate wheelchair pushes instead of steps. It takes me about 30 minutes to implement this feature, but my users were pleased.

#### Conclusion
I suggest you watch this two sessions from WWDC18:

1. [VoiceOver: App Testing Beyond The Visuals](https://developer.apple.com/videos/play/wwdc2018/226/)
2. [Deliver an Exceptional Accessibility Experience](https://developer.apple.com/videos/play/wwdc2018/230/)

Accessibility isn’t a feature or a “nice to have.” **It’s a necessity**. So let’s make your app accessible for everyone.
