# Secure Coding Guidelines — 5SOffice Web3 Vouchers (A.8.28)
Owner: Nguyễn Đăng Quang  
Last updated: 2026-02-28  
Status: Draft (Phase 1 MVP)

---

## 1. Purpose
This guideline defines minimum secure coding practices for the project to support ISO/IEC 27001:2022 control **A.8.28 Secure Coding**.

Audience:
- Developers (in-house or vendor)
- AI coding tools users (Copilot/Cursor/etc.)
- Reviewers (Project Owner)

---

## 2. General rules (must follow)
### 2.1 No secrets in code
- Never commit:
  - private keys, seed phrases
  - API keys, DB passwords
  - RPC provider tokens
- Use `.env` and secret managers (GitHub Secrets, vault later).
- Add/keep `.env` in `.gitignore`.

### 2.2 Input validation everywhere
Backend must validate:
- tokenId exists in catalog
- amount bounds (default 1 for MVP)
- orderId format
- siteId format (if used)
- customerMeta fields format (email/username constraints)

Return consistent errors:
- HTTP 400/401/403/429/500 with `code`, `message`, `requestId`

### 2.3 Fail safe by default
- Deny if uncertain:
  - if order is not VALID, do not deliver
  - if listener mismatch, do not mark VALID
- Do not “auto-fix” mismatches silently.

### 2.4 Idempotency for delivery
- `POST /deliver` must be idempotent.
- If already DELIVERED, return the existing result.

### 2.5 Logging hygiene
- Include `requestId` for traceability.
- Never log:
  - secrets
  - full customer private data
- Mask wallet addresses in reception views/logs where possible.

---

## 3. Web (Portal / Reception Console)
### 3.1 Wallet interactions
- Always show network/chain info (BNB testnet/mainnet).
- Prevent wrong-chain transactions with clear UI warnings.
- Do not store wallet secrets.

### 3.2 XSS / Injection
- Escape user-controlled content.
- Do not render raw HTML from API responses.
- Validate QR payload parsing (accept only `5SOV://order/{orderId}`).

### 3.3 Session security (Reception)
- Require login (even MVP).
- Use secure cookies or tokens with short expiry.
- Logout function available.

---

## 4. Backend (Order API)
### 4.1 OrderHash correctness (critical)
- Use canonical orderHash:
  - `keccak256(abi.encode(orderId, customerAddress, tokenId, amount, siteId, expiresAt))`
- Ensure field types and ordering are fixed and tested.

### 4.2 Rate limiting & abuse protection
- Apply rate limits on:
  - create order endpoint
  - validate endpoint (optional)
- Use short TTL orders (e.g., 30 minutes).
- Optional: cooldown per wallet.

### 4.3 Authentication & authorization
- Reception endpoints require authentication.
- Enforce role-based permissions (RECEPTION vs ADMIN).
- If site enforcement enabled:
  - reception can deliver only for assigned site.

### 4.4 Database safety
- Use parameterized queries / ORM.
- Apply migrations.
- Restrict DB network access (VPS firewall).

---

## 5. Chain Listener (Worker)
### 5.1 Confirmations
- Must wait CONFIRMATIONS=3 before marking VALID.
- Never mark VALID immediately on first seen event.

### 5.2 Matching rules
- Match by:
  - orderHash
  - customerAddress
  - tokenId
  - amount
  - expiresAt
- If mismatch → log and ignore.

### 5.3 Reliability
- Retry RPC calls with exponential backoff.
- Persist last processed block (DB) to resume after restart.

---

## 6. Smart contract (ERC-1155)
### 6.1 Access control
- Admin and minter roles separated.
- Enforce maxSupply per tokenId.

### 6.2 Redemption
- Use `redeemAndBurn(tokenId, amount, orderHash)` and emit VoucherRedeemed.
- Redeem must fail when paused.

### 6.3 Use audited libraries
- Use OpenZeppelin contracts where possible.
- Keep contract minimal.

---

## 7. Code review checklist (security)
Reviewers must check:
- [ ] No secrets introduced
- [ ] Input validation present
- [ ] Deliver endpoint idempotent
- [ ] Listener confirmations and matching correct
- [ ] orderHash encoding unchanged or updated with tests
- [ ] Error handling does not leak internals
- [ ] Logging masks sensitive data

---

## 8. Minimum security tests (Phase 1)
- Unit test: orderHash deterministic (backend)
- Integration test: redeem event → VALID
- Idempotency test: deliver twice
- CI scans: dependency + secret scan

---

## 9. References
- docs/01-product/requirements/appsec-requirements.md
- docs/02-architecture/architecture.md
- docs/02-architecture/redeem-flow.md
- docs/00-overview/definition-of-done.md
- docs/03-operations/security-baseline.md
