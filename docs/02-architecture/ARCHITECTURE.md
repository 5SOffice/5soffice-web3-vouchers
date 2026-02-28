Architecture — 5SOffice Web3 Vouchers (BNB Chain)

This document is the single source of truth for the MVP architecture.
It is designed so that a developer (or AI coder) can implement the system without guessing.

⸻

1. System components

1) Smart Contract (BNB Chain)
	•	ERC-1155 voucher contract
	•	Supports creating voucher types, minting, and redeeming (burn) with orderHash
	•	Emits events consumed by the backend listener

2) Customer Portal (Web)
	•	Connect MetaMask
	•	View voucher balances (by tokenId)
	•	Create redeem orders and show QR code
	•	Trigger on-chain redeem transaction: redeemAndBurn(tokenId, amount, orderHash)

3) Backend API (Order Service)
	•	Creates orders and generates orderHash
	•	Stores order states and service mapping rules
	•	Provides validation API for reception console

4) Chain Listener (Worker)
	•	Subscribes to contract events (VoucherRedeemed)
	•	Waits for confirmations
	•	Matches event → updates order status to VALID

5) Reception Console (Web)
	•	Scans QR
	•	Validates order state with backend
	•	Marks order as DELIVERED after service is provided

⸻

2. Data model (minimum)

2.1 Order
	•	orderId (string, unique)
	•	customerAddress (0x…)
	•	tokenId (uint256)
	•	amount (uint256)
	•	siteId (string, recommended; use "" if not enforcing sites yet)
	•	status (enum): CREATED | PENDING_CHAIN | VALID | DELIVERED | EXPIRED | CANCELLED
	•	orderHash (bytes32)
	•	createdAt (ISO datetime)
	•	expiresAt (unix seconds, uint64; deterministic for hashing)
	•	chainTxHash (optional)
	•	chainBlockNumber (optional)
	•	deliveredAt (optional)
	•	deliveredBy (optional)
	•	deliveredNote (optional)
	•	customerMeta (optional; depends on voucher type)
	•	aiAccount (for AI Suite)
	•	jobProvider (for Job VIP posting)

2.2 Voucher Type Config
	•	tokenId
	•	name
	•	group (A | B | C)
	•	unit (e.g., 1h, 10 pages, 30 days)
	•	siteApplicability:
	•	ALL, or
	•	ALLOWLIST (array of siteId)
	•	maxSupply
	•	uri (metadata)
	•	conditions (optional; for Group B/C)

2.3 Staff user (Reception)
	•	staffId
	•	displayName
	•	role (RECEPTION | ADMIN)
	•	siteIds (array) or single siteId

⸻

3. QR payload format (MVP)

QR encodes only orderId (no sensitive content).

QR string format
	•	5SOV://order/{orderId}

Example
	•	5SOV://order/ORD_20260224_X7K9P1

Reception Console extracts orderId and calls backend to validate.

⸻

4. API Contract (minimum set)

All API responses SHOULD include a requestId for debugging and correlation.

4.1 Create order

Endpoint
	•	POST /api/v1/orders

Request

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

Response

{
  "requestId": "req_01HZY...123",
  "orderId": "ORD_20260224_X7K9P1",
  "status": "CREATED",
  "expiresAt": 1771929000,
  "orderHash": "0x9f... (bytes32)",
  "qrPayload": "5SOV://order/ORD_20260224_X7K9P1"
}

Rules
	•	Validate tokenId exists in catalog
	•	Validate amount is allowed (normally 1 for MVP)
	•	If site enforcement enabled: validate siteId is allowed for that tokenId
	•	Apply rate limit per wallet/IP

⸻

4.2 Get order (reception validation)

Endpoint
	•	GET /api/v1/orders/{orderId}

Response

{
  "requestId": "req_01HZY...456",
  "orderId": "ORD_20260224_X7K9P1",
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
  "expiresAt": 1771929000,
  "chainProof": {
    "txHash": "0x123...",
    "blockNumber": 12345678,
    "confirmed": true,
    "confirmations": 3
  }
}


