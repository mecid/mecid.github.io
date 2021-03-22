---
title: UI Testing using Page Object pattern in Swift
layout: post
category: Testing
image: /public/ui.png
---

We talked a lot about different design patterns, which help us maintain the codebase by solving various issues. But what about testing? What can we do to keep our UI tests in a maintainable and consistent state? This week we will talk about the Page Object pattern that allows us to build a foundation for our UI tests.

#### Basic UI test
Let's start first by tackling the problems of a simple UI test.

=====================================================

Here is an example of a typical UI test where we have a code that runs the app, propagates some launching arguments, navigates through the app, interacts with your views, and validates the state of UI.

This code has a few downsides. First of all, it has an app launching logic that we will use in most of our UI tests. I think it is better to extract it into the UITestCase class that defines a UI test and handles common logic like app launching and making screenshots at the end of the UI test.

=====================================================

Here we have the new UITestCase class that defines a UI test case and handles typical setup and teardown logic. For example, we have to launch the app before every test and terminate the app after every test. We also make a screenshot of the app whenever the test finishes its work. We keep it only for failing tests. It might be handy to look at the failing test screenshot. Usually, it helps to understand what is wrong with the state of UI.

=====================================================

Now it looks much better but still has other problems. The example above mixes what we do and how we do it. It exposes the things which should be hidden, like accessibility identifiers and interaction logic.

The code above is not reusable. We might need to use the same login flow in other UI tests, but we don't have a way to reuse it. Even if we copy this code and paste it into another UI test, we will have a classical code duplication problem. What if the view hierarchy of the login screen will change in the future? Should I fix all the UI tests that use login flow?

#### Page Object pattern
Page Object is a type that defines all the interactions of the particular screen and provides you all the needed functions to verify the UI state of that screen. This term appeared first in web page testing, which is why it is called Page Object. Let's try to introduce this pattern in our codebase.

=====================================================

In the example above, we have the LoginScreen struct. I decide to use the word screen because we don't have pages in iOS apps, and the screen sounds more familiar.

As you can see, LoginScreen hides all the complexity of the UI test under the hood and provides you user-friendly functions to interact with the login screen.

Methods of a Page Object should return itself or an instance of another Page Object. It allows us to build readable and chainable UI tests. For example, the tapLogin method navigates you to another screen. That's why it is responsible for creating and returning the MessageScreen Page Object.

=====================================================

#### Conclusion
UI tests are expensive and fragile but vital and usable. That's why you should take care of them as much as you take care of your main codebase. The Page Object pattern is a great way to simplify your UI tests and reuse them.