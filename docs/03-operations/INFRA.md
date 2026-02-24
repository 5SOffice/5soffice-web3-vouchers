# Infrastructure Guide (INFRA.md) — 5SOffice Web3 Vouchers

This document describes practical infrastructure options to run the MVP on **BNB Chain** with **MetaMask**, focusing on low-cost VPS and local development.

> Scope: Phase 1 MVP (Portal + Backend + DB + Chain Listener + Reception Console)

---

## 1) System Components (MVP)

**Required services**
- **Frontend (Customer Portal)**: static web (connect MetaMask, create order, redeem)
- **Frontend (Reception Console)**: static web (QR scan, validate, deliver)
- **Backend API (Order Service)**: creates orders, returns QR payload & orderHash, exposes validation endpoints
- **Database**: stores orders, statuses, site rules, audit logs
- **Chain Listener (Worker)**: subscribes to `VoucherRedeemed` events, waits confirmations, marks orders VALID
- **Reverse Proxy**: Nginx + HTTPS

**Not required**
- Running a BNB full node (too heavy). Use RPC providers.

---

## 2) Local Development (fastest iteration)

**Minimum dev machine**
- CPU: 4 cores
- RAM: 8 GB (recommended 16 GB for Docker)
- Disk: 30–50 GB free SSD
- Tools: Docker, Git, Node.js (or Python), MetaMask

**Local stack**
- Docker Compose: `backend + listener + db + nginx`
- Frontends served by Nginx or local dev server

Best for: rapid development, internal testing, quick demos.

---

## 3) VPS for Testnet MVP (low-cost)

### Option A — Minimal but stable (recommended for cheap VPS)
- CPU: 1 vCPU
- RAM: 2 GB
- Disk: 30–40 GB SSD
- OS: Ubuntu 22.04 LTS
- Docker: Yes

Runs:
- backend API
- chain listener
- Postgres (small)
- nginx
- static frontends

### Option B — Comfortable (recommended for pilot/demo)
- CPU: 2 vCPU
- RAM: 4 GB
- Disk: 50 GB SSD

Best for: smoother performance, less tuning, more logs/monitoring.

> Note: 1 GB RAM can work with SQLite and minimal logs, but it’s easy to OOM with Docker + Postgres. Prefer 2 GB.

---

## 4) One VPS vs Two VPS

### Single VPS (Phase 1 MVP)
- Simplest, cheapest, easiest to operate
- Docker Compose runs everything

Recommended for:
- testnet MVP
- internal demo
- early pilot at 1 site

### Two VPS (Phase 2+ / pilot scaling)
- VPS A: `nginx + backend + db`
- VPS B: `listener + monitoring`
- Listener failures won’t affect reception UI and order API

Recommended for:
- multi-site pilot
- higher uptime needs

---

## 5) Database Choice

### SQLite (fastest MVP)
- Pro: simplest deployment (single file), low resource
- Con: concurrency limits, scaling harder

Use for:
- internal testnet demo
- very small pilot

### PostgreSQL (recommended for real pilot)
- Pro: stable, concurrent, standard
- Con: slightly more resource

Use for:
- any production-like pilot
- multi-site operations

---

## 6) RPC Strategy (BNB Chain)

You do **not** need a full node.
- Use **public RPC** for early testing (may be unstable).
- For pilot/prod, use a **reliable RPC provider**.

Listener requirements:
- stable websocket or polling support
- low-latency and consistent indexing

Config:
- `CONFIRMATIONS=3` (BNB MVP default; tune later)
- retry with backoff on RPC errors

---

## 7) Resource Sizing by Usage (rough guidance)

- **< 500 orders/day**: 1–2 vCPU, 2 GB RAM, Postgres OK
- **500–5,000 orders/day**: 2 vCPU, 4–8 GB RAM, Postgres + caching
- **> 5,000 orders/day**: separate listener, add cache (Redis), monitoring & alerting

Reception scanning load is usually small; backend/DB is the main factor.

---

## 8) Security Baseline (VPS)

**Server**
- Enable firewall: allow only `22`, `80`, `443`
- SSH key login only (disable password login)
- Run services under non-root where possible
- Regular security updates

**Secrets**
- `.env` stored on server only
- Never commit private keys or RPC secrets
- Admin/minter keys stay in wallets; server should not store them

**HTTPS**
- Use Let’s Encrypt for TLS certificates
- Force HTTPS redirects

**Logging**
- Rotate logs (logrotate)
- Keep audit logs for deliver actions

**Backups**
- Postgres: daily dump + 7–30 days retention
- SQLite: daily snapshot of DB file
- Store backups off-VPS if possible

---

## 9) Recommended Docker Compose Services (MVP)

Suggested services:
- `nginx` (reverse proxy + TLS)
- `backend` (order API)
- `listener` (event worker)
- `db` (postgres) OR `sqlite` volume
- optional: `adminer` (DB UI) for dev only

Folder structure:
- `/ops/docker-compose.yml`
- `/ops/.env.example`

---

## 10) Deployment Checklist (quick)

### Testnet
- [ ] VPS provisioned (Ubuntu 22.04)
- [ ] Docker installed
- [ ] `.env` configured (RPC URL, DB, contract address)
- [ ] Nginx + HTTPS configured
- [ ] Backend health endpoint OK
- [ ] Listener connected + consuming events
- [ ] End-to-end test: mint → create order → redeem → VALID → DELIVERED

### Mainnet (later)
- [ ] Admin role moved to multisig (recommended)
- [ ] Monitoring + alerting enabled
- [ ] Backups verified restore works
- [ ] Contract pause runbook tested

---

## 11) Suggested Default Specs for 5SOffice MVP
If you want the simplest “works and doesn’t break” setup:

**VPS:** 2 vCPU / 2 GB RAM / 40 GB SSD  
**DB:** Postgres  
**Stack:** Nginx + Backend + Listener + Postgres via Docker Compose  
**Network:** BNB Testnet (then mainnet when ready)

---
