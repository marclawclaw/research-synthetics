---
tags: [project, ethereum, optimism, base, cdp, synthetics, impaired]
ecosystem: Ethereum, Optimism, Base (msUSD also bridged to Plasma)
category: Synthetics/Multi-asset CDP
status: live but impaired (swaps paused and about $15.7M of synths disclosed as unbacked on 2026-07-30; msUSD trades well below $1)
status_date: 2026-11-16
website: https://metronome.io
docs: https://docs.metronome.io
launched: 2022-12 (first DefiLlama TVL datapoint 2022-12-22)
---

# Metronome Synth (msUSD, msETH, msBTC, msOP)

Status: live but impaired. Metronome Synth is a multi-collateral, multi-synthetic CDP protocol: users deposit crypto assets and mint msUSD, msETH, msBTC (and msOP on Optimism) that track a USD, ETH, BTC or OP reference, and swap between synths on an oracle-priced Synth Marketplace. On 2026-07-30 MetronomeDAO disclosed that the swap module had created roughly 6,367 msETH and 4.57 million msUSD without adequate collateral. On 2026-11-16 DefiLlama prices msUSD at $0.818.

## Adoption metrics

| Metric | Value | Date | Source |
|--------|-------|------|--------|
| TVL (Ethereum $5.79M, Optimism $1.59M, Base $0.51M) | $7.89M | 2026-11-16 | [DefiLlama Metronome Synth](https://api.llama.fi/protocol/metronome-synth) |
| TVL peak | $33.04M | 2025-10-07 | [DefiLlama Metronome Synth](https://api.llama.fi/protocol/metronome-synth) |
| msUSD circulating (Ethereum, Base, OP Mainnet, Plasma) | $41.95M | 2026-11-16 | [DefiLlama stablecoins](https://stablecoins.llama.fi/stablecoins) |
| msUSD price | $0.818 | 2026-11-16 | [DefiLlama stablecoins](https://stablecoins.llama.fi/stablecoins) |
| Fees: 30d / 1y / all time (see caveat) | $46,261 / $8.41M / $19.94M | 2026-11-16 | [DefiLlama fees overview](https://api.llama.fi/overview/fees) |
| Peak monthly fees | $2.50M (2026-02); falling to $0.14M (2026-08), $0.085M (2026-09), $0.069M (2026-10) | monthly sums of daily series | [DefiLlama fees: Metronome Synth](https://api.llama.fi/summary/fees/metronome-synth) |
| Disclosed shortfall | about 6,367 msETH (about 31% of msETH supply) and 4.57M msUSD (about 16% of msUSD supply), about $15.7M to $16M | 2026-07-30 | [Crypto Briefing](https://cryptobriefing.com/metronome-dao-16m-oracle-shortfall/); [The Defiant](https://thedefiant.io/news/defi/metronome-discloses-usd15-7-million-synth-shortfall-blames-oracle-lag-in-swap-module) (headline and summary only; article body blocked) |

Cross-check problem: msUSD supply ($41.95M) is about five times the protocol TVL ($7.89M). With ordinary overcollateralisation the supply should be below TVL. The gap is consistent with a large unbacked float (the shortfall disclosure) and bridged or swap-created supply, but no source retrieved reconciles the figures: [NOT FOUND]. The 16% unbacked share of msUSD supply stated in the disclosure implies roughly $28.6M of supply at 2026-07-30, so supply apparently grew afterwards.

Fee caveat: DefiLlama defines Metronome fees as synth interest and swap fees minted to the treasury, plus AMO harvest profits, MetBasis LP fees, LP rewards (AERO, xAERO, VELO, KITE, CRV, OETH, FXN) and Uniswap V3 and Aerodrome LP fees on treasury-owned positions. It is therefore not a borrower cost; much of it is treasury strategy income. The split between borrower interest and swap fees is [NOT FOUND].

## How it works

### User perspective

A user deposits eligible collateral, receives a collateral-factor-limited borrowing power, and mints a synth up to that limit. Synth debt tracks the reference asset price, so debt in USD terms rises if the user mints msETH or msBTC and the reference asset rallies. Synths can be swapped on the Synth Marketplace. A "Smart Farming" product loops collateral into yield and auto-repays.

### Protocol perspective

- **Collateral and ratios (docs, undated, last updated about nine months before 2026-11-16):** collateral factors USDC 85%, DAI 85%, FRAX 83%, vaUSDC 82%, vaFRAX 80%, ETH 83%, vaETH 80%, sfrxETH 80%, vastETH 78%, varETH 75%, vacbETH 67%, WBTC 80%. Equivalent minimum collateral ratios are the reciprocal (for example USDC 85% gives about 118%; ETH 83% gives about 120%). Ratios come from a five-factor risk score (issuance, liquidity, Lindy, peg volatility, rehypothecation). Source: [Liquidations and collateral factors](https://docs.metronome.io/metronome-synth/metronome-synth-protocol/liquidations-and-collateral-factors.md).
- **Deposit caps (docs):** USDC 10M, DAI 10M, FRAX 9M, ETH 4,286, WBTC 311 and others; the msUSD mintage cap is the sum of stablecoin deposit caps times their collateral factors. Source: [Mintage and deposit caps](https://docs.metronome.io/metronome-synth/metronome-synth-protocol/metronome-synth-mintage-and-deposit-caps.md). Whether these caps are current after the 2026 incident: [NOT FOUND].
- **Liquidation:** triggered when collateral value times collateral factor falls below synth debt value; partial or full. Liquidation fee is an extra 18%, of which 10% goes to the liquidator and 8% to Metronome. Source: [Liquidation risk page](https://docs.metronome.io/metronome-synth/risks/liquidation) and [revenue model](https://docs.metronome.io/metronome-synth/metronome-revenue-model).
- **Peg and oracle:** msUSD is valued at exactly $1.00 inside the protocol; msETH and msBTC use Chainlink prices with a deviation threshold. No direct redemption of msUSD for $1 of collateral is described in the docs retrieved; the peg appears to rely on the marketplace and treasury liquidity (inference). See [[patterns/oracle-pricing-model]].

## Key behaviours

- [[patterns/oracle-pricing-model]] : the 2026 shortfall is an oracle latency failure at the swap layer, not at liquidation.
- [[patterns/synthetic-collateral-models]] : multi-collateral CDP with per-synth debt tokens, closest to the per-position CDP family.
- [[patterns/cdp-synthetics-landscape]] : see for comparison with debt-pool Synthetix and redemption-floor CDPs.

## Architecture decisions

- **Zero-slippage oracle swaps between synths:** the same design as Synthetix exchange. The swap module mints and burns synths at oracle prices, so by inference any oracle lag can become an unbacked-supply source (this is the failure MetronomeDAO disclosed).
- **Productive collateral:** supported collateral includes yield-bearing vault tokens (the "va" prefixed assets such as vaETH and vaUSDC) so collateral earns yield; see [Productive collateral](https://docs.metronome.io/metronome-synth/metronome-synth-protocol/productive-collateral.md) (page listed in the docs index, not retrieved).
- **Treasury-owned liquidity (AMOs, POL):** supports peg and generates fee income (see fee caveat).
- **Governance:** MET token and a DAO; esMET revenue share exists per docs (not retrieved in detail).

## Differentiators

- Multi-synth set (USD, ETH, BTC, OP) with one set of collateral.
- Productive collateral and treasury-managed liquidity.
- Relatively low collateral ratios (about 118 to 150% depending on asset) compared with Synthetix V2.

## Limitations and criticisms

- **Disclosed shortfall (2026-07-30):** about 6,367 msETH and 4.57M msUSD unbacked, attributed to Chainlink ETH/USD oracle latency on Base (median latency breach of about 54 seconds and up to 5 minutes 37 seconds in extreme cases per Crypto Briefing); treasury says it staged over $34M in defensive looped positions and injected $6.5M of protocol-owned liquidity; swaps are paused pending architectural upgrades (Crypto Briefing; the primary Metronome post-mortem was not retrieved). One search summary adds that if msETH and msUSD decline about 30% the treasury positions would generate enough profit to buy back and burn the unbacked tokens; not verified.
- **Price:** msUSD at $0.818 on 2026-11-16 (DefiLlama), about 18% below peg.
- **No audits listed:** DefiLlama shows zero audit links for Metronome Synth ([API](https://api.llama.fi/protocol/metronome-synth)); audits elsewhere: [NOT FOUND].
- Revenue is dominated by treasury strategy income, which fell sharply after the incident.

### Fees and borrower cost

| Item | Value | Date | Source |
|------|-------|------|--------|
| Synth balance fee (borrower interest) | Annualised, charged per second, added to the user's outstanding synth debt; set per synth by risk profile; numeric rates are shown only in the app dashboard: [NOT FOUND] | docs accessed 2026-11-16 | [Revenue model](https://docs.metronome.io/metronome-synth/metronome-revenue-model) |
| Marketplace swap fee | 0.45% (0.55% on mainnet) in the docs page; a search snippet of an earlier page version quoted 0.25% (conflict, below) | docs accessed 2026-11-16 | [Revenue model](https://docs.metronome.io/metronome-synth/metronome-revenue-model) |
| Liquidation fee | 18% additional premium: 10% to liquidator, 8% to Metronome | docs accessed 2026-11-16 | [Revenue model](https://docs.metronome.io/metronome-synth/metronome-revenue-model) |
| Issuance, burn and deposit fee | A FeeProvider contract manages deposit, withdrawal, issuance and liquidation fees, but no numeric values are published in the docs retrieved: [NOT FOUND] | 2026-11-16 | [Contract overview: Fees](https://docs.metronome.io/contract-overview/fees.md) |
| Where revenue goes | All revenue to the Metronome Treasury | 2026-11-16 | [Revenue model](https://docs.metronome.io/metronome-synth/metronome-revenue-model) |
| Fee run rate | $8.41M in the year to 2026-11-16, but only $46,261 in the last 30 days | 2026-11-16 | [DefiLlama fees overview](https://api.llama.fi/overview/fees) |
| Post-incident fee changes | "Higher fees on all synthetic pairs" planned (Crypto Briefing); search snippet claimed 0.45% on Base and 0.55% on Ethereum | 2026-07-31 | [Crypto Briefing](https://cryptobriefing.com/metronome-dao-16m-oracle-shortfall/) |

Conflict: swap fee 0.25% (search snippet of a version of the revenue-model page) versus 0.45% and 0.55% (the page as fetched on 2026-11-16, last edited about nine months earlier). The later figure is used.

## Sources

- [DefiLlama Metronome Synth API](https://api.llama.fi/protocol/metronome-synth) : accessed 2026-11-16
- [DefiLlama fees: Metronome Synth](https://api.llama.fi/summary/fees/metronome-synth) : accessed 2026-11-16
- [DefiLlama stablecoins list](https://stablecoins.llama.fi/stablecoins) : accessed 2026-11-16
- [Metronome docs: revenue model](https://docs.metronome.io/metronome-synth/metronome-revenue-model) : accessed 2026-11-16
- [Metronome docs: liquidations and collateral factors](https://docs.metronome.io/metronome-synth/metronome-synth-protocol/liquidations-and-collateral-factors.md) : accessed 2026-11-16
- [Metronome docs: mintage and deposit caps](https://docs.metronome.io/metronome-synth/metronome-synth-protocol/metronome-synth-mintage-and-deposit-caps.md) : accessed 2026-11-16
- [Metronome docs: oracle disruptions](https://docs.metronome.io/metronome-synth/risks/oracle-disruptions.md) : accessed 2026-11-16
- [Metronome docs: liquidation risk](https://docs.metronome.io/metronome-synth/risks/liquidation) : accessed 2026-11-16
- [Crypto Briefing: MetronomeDAO 16M shortfall](https://cryptobriefing.com/metronome-dao-16m-oracle-shortfall/) : accessed 2026-11-16
- [The Defiant: Metronome discloses $15.7M synth shortfall](https://thedefiant.io/news/defi/metronome-discloses-usd15-7-million-synth-shortfall-blames-oracle-lag-in-swap-module) : accessed 2026-11-16 (headline and summary only)
