# Research Index: Synthetic Assets

Research scope: on-chain synthetic asset protocols across Ethereum (L1 and L2s), Solana, and Bitcoin L2s. Includes both pure synthetics (CDP-backed price exposure) and perpetuals platforms that produce synthetic exposure to BTC, ETH, SOL and RWAs.

Research date range: data accessed May 2026.

## Discovered protocols

| Rank | Protocol | Ecosystem | Primary metric | Value | Selected |
|------|----------|-----------|---------------|-------|----------|
| 1 | Hyperliquid | Custom L1 (USDC bridge on Arbitrum) | 30d perp volume | $176.06B | yes |
| 2 | GMX V2 | Arbitrum, Avalanche, MegaETH, Botanix | TVL | $203.47M | yes |
| 3 | Jupiter Perps | Solana | JLP TVL | ~$1.3B | yes |
| 4 | Drift Protocol | Solana | Open interest | ~$400M | yes |
| 5 | dYdX v4 | dYdX Chain (Cosmos) | Daily volume | ~$200M (avg) | yes |
| 6 | Synthetix V3 | Ethereum, Base, Arbitrum | Cumulative perp volume | $6.96B | yes |
| 7 | sBTC / Stacks | Bitcoin L2 | sBTC TVL (peak Q1 2026) | $545M | yes |
| 8 | Ostium | Arbitrum | Monthly volume | ~$6B | yes |
| 9 | Derive (Lyra) | Derive Chain (OP Stack) | TVL | [see project note] | no (options, narrower scope) |
| 10 | Parcl | Solana | TVL | [DATA NEEDED] | no (real estate niche) |
| 11 | PreStocks | Solana | TVL | ~$10M (Feb 2026) | no (equity perps, small) |

## Selection rationale

The eight selected protocols cover the design space:

- **CDP-style synthetics:** [[projects/synthetix-v3]]
- **Liquidity-pool perps (LP as counterparty):** [[projects/gmx-v2]], [[projects/jupiter-perps]]
- **CLOB / orderbook:** [[projects/hyperliquid]], [[projects/dydx-v4]]
- **Hybrid vAMM + orderbook:** [[projects/drift]]
- **RWA-extended perps:** [[projects/ostium]]
- **BTC-backed synthetic:** [[projects/sbtc-stacks]]

This covers the full spectrum of trust models (threshold multisigs, validator sets, oracle dependencies, LP risk, decentralised orderbooks) which is the analytical core of the brief.

## Research status

- [ ] [[projects/synthetix-v3]]
- [ ] [[projects/gmx-v2]]
- [ ] [[projects/hyperliquid]]
- [ ] [[projects/dydx-v4]]
- [ ] [[projects/drift]]
- [ ] [[projects/jupiter-perps]]
- [ ] [[projects/ostium]]
- [ ] [[projects/sbtc-stacks]]

## Cross-cutting notes (planned)

- [[metrics/tvl-comparison]]
- [[metrics/volume-comparison]]
- [[patterns/oracle-pricing-model]]
- [[patterns/lp-counterparty-risk]]
- [[patterns/bridge-trust-assumptions]]
- [[patterns/synthetic-collateral-models]]
