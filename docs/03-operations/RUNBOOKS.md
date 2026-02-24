# docs/RUNBOOKS.md — Operational Runbooks

This document provides operational procedures for common incidents and routine tasks.

---

## 1) Routine operations

### 1.1 Daily checklist (Reception)
- Scan QR → confirm status VALID → deliver → mark DELIVERED
- If not VALID: do not deliver; instruct customer to redeem again or contact support

### 1.2 Daily checklist (Ops/Admin)
- Check dashboard:
  - orders created/valid/delivered/expired
  - listener health
  - error logs
- Verify backups (DB) completed (pilot/prod)

---

## 2) Incident: Listener is down (no orders become VALID)

Symptoms:
- Customer redeems on-chain but reception shows CREATED/PENDING_CHAIN
- Listener health endpoint fails

Actions:
1) Verify RPC availability
2) Restart listener service (systemd/docker)
3) Check logs for:
   - RPC disconnect
   - contract address mismatch
   - event parsing errors
4) Confirm latest block height is progressing
5) Reprocess missed blocks (if implemented) or replay from last saved block

Mitigation:
- Reception should only deliver when status is VALID.

---

## 3) Incident: RPC provider outage / instability

Symptoms:
- Listener can't fetch blocks/events
- Backend may still work but cannot validate

Actions:
1) Switch to backup RPC endpoint (staging/prod should have 2 providers)
2) Increase retry/backoff temporarily
3) Monitor confirmation times

---

## 4) Incident: Contract emergency (suspected exploit or wrong mint)

Actions (highest priority):
1) **PAUSE contract** immediately (ADMIN_ROLE)
2) Stop minting operations (disable MINTER processes)
3) Communicate internally: “deliver only if order is VALID”
4) Investigate:
   - chain events: abnormal mint/redeem
   - admin/minter wallet activity
5) Fix + rotate keys if needed
6) Unpause only after confirmation

---

## 5) Incident: Reception delivered twice / dispute

Prevention:
- DELIVER endpoint must be idempotent
- Order state must be one-way (VALID → DELIVERED)

Actions:
1) Check order audit log:
   - deliveredBy, deliveredAt, deliveredNote
2) Verify chain proof (txHash) and orderHash match
3) If operational mistake:
   - log an incident
   - apply internal resolution (voucher replacement optional)

---

## 6) Incident: Customers lack BNB gas

Symptoms:
- Customer cannot redeem due to insufficient BNB

Actions:
- Provide help guide:
  - how to top up small BNB
  - how to switch to BNB network
- Optional (future): “gas top-up” policy for VIP customers

---

## 7) Key rotation runbook (minter)
When minter key is suspected compromised:
1) Pause contract (optional but recommended)
2) Revoke MINTER_ROLE from old wallet
3) Grant MINTER_ROLE to new wallet
4) Update internal procedures and `.env` if any tooling references it
5) Unpause (if paused)

---

## 8) Backup & restore test (pilot/prod)
- Perform monthly restore test:
  1) Restore DB snapshot to a test instance
  2) Validate sample orders and statuses
  3) Document results
