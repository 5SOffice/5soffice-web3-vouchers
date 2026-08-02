# Optional Web3 Adapter

## Purpose

Preserve the original ERC-1155 innovation as an optional representation or proof layer without making normal customer operations blockchain-dependent.

## Supported scenarios

- branded Web3 campaigns;
- partner-issued vouchers;
- demonstrable ownership or redemption proof;
- controlled technical pilots;
- interoperability experiments.

## Boundary rules

1. The core engine owns catalog, eligibility, reservation, service delivery and operational ledger.
2. The adapter owns chain mapping, transaction submission, event listening and reconciliation.
3. No personal data, booking details or sensitive business information is placed on-chain.
4. Token IDs map to versioned entitlement definitions.
5. Chain events do not directly deliver a service; they update a reconciliation workflow.
6. Contract pause, key management, role separation and incident runbooks are mandatory.
7. Customers using the default account/QR flow do not need a wallet.

## Possible modes

- **Mirror mode:** off-chain entitlement is authoritative; token mirrors a selected right.
- **Claim mode:** token proves eligibility to claim an off-chain entitlement.
- **Burn-proof mode:** redemption triggers burn after business confirmation.

The mode must be defined per campaign. Mixed or ambiguous authority is prohibited.
