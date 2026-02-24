# Token Spec — ERC-1155 Voucher Contract (BNB Chain)

## 1. Standard
- ERC-1155 (multi-token)
- One contract supports multiple voucher types (`tokenId`)

## 2. Roles & permissions
- ADMIN_ROLE
  - Create voucher types
  - Set metadata URIs
  - Pause/unpause
  - Grant/revoke roles
- MINTER_ROLE
  - Mint/airdrop vouchers

> Phase 1 assumes customers burn their own vouchers via MetaMask.
> No custodial redeemer role in MVP.

## 3. Voucher type definition
Each voucher type is defined by:
- tokenId: uint256
- maxSupply: uint256
- mintedSupply: uint256 (tracked)
- uri: string (metadata per tokenId)

Constraints:
- tokenId must be created once before minting
- mintedSupply + mintAmount <= maxSupply

## 4. Required functions (MVP)
### Admin
- createVoucherType(tokenId, maxSupply, uri)
- setTokenURI(tokenId, uri)
- pause()
- unpause()

### Minting
- mint(to, tokenId, amount)
- mintBatch(to, tokenIds[], amounts[])

### Redemption (recommended)
- redeemAndBurn(tokenId, amount, orderHash)
  - burns `amount` of `tokenId` from msg.sender
  - emits VoucherRedeemed(msg.sender, tokenId, amount, orderHash)

## 5. Events
- VoucherTypeCreated(tokenId, maxSupply, uri)
- VoucherMinted(to, tokenId, amount)
- VoucherRedeemed(user, tokenId, amount, orderHash)
- TokenURIUpdated(tokenId, uri)
- Paused(account), Unpaused(account) (standard)

## 6. orderHash format (off-chain)
Backend computes `orderHash` as:
- orderHash = keccak256(
    orderId || customerAddress || tokenId || amount || siteId || expiresAt
  )

Notes:
- Use fixed encoding (ABI encoding) to avoid ambiguity.
- siteId can be an integer or bytes32 mapping if needed.

## 7. Security requirements (MVP)
- Pausable contract to stop redeem/mint during incidents
- ADMIN key should be protected (hardware wallet); multisig recommended for production
- Prevent mint beyond maxSupply
- Validate tokenId exists before mint/redeem

## 8. Upgradeability
MVP: non-upgradeable recommended for simplicity.
If future upgrades required: deploy v2 contract and migrate (phase later).

## 9. Metadata (URI)
Metadata should include:
- name
- description (service unit, redemption rules)
- image (square icon)
- attributes:
  - group: A/B/C
  - unit: "1h" / "10 pages" / "30 days"
  - siteApplicability: "ALL" or site list
