---
tags: [pattern, collateral, design]
seen_in: [synthetix-v3, gmx-v2, jupiter-perps, sbtc-stacks, ostium]
---

# Synthetic Collateral Models

Synthetic assets need backing. The choice of collateral model defines the protocol's economic security, capital efficiency and trust assumptions. Across the eight projects researched, five distinct models emerge.

## Models observed

### 1. Native-token CDP (Synthetix V1/V2)

- Backing: protocol's native token (SNX) staked at a high collateral ratio (typically >500%).
- Issuance: stakers mint Synths against their SNX.
- Pros: pure on-chain, no external custody.
- Cons: reflexive (SNX price down → C-ratio down → forced unwinds), capital-inefficient.
- Status in 2026: legacy. V3 moves away from SNX-only.

### 2. Multi-collateral CDP (Synthetix V3)

- Backing: pluggable collateral (USDC, sUSD, ETH derivatives) staked into debt pools per market.
- Issuance: synthetic markets draw credit from the pool.
- Pros: more capital-efficient, less reflexive than native CDP. Stakers can pick risk.
- Cons: still complex; LPs absorb aggregate trader PnL.
- See: [[projects/synthetix-v3]].

### 3. LP-pool counterparty (GMX, Jupiter, Ostium)

- Backing: LPs deposit basket of supported assets into a pool that is the counterparty to traders.
- Issuance: traders take positions against the pool; pool absorbs PnL.
- Pros: simple, predictable execution, scales linearly with pool size.
- Cons: LPs carry directional risk; see [[patterns/lp-counterparty-risk]].
- See: [[projects/gmx-v2]], [[projects/jupiter-perps]], [[projects/ostium]].

### 4. Bridged BTC peg (sBTC)

- Backing: native BTC locked on Bitcoin L1.
- Issuance: 1:1 mint on Stacks against locked BTC.
- Pros: 1:1 backing by the real asset, not a derivative price.
- Cons: bridge trust assumption dominates; signer set integrity is the single point of failure. See [[patterns/bridge-trust-assumptions]].
- See: [[projects/sbtc-stacks]].

### 5. Synthetic via orderbook matching (Hyperliquid, dYdX)

- Backing: USDC collateral per trader; PnL exchanged peer-to-peer via the orderbook.
- Issuance: there is no synthetic asset token; exposure is held as an open position.
- Pros: no LP counterparty, no synthetic-asset peg to maintain.
- Cons: requires deep liquidity (market makers) to function; the orderbook is the product.
- See: [[projects/hyperliquid]], [[projects/dydx-v4]].

## Selection criteria

| Goal | Recommended model |
|------|-------------------|
| Maximum decentralisation of BTC representation | Bridged BTC peg with bonded threshold signers (sBTC) |
| Highest capital efficiency for perp markets | Orderbook matching (Hyperliquid, dYdX) |
| Lowest oracle dependency for synthetic exposure | Orderbook matching (oracle only for liquidation, not execution) |
| Easiest to bootstrap with no professional market makers | LP-pool counterparty (Jupiter, GMX) |
| Long-tail asset coverage | LP-pool with oracle pricing (GMX, Jupiter, Ostium) |
| Cross-margined unified trading | Hybrid (Drift) or orderbook (Hyperliquid, dYdX) |

## Relevance to RFP

The collateral model decides almost every downstream design question (oracle reliance, LP yield model, liquidation engine, risk caps). It should be specified early. Typical RFP requirements:

- Collateral asset(s) and acceptance criteria.
- C-ratio or pool composition rules.
- Withdrawal/redemption mechanism.
- Insurance fund / backstop vault.
- Slashing or socialised loss rules in tail events.
