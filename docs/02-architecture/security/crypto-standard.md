# Cryptography Standard — 5SOffice Web3 Vouchers
Owner: Nguyễn Đăng Quang  
Last updated: 2026-02-27  
Status: Draft (Phase 1 MVP)

---

## 1. Purpose
Define cryptographic requirements for the project to support:
- **A.8.24 Use of Cryptography**
and ensure consistent, secure configuration across environments.

---

## 2. Scope
Applies to:
- Web frontends (Portal, Reception Console)
- Backend API
- Database
- Secret storage
- Deployment pipelines and CI secrets

Note:
- Blockchain cryptography (wallet signatures) is provided by MetaMask and the chain.
- This standard focuses on project-controlled crypto (TLS, storage, secrets).

---

## 3. Transport security (TLS)
### Requirements
- All public endpoints in staging/prod MUST use HTTPS.
- TLS minimum:
  - Prefer TLS 1.3
  - Allow TLS 1.2 if required for compatibility
- Use modern ciphers (default strong nginx settings are acceptable).

### Certificates
- Use Let’s Encrypt for MVP (staging/prod).
- Renew automatically.
- Private keys for TLS certificates must be protected on server.

Evidence:
- TLS configuration and certificate status (screenshot / command output).

---

## 4. Data at rest protection
### Database
- For VPS:
  - Prefer encrypted disks/volumes if supported.
  - At minimum, restrict file permissions and network access.
- Database credentials stored as secrets (not in code).

### Backups
- Backups must be stored securely:
  - restricted access
  - avoid public links
- For production-like pilots:
  - encrypt backups if stored off-server (optional for MVP, recommended for pilot)

Evidence:
- backup job config + sample backup file presence.

---

## 5. Secret management (critical)
### Prohibited
- No secrets in Git repository (no private keys, RPC tokens, DB passwords).
- No secrets in logs.

### Allowed storage (MVP)
- GitHub Secrets for CI
- `.env` on server with restricted permissions (chmod 600)
- Optional: vault/KMS (phase later)

### Rotation
- Rotate immediately if suspected leak.
- Keep separate secrets per environment.

Evidence:
- `.env.example` exists
- secrets present only in runtime environment

---

## 6. Application cryptography usage
### Password hashing (if accounts exist later)
- If reception login uses passwords:
  - use strong hashing (bcrypt/argon2)
- MVP can use simple auth; however production should use stronger mechanisms.

### Token/Session signing
- If using JWT sessions:
  - use strong random secret key (>= 32 bytes)
  - set expiration
  - store secret outside code

---

## 7. Cryptographic baseline for external integrations
- RPC endpoints must be HTTPS (preferred).
- If using third-party services (monitoring, email):
  - store API keys as secrets
  - use TLS endpoints

---

## 8. Minimum checklist (Phase 1)
- [ ] HTTPS enabled on staging/prod
- [ ] Secrets are not committed and not logged
- [ ] `.env.example` present
- [ ] Firewall restricts DB access
- [ ] Backups are protected (not publicly accessible)

---

## 9. References
- docs/03-operations/security-baseline.md
- docs/03-operations/deployment.md
- docs/02-architecture/environments.md
