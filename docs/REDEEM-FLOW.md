# Redeem Flow — OrderID + QR + Redeem & Burn

## 1. States
Order status lifecycle:
- CREATED: order created in backend
- PENDING_CHAIN: customer initiated chain tx (optional intermediate)
- VALID: chain event confirmed and matched to order
- DELIVERED: reception delivered the service
- EXPIRED: order expired before redemption
- CANCELLED: order cancelled by customer/admin (optional)

Voucher lifecycle:
- Minted → Held by customer → Redeemed (burned)

---

## 2. Step-by-step flow
### Step 1 — Customer creates order (Portal)
Input:
- tokenId
- amount
- siteId (optional but recommended)
- service-specific info:
  - For AI Suite: account identifier (email/username)
  - For Job VIP: provider selection (optional)

Backend output:
- orderId
- expiresAt (e.g., 30 minutes)
- orderHash (bytes32)
- QR payload (contains orderId + minimal info)

Order status: CREATED

### Step 2 — Customer redeems on-chain (MetaMask)
Portal calls contract:
- redeemAndBurn(tokenId, amount, orderHash)

Customer signs tx and pays gas in BNB.

Order status: (optionally) PENDING_CHAIN

### Step 3 — Chain Listener confirms redemption
Listener watches:
- VoucherRedeemed(user, tokenId, amount, orderHash)

Listener verifies:
- orderHash exists in DB
- user == customerAddress
- tokenId & amount match
- now <= expiresAt
- sufficient confirmations reached

If valid:
- update order status → VALID
- record chainTxHash

### Step 4 — Reception validates and delivers
Reception scans QR (orderId):
- Backend returns:
  - status
  - service details
  - customer address (masked)
  - tokenId/unit
  - expiry and chain proof (optional link)

If status == VALID:
- reception delivers service
- reception marks DELIVERED with:
  - deliveredAt
  - deliveredBy
  - note (optional)

---

## 3. QR payload (recommended)
QR contains only:
- orderId
Optionally:
- short checksum

Reason:
- Reception relies on backend for full validation.
- Avoid embedding sensitive info in QR.

---

## 4. Expiry rules
- Each order has an expiry window (e.g., 30 minutes).
- If redemption happens after expiresAt:
  - Listener does NOT mark VALID
  - Order becomes EXPIRED
- Customer must create a new order if needed.

---

## 5. Failure scenarios & handling
### A) Customer has no gas
- Portal shows message: need small BNB for gas
- Provide help link/FAQ

### B) Wrong tokenId/amount burned
- Listener mismatch → order not VALID
- Customer support required (rare if UI prevents errors)

### C) Reception scan shows PENDING_CHAIN
- Ask customer to wait for confirmation
- Refresh scan until VALID or expiry

### D) Replay / reuse attempt
- Burn ensures voucher cannot be reused
- Order status prevents double delivery (VALID→DELIVERED is one-way)

---

## 6. Operational checklist for reception
- Scan QR
- Confirm status VALID
- Deliver service
- Mark DELIVERED
- If not VALID: do not deliver; instruct customer to redeem again or contact support
