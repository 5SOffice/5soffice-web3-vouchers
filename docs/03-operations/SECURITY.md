# Security — 5SOffice Web3 Vouchers

## 1. Security goals
- Protect admin/minter keys (prevent unauthorized minting or contract control).
- Ensure redemption is one-time and dispute-resistant.
- Provide fast incident response (pause/unpause).
- Reduce abuse (bot airdrops, spam orders, replay attempts).

---

## 2. Key management (Phase 1 MVP)
### Admin wallet (highest privilege)
- Use a hardware wallet (Ledger/Trezor) for ADMIN_ROLE.
- Store seed phrase offline, never in cloud notes.
- Restrict usage: admin wallet should rarely sign transactions.

### Minter wallet (operational)
- MINTER_ROLE can mint vouchers.
- Use a dedicated wallet separate from admin.
- Prefer a hardware wallet; if not possible, use a secured browser profile + device encryption.
- Keep mint functions behind an internal process (approval checklist).

### Environment separation
- Use separate wallets for:
  - Testnet (development)
  - Mainnet (production)
- Never reuse the same seed phrase across environments.

---

## 3. Multisig (Phase later, recommended before scaling)
Before large-scale mainnet operations:
- Move ADMIN_ROLE to a multisig (e.g., Safe).
- Suggested policy: 2-of-3 or 3-of-5 signers.
- Signers: Owner (Quang), Operations lead, Tech lead (or trusted partner).

Rationale:
- Prevent single-point-of-failure and reduce insider risk.

---

## 4. Contract-level protections
### Pausable
- Contract must support pause/unpause.
- When paused:
  - minting and redeeming should be disabled (or at least redeem disabled).
- Use pause during:
  - suspected key compromise
  - unexpected minting
  - redemption bugs
  - abnormal activity spikes

### Supply caps
- Each tokenId has maxSupply.
- Mint must enforce mintedSupply + amount <= maxSupply.

### TokenId existence check
- Only allow mint/redeem for created tokenIds.

---

## 5. Redemption integrity
### Recommended: redeemAndBurn(orderHash)
- Redemption event should include orderHash.
- Backend matches:
  - orderHash
  - customer address
  - tokenId
  - amount
  - expiry window

### Prevent double delivery
- Backend order state must be one-way:
  - VALID -> DELIVERED cannot revert
- Reception console must always validate server-side status.

---

## 6. Anti-abuse (off-chain)
### Airdrop abuse
- Limit airdrops per wallet per campaign (rate limit).
- For “claim” campaigns:
  - signature-based claim with nonce + expiry
  - optional allowlist by customer identity (email/phone) in backend (no OTP required)

### Order spam
- Rate limit order creation per wallet/IP.
- Require wallet connection to create order.
- Use short expiry (e.g., 30 minutes).
- Optional: small “cooldown” per wallet for repeated order creation.

### Reception misuse
- Reception login required (staff accounts).
- Log every DELIVERED action with user, timestamp, site.
- Admin view for audit.

---

## 7. Monitoring & alerting (minimum)
- Alert on:
  - unexpected mint volume
  - repeated failed order matches
  - redemption spikes per minute
- Maintain basic dashboards:
  - orders created, valid, delivered, expired
  - redemption confirmation time

---

## 8. Incident response (simple runbook)
1) Pause contract immediately (ADMIN_ROLE).
2) Disable minting operations and rotate MINTER wallet if needed.
3) Investigate logs:
   - chain events (mint/redeem)
   - backend audit logs
4) Communicate to internal ops:
   - stop deliveries unless order VALID
5) Resume (unpause) only after fix and confirmation.

---

## 9. Future hardening (Phase later)
- Multisig admin
- Dedicated deployment machine
- Formal smart contract audit (if scaling)
- Allowlist/transfer restrictions (only if needed)
