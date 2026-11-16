---
tags: [metrics, tvl]
updated: 2026-11-16
---

# TVL Comparison

Snapshot of protocol TVL (total value locked, pool AUM or synthetic supply) across Ethereum, Solana and the CDP synthetics landscape. Refreshed 2026-11-16; the May 2026 values are kept in a separate column so the trend is visible. DefiLlama web pages returned HTTP 403 during this refresh, so DefiLlama figures were read from its public API (`api.llama.fi`), which is cited per row.

## Ethereum and L2s (perps and synthetic exposure)

| Protocol | Ecosystem | TVL, 2026-11-16 | May 2026 | Source |
|----------|-----------|-----------------|----------|--------|
| [[projects/lighter]] | Ethereum zk-rollup, Robinhood Chain | $549.6M bridge collateral (LLP $97.1M; Robinhood Chain instance $111.7M) | not in vault | [DefiLlama](https://api.llama.fi/protocol/lighter-bridge) |
| [[projects/gmx-v2]] | Arbitrum, Avalanche, MegaETH | $187.05M (Arbitrum $180.07M, Avalanche $6.74M, MegaETH $0.23M, Botanix $0) | $203.47M | [DefiLlama](https://api.llama.fi/protocol/gmx-v2-perps) |
| [[projects/variational]] | Arbitrum | [NOT FOUND] (DefiLlama has no TVL series; OI $697M) | not in vault | [DefiLlama OI](https://api.llama.fi/overview/open-interest) |
| [[projects/ostium]] | Arbitrum | $2.26M (exploited 2026-07-15) | $46.76M (DefiLlama series; old note had [DATA NEEDED]) | [DefiLlama](https://api.llama.fi/protocol/ostium) |
| [[projects/synthetix-v3]] | Ethereum (Base and Arbitrum already gone) | $0 (last non-zero $29.10M on 2026-06-25); parent Synthetix mainnet exchange $0.887M | $41.19M | [DefiLlama V3](https://api.llama.fi/protocol/synthetix-v3), [DefiLlama parent](https://api.llama.fi/protocol/synthetix) |

## Solana

| Protocol | TVL, 2026-11-16 | May 2026 | Source |
|----------|-----------------|----------|--------|
| [[projects/jupiter-perps]] | $572.9M (DefiLlama; plus $119.3M listed as borrowed); JLP pool AUM $683.5M on Jupiter's own API (2026-11-14) | ~$1.3B (Eco support; DefiLlama showed $677M on 2026-05-19) | [DefiLlama](https://api.llama.fi/protocol/jupiter-perpetual-exchange), [Jupiter](https://perps-api.jup.ag/v1/jlp-info) |
| GMTrade (GMX on Solana, separate deployment) | $22.61M | $41.1M | [DefiLlama](https://api.llama.fi/protocol/gmx-solana) |
| [[projects/hylo]] (collateral-split synthetic) | $26.72M | not in vault | [DefiLlama](https://api.llama.fi/protocols) |
| Pacifica | $24.2M (bridge TVL) | not in vault | [DefiLlama](https://api.llama.fi/protocol/pacifica) |
| [[projects/drift]] (Drift Trade series; relaunched as Velocity 2026-09-29) | $0.64M (Velocity TVL [NOT FOUND]) | "$150M-$400M" (not reproducible; DefiLlama Drift Trade was $311M on 2026-04-01 and $6.25M on 2026-05-19) | [DefiLlama](https://api.llama.fi/protocol/drift-trade) |

## CDP, debt-pool and synthetic stablecoin supply

| Protocol | Design | TVL or supply, 2026-11-16 | Source |
|----------|--------|---------------------------|--------|
| [[projects/sky-maker]] | Per-position CDP | $3.756B TVL (peak $19.82B, 2021-12-01) | [DefiLlama](https://api.llama.fi/protocol/sky-lending) |
| [[projects/ethena-usde]] | Delta-neutral | USDe supply $4.887B (peak $14.82B, 2025-10-04; $4.35B on 2026-05-19); USDtb $317.9M | [DefiLlama stablecoins](https://stablecoins.llama.fi/stablecoin/146) |
| [[projects/liquity]] | Per-position CDP | V1 $130.33M; V2 $70.73M (peak $180.03M, 2025-10-07) | [DefiLlama V1](https://api.llama.fi/protocol/liquity-v1), [V2](https://api.llama.fi/protocol/liquity-v2) |
| [[projects/alchemix]] | Self-repaying CDP | V3 $32.75M; V2 $71K after migration | [DefiLlama V3](https://api.llama.fi/protocol/alchemix-v3) |
| [[projects/metronome-synth]] | Per-position CDP, multi-synth | $7.89M TVL; msUSD supply $41.95M at $0.818 (impaired) | [DefiLlama](https://api.llama.fi/protocols) |
| [[projects/synthetix-v2]] | Debt pool (legacy) | peak $2.97B; effectively zero after solo staking ended April 2025 | [DefiLlama](https://api.llama.fi/protocol/synthetix-v1+v2) |

The full 50-row comparison is in [[patterns/cdp-synthetics-landscape]].

## Not refreshed (outside the Ethereum and Solana scope of this update)

| Protocol | Ecosystem | TVL (May 2026) | Source |
|----------|-----------|----------------|--------|
| [[projects/hyperliquid]] | Custom L1 | $5.111B | [DefiLlama](https://defillama.com/protocol/hyperliquid) |
| [[projects/sbtc-stacks]] | Bitcoin L2 (Stacks) | $437M (end Q1 2026, peak $545M) | [Stacks blog](https://www.stacks.co/blog/q1-2026-snapshot) |
| [[projects/dydx-v4]] | Cosmos appchain | [DATA NEEDED; OI ~$175M] | [Coinbureau](https://coinbureau.com/review/dydx) |

## Observations

- **Two of the May venues largely collapsed.** Ostium fell from $46.8M to $2.3M after the July exploit, and Drift from $311M (pre-hack, 2026-04-01) to under $1M. Synthetix V3 went to zero on 2026-06-26.
- **Solana perp TVL is now about $0.6B to $0.7B, not the $1.7B to $1.9B quoted in May.** That May figure rested on a Jupiter value roughly double DefiLlama's and a Drift range that the DefiLlama series does not support.
- **GMX V2 is the only May Ethereum-side perp venue holding its TVL (down 8%).** The growth is in orderbook venues: Lighter's bridge collateral is $550M, roughly three times GMX.
- **Delta-neutral and CDP stablecoin supply dwarfs perp TVL.** Sky holds $3.8B and USDe $4.9B, so synthetic dollars remain the largest synthetic category by value locked, even after USDe fell two-thirds from its peak.

## Methodology notes

- "TVL" definitions differ: Lighter is bridge collateral, Jupiter is the JLP pool, GMX is GM/GLV pool value, stablecoins are circulating supply. Open interest (OI) is a complementary metric: see [[metrics/volume-comparison]].
- Jupiter's two figures differ by about $110M (AUM on Jupiter's API versus DefiLlama TVL), probably a scope difference; both are shown.
- Numbers are point-in-time; source endpoints give live values.
