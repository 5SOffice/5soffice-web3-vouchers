# Security and Operational Controls v2

## Identity and authorization

- authenticate customers, staff and service accounts;
- enforce tenant, location and role boundaries;
- use least privilege and short-lived credentials;
- require stronger authorization for grants, adjustments, bulk issuance and campaign activation.

## Transaction integrity

- idempotency for all mutations;
- signed server-side QR payloads;
- replay protection;
- bounded reservation expiry;
- reconciliation jobs for partial failures;
- append-only ledger and compensating entries.

## Fraud and abuse controls

- velocity and quantity limits;
- suspicious redemption detection;
- device/session and campaign risk signals;
- duplicate delivery prevention;
- staff override reason and approval;
- monitoring of bulk grants and adjustments.

## Privacy

- minimize personal data;
- separate public token data from operational data;
- define retention and deletion;
- restrict logs and reports;
- avoid secrets and sensitive payloads in chain metadata.

## Operational readiness

- health and reconciliation dashboards;
- failed transaction queue;
- support and refund/reissue procedure;
- incident response and chain-adapter disablement;
- backup and recovery for business records;
- periodic access and rule review.
