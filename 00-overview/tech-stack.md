# Tech Stack (Proposed) — Phase 1 MVP

This file locks the tech choices to reduce ambiguity for devs and AI coding tools.

---

## 1) Blockchain
- Chain: **BNB Chain**
- Environment: **BNB Testnet (Phase 1)** → Mainnet later
- Token standard: **ERC-1155**
- Wallet: **MetaMask**
- Confirmations: **3** (default)

---

## 2) Smart contract tooling
Choose one:
- Option A: **Hardhat (TypeScript)**
- Option B: **Foundry**

Requirement:
- Unit tests included
- Contract verification script (optional)

---

## 3) Backend
Choose one:
- Option A: **Node.js + TypeScript (Fastify/NestJS/Express)**
- Option B: **Python (FastAPI)**

Minimum requirements:
- REST API
- DB layer + migrations
- Structured logging (requestId)
- Rate limiting

---

## 4) Database
- Pilot/Prod: **PostgreSQL**
- Early demo allowed: SQLite (documented)

---

## 5) Chain Listener
- Runs as a separate worker process
- Uses RPC (HTTP or WS)
- Retries with backoff
- Stores last processed block (DB)

---

## 6) Frontend
- Customer Portal: **React / Next.js** (recommended)
- Reception Console: **React / Next.js** (can be same app with routes)
- QR scanning library (web camera)

---

## 7) Ops
- Reverse proxy: **Nginx**
- Deployment: Docker Compose (MVP)
- TLS: Let’s Encrypt
- Basic monitoring: health checks + logs

---

## 8) Non-goals (Phase 1)
- No account abstraction / gasless transactions
- No custodial wallets / OTP
- No secondary market features
