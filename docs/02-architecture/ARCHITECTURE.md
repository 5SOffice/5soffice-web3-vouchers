# Architecture — 5SOffice Web3 Vouchers (BNB Chain)

This document is the single source of truth for the MVP architecture.
It is designed so that a developer (or AI coder) can implement the system without guessing.

---

## 1. System components

### 1) Smart Contract (BNB Chain)
- ERC-1155 voucher contract
- Supports creating voucher types, minting, and redeeming (burn) with `orderHash`
- Emits events consumed by the backend listener

### 2) Customer Portal (Web)
- Connect MetaMask
- View voucher balances (by tokenId)
- Create redeem orders and show QR code
- Trigger on-chain redeem transaction: `redeemAndBurn(tokenId, amount, orderHash)`

### 3) Backend API (Order Service)
- Creates orders and generates `orderHash`
- Stores order states and service mapping rules
- Provides validation API for reception console

### 4) Chain Listener (Worker)
- Subscribes to contract events (VoucherRedeemed)
- Waits for confirmations
- Matches event → updates order status to `VALID`

### 5) Reception Console (Web)
- Scans QR
- Validates order state with backend
- Marks order as `DELIVERED` after service is provided

---

## 2. Data model (minimum)

### 2.1 Order
- `orderId` (string, unique)
- `customerAddress` (0x...)
- `tokenId` (uint256)
- `amount` (uint256)
- `siteId` (string, recommended; use `""` if not enforcing sites yet)
- `status` (enum): `CREATED | PENDING_CHAIN | VALID | DELIVERED | EXPIRED | CANCELLED`
- `orderHash` (bytes32)
- `createdAt` (ISO datetime)
- `expiresAt` (unix seconds, uint64; deterministic for hashing)
- `chainTxHash` (optional)
- `chainBlockNumber` (optional)
- `deliveredAt` (optional)
- `deliveredBy` (optional)
- `deliveredNote` (optional)
- `customerMeta` (optional; depends on voucher type)
  - `aiAccount` (for AI Suite)
  - `jobProvider` (for Job VIP posting)
  - etc.

### 2.2 Voucher Type Config
- `tokenId`
- `name`
- `group` (`A | B | C`)
- `unit` (e.g., `1h`, `10 pages`, `30 days`)
- `siteApplicability`:
  - `ALL`, or
  - `ALLOWLIST` (array of `siteId`)
- `maxSupply`
- `uri` (metadata)
- `conditions` (optional; for Group B/C)

### 2.3 Staff user (Reception)
- `staffId`
- `displayName`
- `role` (`RECEPTION | ADMIN`)
- `siteIds` (array) or single `siteId`

---

## 3. QR payload format (MVP)

QR encodes only `orderId` (no sensitive content).

**QR string format**
- `5SOV://order/{orderId}`

**Example**
- `5SOV://order/ORD_20260224_X7K9P1`

Reception Console extracts `orderId` and calls backend to validate.

---

## 4. API Contract (minimum set)

All API responses SHOULD include a `requestId` for debugging and correlation.

### 4.1 Create order
**Endpoint**
- `POST /api/v1/orders`

**Request**
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
