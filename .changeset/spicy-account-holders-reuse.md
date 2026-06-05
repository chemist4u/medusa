---
"@medusajs/payment": patch
---

fix(payment): make createAccountHolder reuse an existing account holder

`PaymentModuleService.createAccountHolder` blindly inserted the `(provider_id, external_id)` row returned by the provider. Because the provider call is idempotent (e.g. Stripe returns the same customer for a given idempotency key), the row could already exist — typically an orphaned holder whose customer link (written separately, by `createRemoteLinkStep`) was never persisted — and the insert then collided with the unique `(provider_id, external_id)` index, throwing "Account holder ... already exists" (a 400 on `POST /payment-sessions`) that never self-healed. It now reuses an existing holder before inserting (get-or-create), so the caller's link upsert re-links it and any later init for the same customer succeeds.
