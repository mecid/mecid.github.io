---
title: Glassifying custom SwiftUI views. Groups
layout: post
---

Glassifying custom views can be very easy using the glassEffect view modifier. It is a one-shot view modifier that handles everything for you. But things can become quite complicated when you try to glassify a group of views. Today, we will talk about glassifying groups of views in SwiftUI.

Why is it not enough to use the glassEffect view modifier on a group of views?

======================================================

You can apply the glassEffect view modifier individually on a per-view basis. Unfortunately, it is not the way to go. Glass should reflect other glass elements around. To achieve that, you should still use the glassEffect view modifier, but also wrap the group of views with the GlassEffectContainer.

======================================================

All the glass effects inside the GlassEffectContainer can interact with each other and reflect the light. GlassEffectContainer not only improves the performance of rendering multiple glass effects but also allows us to morph in and out shapes of glasses.

There is a special spacing parameter on the GlassEffectContainer allowing you to control the distance between two glasses that will morph after this distance.

======================================================

The spacing parameter allows us to tune the spacing amount in the layout after which glass shapes should morph in or out. SwiftUI allows us to animate the morphing behavior easily, as many other properties of views.

There is another option allowing us to combine glass shapes together without relying on spacing. The glassEffectUnion view modifier allows us to combine a set of glass effects with the same identifier. It might be useful whenever views have too big a distance between them to rely on the spacing parameter and you want to manually indicate that glass effects of these particular views must be combined.

======================================================

The glassEffectUnion view modifier combines glasses only when they have the same effect types, similar shapes, and identifiers. Keep this in mind when you try to combine glass effects manually.
