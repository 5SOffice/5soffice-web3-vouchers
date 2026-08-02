# Redemption State Machine

## States

- `DRAFT` — preview created, no entitlement reserved.
- `RESERVED` — entitlement held until expiry.
- `CONFIRMED` — user confirmed and business transaction accepted.
- `DELIVERED` — service delivered or digital action completed.
- `REDEEMED` — entitlement consumption finalized.
- `RELEASED` — reservation returned without consumption.
- `EXPIRED` — reservation expired.
- `CANCELLED` — transaction cancelled according to policy.
- `FAILED` — processing failed and requires retry or compensation.

## Normal flow

```text
DRAFT -> RESERVED -> CONFIRMED -> DELIVERED -> REDEEMED
```

## Failure flow

```text
RESERVED -> EXPIRED -> RELEASED
CONFIRMED -> FAILED -> RELEASED or manual resolution
DELIVERED -> FAILED -> reconciliation queue
```

## Rules

- Every mutation requires an idempotency key.
- Reservation has an explicit expiry.
- Redemption occurs only after the defined success condition.
- Reception scanning validates server-side state, signature, expiry and service/location rules.
- Manual adjustments never rewrite history; they create compensating ledger entries.
- Optional on-chain actions follow the business state and are reconciled asynchronously unless the campaign explicitly requires chain confirmation.