⸻

4.3 Mark delivered (reception action)

Endpoint
	•	POST /api/v1/orders/{orderId}/deliver

Request

{
  "deliveredBy": "staff_001",
  "deliveredNote": "Room A - 10:00 to 11:00"
}

Response

{
  "requestId": "req_01HZY...789",
  "orderId": "ORD_20260224_X7K9P1",
  "status": "DELIVERED",
  "deliveredAt": "2026-02-24T10:05:12Z"
}

Idempotency rule
	•	If order is already DELIVERED, repeated calls must return the existing delivered state (no duplicates).

Authorization rule
	•	Only staff with RECEPTION role can call this endpoint.
	•	If site enforcement enabled: staff site must match order site.

⸻

4.4 Voucher catalog (optional but recommended)

Endpoint
	•	GET /api/v1/catalog

Response

{
  "requestId": "req_01HZY...999",
  "version": "2026.02.24",
  "items": [
    {
      "tokenId": 1001,
      "name": "Meeting Room 1h",
      "group": "A",
      "unit": "1h",
      "siteApplicability": {
        "mode": "ALLOWLIST",
        "siteIds": ["HCM_ROX_T09", "HCM_IMC_T04"]
      }
    }
  ]
}


⸻

5. orderHash encoding (MUST be consistent)

Backend and contract must use the same deterministic encoding.

Canonical definition
	•	orderHash = keccak256(abi.encode(orderId, customerAddress, tokenId, amount, siteId, expiresAt))

Field types
	•	orderId: string
	•	customerAddress: address
	•	tokenId: uint256
	•	amount: uint256
	•	siteId: string (use "" if not enforcing sites yet)
	•	expiresAt: uint64 unix seconds

Important
	•	Use abi.encode(...) (NOT abi.encodePacked(...)) to reduce collision risks.

⸻

6. End-to-end sequence
	1.	Portal → Backend: Create Order (POST /orders)
	2.	Backend → Portal: returns orderId, orderHash, expiresAt, qrPayload
	3.	Portal → Contract: redeemAndBurn(tokenId, amount, orderHash) via MetaMask
	4.	Listener: receives VoucherRedeemed(...) event
	5.	Listener → Backend: matches order, waits confirmations, sets status VALID
	6.	Reception → Backend: scans QR, checks order status
	7.	Reception → Backend: marks DELIVERED

⸻

7. Listener matching logic

Listener matches using:
	•	orderHash (primary)
	•	user == customerAddress
	•	tokenId and amount match
	•	current time <= expiresAt
	•	order status in {CREATED, PENDING_CHAIN}

On success:
	•	set status = VALID
	•	store chainTxHash, chainBlockNumber

On mismatch:
	•	no status change
	•	log mismatch with event details

⸻

8. Confirmations and reorg handling

Confirmations (BNB Chain MVP)
	•	CONFIRMATIONS = 3 (tunable)

Rule
	•	Listener MUST NOT mark VALID until confirmations are met.

Reorg strategy (simple)
	•	If event disappears before confirmations, ignore and continue listening.

⸻

9. Authentication & authorization (MVP)

Customer Portal
	•	No accounts in MVP; identity is MetaMask address.
	•	Rate limit order creation by wallet/IP.

Reception Console
	•	Staff login required (simple username/password or magic link).
	•	Roles: RECEPTION, ADMIN.
	•	Staff is associated with siteId (or siteIds) for delivery authorization.

⸻

10. Site applicability (recommended)

Each voucher type:
	•	applies to ALL sites, or
	•	applies to an ALLOWLIST of siteIds

Backend enforces:
	•	order.siteId must be valid for tokenId
	•	reception staff must be assigned to that site

⸻

11. Observability (minimum)
	•	Backend: /health endpoint
	•	Listener: heartbeat + last processed block
	•	Metrics:
	•	orders created / valid / delivered / expired
	•	redemption confirmation time (p50/p95)
	•	mismatch count

⸻
