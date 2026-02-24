# docs/SLA-SLO.md — SLA/SLO Targets (Pilot & Production)

This document defines reliability targets for the MVP during pilot and future production.

---

## 1) Definitions
- **SLA**: contractual or externally communicated uptime guarantee (usually stricter, legal implications).
- **SLO**: internal target for reliability/performance.
- **SLI**: measured indicator (uptime, latency, validation time).

For Phase 1 MVP:
- We use **SLOs** (internal targets). Avoid publishing strict SLAs until stable.

---

## 2) Key user journeys (what we must protect)
1) Create order (Portal → Backend)
2) Redeem on-chain (Customer tx)
3) Listener validates event → Order becomes VALID
4) Reception scan → deliver → mark DELIVERED

---

## 3) Proposed SLOs (Pilot)
### 3.1 Backend API availability
- **SLO**: 99.5% monthly availability (pilot)
- Measurement: % of successful responses for key endpoints

### 3.2 Reception validation latency
- **SLO**: p95 < 500 ms for `GET /orders/{orderId}` (within VN)
- Note: Reception UX must be fast.

### 3.3 Redemption confirmation time (end-to-end)
- **SLO**: p95 < 2 minutes from on-chain tx to order VALID
- Assumption: BNB chain stable + RPC stable + confirmations=3

### 3.4 Listener uptime
- **SLO**: 99.5% monthly
- Measurement: worker heartbeat + block height progression

---

## 4) Error budgets (pilot)
- API downtime allowed per month (99.5%):
  - ~3h 36m downtime/month
- Use downtime events to prioritize fixes.

---

## 5) Alerts (minimum)
- Listener no progress for > 3 minutes
- API 5xx rate > 2% for 5 minutes
- DB connection failures
- Spike in mismatched events or failed validations

---

## 6) Future (production targets)
After stable pilot:
- Backend availability SLO: 99.9%
- Redemption p95: < 90 seconds
- Add multi-RPC failover and multi-instance backend
