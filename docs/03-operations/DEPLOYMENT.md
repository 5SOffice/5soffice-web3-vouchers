# Deployment — Checklist (BNB Testnet → Mainnet)

> This is a living document. Start with testnet and only move to mainnet after pilot validation.

## 1. Prerequisites
- MetaMask installed
- Dedicated wallets:
  - Admin wallet (hardware wallet recommended)
  - Minter wallet (separate from admin)
- Small BNB balance for gas:
  - Testnet BNB (from faucet)
  - Mainnet BNB (small amount for deployments and operations)

## 2. Network selection
- Phase 1 build and test on **BNB Testnet**
- Pilot can run on testnet or mainnet with limited supply (decision later)
- Production on **BNB Mainnet** after security readiness

## 3. Contract deployment steps (high level)
### Testnet
1) Configure RPC + chainId in your deployment tool (Foundry/Hardhat).
2) Deploy ERC-1155 voucher contract.
3) Verify contract on explorer (optional but recommended).
4) Create voucher types:
   - tokenId, maxSupply, URI
5) Assign roles:
   - ADMIN_ROLE (admin wallet)
   - MINTER_ROLE (minter wallet)
6) Mint a small set of vouchers for internal testing wallets.

### Mainnet (later)
- Repeat with new addresses and stricter key management.
- Use multisig for admin before scaling.

## 4. Post-deploy configuration
- Record:
  - contract address
  - chainId
  - deployment tx hash
  - role assignments
- Update backend config:
  - contract address
  - ABI
  - event signature
  - confirmations requirement (e.g., 3–10 blocks)

## 5. Backend & listener deployment
- Deploy backend + listener (Docker recommended)
- Environment variables:
  - DB connection string
  - RPC endpoint
  - contract address
  - confirmations required
- Health checks:
  - listener connected to RPC
  - event subscription working
  - order create and status update working

## 6. End-to-end test checklist
- [ ] Mint voucher to a test wallet
- [ ] Portal shows correct balance
- [ ] Create redeem order (OrderID + QR)
- [ ] Redeem & burn on-chain (MetaMask)
- [ ] Listener confirms event and sets order VALID
- [ ] Reception scans QR and marks DELIVERED
- [ ] Logs/audit entries recorded

## 7. Rollback / emergency
- [ ] Pause contract (admin)
- [ ] Disable minting operations
- [ ] Stop deliveries unless order is VALID
- [ ] Investigate and fix
- [ ] Unpause after verification

## 8. To be added later
- Verified contract source + releases
- Multisig setup steps
- Monitoring/alerting setup
- Backup and DR plans
