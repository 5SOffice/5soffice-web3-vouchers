# Redeem Flow — OrderID + QR + Redeem & Burn (BNB Chain)

## 1. States

### 1.1 Order status lifecycle
- CREATED: order created in backend
- PENDING_CHAIN: customer submitted on-chain tx (optional intermediate)
- VALID: chain event confirmed (N confirmations) and matched to order
- DELIVERED: reception delivered the service
- EXPIRED: order expired before successful redemption match
- CANCELLED: order cancelled by customer/admin (optional)

### 1.2 Voucher lifecycle
- Minted → Held by customer → Redeemed (burned)

---

## 2. QR payload (MVP)

QR encodes only the `orderId` in a prefixed string:

- Format: `5SOV://order/<orderId>`
- Example: `5SOV://order/ORD_20260223_ABC123`

Reception Console parses `<orderId>` and calls backend to validate.

---

## 3. orderHash rules (MUST be consistent)

Backend computes `orderHash` using the Solidity equivalent of:

- `orderHash = keccak256(abi.encode(orderId, customerAddress, tokenId, amount, siteId, expiresAt))`

Field definitions:
- `orderId`: string
- `customerAddress`: address (0x…)
- `tokenId`: uint256
- `amount`: uint256
- `siteId`: string (use `""` if not enforcing sites yet)
- `expiresAt`: uint64 unix timestamp (seconds)

Notes:
- Use `abi.encode(...)` (NOT `abi.encodePacked`) to reduce collision risk.
- `expiresAt` must be stored and hashed as unix seconds for deterministic hashing.

---

## 4. Step-by-step flow

### Step 1 — Customer creates order (Portal → Backend)
**Input**
- tokenId
- amount
- siteId (recommended; optional only for simplest pilot)
- service-specific info (optional, in `customerMeta`)
  - AI Suite: `aiAccount` (email/username)
  - Job VIP: `jobProvider` (optional)

**Backend actions**
- Validates:
  - voucher exists in catalog
  - site applicability (if enabled)
  - rate limits / spam prevention
- Generates:
  - orderId
  - expiresAt (recommend: 30 minutes from creation)
  - orderHash
  - qrPayload

**Output**
- orderId, expiresAt, orderHash, qrPayload
- status = CREATED

---

### Step 2 — Customer redeems on-chain (MetaMask)
Portal calls contract:
- `redeemAndBurn(tokenId, amount, orderHash)`

Customer signs tx and pays gas in BNB.

Backend may set status to PENDING_CHAIN immediately after the portal submits tx (optional).

---

### Step 3 — Chain Listener confirms and matches redemption
Listener watches contract events:
- `VoucherRedeemed(user, tokenId, amount, orderHash)`

**Confirmations**
- Wait for `N` block confirmations before marking VALID.
- Recommended MVP value on BNB Chain: `N = 3` (tunable later).

**Matching conditions (ALL required)**
- orderHash exists in DB
- order status is CREATED or PENDING_CHAIN
- user == customerAddress
- tokenId and amount match the order
- current time <= expiresAt
- (if enforcing site) order.siteId is valid for the tokenId

If matched:
- update order status → VALID
- store chainTxHash and chainBlockNumber

If not matched:
- no status change
- log mismatch for investigation

---

### Step 4 — Reception validates and delivers (QR scan)
Reception scans QR and extracts `orderId`, then calls:
- `GET /api/v1/orders/{orderId}`

If status == VALID:
- deliver the service
- call `POST /api/v1/orders/{orderId}/deliver`

Deliver request includes:
- deliveredBy (staffId)
- deliveredNote (optional)

System sets:
- status = DELIVERED
- deliveredAt timestamp

**Idempotency rule**
- If order is already DELIVERED, repeated deliver calls must not create duplicates.
- Response should return current DELIVERED state.

---

## 5. Expiry rules
- Order expires at `expiresAt`.
- If redemption event arrives after `expiresAt`:
  - Listener must NOT mark VALID.
  - Order becomes EXPIRED.
- Customer must create a new order if still wants to redeem.

---

## 6. Failure scenarios & handling

### A) Customer has no gas (BNB)
- Portal shows: “Need a small amount of BNB for network fee”
- Provide a simple help guide (where to get BNB, how to send to wallet)

### B) Customer tries wrong voucher / wrong amount
- Listener mismatch → order not VALID
- Customer can create a new order with correct selection

### C) Reception sees PENDING_CHAIN
- Ask customer to wait for confirmations
- Refresh until VALID or expiry

### D) Replay / reuse attempt
- Burn prevents voucher reuse
- Order state prevents double delivery (VALID→DELIVERED one-way)

### E) Site mismatch (if enforced)
- Order created for site A but reception at site B:
  - validation fails (not deliver)
  - customer must redeem at the correct site or create a new order if policy allows

---

## 7. Operational checklist (Reception)
1) Scan QR
2) Confirm status is VALID
3) Deliver service
4) Mark DELIVERED with note if needed
5) If not VALID: do not deliver; instruct customer to redeem again or contact support
