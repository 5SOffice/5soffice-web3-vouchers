# Voucher Catalog (Phase 0)

## 1. Purpose
This document defines:
- Initial starter vouchers (6)
- Voucher grouping rules (A/B/C)
- TokenId scheme for scalable expansion

---

## 2. Voucher Groups
### Group A — Countable services (recommended for MVP)
- 1 voucher = 1 service unit
- Redeem = burn once = done
Examples: meeting room hours, printing pages, day passes, job posting (1x)

### Group B — Discounts / Promotions (conditions handled by backend)
- Voucher redemption triggers a discount or one-time promo
- Requires order validation rules (e.g., applies only to certain packages)
Examples: 5% off month-1, setup fee waiver, discount amount

### Group C — Time/quota-based benefits (simple version allowed)
- Voucher redemption grants time-bound access (e.g., 30 days)
- Backend grants entitlement and manages expiry
Examples: AI Suite 1 month, membership trials

---

## 3. TokenId scheme
Recommended:
- 1xxx: Space usage (meeting rooms, booths)
- 2xxx: Print/Scan
- 3xxx: Access passes (coworking)
- 4xxx: Discounts/Promos
- 5xxx: Partner services (job VIP posting)
- 6xxx: Digital services (AI Suite)

---

## 4. Starter voucher set (initial 6)
> TokenIds below are suggested defaults; adjust if needed.

### 1001 — Meeting Room 1h
- Group: A
- Unit: 1 hour
- Redeem: burn 1 voucher
- Site applicability: configurable per site
- Notes: delivery = room booking confirmation

### 2001 — Color Print 10 pages
- Group: A
- Unit: 10 pages
- Redeem: burn 1 voucher
- Notes: delivery = print at reception (or partner print shop)

### 2002 — B/W Print 50 pages
- Group: A
- Unit: 50 pages
- Redeem: burn 1 voucher

### 3001 — Coworking Day-pass (1 day)
- Group: A
- Unit: 1 day
- Redeem: burn 1 voucher

### 5001 — Job VIP Posting (1x)
- Group: A (or B if conditions apply)
- Unit: 1 posting
- Redeem: burn 1 voucher
- Notes: backend may record provider + package details per Order

### 6001 — AI Suite (1 month)
- Group: C (simple time-based entitlement)
- Unit: 30 days access
- Redeem: burn 1 voucher
- Notes: backend grants access to an account identifier (email/username)

---

## 5. Rules for adding new voucher types
When adding a new voucher:
1) Assign a new tokenId following the scheme
2) Define:
   - Name, group (A/B/C)
   - Unit
   - Site applicability (ALL or allowlist)
   - Conditions (for B/C)
   - maxSupply
   - metadata URI
3) Update:
   - This catalog
   - Backend mapping/config
4) Create voucher type on-chain (createVoucherType)
5) Mint/airdrop as needed

---

## 6. Supply planning (high level)
- Total program cap can be controlled by:
  - maxSupply per tokenId (recommended)
- Keep supplies small for pilot; increase after validation.

---

## 7. Optional: transfer policy
Default (Phase 1): vouchers are transferable (ERC-1155 default).
If needed later: implement transfer restrictions (phase later; not in MVP).
