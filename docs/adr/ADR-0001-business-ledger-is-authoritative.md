# ADR-0001 — Business Ledger Is Authoritative

- Status: Accepted
- Date: 2026-08-02

## Context

The original design used on-chain burn as the central redemption event. Most 5SOffice customers do not need a blockchain wallet, and operational delivery requires private rules, reservations, cancellations and reconciliation.

## Decision

The Voucher & AI Credit Engine's business ledger is the authoritative operational record for entitlement availability and service redemption. Blockchain is an optional adapter with a defined mode per campaign.

## Consequences

Positive:

- normal customers use accounts and QR codes;
- business operations continue without chain availability;
- private rules and data remain off-chain;
- reservations and compensation are manageable;
- AI agents can use one consistent entitlement API.

Trade-off:

- optional chain state requires reconciliation;
- campaign documentation must clearly define whether the token is a mirror, claim proof or burn proof.
