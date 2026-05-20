---
tags: [metrics, volume, perpetuals]
updated: 2026-05-19
---

# Volume Comparison

Trading volume across the protocols, where "volume" generally means notional perpetuals volume.

| Protocol | 24h volume | 30d volume | Cumulative volume | Date | Source |
|----------|-----------|------------|------------------|------|--------|
| [[projects/hyperliquid]] | $8.516B | $176.06B | [DATA NEEDED for all-time] | 2026-05 | [DefiLlama](https://defillama.com/protocol/hyperliquid) |
| [[projects/jupiter-perps]] | [DATA NEEDED for current] | [DATA NEEDED for current] | $263B (2025 full year) | 2025 | [Yellow.com](https://yellow.com/research/jupiter-solana-dex-aggregator-market-cap-2026) |
| [[projects/gmx-v2]] | $106.46M | [DATA NEEDED] | $345B+ (since v1 launch, all chains) | 2026-05 | [DefiLlama](https://defillama.com/protocol/gmx-v2-perps), [Coinbureau](https://coinbureau.com/review/gmx-review/) |
| [[projects/ostium]] | ~$200M | ~$6B (monthly) | [DATA NEEDED] | 2026-04 | [MEXC](https://blog.mexc.com/ostium-airdrop-2026-how-to-farm-points-on-the-rwa-perp-dex-bringing-gold-oil-and-stocks-on-chain/) |
| [[projects/dydx-v4]] | ~$200M average, peak $500M | ~$6B (extrapolated avg) | [DATA NEEDED for v4 since launch] | 2026-04 | [Coinbureau](https://coinbureau.com/review/dydx) |
| [[projects/synthetix-v3]] | $0 (winding down non-Base) | [DATA NEEDED] | $6.962B (V3 cumulative), $62.72B (all versions) | 2026-05 | [DefiLlama](https://defillama.com/protocol/synthetix-v3), [CoinMarketCap](https://coinmarketcap.com/cmc-ai/synthetix/price-prediction/) |
| [[projects/drift]] | [DATA NEEDED for current] | [DATA NEEDED] | [DATA NEEDED] | 2026 | (open interest ~$400M serves as proxy) |
| [[projects/sbtc-stacks]] | N/A (synthetic asset, not a venue) | N/A | N/A | 2026 | - |

## Observations

- **Hyperliquid is in a class of its own:** 30-day volume of $176B is larger than every other venue combined, and approaches centralised-exchange territory. It flipped Coinbase's 2025 notional volume per The Market Periodical.
- **Jupiter Perps' 2025 volume of $263B** is roughly proportional to Hyperliquid's run rate, but most of it predates 2026; the relative growth gap has widened.
- **GMX cumulative volume of $345B+** spans roughly five years (2021-2026); its 24h pace ($106M) implies a long-tail revenue stream rather than rapid growth.
- **dYdX and Ostium are similar in monthly throughput** (~$6B), despite Ostium being two years younger and focused on RWA pairs.
- **Synthetix V3's $0 24h on non-Base chains** reflects the active wind-down of Arbitrum perps. The cumulative figure ($6.96B V3 / $62.72B all-time) shows the protocol has been a significant historical contributor even as its current share is small.

## Methodology notes

- Daily volume is highly variable; favour 7d or 30d averages where possible.
- "Notional volume" varies by venue: Hyperliquid reports gross taker volume; GMX reports executed position size in USD; dYdX reports matched trade notional. Direct cross-comparison is approximate.
- For LP-pool venues (GMX, Jupiter), the headline volume figure includes both opening and closing of positions, so $X in "volume" implies roughly $X/2 in unique trader exposure.
