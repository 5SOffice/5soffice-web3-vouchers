# Application Security Requirements — 5SOffice Web3 Vouchers
Owner: Nguyễn Đăng Quang  
Last updated: 2026-02-27  
Status: Draft (Phase 1 MVP)

---

## 1. Purpose
This document defines **minimum application security requirements** for the Web3 Vouchers system.  
Supports:
- **A.8.26 Application Security Requirements**
- Related controls: secure SDLC, testing, environment separation, change management

---

## 2. Security scope (system components)
- Customer Portal (web)
- Reception Console (web)
- Backend Order API + DB
- Chain Listener worker
- Smart contract (ERC-1155)

---

## 3. Core security principles
- Least privilege (roles and access)
- Defense-in-depth (backend validation even if chain is correct)
- Secure defaults (deny by default, allowlist sites)
- Auditability (logs, requestId, delivery log)

---

## 4. Functional security requirements

### 4.1 Identity & access
**Customer**
- Identity is MetaMask address.
- No account system in MVP.

**Reception**
- Reception Console MUST require staff authentication.
- Role `RECEPTION` can validate and deliver orders.
- If site enforcement is enabled:
  - RECEPTION can deliver only for assigned `siteId`.

**Admin**
- Admin actions (mint/type creation/deploy) are off-chain via wallet and controlled by Project Owner.

Acceptance criteria:
- Reception cannot mark DELIVERED without valid login.
- Unauthorized requests return 401/403.

---

### 4.2 Order integrity (anti-fraud)
- Backend MUST generate deterministic `orderHash` using the canonical encoding:
  - `keccak256(abi.encode(orderId, customerAddress, tokenId, amount, siteId, expiresAt))`
- Listener MUST match on:
  - orderHash, customerAddress, tokenId, amount, expiry
- Backend MUST enforce state transitions:
  - VALID → DELIVERED is one-way
  - DELIVERED is terminal

Acceptance criteria:
- If redeem happens after expiresAt, order never becomes VALID.
- If tokenId/amount mismatch, order never becomes VALID.

---

### 4.3 Idempotency & double-delivery prevention
- `POST /orders/{orderId}/deliver` MUST be idempotent:
  - If already DELIVERED, return current state without duplicates.
- Backend MUST store `deliveredAt` and `deliveredBy`.

Acceptance criteria:
- Two deliver calls result in a single delivered record.

---

### 4.4 Input validation
Backend MUST validate:
- tokenId exists in catalog
- amount is allowed (default 1 for MVP)
- siteId format (if used)
- customerMeta fields (email/username constraints)

Acceptance criteria:
- Invalid payload returns 400 with error code.

---

### 4.5 Rate limiting & abuse prevention
- Rate limit order creation per wallet/IP.
- Short order expiry default: 30 minutes.
- Optional cooldown on repeated order creation.

Acceptance criteria:
- Excessive requests return 429.

---

### 4.6 Sensitive data & logging
- Never log secrets (private keys, db passwords, rpc secrets).
- Logs MUST include `requestId`.
- Store audit logs for:
  - deliver actions (who/when/note)
  - order status transitions (optional)

Acceptance criteria:
- No secrets in logs or repo.

---

### 4.7 Secure communications
- All web traffic MUST use HTTPS in staging/prod.
- Backend internal traffic should be protected by network controls (VPS firewall).

Acceptance criteria:
- HTTP redirects to HTTPS.

---

### 4.8 Error handling
- Error responses should be consistent:
  - `code`, `message`, `requestId`
- Avoid leaking internal stack traces to clients.

---

## 5. Security requirements for smart contract
- Enforce maxSupply per tokenId.
- Pausable contract (pause blocks redeem; mint policy as defined).
- Emit `VoucherRedeemed(user, tokenId, amount, orderHash)`.

Acceptance criteria:
- redeem fails when paused.
- mint fails beyond maxSupply.

---

## 6. Testing requirements (security)
Minimum for Phase 1:
- Unit tests for orderHash generation (backend)
- Integration test for: redeem event → VALID
- Test for deliver idempotency
- CI scans:
  - dependency scan
  - secret scan

Evidence:
- CI logs + sample txHash/orderId

---

## 7. Change management requirements
- All changes via PR.
- Protected main branch.
- PR template must include test evidence.
- Security-related docs updated when behavior changes.

---

## 8. References
- docs/02-architecture/architecture.md
- docs/02-architecture/redeem-flow.md
- docs/03-operations/security-baseline.md
- docs/03-operations/sla-slo.md
