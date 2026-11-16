# Research Index: Synthetic Assets

Research scope: on-chain synthetic asset protocols across Ethereum (L1 and L2s), Solana, and Bitcoin L2s. Includes both pure synthetics (CDP-backed price exposure) and perpetuals platforms that produce synthetic exposure to BTC, ETH, SOL and RWAs.

Research date range: first snapshot May 2026; Ethereum, Solana and CDP refresh 2026-11-16. Hyperliquid, dYdX and sBTC were not refreshed in November (outside the Ethereum and Solana scope of that update) and still carry May 2026 data.

## Discovered protocols

Ranked by current relevance and size as of 2026-11-16 where refreshed. Prior May rank in brackets.

| Rank | Protocol | Ecosystem | Primary metric | Value | Status | Selected |
|------|----------|-----------|---------------|-------|--------|----------|
| 1 | Hyperliquid [1] | Custom L1 (USDC bridge on Arbitrum) | 30d perp volume | $176.06B (May; not refreshed) | live | yes |
| 2 | Ethena USDe | Ethereum (plus Base, Robinhood Chain, Solana) | USDe supply | $4.887B (peak $14.82B Oct 2025) | live, shrinking | yes (new) |
| 3 | Lighter | Ethereum zk-rollup, Robinhood Chain | Q3 2026 volume | $126.9B; bridge collateral $549.6M | live | yes (new) |
| 4 | Variational | Arbitrum | Q3 2026 volume | $104.0B; OI $697M | live | yes (new) |
| 5 | Sky / Maker | Ethereum | TVL | $3.756B | live | yes (new, CDP) |
| 6 | Jupiter Perps [3] | Solana | JLP TVL | $573M DefiLlama, $683M Jupiter API (May: ~$1.3B) | live, shrinking | yes |
| 7 | sBTC / Stacks [7] | Bitcoin L2 | sBTC TVL | $437M (May; not refreshed) | live | yes |
| 8 | GMX V2 [2] | Arbitrum, Avalanche, MegaETH | TVL | $187.05M (May: $203.47M) | live, shrinking | yes |
| 9 | Liquity V1 and V2 | Ethereum | TVL | $130.3M and $70.7M | live | yes (new, CDP) |
| 10 | Alchemix V3 | Ethereum, Base, Optimism | TVL | $32.75M | live | yes (new, CDP) |
| 11 | Hylo | Solana | TVL | $26.7M | live, young | yes (new, CDP-adjacent) |
| 12 | Metronome Synth | Ethereum, Base | msUSD supply | $41.95M at $0.818 | live but impaired (July 2026 shortfall) | yes (new, CDP) |
| 13 | Ostium [8] | Arbitrum | TVL | $2.26M (May: $46.8M) | exploited 2026-07-15 ($23.75M), partial relaunch | yes |
| 14 | Drift [4] | Solana | TVL (Drift Trade series) | $0.64M (pre-hack $311M on 2026-04-01) | exploited 2026-04-01 (~$285M), relaunched as Velocity 2026-09-29 | yes |
| 15 | dYdX v4 [5] | dYdX Chain (Cosmos) | Daily volume | ~$200M (May; not refreshed) | live | yes |
| 16 | Synthetix V3 [6] | Ethereum | TVL | $0 (May: $41.19M) | effectively retired June 2026; small mainnet CLOB exchange remains | yes |
| 17 | Synthetix V1/V2 | Ethereum, Optimism | Peak TVL | $2.97B | legacy, solo staking retired April 2025 | yes (new, CDP) |
| 18 | Youves | Tezos | TVL | $16.2M (peak $43.7M, 2024-03) | live | yes (new, CDP) |
| 19 | Mirror Protocol | Terra | status | n/a | dead (2022) | yes (new, historical) |
| 20 | Derive (Lyra) [9] | Derive Chain (OP Stack) | TVL | $167.1M (options OI $1.7B) | live | no (options, narrower scope; largest on-chain options venue) |
| n/a | Aster | BNB Chain centred | Q3 volume | $154.8B; TVL $767.5M | live | no (BNB centred, only partly in scope) |
| n/a | Parcl [10] | Solana | V3 TVL | $0.3M | V4 in closed alpha on its own chain | no (real estate niche, near zero) |
| n/a | PreStocks [11] | Solana | cumulative volume | $414.7M (Allium, search extract) | live | no (SPV-backed, not strictly synthetic; pattern-level mention) |

## Selection rationale

The twenty selected protocols cover the design space:

- **Debt-pool and CDP synthetics:** [[projects/synthetix-v2]], [[projects/synthetix-v3]], [[projects/sky-maker]], [[projects/liquity]], [[projects/alchemix]], [[projects/metronome-synth]], [[projects/youves]], [[projects/mirror-protocol]] (historical), [[projects/frankencoin]], [[projects/crvusd]]. Comparison in [[patterns/cdp-synthetics-landscape]].
- **Collateral-split and delta-neutral synthetic dollars:** [[projects/hylo]], [[projects/ethena-usde]]
- **Liquidity-pool perps (LP as counterparty):** [[projects/gmx-v2]], [[projects/jupiter-perps]]
- **CLOB / orderbook:** [[projects/hyperliquid]], [[projects/dydx-v4]], [[projects/lighter]], [[projects/variational]]
- **Hybrid vAMM + orderbook:** [[projects/drift]]
- **RWA-extended perps:** [[projects/ostium]]
- **BTC-backed synthetic:** [[projects/sbtc-stacks]]

This covers the full spectrum of trust models (threshold multisigs, validator sets, oracle dependencies, LP risk, decentralised orderbooks, debt pools) which is the analytical core of the brief.

## Research status

First pass done 2026-05; Ethereum, Solana and CDP update done 2026-11-16 (agent-assisted, with spot checks against DefiLlama's API). A formal fact-check pass (see `rfp/agents/fact-checker.md`) has not been run on the November changes.

- [x] [[projects/synthetix-v3]] (refreshed 2026-11)
- [x] [[projects/gmx-v2]] (refreshed 2026-11)
- [ ] [[projects/hyperliquid]] (May 2026 data; not refreshed)
- [ ] [[projects/dydx-v4]] (May 2026 data; not refreshed)
- [x] [[projects/drift]] (refreshed 2026-11)
- [x] [[projects/jupiter-perps]] (refreshed 2026-11)
- [x] [[projects/ostium]] (refreshed 2026-11)
- [ ] [[projects/sbtc-stacks]] (May 2026 data; not refreshed)
- [x] New 2026-11: [[projects/lighter]], [[projects/variational]], [[projects/ethena-usde]], [[projects/synthetix-v2]], [[projects/liquity]], [[projects/sky-maker]], [[projects/alchemix]], [[projects/metronome-synth]], [[projects/youves]], [[projects/mirror-protocol]], [[projects/hylo]]

Related: [[projects/monero-zcash-synthetics-community-history]] (ecosystem history for the sXMR RFP).

## Cross-cutting notes

- [[metrics/tvl-comparison]]
- [[metrics/volume-comparison]]
- [[patterns/oracle-pricing-model]]
- [[patterns/lp-counterparty-risk]]
- [[patterns/bridge-trust-assumptions]]
- [[patterns/synthetic-collateral-models]]
- [[patterns/cdp-synthetics-landscape]]
- [[patterns/cdp-vs-morpho-trust]]
- [[patterns/collateral-debt-units]]
- [[rfp/appendix/cdp-isolation]]
