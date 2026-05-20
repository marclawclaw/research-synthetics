---
tags: [pattern, trust, bridge, custody]
seen_in: [hyperliquid, sbtc-stacks, dydx-v4]
---

# Bridge Trust Assumptions

Synthetic asset systems frequently rely on a bridge or custody layer that holds the underlying asset (USDC, BTC, or other collateral) while a derived asset is used on the application chain. The trust model of that custody layer is the dominant single risk in many designs.

## Implementations and trust models

- **[[projects/hyperliquid]] USDC bridge:** USDC is custodied on Arbitrum by a 3-of-4 effective multisig of validator-controlled keys. Withdrawals require 2/3 of staking power to sign, then go through a ~200 second dispute window where only validators (not the public) can dispute. Independent reviews note the same team controls both hot and cold validator key sets.
- **[[projects/sbtc-stacks]] BTC peg:** Bitcoin is custodied on the Bitcoin L1 by a 70%-weighted threshold signature over a permissionless signer set (currently 15, design ceiling 150). Signers post STX bond as economic security. Peg-outs are contract-bound: signers can only co-sign transactions matching the on-chain withdrawal request.
- **[[projects/dydx-v4]] no-bridge USDC:** dYdX Chain holds USDC natively (via Noble + IBC). There is no external bridge contract holding user collateral; the security model is the 2/3 of the dYdX Chain validator set.

## Risk spectrum

| Model | Trust assumption | Decentralisation | Speed of finality |
|-------|------------------|-----------------|-------------------|
| Federated multisig (WBTC) | Trust the custodian operators | Low (5-15 operators) | Slow (manual) |
| Validator multisig (Hyperliquid) | Trust 2/3 of small validator set | Low-medium (4 nodes) | Fast (~200s) |
| Threshold signatures (sBTC) | Trust 70% of permissionless signers | Medium-high (15-150 bonded signers) | Medium (Bitcoin blocks) |
| Native chain custody (dYdX) | Trust 2/3 of L1 validator set | Depends on validator set diversity | Block time |
| Optimistic rollup (Arbitrum/OP) | Trust at least one honest fraud-prover | High (anyone can prove fraud) | Slow (~7 day window) |

## Failure modes

- **Hot key compromise:** if the multisig hot signers' keys are stolen, attacker drains the bridge.
- **Signer collusion:** if more than the threshold colludes, peg can be drained or peg-outs censored.
- **Bridge contract bug:** the smart contract itself may have flaws independent of the signer set.
- **Liveness failure:** in threshold schemes, if too many signers go offline, withdrawals stall even though funds are safe.
- **Dispute window too short to detect malice:** Hyperliquid's 200s window is materially shorter than Arbitrum's 7 days. If the team itself is compromised, the public has no recourse.

## Mitigations

- **Bond + slashing:** sBTC signers post STX. If they misbehave, bond is slashed. Economic disincentive scales with TVL.
- **Permissionless signer entry:** allows the set to grow and rotate without manual onboarding.
- **Contract-bound co-signing:** signers cannot redirect funds outside the contract's withdrawal logic.
- **Auditor + bug bounty + insurance fund:** standard practice. Hyperliquid's bridge contract was audited by Cyfrin.
- **Dispute window length:** longer windows give the public more time to detect and prove malicious activity at the cost of capital efficiency.

## Relevance to RFP

If an RFP includes a synthetic asset backed by collateral on another chain, the bridge or custody layer is the most important trust assumption to specify. Requirements should make explicit:

- Number and identity of signers/validators.
- Threshold and signing scheme.
- Whether signers are permissionless and bonded.
- Dispute window length and who can dispute.
- Contract-binding of signer authority.
- Disclosure of key custody (hot/cold, multi-org).
