---
tags: [metrics, tvl]
updated: 2026-05-19
---

# TVL Comparison

Snapshot of protocol TVL (total value locked or pool equivalent) across the synthetic asset and perpetuals landscape, accessed May 2026.

| Protocol | Ecosystem | TVL | Date | Source |
|----------|-----------|-----|------|--------|
| [[projects/hyperliquid]] | Custom L1 | $5.111B | 2026-05 | [DefiLlama](https://defillama.com/protocol/hyperliquid) |
| [[projects/jupiter-perps]] | Solana | ~$1.3B (JLP pool) | 2026-04 | [Eco support](https://eco.com/support/en/articles/15083164-jupiter-perps-fees-leverage-how-jlp-works) |
| [[projects/sbtc-stacks]] | Bitcoin L2 (Stacks) | $437M (end Q1 2026, peak $545M) | 2026 Q1 | [Stacks blog](https://www.stacks.co/blog/q1-2026-snapshot) |
| [[projects/drift]] | Solana | $150M-$400M | 2026 | [Eco support](https://eco.com/support/en/articles/15083167-drift-protocol-solana-perpetuals-dex-deep-dive) |
| [[projects/gmx-v2]] | Arbitrum + Avalanche + MegaETH + Botanix | $203.47M | 2026-05 | [DefiLlama](https://defillama.com/protocol/gmx-v2-perps) |
| [[projects/dydx-v4]] | dYdX Chain (Cosmos) | [DATA NEEDED for current TVL; OI ~$175M] | 2026 | [Coinbureau](https://coinbureau.com/review/dydx) |
| [[projects/synthetix-v3]] | Ethereum + Base | $41.19M (V3) | 2026-05 | [DefiLlama](https://defillama.com/protocol/synthetix-v3) |
| [[projects/ostium]] | Arbitrum | [DATA NEEDED for pure TVL; OI $213M, OLP vault available] | 2026-04 | [MEXC](https://blog.mexc.com/ostium-airdrop-2026-how-to-farm-points-on-the-rwa-perp-dex-bringing-gold-oil-and-stocks-on-chain/) |

## Observations

- Hyperliquid's $5.1B TVL is an order of magnitude larger than any other on-chain perp venue, reflecting its dominance.
- Solana represents roughly $1.7B-$1.9B in combined synthetic perp TVL (Jupiter + Drift), which is competitive with all of Arbitrum's perp venues combined.
- Synthetix V3 TVL of $41M is a fraction of what it was during 2021 peaks (~$3B+), reflecting the structural shift away from CDP-style synthetics toward perp-pool synthetics.
- sBTC TVL of $437M makes it materially larger than the Stacks-native DeFi protocols built on top of it (Zest at $76M, Granite at $26M), implying most sBTC sits idle or in low-utility venues.

## Methodology notes

- "TVL" definitions differ: Hyperliquid TVL includes USDC bridge balance and HLP. Jupiter TVL is the JLP pool value. GMX TVL is GM/GLV pool value.
- Open interest (OI) is a complementary metric for perp venues: see [[metrics/volume-comparison]].
- Numbers are point-in-time and fluctuate daily. Source pages are linked for live values.
