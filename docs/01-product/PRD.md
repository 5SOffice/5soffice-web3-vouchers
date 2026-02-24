# PRD — 5SOffice Web3 Vouchers (Phase 0 → Phase 1)

## 1. Overview
5SOffice Web3 Vouchers is a promotional program using **NFT vouchers (ERC-1155)** on **BNB Chain**. Customers receive vouchers and redeem them for 5SOffice services (meeting rooms, printing, day passes) and digital perks (AI Suite).  
Primary goal: **brand promotion** and demonstrating 5SOffice as a tech-forward SME.

**Not a financial token.** No cash-out, no investment promise.

## 1.1 Project ownership
- **Project Owner / Concept Creator:** Nguyễn Đăng Quang
- **Decision authority:** Product scope, voucher catalog, redemption policy, and rollout roadmap
- **Primary objective:** Brand promotion and real-world utility for 5SOffice customers

---

## 2. Goals
### Business goals
- Increase brand awareness and differentiation (“5SOffice uses Web3”).
- Improve conversion for office services by offering redeemable benefits.
- Create a scalable promo mechanism for future campaigns and partners.

### Product goals
- Deliver a simple, operable redemption flow for reception staff.
- Provide clear on-chain proof of redemption (redeem + burn).
- Support adding new voucher types without rewriting the system.

---

## 3. Non-goals (Phase 0–1)
- Not building a tradable investment token.
- No fiat redemption or exchange listing.
- No advanced membership/quota systems (Group C advanced) in Phase 1.
- No gasless relayer/account abstraction in Phase 1 (customers pay gas via MetaMask).

---

## 4. Target users & Personas
### Persona A — New SME Founder
- Wants to try services with low commitment.
- Motivated by promotions/bonuses.
- Not highly technical; can use MetaMask if guided.

### Persona B — Existing 5SOffice Customer
- Already uses meeting rooms/printing occasionally.
- Likes “free hours/pages” perks.
- Will redeem frequently if UX is simple.

### Persona C — Reception/Operations Staff
- Needs a fast and reliable validation method.
- Not expected to understand blockchain details.
- Must avoid disputes and ensure one-time use.

---

## 5. Scope
### Phase 0 (Docs-first)
- Define catalog, flows, contract spec, architecture.

### Phase 1 (MVP on BNB Testnet)
- ERC-1155 contract with redeemAndBurn(orderHash).
- Customer Portal (MetaMask connect, view vouchers, create order, redeem).
- Backend Order Service + Chain Listener.
- Reception Console (scan QR, validate, deliver).
- Basic admin tooling (configure voucher types, mint/airdrop).

---

## 6. MVP Voucher Set (initial 6)
1) Meeting Room 1h  
2) Color Print 10 pages  
3) B/W Print 50 pages  
4) Coworking Day-pass (1 day)  
5) Job VIP Posting (1x)  
6) AI Suite (1 month)

---

## 7. Key user stories
### Customer
- As a customer, I can connect MetaMask and see my voucher balances.
- As a customer, I can create a redeem order for a voucher and receive a QR code.
- As a customer, I can redeem a voucher by signing one on-chain transaction.
- As a customer, I can view redemption history and order status (Pending/Valid/Delivered/Expired).

### Reception
- As reception, I can scan a QR and instantly see if the order is valid.
- As reception, I can mark the service as delivered with a timestamp and notes.
- As reception, I can reject invalid/expired orders and see the reason.

### Admin/Marketing
- As admin, I can create new voucher types and set max supply + metadata.
- As admin, I can mint/airdrop vouchers to customer wallets.
- As admin, I can configure which voucher applies to which site and any special conditions.

---

## 8. User journey (high level)
1) Customer receives voucher NFT (airdrop).
2) Customer chooses service → creates order (OrderID + QR).
3) Customer redeems on-chain (burn) with orderHash.
4) Backend confirms event → order becomes VALID.
5) Reception scans QR → delivers service → marks DELIVERED.

---

## 9. Success metrics (Phase 1 pilot)
- % of issued vouchers redeemed (redeem rate).
- Avg time at reception to validate and deliver (target < 30s).
- Number of disputes/failed validations (target near 0).
- Lead-to-conversion uplift for campaigns with vouchers.

---

## 10. Risks & mitigations
- Customers don’t have BNB for gas → provide simple guide; optionally a tiny “gas top-up” for VIP customers.
- MetaMask complexity → clear UI instructions; keep steps minimal.
- Voucher misuse/disputes → use redeemAndBurn + orderHash + QR validation.
- Operational mismatch across sites → include site applicability rules in catalog/order validation.

---

## 11. Open questions (to finalize before build)
- Which sites support each voucher type (site applicability)?
- Does 5SOffice allow voucher transfers between customers (default yes in Phase 1)?
- What is the initial max supply per voucher type for the pilot?
