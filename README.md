# Elevate Labs Android notes

Practical guides for Android developers working with Google Play subscriptions, backend entitlements, and app UI states. Each note stands on its own and includes sources, examples, and clear limits.

## Guides

### Google Play subscription access after cancellation

[Read the guide](google-play-subscription-access.md) for an access checklist and a reproducible cancellation-to-expiry test.

### When a screen gets an error, give it something to show

[Read the guide](when-a-screen-gets-an-error-give-it-something-to-show.md) for a Kotlin UI-state example and a focused error-state test checklist. This topic is useful across Android apps and is not specific to our SDK.

### An RTDN arrived. Is the subscription entitlement correct?

[Read the guide](rtdn-is-a-signal-not-an-entitlement.md) for a five-check reconciliation trace and a reproducible expiry test.

## About the project

Elevate Labs is developing an Android Monetization and Entitlement SDK for apps that sell subscriptions through Google Play. The SDK is in development and internal validation; it is not yet a production release. This repository contains technical notes, not a released SDK package.

We share what we have tested, distinguish staging results from general platform guidance, and update examples as the work changes. If you find a technical issue, open an issue with a minimal reproduction. Please leave out purchase tokens, customer identifiers, and private service details.
