# Definition of Done (DoD) — Phase 1 MVP

This checklist defines what “done” means for the Phase 1 MVP so that devs and AI tools deliver consistently.

---

## 1) Contract (ERC-1155)
- [ ] Contract deployed on **BNB Testnet**
- [ ] Supports: createVoucherType, mint/mintBatch, redeemAndBurn(orderHash), pause/unpause
- [ ] Enforces maxSupply per tokenId
- [ ] Emits VoucherRedeemed(user, tokenId, amount, orderHash)
- [ ] Basic unit tests for:
  - [ ] cannot mint beyond maxSupply
  - [ ] redeemAndBurn burns correct balance
  - [ ] pause blocks redeem

---

## 2) Backend (Order API)
- [ ] Implements endpoints:
  - [ ] POST /api/v1/orders
  - [ ] GET /api/v1/orders/{orderId}
  - [ ] POST /api/v1/orders/{orderId}/deliver (idempotent)
  - [ ] GET /api/v1/catalog (optional but recommended)
- [ ] Generates deterministic orderHash using abi.encode field order
- [ ] Enforces expiry window (default 30 min)
- [ ] Rate limiting for order creation
- [ ] Logs include requestId
- [ ] DB migrations / schema documented

---

## 3) Chain Listener
- [ ] Consumes VoucherRedeemed events from contract address
- [ ] Waits CONFIRMATIONS=3 before marking VALID
- [ ] Matching rules implemented:
  - orderHash, customerAddress, tokenId, amount, expiresAt
- [ ] Handles RPC retries with backoff
- [ ] Stores txHash + blockNumber
- [ ] Exposes heartbeat / lastProcessedBlock metrics (or logs)

---

## 4) Customer Portal (Web)
- [ ] Connect MetaMask
- [ ] Displays voucher balances for starter set
- [ ] Create order (POST /orders)
- [ ] Initiate redeemAndBurn via MetaMask
- [ ] Shows order status (CREATED/PENDING/VALID/EXPIRED)
- [ ] Clear user guidance for “need small BNB gas”

---

## 5) Reception Console (Web)
- [ ] QR scan reads format: 5SOV://order/{orderId}
- [ ] Calls GET /orders/{orderId} and displays status clearly
- [ ] Only allows DELIVER when status is VALID
- [ ] Mark deliver is idempotent and logs staffId + timestamp
- [ ] Basic staff login (MVP acceptable)

---

## 6) End-to-end acceptance tests (must pass)
- [ ] Mint voucher to test wallet
- [ ] Portal shows correct balance
- [ ] Create order returns orderId + qrPayload + orderHash
- [ ] Redeem burns voucher and emits VoucherRedeemed
- [ ] Listener marks order VALID after confirmations
- [ ] Reception scan shows VALID and can mark DELIVERED
- [ ] Double deliver attempt does not duplicate delivery

---

## 7) Ops & Security minimum
- [ ] HTTPS enabled on staging domain (or documented for local)
- [ ] Secrets not committed; .env.example provided
- [ ] Admin/minter wallet separation documented
- [ ] Basic backup plan for DB (even if manual)

---

## 8) Documentation updated
- [ ] Any API/flow change reflected in docs:
  - architecture.md
  - redeem-flow.md
  - token-spec.md
- [ ] Release notes / changelog entry (optional)
