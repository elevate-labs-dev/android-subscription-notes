# An RTDN arrived. Is the subscription entitlement correct?

A Google Play real-time developer notification (RTDN) tells your backend that a purchase state changed. It does not contain the complete current subscription state. Google recommends querying the Play Developer API after an RTDN and using the returned subscription resource to update your backend. [RTDN reference](https://developer.android.com/google/play/billing/rtdn-reference) · [Subscription lifecycle](https://developer.android.com/google/play/billing/lifecycle/subscriptions)

A successful HTTP response from your notification endpoint is useful delivery evidence. By itself, it does not show whether the correct account's entitlement was updated, whether another valid purchase still grants access, or whether the app has refreshed and enforced the decision.

## Follow the event through five checks

| Check | Ask | Keep as evidence |
| --- | --- | --- |
| Delivery | Did the endpoint receive and acknowledge this message? | Message ID, event time, notification type, response code |
| Current Play state | Did the backend query the current subscription after the notification? | API result category, subscription state, expiry time, query time |
| Purchase source | Was the state saved against the right app account and product? | Redacted account reference, product ID, source state and update time |
| Entitlement | Does the account have any other valid purchase that grants the same access? | Aggregate state, valid-until time, decision reason |
| App | Did the app refresh that decision and enforce it on the paid feature? | App state after refresh or relaunch, feature result |

Keep these records correlated with timestamps. A message ID identifies a notification; it does not by itself prove that the same purchase was processed twice. A notification type describes an event; it is not the full current entitlement.

Before sharing logs, remove purchase tokens, order IDs, email addresses, service-account details, project identifiers and other personal or secret data. A small redacted trace is easier to review and safer than a complete log dump.

## Reproduce an expiry check

Use a license tester, a test subscription, and a non-production backend with RTDN enabled. Google's [test guide](https://developer.android.com/google/play/billing/test) explains accelerated test renewals and their limits. Test timing differs from production, so check the current guidance rather than assuming a fixed interval.

1. Make a test purchase and confirm that Play reports it as active, the backend has verified it, and the app unlocks a real paid feature.
2. Record the subscription expiry, saved purchase source, combined entitlement, and app access state. Keep the app closed through the test expiry if you want to exercise the backend notification path.
3. When the expiry notification arrives, query Play for the current subscription state. Compare the API result with the saved source and the account's aggregate entitlement.
4. Remove access for an expired source. Preserve access only if another valid source still grants it under your product rules.
5. Refresh or relaunch the app and test the paid feature itself. Confirm that an expired account is locked and that the decision remains correct after another launch.

If your test harness can safely delay delivery, use a separate non-production test to check how your reconciliation path recovers. Do not treat a missing notification, an API error, an unassociated purchase, a stale app cache, and an expired subscription as the same failure. Record which step failed; each points to a different fix.

## A useful rule

Use RTDN to trigger reconciliation. Use the current verified purchase state and your entitlement rules to decide access. Then test what the app actually lets the user do.

### References

- [Real-time developer notifications reference](https://developer.android.com/google/play/billing/rtdn-reference)
- [Subscription lifecycle](https://developer.android.com/google/play/billing/lifecycle/subscriptions)
- [Test your Google Play Billing integration](https://developer.android.com/google/play/billing/test)
