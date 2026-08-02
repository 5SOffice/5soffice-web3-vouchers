# API Contracts

Concrete schemas should be published as OpenAPI during implementation.

## Required context

- correlation ID;
- authenticated actor;
- tenant/customer IDs;
- channel;
- locale;
- idempotency key for mutations.

## Core API operations

### Catalog

- list eligible entitlements;
- retrieve entitlement details and conditions;
- preview benefit for a proposed service.

### Wallet

- retrieve balances and expiry;
- retrieve usage history;
- grant, suspend or adjust through authorized administration.

### Redemption

- create preview;
- reserve entitlement;
- confirm transaction;
- mark delivery;
- finalize redemption;
- release or cancel;
- retrieve status.

### QR

- issue signed redemption QR/reference;
- validate QR/reference;
- confirm authorized delivery.

### Web3 adapter

- create tokenization request;
- retrieve mapping and chain status;
- submit burn/proof request;
- reconcile chain events.

## Error requirements

Errors must distinguish validation, eligibility, insufficient balance, expiry, conflict, authorization, downstream failure and reconciliation-required states. User interfaces must not display technical success until the authoritative operation is committed.
