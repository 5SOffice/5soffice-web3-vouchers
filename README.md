# 5SOffice Voucher & AI Credit Engine

**Entitlements, service credits and redemptions for 5SOffice customers and SME tenants — with optional Web3 representation.**

This repository is being repositioned from a Web3-first NFT voucher application into a business-first **Voucher & AI Credit Engine**. The core customer journey uses a normal 5SOffice account, QR code and entitlement ledger. ERC-1155 on BNB Chain remains available as an optional adapter for selected campaigns, demonstrations or partners.

## Product direction

```text
5SOffice Webapp / Tenant Portal / Tenant Agent
                     |
                     v
          Voucher & AI Credit Engine
 Catalog | Rules | Wallet | Reservation | Redemption | Ledger
                     |
          +----------+-----------+
          |                      |
          v                      v
  Business Services       Optional Web3 Adapter
 Booking / Printing / AI   ERC-1155 / BNB Chain
```

## Why this change

A wallet-first and on-chain-only flow adds friction for most SME customers. Business operations must not depend on MetaMask, network fees or blockchain availability. The new design keeps the reusable value of the original voucher concept while making normal account and QR-based use the default.

## Core capabilities

- voucher and service-credit catalog;
- eligibility, validity and campaign rules;
- customer/tenant entitlement wallet;
- reservation before service delivery;
- confirmation, cancellation and release;
- immutable-enough usage ledger and audit trail;
- QR redemption for reception and service staff;
- AI usage credits and expert-escalation credits;
- API integration with Webapp, Tenant Agent and business services;
- optional mint, transfer and redemption proof through a Web3 adapter.

## AI Service Credits

Examples of rights that can be granted by tenancy packages or promotions:

- document actions;
- meeting summaries;
- marketing workflow actions;
- compliance checks;
- expert review or escalation;
- AI Suite access periods.

Credits may be included in a service package, purchased, granted by campaign or converted from a voucher. Users should see a simple balance and usage history without needing blockchain knowledge.

## Default customer journey

```text
Sign in to 5SOffice account
  -> view eligible vouchers and credits
  -> choose service or ask the Tenant Agent
  -> preview conditions and balance impact
  -> confirm
  -> reserve entitlement
  -> complete business transaction
  -> confirm redemption
  -> receive QR/reference and ledger record
```

## Optional Web3 journey

Selected entitlements may be mirrored or issued as ERC-1155 tokens. The adapter may provide ownership or redemption proof, but the core ledger and business service remain authoritative for operational delivery unless a future approved design states otherwise.

## Documentation

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

## Non-goals

- A financial token, investment product or promise of appreciation.
- Cash-out or speculative trading features.
- Requiring customers to use MetaMask.
- Making business-service availability depend on blockchain uptime.
- Storing personal or operationally sensitive data on-chain.

## Project owner

**Nguyễn Đăng Quang** — Project Owner and Concept Creator; responsible for product direction, governance and quality review.
