# Update Manifest — Voucher & AI Credit Engine v2.0

Copy the contents of this package to the root of `5soffice-web3-vouchers`.

## Replace

- `README.md`

## Add

- `docs/00-overview/product-direction-v2.md`
- `docs/01-product/entitlement-catalog-v2.md`
- `docs/01-product/ai-service-credits.md`
- `docs/02-architecture/target-architecture-v2.md`
- `docs/02-architecture/redemption-state-machine.md`
- `docs/02-architecture/api-contracts.md`
- `docs/02-architecture/web3-adapter.md`
- `docs/03-operations/security-and-controls-v2.md`
- `docs/04-roadmap/implementation-roadmap-v2.md`
- `docs/adr/ADR-0001-business-ledger-is-authoritative.md`

## Preserve

Keep the existing ERC-1155 specifications and legacy docs as historical or adapter-specific material. Where they conflict, this update defines the current product direction.

## Suggested repository description

`Voucher, entitlement and AI service credit engine for 5SOffice, with optional ERC-1155 proof on BNB Chain.`

## Suggested commit message

`docs: reposition Web3 vouchers as entitlement and AI credit engine`
