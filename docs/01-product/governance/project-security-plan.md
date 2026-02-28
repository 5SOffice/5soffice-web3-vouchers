# Project Security Plan — 5SOffice Web3 Vouchers
Owner: Nguyễn Đăng Quang  
Last updated: 2026-02-27  
Status: Draft (Phase 1 MVP)

---

## 1. Purpose
This document defines how information security is embedded into project management for the **5SOffice Web3 Vouchers** project.  
It supports **A.5.8 Information Security in Project Management** and provides security gates, roles, and minimum evidence.

---

## 2. Project scope (MVP)
### In scope (Phase 1)
- ERC-1155 voucher contract on **BNB Testnet**
- Customer Portal (MetaMask connect, create order, redeem)
- Backend Order Service + Database
- Chain Listener (event processing)
- Reception Console (QR validate, deliver)
- Basic ops: deployment, monitoring, backups (lightweight)

### Out of scope (Phase 1)
- Gasless transactions / account abstraction
- Custodial wallets / OTP
- Secondary market / royalty
- DAO voting / membership quota systems (advanced Group C)

---

## 3. Security objectives (project-level)
1) Prevent unauthorized minting, redemption manipulation, or double delivery.
2) Ensure **one-time redemption**: voucher burn + order status gate.
3) Protect secrets (keys, RPC secrets, DB creds) and avoid leakage.
4) Maintain operational reliability for reception validation (fast, stable).
5) Provide auditable evidence (PRs, approvals, logs) for ISMS alignment.

---

## 4. Roles & responsibilities (RACI summary)
### Key roles
- **Project Owner / Concept Creator (Nguyễn Đăng Quang)**
  - Approves scope, security requirements, architecture, and release gates
- **Developer / Vendor**
  - Implements code and tests according to docs/specs
- **Ops / Reception Lead**
  - Operates reception console, ensures delivery process compliance
- **Security reviewer (can be Project Owner in MVP)**
  - Reviews security-related changes, approves release readiness

### Decision authority (MVP)
- Architecture + orderHash rules + redeem flow changes require approval by Project Owner.
- Contract deploys and role assignments require Project Owner approval.

---

## 5. Security gates (minimum)
### Gate 0 — Planning (Phase 0/Start)
Must exist:
- PRD, Architecture, Token Spec, Redeem Flow, Catalog
- This Project Security Plan
- Security baseline + deployment checklist

Evidence:
- Docs committed and reviewed in Git

### Gate 1 — Design review (before build)
Checklist:
- orderHash encoding fixed and documented
- QR format fixed and documented
- API contract defined (endpoints + JSON samples)
- Threat model lite created (basic threats + mitigations)

Evidence:
- Approved architecture docs / ADR (optional)

### Gate 2 — Implementation review (per PR)
Required:
- Protected main branch + PR reviews
- No secrets committed
- Tests updated (unit/integration where applicable)

Evidence:
- PR template filled, approvals recorded

### Gate 3 — Security testing gate (before testnet pilot)
Required:
- Dependency scan / secret scan runs in CI
- Listener confirmation logic tested (CONFIRMATIONS=3)
- Deliver endpoint idempotency verified

Evidence:
- CI logs + test evidence + sample tx hash / orderId

### Gate 4 — Pilot readiness (before real reception use)
Required:
- Reception workflow trained
- Runbooks available (listener down, RPC outage, pause)
- Backup plan exists (DB snapshot)

Evidence:
- Training note + runbook link + backup evidence

---

## 6. Key project risks (starter list)
| Risk | Impact | Mitigation | Owner |
|---|---|---|---|
| Admin/minter key compromise | Unauthorized mint/pause misuse | Hardware wallet, separate roles, multisig later | Project Owner |
| Wrong orderHash encoding | Orders never become VALID / disputes | Freeze encoding in docs + tests | Dev + Owner |
| Double delivery | Loss / fraud | VALID→DELIVERED one-way + idempotent deliver | Dev + Ops |
| RPC instability | Slow/failed validation | Backup RPC, retries, monitoring | Dev/Ops |
| Customer lacks gas | Cannot redeem | Clear guide; optional small gas top-up policy later | Ops |

---

## 7. Minimum evidence to retain (for audit/ISMS alignment)
- Branch protection + CODEOWNERS screenshot
- 2 sample PRs with review approval and checklist
- CI run logs (dependency/secret scan, tests)
- Contract deployment tx hash (testnet)
- Listener logs showing event → VALID
- Reception delivery log (order delivered sample)
- Backup evidence (DB snapshot)

---

## 8. References
- docs/02-architecture/architecture.md
- docs/02-architecture/redeem-flow.md
- docs/02-architecture/token-spec.md
- docs/03-operations/security-baseline.md
- docs/03-operations/runbooks.md
