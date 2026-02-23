# 5SOffice Web3 Vouchers (BNB Chain)

**Docs-first repo** for the 5SOffice Web3 Voucher program.

This project introduces **NFT vouchers (ERC-1155)** on **BNB Chain**, redeemable for 5SOffice services (e.g., meeting rooms, printing, day passes) and digital perks (e.g., AI Suite).  
Primary goal: **brand promotion** and “tech-forward” customer experience — **not** an investment product.

---

## Why NFT Vouchers (ERC-1155)?

We choose **ERC-1155** because it is:
- Simple to operate: **1 voucher = 1 service unit**, redemption = **burn**
- Easy to scale: many voucher types in **one contract**
- Clear for customers: less confusion than “coin points”
- Suitable for promotions: controlled supply per voucher type

---

## MVP (Phase 0 → Phase 1)

### Phase 0 (current): Docs-first
Focus on product specs, architecture, voucher catalog, and redeem flow.

### Phase 1: Minimal working MVP on BNB Testnet
- ERC-1155 contract (voucher types + mint + `redeemAndBurn(orderHash)` + pause)
- Customer portal (MetaMask connect, view vouchers, create redeem order, redeem)
- Backend order service + chain listener
- Reception console (scan QR → validate order → deliver service)

---

## Core Concept

- Each voucher type is a `tokenId` (ERC-1155).
- Customers hold vouchers in MetaMask.
- Redemption flow:
  1) Customer creates a **Redeem Order** (OrderID) on portal
  2) Customer calls **redeem & burn** on-chain with an `orderHash`
  3) Backend listens to `VoucherRedeemed` event → marks Order **VALID**
  4) Reception scans QR → **DELIVERED**

This gives a clean on-chain proof of redemption while keeping real operations simple.

---

## Starter Voucher Set (initial 6)

1. **Meeting Room 1h**
2. **Color Print 10 pages**
3. **B/W Print 50 pages**
4. **Coworking Day-pass (1 day)**
5. **Job VIP Posting (1x)**
6. **AI Suite (1 month)**

> More vouchers will be added later across 3 groups:
> - Group A: countable services (burn once = done)
> - Group B: discounts/promotions (conditions handled by backend Order rules)
> - Group C: time/quota-based benefits (phase later)

---

## Tech Stack (target)

- **Chain:** BNB Chain
- **Wallet:** MetaMask
- **Token standard:** ERC-1155
- **Backend:** Order service + Chain listener (events)
- **Frontend:**
  - Customer Portal
  - Reception Console (QR validation)

---

## Repository Structure (Phase 0)
