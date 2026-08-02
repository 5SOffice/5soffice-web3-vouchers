# Target Architecture v2

```text
Channels
 Webapp | Tenant Portal | Tenant Agent | Reception Console
                          |
                          v
                 Entitlement API Gateway
                          |
        +-----------------+------------------+
        |                 |                  |
        v                 v                  v
 Catalog & Rules      Wallet Service     Redemption Service
        |                 |                  |
        +-----------------+------------------+
                          |
                          v
               Append-only Usage Ledger
                          |
        +-----------------+------------------+
        |                 |                  |
        v                 v                  v
 Booking/Services     Reporting/Audit    Web3 Adapter (optional)
```

## Core components

- **Catalog & Rules:** versioned entitlement definitions and eligibility evaluation.
- **Wallet Service:** calculates available, reserved, consumed and expired balances.
- **Reservation Service:** holds entitlement for a bounded time.
- **Redemption Service:** coordinates service transaction and balance finalization.
- **Ledger:** records grants, reservations, consumption, release, expiry and adjustment.
- **QR Service:** creates signed, short-lived redemption references.
- **Admin Console API:** campaigns, grants, suspensions and adjustments with authorization.
- **Web3 Adapter:** isolated chain-specific integration.

## Architecture properties

- idempotent mutations;
- atomic or compensating updates between service and entitlement state;
- signed short-lived QR references;
- no reliance on client-provided balance;
- tenant and customer scoping;
- complete correlation IDs;
- append-only ledger with controlled corrections;
- no PII or sensitive order details on public chains;
- Web3 adapter failure does not block core service unless explicitly required by a campaign.
