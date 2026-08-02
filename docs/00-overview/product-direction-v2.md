# Product Direction v2

## Decision

The repository will provide the **5SOffice Voucher & AI Credit Engine**. Normal account and QR-based entitlement use is the default. Blockchain integration becomes optional and isolated behind a Web3 adapter.

## Strategic role

| Component | Responsibility |
|---|---|
| 5SOffice Webapp | Customer and tenant interface |
| Tenant Agent / AI Suite | Conversational discovery and redemption workflow |
| Voucher & AI Credit Engine | Catalog, eligibility, wallet, reservation, redemption and ledger |
| Booking and service systems | Actual business-service delivery |
| Optional Web3 Adapter | Token mint/burn or proof for selected entitlements |

## Product principles

1. **Business service first.** The entitlement exists to support a real service.
2. **No wallet requirement.** A normal account is sufficient.
3. **Reserve then confirm.** Prevent double-spend while avoiding irreversible redemption before service success.
4. **One operational ledger.** Every balance change is traceable and idempotent.
5. **Privacy off-chain.** Personal and sensitive business data must not be written to a public chain.
6. **Optional tokenization.** Web3 is used only when it creates demonstrable value.
7. **Clear non-financial positioning.** No investment or appreciation claims.

## Primary use cases

- tenancy package benefits;
- promotional service vouchers;
- meeting room and coworking passes;
- printing and administrative services;
- AI Suite usage credits;
- expert consultation credits;
- partner or customer gifting campaigns.
