# What access should an Android subscription keep after cancellation?

A Google Play subscription can be **canceled and still entitled to access**. Cancellation generally stops a future renewal; it does not necessarily end the current paid period. An app that treats every cancellation signal as “remove premium now” may lock out someone who still has time remaining. Trusting an old local “active” flag after expiry can make the opposite mistake.

A reliable access decision uses the current, verified subscription state. Google recommends checking purchase state through the Play Developer API. Real-time developer notifications (RTDN) tell the backend that something changed, but the notification is not the complete state; query the Developer API before reconciling access. [Subscription lifecycle](https://developer.android.com/google/play/billing/lifecycle/subscriptions) · [RTDN reference](https://developer.android.com/google/play/billing/rtdn-reference)

## A small access model

| Verified state | Access decision to test | What the app should explain |
| --- | --- | --- |
| Active | Grant access | Premium is active |
| Canceled but still within the paid period | Keep access through verified expiry | Cancellation is scheduled; show when access ends |
| Grace period | Grant access while the verified grace state permits it | Payment needs attention |
| Account hold | Withhold paid access | Payment needs correction; explain recovery |
| Expired | Withhold paid access | Premium has ended |
| Pending purchase | Do not grant a new entitlement yet | Payment has not completed |

This is a test checklist, not a substitute for Google's current API definitions or your app's policy. Do not infer access from a notification name alone. Check the canonical subscription state, expiry, product mapping, account association and any other valid entitlement sources. A pending purchase should not unlock a paid feature. [Play Billing integration guidance](https://developer.android.com/google/play/billing/integrate)

## Reproduce cancellation and expiry

You need a Play Console subscription product and base plan, a matching installed app, a Play license tester, and a backend able to query purchases.subscriptionsv2.get and record its entitlement decision. Use Play test instruments. Google's [test guide](https://developer.android.com/google/play/billing/test) explains license testers and accelerated renewals.

1. Start with no active entitlement for the test account. Record the app state and backend record.
2. Buy the test subscription with **Test card, always approves**. Check that the backend verifies the purchase before granting access. Record Play state, backend state and app state with timestamps.
3. Cancel the subscription in Play Store while the paid period remains. Reopen the app. Expect access to remain until the verified expiry. The cancellation notification alone is not an immediate revocation command.
4. Wait for the accelerated test period to end. A monthly test subscription renews on an approximately five-minute cadence, with timing variation; check the actual canonical state rather than assuming an exact minute. Correlate RTDN receipt, Developer API read, durable source, combined entitlement and app state. Once Play reports expiry and no other valid entitlement exists, expect paid access to end.
5. Force-stop and relaunch. Check that the app still shows no paid access. This catches a stale local cache restoring an old decision.

Capture a row for each transition: time, Play state, RTDN event ID, backend source state, combined entitlement and app-visible access. Redact purchase tokens and user identifiers. If RTDN is delayed or absent, use a reconciliation path around expiry and on app restore. An HTTP success from the RTDN endpoint proves delivery handling, not the final access decision.

## Check the paid feature itself

A purchase screen ending in “Subscribed” proves only one point in the lifecycle. Test the actual paid feature before cancellation, after cancellation, after expiry and after relaunch. It can be correct for the app to say “canceled” while access remains active until the paid period ends. After expiry, a fresh backend check should lock the feature.

In our October 2026 staging Android tests, backend-verified access remained after cancellation and ended after expiry. A later corrected staging build also showed its premium demo locked after expiry. These observations cover the tested paths; they do not establish every Play lifecycle edge case or production reliability.

The practical rule: model **subscription status** and **right to access** separately. Refresh the access decision from a verified backend at purchase, app restore and relevant RTDN changes, then test transitions with Play, backend and app evidence.
