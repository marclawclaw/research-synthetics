---
tags: [project, ethereum, base, synthetics, perpetuals]
ecosystem: Ethereum, Base, Arbitrum
category: Synthetics/Perpetuals
website: https://synthetix.io
docs: https://docs.synthetix.io
launched: 2018 (v1), 2023 (v3)
---

# Synthetix V3

Synthetix is the original on-chain synthetic asset protocol. V1/V2 ran a debt-pool model where SNX stakers minted Synths (sUSD, sBTC, sETH) collateralised by SNX. V3, deployed across Ethereum, Base and Arbitrum, generalised the architecture into a "liquidity layer" that supports perpetuals, spot synths and arbitrary market types, with pluggable oracles and pool-specific collateral.

## Adoption metrics

| Metric | Value | Date | Source |
|--------|-------|------|--------|
| TVL (V3) | $41.19M | May 2026 | [DefiLlama Synthetix V3](https://defillama.com/protocol/synthetix-v3) |
| Cumulative perp volume (V3) | $6.962B | May 2026 | [DefiLlama Synthetix V3](https://defillama.com/protocol/synthetix-v3) |
| 24h perp volume | $0 (winding down on non-Base venues) | May 2026 | [DefiLlama Synthetix V3](https://defillama.com/protocol/synthetix-v3) |
| Daily average trading volume (CMC summary) | ~$42M | 2026 | [CoinMarketCap Synthetix](https://coinmarketcap.com/cmc-ai/synthetix/price-prediction/) |
| Cumulative perp volume (CMC summary, all versions) | $62.72B | 2026 | [CoinMarketCap Synthetix](https://coinmarketcap.com/cmc-ai/synthetix/price-prediction/) |
| Active deployment chains | Ethereum, Base, Arbitrum (winding down) | 2026 | [The Defiant](https://thedefiant.io/news/defi/synthetix-to-pivot-v3-perps-from-arbitrum-to-base) |

## How it works

### User perspective

A trader deposits accepted collateral (USDC, sUSD, ETH derivatives depending on market) into a Synthetix V3 perp account, then opens a leveraged position on BTC, ETH or supported altcoins. Pricing is fetched from a configured oracle aggregator (Chainlink Data Streams plus Pyth pull oracles) at execution time. Settlement is two-step to mitigate front-running: commit, then settle after an oracle update window.

LPs deposit collateral into specific debt pools, which in turn back specific markets. LP returns come from a share of trading fees and from absorbing trader PnL.

### Protocol perspective

V3 separates the protocol into three layers:

- **Core (Liquidity Layer):** account registry, collateral management, pool accounting, market registry.
- **Markets:** independent modules (perps, spot synths, options via Derive) that pull liquidity credit from one or more pools.
- **Oracles:** Oracle Manager allows market creators to assemble price feeds (e.g. min of Chainlink, Pyth and a Uniswap TWAP).

The differentiated debt pools let stakers pick risk exposure (which markets back which pool) instead of socialising all market risk across one global SNX pool.

## Key behaviours

- [[patterns/synthetic-collateral-models]] : V3 moved from SNX-only collateral to multi-collateral pools.
- [[patterns/oracle-pricing-model]] : oracle-aggregated mark price with on-chain commit-reveal settlement.
- [[patterns/lp-counterparty-risk]] : LPs are the counterparty; debt grows when traders win.

## Architecture decisions

- **Liquidity Layer abstraction:** V3 is a primitive for any synthetic market, not just Synths. Markets plug in.
- **Move from Optimism back to Ethereum and Base:** December 2025 strategic shift to mainnet for deeper liquidity and composability; Base named as growth target. Arbitrum perps are being wound down.
- **420 Pool simplified staking:** introduced 2025/26, lets users stake SNX without managing C-ratio or debt manually.
- **Fee buyback:** May 2026 governance committed all trading fee revenue to SNX and sUSD buybacks.
- **SnaXchain governance L2:** Optimism Superchain rollup launched late Q3 2024 as the home of Synthetix governance.

## Differentiators

- Pluggable per-market oracle aggregation (Chainlink + Pyth + TWAP min).
- Debt pool model: LP credit, not orderbook.
- Native support for spot synths (sUSD, sBTC, sETH) alongside perps.
- 7+ years of production iteration on synthetic asset economics.

## Limitations and criticisms

- TVL has compressed sharply: $41M V3 TVL in May 2026 is a fraction of historical Synthetix peak (~$3B+ across V1/V2 during 2021).
- Debt pool model is conceptually complex: stakers' debt changes with aggregate trader PnL, which has historically deterred passive LPs.
- sUSD has experienced multiple depeg episodes; stability of the spot synth has lagged competing stablecoins.
- Strategic flux (multiple chain pivots Optimism → Arbitrum → Base) creates user friction and fragmented liquidity.

## Sources

- [DefiLlama Synthetix V3 protocol page](https://defillama.com/protocol/synthetix-v3) : accessed 2026-05-19
- [DefiLlama Synthetix v1+v2 page](https://defillama.com/protocol/synthetix-v1+v2) : accessed 2026-05-19
- [Synthetix V3 blog overview](https://blog.synthetix.io/what-is-synthetix-v3/) : accessed 2026-05-19
- [Synthetix V3 oracle docs](https://docs.synthetix.io/synthetix-protocol/the-synthetix-protocol/oracles) : accessed 2026-05-19
- [The Defiant : Synthetix pivots to Base](https://thedefiant.io/news/defi/synthetix-to-pivot-v3-perps-from-arbitrum-to-base) : accessed 2026-05-19
- [CoinMarketCap CMC AI : Synthetix 2026 updates](https://coinmarketcap.com/cmc-ai/synthetix/latest-updates/) : accessed 2026-05-19
- [Zipmex : Synthetix 2026 guide](https://zipmex.com/blog/synthetix-snx-guide-2026/) : accessed 2026-05-19
