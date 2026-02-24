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
- Waits for confirmations
- Matches event to orders and updates status to VALID

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
- siteId (string) — recommended (can be optional in MVP but should exist for multi-site operations)
- status: CREATED | PENDING_CHAIN | VALID | DELIVERED | EXPIRED | CANCELLED
- orderHash (bytes32)
- createdAt, expiresAt
- deliveredAt, deliveredBy, deliveredNote
- chainTxHash (optional)
- chainBlockNumber (optional)

### Voucher Type Config
- tokenId
- name
- group: A | B | C
- unit (e.g., 1h, 10 pages, 1 month)
- siteApplicability: ALL | ALLOWLIST(siteIds)
- maxSupply
- uri (metadata)
- conditions (optional, for Group B/C only)

### Staff user (Reception)
- staffId
- displayName
- role: RECEPTION | ADMIN
- siteId(s)

---

## 3. QR payload format (MVP)
QR should encode **only orderId** (and a prefix) to keep it simple and safe.

Recommended QR string:
- `5SOV://order/<orderId>`

Example:
- `5SOV://order/ORD_20260223_ABC123`

Reception Console:
- parses `<orderId>` and calls backend to validate status.

---

## 4. API Contract (minimum set)
> All responses should include a `requestId` for logging/debug.

### 4.1 Create Order
`POST /api/v1/orders`

Request (Portal → Backend):
```json
{
  "customerAddress": "0xabc...",
  "tokenId": 1001,
  "amount": 1,
  "siteId": "HCM_ROX_T09",
  "customerMeta": {
    "aiAccount": "user@email.com",
    "jobProvider": "providerA"
  }
}

Response:

{
  "orderId": "ORD_20260223_ABC123",
  "status": "CREATED",
  "expiresAt": "2026-02-23T10:30:00Z",
  "orderHash": "0x9f... (bytes32)",
  "qrPayload": "5SOV://order/ORD_20260223_ABC123"
}

Notes:
	•	customerMeta is optional and depends on voucher type (e.g., AI Suite needs account identifier).
	•	Backend must validate site applicability for the voucher (if configured).

⸻

4.2 Get Order Status (Reception validation)

GET /api/v1/orders/{orderId}

Response:

{
  "orderId": "ORD_20260223_ABC123",
  "status": "VALID",
  "siteId": "HCM_ROX_T09",
  "customerAddressMasked": "0xabc...789",
  "tokenId": 1001,
  "amount": 1,
  "service": {
    "name": "Meeting Room 1h",
    "unit": "1h",
    "group": "A"
  },
  "expiresAt": "2026-02-23T10:30:00Z",
  "chainProof": {
    "txHash": "0x123...",
    "blockNumber": 12345678,
    "confirmed": true
  }
}


⸻

4.3 Mark Delivered (Reception action)

POST /api/v1/orders/{orderId}/deliver

Request:

{
  "deliveredBy": "staff_001",
  "deliveredNote": "Room A - 10:00 to 11:00"
}

Response:

{
  "orderId": "ORD_20260223_ABC123",
  "status": "DELIVERED",
  "deliveredAt": "2026-02-23T10:05:12Z"
}

Rules:
	•	Only allowed if current status == VALID
	•	Must be idempotent (repeat calls do not create duplicates)

⸻

4.4 Voucher Catalog (optional but recommended)

GET /api/v1/catalog

Response:

{
  "version": "2026.02.23",
  "items": [
    {
      "tokenId": 1001,
      "name": "Meeting Room 1h",
      "group": "A",
      "unit": "1h",
      "siteApplicability": { "mode": "ALLOWLIST", "siteIds": ["HCM_ROX_T09", "HCM_IMC_T04"] }
    }
  ]
}


⸻

5. orderHash encoding (MUST be consistent)

To avoid mismatches, backend and contract must use the same encoding.

Recommended (MVP):
	•	Use Solidity ABI encoding equivalent of abi.encode(...) (NOT encodePacked).

Canonical fields:
	•	orderId (string)
	•	customerAddress (address)
	•	tokenId (uint256)
	•	amount (uint256)
	•	siteId (string)
	•	expiresAt (uint64 unix timestamp)

Definition:
	•	orderHash = keccak256(abi.encode(orderId, customerAddress, tokenId, amount, siteId, expiresAt))

Notes:
	•	expiresAt should be stored as unix seconds for deterministic hashing.
	•	If siteId is omitted in MVP, pass empty string "" (still keep the field).

⸻

6. Sequence (end-to-end)
	1.	Portal → Backend: Create Order (tokenId, amount, siteId, customerMeta)
	2.	Backend → Portal: returns orderId, orderHash, expiresAt, QR payload
	3.	Portal → Contract: redeemAndBurn(tokenId, amount, orderHash) signed by MetaMask
	4.	Listener: receives VoucherRedeemed event
	5.	Listener → Backend: validate match (orderHash, customerAddress, tokenId, amount, expiry) → set order VALID
	6.	Reception → Backend: scan QR (orderId) → check status VALID
	7.	Reception → Backend: mark DELIVERED

⸻

7. Matching logic (listener)

Listener matches on:
	•	orderHash (primary key)
	•	user == customerAddress
	•	tokenId & amount match
	•	current time <= expiresAt
	•	order status in {CREATED, PENDING_CHAIN} only

If match succeeds:
	•	set status VALID
	•	store txHash + blockNumber
Else:
	•	log as mismatch (no status change)

⸻

8. Confirmations / Reorg handling
	•	Listener should wait for N confirmations before marking VALID.
	•	Recommended for BNB Chain MVP: N = 3 (can tune later).

Reorg approach (simple):
	•	do not mark VALID until confirmed
	•	if event disappears before confirmations, ignore and continue listening

⸻

9. Authentication & Authorization (MVP)

Customer Portal
	•	No account system in MVP; relies on MetaMask address.
	•	Rate-limit order creation per address/IP to prevent spam.

Reception Console
	•	Requires staff login (simple username/password or magic link).
	•	Staff account is mapped to a siteId (or list of siteIds).
	•	Reception can only deliver orders for their site (if site enforcement enabled).

Admin actions (mint/create type):
	•	Off-chain admin panel optional; can be done via scripts in Phase 1.

⸻

10. Site applicability (recommended)
	•	Each voucher type has applicability:
	•	ALL sites, or
	•	ALLOWLIST of siteIds
	•	Backend must enforce:
	•	order.siteId is allowed for tokenId
	•	reception staff belongs to that siteId (if enforcement enabled)

⸻

11. Observability & logs
	•	Listener logs: event received, confirmations, matched/unmatched, errors
	•	Backend logs: order creation, status transitions, deliver actions
	•	Suggested dashboards:
	•	orders created/valid/delivered/expired
	•	redemption confirmation time (p50/p95)
	•	mismatch count
