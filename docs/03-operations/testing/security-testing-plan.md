# Security Testing Plan — 5SOffice Web3 Vouchers (A.8.29)
Owner: Nguyễn Đăng Quang  
Last updated: 2026-02-28  
Status: Draft (Phase 1 MVP)

---

## 1. Purpose
This plan defines security testing activities for the 5SOffice Web3 Vouchers project to support ISO/IEC 27001:2022 control **A.8.29 Security Testing in Development and Acceptance**.

Goals:
- Detect security issues early (shift-left).
- Prevent regressions via CI and repeatable checks.
- Provide auditable evidence (CI logs, reports, test results).

---

## 2. Scope
Applies to:
- Smart contract (ERC-1155)
- Backend Order API + DB
- Chain Listener worker
- Customer Portal (web)
- Reception Console (web)
- Deployment configuration (infrastructure/security baseline)

Environments:
- Local/dev
- Staging (BNB Testnet)
- Production (BNB Mainnet, later)

---

## 3. Testing principles
- Prefer automated checks in CI whenever possible.
- “Block release” criteria for critical/high findings (defined below).
- Maintain minimal, realistic threat coverage for MVP (do not over-engineer).

---

## 4. Security testing types (MVP)

### 4.1 Static Application Security Testing (SAST)
Target:
- Backend, listener, frontend (as applicable)

Minimum checks:
- Lint + type checks (TypeScript) or static checks (Python)
- Detect common insecure patterns (unsafe eval, SSRF patterns, injection risk indicators)

Evidence:
- CI job logs + pass/fail status

Acceptance (MVP):
- SAST runs on every PR to main.
- PR cannot merge if critical issues detected (policy configurable).

---

### 4.2 Dependency / Supply Chain Scanning (SCA)
Target:
- npm/pnpm/yarn dependencies
- pip dependencies (if Python)

Minimum checks:
- `npm audit` / `pnpm audit` or equivalent
- GitHub Dependabot alerts enabled (recommended)
- License awareness (optional MVP)

Evidence:
- CI logs, Dependabot screenshots/alerts

Acceptance (MVP):
- Block merge if **Critical** vulnerabilities exist with known exploits and no mitigation.
- High vulnerabilities must have a triage record (issue/ticket) with target fix date.

---

### 4.3 Secret Scanning
Target:
- Entire repository

Minimum checks:
- Gitleaks (recommended) or GitHub secret scanning (if available)
- Ensure `.env` and secrets are not committed

Evidence:
- CI logs and “no leaks” status

Acceptance (MVP):
- Any real secret found → immediate block + rotate secret.

---

### 4.4 Configuration / Baseline checks
Target:
- Deployment configuration, environment variables, server settings (when on VPS)

Minimum checks:
- HTTPS enforced (staging/prod)
- DB not publicly exposed
- Firewall rules documented
- `.env.example` exists

Evidence:
- Deployment checklist completion + screenshots/commands

Acceptance (MVP):
- Staging must use HTTPS (where public).
- DB access restricted (no public IP exposure).

---

### 4.5 Smart Contract Testing (unit + integration)
Target:
- ERC-1155 voucher contract

Minimum unit tests:
- maxSupply enforced per tokenId
- cannot mint beyond maxSupply
- redeemAndBurn burns correct amount
- emits `VoucherRedeemed(user, tokenId, amount, orderHash)`
- pause blocks redeem (and mint behavior as designed)

Optional (nice-to-have for MVP):
- Access control tests (only admin can create type, only minter can mint)

Evidence:
- Test logs + coverage report (optional)

Acceptance (MVP):
- All contract tests pass on PR.
- Contract build artifacts are reproducible.

---

### 4.6 Backend Security Tests (functional security)
Target:
- Order API + DB

Minimum tests:
- Deterministic orderHash generation:
  - exact field order and type (abi.encode equivalent)
- Order expiry enforcement:
  - redeem after expiry must not become VALID
- Deliver endpoint idempotency:
  - deliver twice results in one DELIVERED record
- Authorization:
  - deliver requires reception auth (401/403 as appropriate)
- Rate limiting:
  - excessive order creation returns 429

Evidence:
- Integration test logs + sample requests

Acceptance (MVP):
- These tests pass in CI or as repeatable test scripts.

---

### 4.7 Listener Security Tests (event matching integrity)
Target:
- Chain listener worker

Minimum tests:
- Wait confirmations before marking VALID (CONFIRMATIONS=3)
- Matching rules require:
  - orderHash
  - customerAddress
  - tokenId
  - amount
  - expiresAt window
- Mismatch does not mark VALID

Resilience checks (MVP minimal):
- RPC retry/backoff behavior (smoke test)

Evidence:
- Listener test logs + run logs with sample tx hash/orderId

Acceptance (MVP):
- Listener never marks VALID on first unconfirmed event.
- Listener never marks VALID when mismatch exists.

---

### 4.8 Web Security (Portal + Reception Console)
Target:
- Web UIs

Minimum checks:
- QR parsing accepts only `5SOV://order/{orderId}` format
- No rendering of raw HTML from API
- Basic XSS prevention patterns (framework defaults)
- Reception requires login

Evidence:
- Manual test checklist + screenshots

Acceptance (MVP):
- Reception cannot deliver without auth.
- QR parser rejects invalid formats.

---

## 5. Acceptance testing (UAT) on BNB Testnet
End-to-end acceptance scenario (must pass):
1) Mint voucher to test wallet
2) Portal shows correct balance
3) Create order returns orderId + orderHash + QR payload
4) Redeem burns voucher and emits VoucherRedeemed
5) Listener marks order VALID after confirmations
6) Reception scans QR and marks DELIVERED
7) Double deliver attempt is blocked/idempotent
8) Expired order cannot become VALID

Evidence required:
- Tx hash
- OrderId
- Screenshot of reception status VALID → DELIVERED
- CI run links/logs

---

## 6. Severity & release blocking rules (MVP)
### Block merge / release if:
- Any secret leakage is detected.
- Any Critical vulnerability in dependencies with known exploit and no mitigation.
- Contract fails tests or cannot enforce maxSupply / pause rules.
- Listener does not wait confirmations or can mark VALID without strict match.
- Deliver endpoint is not idempotent.

### Triage required (not necessarily blocking) for:
- High vulnerabilities (with mitigation plan + due date)
- Medium issues (track and schedule)
- Low issues (fix when convenient)

---

## 7. Evidence retention (for audit)
Store evidence in:
- GitHub Actions logs (preferred)
- `docs/03-operations/evidence/` (optional pointers only; avoid storing secrets)

Minimum audit samples:
- 2 PRs with CI passing (SAST/SCA/secret scan)
- Contract test run output
- One E2E test record (txHash + orderId + screenshot)
- One vulnerability triage example (issue/ticket)

---

## 8. References
- docs/01-product/sdlc/sdlc-policy.md
- docs/01-product/sdlc/secure-coding-guidelines.md
- docs/01-product/requirements/appsec-requirements.md
- docs/02-architecture/architecture.md
- docs/02-architecture/redeem-flow.md
- docs/03-operations/security-baseline.md
- docs/00-overview/definition-of-done.md
