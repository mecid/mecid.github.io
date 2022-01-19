---
title: Microapps architecture in Swift. Feature modules.
layout: post
image: /public/xcode-spm.png
category: Architecture
---

In the first post of the current series, I talked about Swift Package Manager basics and how we can maintain the project with many Swift modules. This week we continue the topic of Microapps architecture by introducing feature modules.

Last week we created a separate module for the design system of our app that contains buttons and other shared UI components. We call them foundation modules because we will import them into many different modules and use their functionality. Another excellent example of the foundation module is the networking layer. We can also extract it into a separate module and import it whenever needed.

In the current post, I want to focus on the feature modules. Feature module provides complete functionality for a dedicated app feature. We can also call them product modules because they usually implement a particular part of the final product. Let's create the first feature module onboarding users on the first app launch. As always, we should start with declaring our module in the Package.swift file.

=====================================================

As you can see, we define the Onboarding module as a separate target with Design System dependency. It allows us to import the Design System module and reuse its functionality. The onboarding screen should present a few items that we define below.

=====================================================

Please look at how we use the public access modifier to make the dedicated parts of code visible outside of the current module. Now we can move to OnboardingView itself.

=====================================================

First, we import the Design System module to use the main button style. Next, we implement the OnboardingView by iterating through onboarding items and presenting them in the vertical stack. We also display a button on the bottom of the screen with the style that we imported from the Design System module.

OK, now we have a separate module representing the onboarding feature. Remember that we should implement all the app logic in the dedicated feature modules. The app target should only provide a thin coordinator layer that instantiates different features and navigates between them.

=====================================================

As you can see in the example above, we have the RootView in the app target that imports both Onboarding and DailySummary modules. RootView doesn't contain any logic. The only thing it does is coordinate between two feature modules.

Dividing the app into many decoupled feature modules allows us to create micro-apps for different user flows and deliver them to the QA team to get early feedback without waiting for other features.

Feature modules can contain more than one screen and should encapsulate the whole feature. For example, it can be a multi-screen checkout feature in the store app. A dedicated team can work on this module and deliver a microapp using TestFlight to test the checkout flow.

This week we learned about the feature modules and how they can improve the development of the big app by delivering a microapp for the dedicated feature set.
