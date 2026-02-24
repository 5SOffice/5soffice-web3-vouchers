# docs/ENVIRONMENTS.md — Environments & Configuration

This document defines environments, naming conventions, and configuration rules for 5SOffice Web3 Vouchers.

---

## 1) Environments

### 1.1 Local (DEV)
Purpose:
- Fast development and testing
- Simulate end-to-end flow

Network:
- BNB Testnet (recommended) or local EVM (optional)

Typical stack:
- Docker Compose: backend + listener + db + nginx
- Frontends served locally or via nginx

---

### 1.2 Testnet (STAGING)
Purpose:
- Shared testing for team/dev/vendor
- Pilot rehearsal before mainnet

Network:
- **BNB Testnet**

Must have:
- Separate wallets (admin/minter) from mainnet
- Separate DB and secrets
- Dedicated RPC endpoint (recommended)

---

### 1.3 Mainnet (PROD)
Purpose:
- Real customer usage

Network:
- **BNB Mainnet**

Must have:
- Stronger key management (hardware wallet + multisig for admin recommended)
- Monitoring and backups enabled
- Strict access controls for reception/admin tools

---

## 2) Naming & IDs

### 2.1 Order IDs
Format suggestion:
- `ORD_<YYYYMMDD>_<RANDOM>`  
Example:
- `ORD_20260224_X7K9P1`

### 2.2 Site IDs
Use stable IDs, not human-facing names:
- `HCM_ROX_T09`
- `HCM_IMC_T04`
- `HCM_ACM_T03`
- `HCM_VIETTEL_T12`

Keep a mapping table in `docs/SITE-MAPPING.md` (optional).

### 2.3 Token IDs (ERC-1155)
Scheme (recommended):
- 1xxx: space usage
- 2xxx: print/scan
- 3xxx: access passes
- 4xxx: discounts/promos
- 5xxx: partner services
- 6xxx: digital services (AI Suite)

---

## 3) Configuration Rules

### 3.1 Secrets handling
- Never commit `.env` to Git
- Provide `.env.example` with placeholders only
- Store secrets in:
  - local: `.env.local` (ignored)
  - VPS: `/opt/5soffice/.env` (restricted permissions)

### 3.2 Required environment variables (minimum)
Backend:
- `APP_ENV` = local | staging | prod
- `BASE_URL` (public)
- `DB_URL`
- `RPC_HTTP_URL`
- `RPC_WS_URL` (optional)
- `CONTRACT_ADDRESS`
- `CONFIRMATIONS` (default 3 on BNB)
- `ORDER_EXPIRY_SECONDS` (default 1800)
- `RATE_LIMIT_PER_MINUTE` (default 10)

Listener:
- `APP_ENV`
- `DB_URL`
- `RPC_HTTP_URL` or `RPC_WS_URL`
- `CONTRACT_ADDRESS`
- `CONFIRMATIONS`

Frontend:
- `APP_ENV`
- `API_BASE_URL`
- `CHAIN_ID` (BNB testnet or mainnet)
- `CONTRACT_ADDRESS`

---

## 4) Deployment separation
- Each environment must have its own:
  - DB
  - RPC endpoints (recommended)
  - contract address
  - domain/subdomain

Suggested domains:
- DEV: `localhost`
- STAGING: `staging-vouchers.5soffice.com.vn`
- PROD: `vouchers.5soffice.com.vn`

---

## 5) Change management
- Use Git tags/releases for milestones:
  - `phase0-docs-v1`
  - `mvp-testnet-v1`
  - `pilot-site1-v1`
- Any change to orderHash encoding or QR format requires:
  - version bump in docs
  - backward compatibility plan
