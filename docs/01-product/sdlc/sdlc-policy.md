# Secure SDLC Policy — 5SOffice Web3 Vouchers (A.8.25)
Owner: Nguyễn Đăng Quang  
Last updated: 2026-02-28  
Status: Draft (Phase 1 MVP)

---

## 1. Purpose
This policy defines the **Secure Development Life Cycle (SDLC)** requirements for the 5SOffice Web3 Vouchers project to ensure security is built into development and delivery.  
It supports ISO/IEC 27001:2022 control **A.8.25 Secure Development Life Cycle**.

---

## 2. Scope
Applies to all work related to:
- Smart contract (ERC-1155)
- Backend Order API + Database
- Chain Listener worker
- Customer Portal
- Reception Console
- Infrastructure as code / deployment scripts

---

## 3. SDLC phases & security gates

### 3.1 Planning (Gate 0)
Required artifacts:
- PRD
- Architecture + Redeem Flow
- Token Spec
- Voucher Catalog
- AppSec Requirements
- Crypto Standard
- Definition of Done (DoD)

Evidence:
- Docs committed in Git and reviewed/approved

---

### 3.2 Design (Gate 1)
Required:
- Threat model (lite) for MVP
- API contract finalized (endpoints + payload + error model)
- OrderHash encoding frozen (abi.encode field order)
- QR format frozen (5SOV://order/{orderId})

Evidence:
- Reviewed PR / approved design notes

---

### 3.3 Implementation (Gate 2 — per PR)
Mandatory rules:
- All changes via Pull Request (PR)
- Main branch protection enabled
- CODEOWNERS required review for high-risk paths:
  - architecture specs
  - contract files
  - listener logic
- PR template completed, including:
  - what changed
  - tests executed
  - rollback notes (when applicable)

Security checks (minimum):
- No secrets committed (secret scanning)
- Dependency checks (at least critical/high)
- Lint + unit tests run

Evidence:
- PR approvals, CI logs, test results

---

### 3.4 Testing & Verification (Gate 3)
Minimum verification before pilot:
- Contract tests:
  - maxSupply enforcement
  - redeemAndBurn works and emits VoucherRedeemed
  - pause blocks redeem
- Backend tests:
  - orderHash generation deterministic
  - deliver endpoint idempotent
- Listener tests:
  - waits CONFIRMATIONS=3
  - match rules correct
- End-to-end test (BNB Testnet):
  - mint → create order → redeem → VALID → DELIVERED

Evidence:
- CI logs + screenshots + tx hash + orderId

---

### 3.5 Release & Deployment (Gate 4)
Release requirements:
- Tagged release (e.g., v0.1.0-testnet)
- Release notes include:
  - changes
  - migration notes (if any)
- Deployment checklist completed (DEPLOYMENT.md)

Operational readiness:
- Runbooks available
- Monitoring/health checks enabled
- Backup plan for DB (pilot/prod)

Evidence:
- Release tag + deployment log + backup proof

---

## 4. Environment separation (A.8.31 alignment)
- Separate environments: local/dev, staging/testnet, production/mainnet
- Separate secrets per environment
- No production secrets in dev/staging

Reference:
- docs/02-architecture/environments.md

---

## 5. Vulnerability handling (A.8.29 alignment)
- All findings must be triaged:
  - severity, owner, target fix date
- Critical issues block release until resolved
- Retest after fixes

Evidence:
- Issue/ticket + patch PR + retest results

---

## 6. Change management (A.8.32 alignment)
- All changes follow PR workflow
- Emergency change procedure:
  - document reason
  - minimum review
  - post-change review required

Reference:
- docs/01-product/sdlc/change-management.md (if present) or PR rules in repo

---

## 7. Minimum records to retain
- 2 sample PRs with review approvals
- CI scan logs (dependency + secret scan)
- E2E redemption proof (tx hash + orderId)
- Release notes/tag history

---

## 8. References
- docs/01-product/prd.md
- docs/02-architecture/architecture.md
- docs/02-architecture/redeem-flow.md
- docs/01-product/requirements/appsec-requirements.md
- docs/02-architecture/security/crypto-standard.md
- docs/00-overview/definition-of-done.md
- docs/03-operations/deployment.md
