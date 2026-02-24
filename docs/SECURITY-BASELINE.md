# docs/SECURITY-BASELINE.md — Security Baseline (MVP)

This document lists minimum security controls for MVP deployment.

---

## 1) Accounts & access
- Reception Console requires login.
- Separate roles:
  - RECEPTION: can validate and deliver
  - ADMIN: can configure catalog, view logs
- Enforce site-based permissions:
  - Reception can only deliver orders for their siteId (if enabled).

---

## 2) Secrets & configuration
- Never commit secrets to Git.
- Store `.env` with restrictive permissions (chmod 600).
- Rotate credentials if leaked.

---

## 3) Server hardening (VPS)
- Ubuntu 22.04 LTS
- Firewall:
  - allow 22 (SSH), 80/443 (web)
  - deny all others
- SSH:
  - key-based auth only
  - disable password login
  - optional: change SSH port (not required but acceptable)
- Updates:
  - enable unattended security updates or weekly patch schedule

---

## 4) HTTPS / TLS
- Enforce HTTPS (Let’s Encrypt)
- Redirect HTTP → HTTPS
- HSTS recommended for prod

---

## 5) Application security
- Rate limiting:
  - order creation per wallet/IP
- Input validation:
  - tokenId, amount bounds
  - siteId format
- Idempotency:
  - deliver endpoint must be idempotent
- Logging:
  - include requestId
  - store audit logs for deliver actions

---

## 6) Blockchain-specific controls
- Admin wallet:
  - hardware wallet recommended
  - minimal usage
- Minter wallet:
  - separate from admin
  - restricted operational procedure
- Contract protections:
  - Pausable
  - maxSupply per tokenId enforced
  - tokenId must exist before mint/redeem

---

## 7) Monitoring & alerting (minimum)
- Health endpoints:
  - backend `/health`
  - listener heartbeat
- Alerts:
  - listener stuck (no new blocks)
  - API error spikes
  - DB unavailable

---

## 8) Backups
- DB backups daily (pilot/prod)
- Verify restore monthly

---

## 9) Incident response baseline
- Pause contract for critical incidents
- Stop minting and deliveries unless order is VALID
- Document incident timeline and remediation actions
