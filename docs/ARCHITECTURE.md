# Architecture — 5SOffice Web3 Vouchers

## 1. Components
### 1) Smart Contract (BNB Chain)
- ERC-1155 voucher contract
- Supports creating voucher types, minting, and redeeming (burn) with `orderHash`
- Emits events consumed by backend

### 2) Customer Portal (Web)
- Connect MetaMask
- View voucher balances (by tokenId)
- Create redeem orders and show QR code
- Trigger on-chain redeem transaction (redeemAndBurn)

### 3) Backend (Order Service)
- Stores orders, statuses, and mapping rules
- Generates OrderID, QR payload, and `orderHash`
- Exposes APIs for portal and reception console

### 4) Chain Listener (Worker)
- Subscribes to contract events (VoucherRedeemed)
- Verifies event confirmations
- Updates order status to VALID when matched

### 5) Reception Console (Web)
- Camera/QR scanner
- Validates order status (VALID/PENDING/EXPIRED/DELIVERED)
- Marks DELIVERED with staff user + timestamp + notes

---

## 2. Data model (minimal)
### Order
- orderId (string, unique)
- customerAddress (0x...)
- tokenId (uint256)
- amount (uint256)
- siteId (string) — optional in MVP but recommended
- status: CREATED | PENDING_CHAIN | VALID | DELIVERED | EXPIRED | CANCELLED
- orderHash (bytes32)
- createdAt, expiresAt
- deliveredAt, deliveredBy, deliveredNote
- chainTxHash (optional)

### Voucher Type Config
- tokenId
- name
- unit (e.g., 1h, 10 pages, 1 month)
- siteApplicability (ALL or allowlist of siteIds)
- maxSupply
- uri (metadata)
- conditions (optional for Group B/C)

---

## 3. Sequence (end-to-end)
1) Portal → Backend: Create Order (tokenId, amount, siteId)
2) Backend → Portal: returns orderId, orderHash, expiresAt, QR payload
3) Portal → Contract: redeemAndBurn(tokenId, amount, orderHash) signed by MetaMask
4) Chain Listener: receives VoucherRedeemed event
5) Listener → Backend: validate match (orderHash, customerAddress, tokenId, amount) → set order VALID
6) Reception → Backend: scan QR (orderId) → check status VALID
7) Reception → Backend: mark DELIVERED

---

## 4. Matching logic (important)
The backend matches a redemption event to an order using:
- orderHash (primary)
- customerAddress
- tokenId
- amount
- expiry window (event time must be before expiresAt)

If any mismatch → order remains invalid; reception should not deliver.

---

## 5. Operational assumptions
- Customers pay gas (BNB) via MetaMask for redeem.
- No OTP system; no custodial wallets in Phase 1.
- Site staff rely on backend order validation (not raw chain scans).

---

## 6. Environments
- Dev: local + BNB testnet
- Pilot: BNB testnet (or mainnet with small supplies)
- Production: BNB mainnet, hardened keys, monitoring

---

## 7. Observability & logs
- Listener logs: event received, order matched, confirmations, failures
- Backend logs: order create, status transitions, reception actions
- Dashboard metrics: redeemed count, failures, avg validation time
