# What happens to subscription access after cancellation?

Canceling a Google Play subscription usually stops the next renewal. The customer may still have access until the current paid period ends. Removing access as soon as a cancellation arrives can lock them out early; relying on an old “active” flag can leave access open after expiry.

The useful distinction is between **subscription status** and **access right now**. A real-time developer notification (RTDN) tells your backend that something changed. The backend can then check the current subscription through the Play Developer API and update its entitlement decision. [Google's subscription lifecycle guide](https://developer.android.com/google/play/billing/lifecycle/subscriptions) and [RTDN reference](https://developer.android.com/google/play/billing/rtdn-reference) explain the underlying states.

## A practical access checklist

| Verified state | Access to test | Helpful message in the app |
| --- | --- | --- |
| Active | Available | Premium is active |
| Canceled, with paid time remaining | Available until verified expiry | Access continues until the shown date |
| Grace period | Available while the verified grace state permits it | Payment needs attention |
| Account hold | Unavailable | Payment needs correction |
| Expired | Unavailable | Premium has ended |
| Pending purchase | No new access yet | Payment has not completed |

This table is a starting point for tests. Your final decision also needs the verified expiry, product and account mapping, and any other valid entitlement sources. Google advises granting benefits only after a purchase is verified and complete. [Play Billing integration guide](https://developer.android.com/google/play/billing/integrate)

## Try the cancellation-to-expiry path

You'll need a Play Console test subscription, a matching installed app, a Play license tester, and a backend that can query purchases.subscriptionsv2.get. The [Play Billing test guide](https://developer.android.com/google/play/billing/test) covers license testers and accelerated renewals.

1. Begin with no active entitlement. Note what the app and backend show.
2. Buy the subscription with **Test card, always approves**. Check that backend verification happens before the paid feature opens.
3. Cancel in Play Store while paid time remains. Reopen the app. Access should continue until the verified expiry.
4. Let the test subscription expire. Monthly test renewals run on an approximately five-minute schedule, with some variation. Compare the Play state, RTDN receipt, follow-up API read, saved entitlement and app access. If no other valid entitlement exists, the paid feature should close after expiry.
5. Force-stop and relaunch the app. Confirm that an old local state does not reopen the feature.

A small timeline makes problems easier to diagnose: record the time, Play state, RTDN event ID, saved entitlement, final access decision and what the app showed. Please remove purchase tokens and user identifiers before sharing it. If a notification is delayed, an expiry check or app restore can still bring the backend up to date. A successful HTTP response from the RTDN endpoint alone does not establish that access was updated.

In our October 2026 staging tests, access continued after cancellation and ended after expiry. A later corrected staging build also kept its premium demo locked after expiry. Those are results for the tested paths, not a claim that every Play lifecycle case or production operation has been validated.

The main check is the paid feature itself: does it remain available during the paid period and close when the verified entitlement ends?


## Another Android note

[When a screen gets an error, give it something to show](when-a-screen-gets-an-error-give-it-something-to-show.md) covers explicit error states and a focused UI test checklist.
