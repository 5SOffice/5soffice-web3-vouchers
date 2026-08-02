# AI Service Credits

## Purpose

Provide a simple entitlement model for metered AI and expert services offered to 5SOffice tenants and SME customers.

## Example monthly package

```text
50 document actions
10 meeting summaries
20 growth-content actions
5 compliance workflows
3 expert escalations
```

These numbers are examples only and require commercial validation.

## Credit lifecycle

```text
Granted or purchased
  -> available
  -> reserved for an action
  -> consumed after successful completion
  -> released if the action fails or expires
  -> expired according to package rules
```

## Metering principles

- A billable action must be defined in plain language.
- The user sees the expected credit cost before confirmation.
- Failed technical execution must not consume the credit.
- A repeated request with the same idempotency key must not double-charge.
- Adjustments require a reason, authorized actor and ledger entry.
- Credits are not cash and are not represented as investment assets.

## Integration with AI agents

Before a metered workflow, the agent:

1. identifies the tenant and user;
2. queries eligible credits;
3. previews the cost and conditions;
4. obtains confirmation;
5. reserves credits;
6. executes the workflow;
7. confirms or releases the reservation;
8. presents the updated balance and reference.
