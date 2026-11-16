---
tags: [project, solana, stablecoin, synthetics, dual-token, leveraged-token]
ecosystem: Solana
category: Synthetics/Dual-token stablecoin (collateral-split, no per-user liquidation)
status: live (launched 2025-07-29; V2 audited July 2026)
status_date: 2026-11-16
website: https://hylo.so
docs: https://docs.hylo.so
launched: 2025-07-29 (DefiLlama listing date; not independently re-checked)
---

# Hylo (Solana: hyUSD and xASSET leverage tokens)

Status: live. Hylo is the main live Solana example of a collateral-split ("dual-token") stablecoin rather than a per-user CDP. Each collateral pool (SOL liquid staking tokens, BTC, HYPE, plus a USDC pool) is split into a leveraged token (xSOL, xBTC, xHYPE) that absorbs all price movement and a virtual dollar unit (vUSD) worth exactly $1. Users see only the stablecoin hyUSD, the xASSET tokens and the collateral. There is no individual debt position, so there is no per-user liquidation. It sits between the CDP family and the delta-neutral family discussed in [[patterns/cdp-synthetics-landscape]]: the stable side is hedged by an in-protocol counterparty (xASSET holders) instead of by a perp short (compare [[projects/ethena-usde]]) or a borrower's own collateral (compare [[projects/liquity]]).

## Adoption metrics

| Metric | Value | Date | Source |
|--------|-------|------|--------|
| TVL (hylo-protocol) | $26.72M | 2026-11-16 | [DefiLlama protocols list](https://api.llama.fi/protocols) |
| TVL peak | $69.47M | 2025-11-03 | [DefiLlama Hylo API](https://api.llama.fi/protocol/hylo-protocol) (as read by the research agent; peak not independently re-computed) |
| hyUSD circulating | $20.43M | 2026-11-16 | [DefiLlama stablecoins](https://stablecoins.llama.fi/stablecoins) |
| hyUSD peak | $39.58M | 2025-11-04 | [DefiLlama stablecoin 302](https://stablecoins.llama.fi/stablecoin/302) (as read by the research agent) |
| Fees: 30d / 1y / all time | $333,585 / $3.49M / $5.34M | 2026-11-16 | [DefiLlama fees overview](https://api.llama.fi/overview/fees) |
| Revenue: 30d / 1y / all time | $177,787 / $1.76M / $2.96M | 2026-11-16 | [DefiLlama revenue overview](https://api.llama.fi/overview/fees?dataType=dailyRevenue) |
| Sister product Hylo LSTs, fees 30d | $78,304 | 2026-11-16 | [DefiLlama fees overview](https://api.llama.fi/overview/fees) |

## How it works

### User perspective

Mint hyUSD by depositing collateral (or USDC), or mint a leveraged xASSET by depositing the same collateral, at prices derived from the pool equation. Redeem back at the current rates. hyUSD can be staked in the Earn Pool as eHYUSD, which accrues the protocol's harvested yield (staking rewards plus borrow rates). Holding xSOL is leveraged long SOL exposure with no liquidation: in the Neutral zone the LST staking yield pays for the leverage, so it is effectively free; above 165% collateral ratio holders pay the excess.

### Protocol perspective

- **Core equation:** for each asset pool, TVL = vUSD supply times $1 plus xASSET supply times xASSET price. The xASSET price is computed so that xASSET absorbs all gains and losses. hyUSD is backed 1:1 by the sum of the vUSDs while pool collateral ratios stay above 100%. Source: [Core mechanism](https://docs.hylo.so/protocol-overview/core-mechanism.md).
- **Pools and ratios:** each pool targets about 150% collateral ratio, with rebalance zones: at or above 175% Buy Zone 2; 165 to 175% Buy Zone 1; 135 to 165% Neutral; 120 to 135% Sell Zone 1; 100 to 120% Sell Zone 2; below 100% Destabilised. hyUSD mints that would push a pool below 150% are blocked. Source: [Risk management](https://docs.hylo.so/protocol-overview/risk-management.md).
- **Loss absorption:** below 100%, first-loss capital in the Earn Pool burns hyUSD to retire unbacked supply and restore 100% backing (same page).
- **Stability mechanisms in order:** dynamic collateral routing through fee curves, collateral rebalancing, xASSET anti-destabilisation fees, the mint block, then Earn Pool loss absorption (same page).
- **Peg:** mint and redeem at $1 against collateral; the USDC pool is 1:1 at flat fees. Oracle: Pyth for SOL, BTC, HYPE; LSTs priced by Sanctum redemption value (research agent reading of [dynamic collateral routing](https://docs.hylo.so/protocol-overview/dynamic-collateral-routing.md) and [stablecoin guide](https://docs.hylo.so/product-guide/stablecoin.md); not re-fetched). See [[patterns/oracle-pricing-model]].

## Key behaviours

- [[patterns/synthetic-collateral-models]] : a collateral-split model that does not fit the five models listed there; closest to a CDP whose borrower is the pooled xASSET holder.
- [[patterns/cdp-synthetics-landscape]] : classed as "dual-token" in the landscape taxonomy.
- [[patterns/oracle-pricing-model]] : Pyth pricing with LST redemption-value adjustment.

## Architecture decisions

- **Multi-asset pools with isolated ratios**, so stress in one collateral is worked out locally while hyUSD stays backed by the aggregate.
- **No liquidations, no debt:** protection comes from fee curves that pay the xASSET side to restore the ratio.
- **LST yield funds the leverage:** the SOL pool charges no interest in the Neutral zone.
- **Earn Pool as the first-loss tranche** shared by all pools.

## Differentiators

- Solana-native stable asset with multiple volatile collateral types and a leveraged token that cannot be liquidated.
- No oracle-triggered per-user liquidation event, which removes one class of liquidation-cascade risk (but not oracle-mispricing risk).
- Three audits across versions (OtterSec February 2025 for V1.1, Accretion December 2025 for V1.2 and July 2026 for V2), per [Hylo audits](https://docs.hylo.so/security/audits) (read by the research agent).

## Limitations and criticisms

- TVL fell from a peak of $69.47M (2025-11-03) to $26.72M; hyUSD circulating roughly halved from its $39.58M peak.
- The stablecoin is only as strong as the aggregate collateral ratio. Below 100% the loss is socialised to the Earn Pool, not to a liquidated borrower.
- Leveraged-token holders face a path-dependent decay and borrow-rate drag above 165% collateral ratio.
- Docs describe the model as "delta-neutral"; this is delta-neutral at the pool level through the split, not a hedge against an external venue. The term should not be confused with Ethena-style perp hedging.
- No incidents are recorded in DefiLlama's hacks database (accessed 2026-11-16). Short operating history (about 16 months).

### Fees and borrower cost

| Item | Value | Date | Source |
|------|-------|------|--------|
| hyUSD mint with USDC | 0% | docs, 2026-11-16 | [Multi-asset architecture](https://docs.hylo.so/protocol-overview/multi-asset-architecture.md) |
| hyUSD redeem to USDC | 0.2% | docs, 2026-11-16 | [Multi-asset architecture](https://docs.hylo.so/protocol-overview/multi-asset-architecture.md) |
| Volatile-pool mint and redeem fees | Piecewise fee curves by collateral ratio; exact numbers [NOT FOUND] | 2026-11-16 | [Dynamic collateral routing](https://docs.hylo.so/protocol-overview/dynamic-collateral-routing.md) |
| Borrow rate, SOL LST pool | None in the Neutral zone; harvest multiplier rises above 165% collateral ratio and the excess is taken from xSOL equity | 2026-11-16 | [Multi-asset architecture](https://docs.hylo.so/protocol-overview/multi-asset-architecture.md) |
| Borrow rate, BTC and HYPE pools | Configurable base rate on xAsset value, multiplier 1x between 135 and 165%, rising to a ceiling of 2x the base rate at 175% or higher; paused below 135%. Base rate number: [NOT FOUND] | 2026-11-16 | [Multi-asset architecture](https://docs.hylo.so/protocol-overview/multi-asset-architecture.md) |
| Fee destination | Harvested yield (staking rewards and borrow rates) compounds to eHYUSD stakers in the Earn Pool; DefiLlama revenue is $177.8k against $333.6k of fees over 30 days (about 53%, derived); where the remaining 47% goes is [NOT FOUND] | 2026-11-16 | [DefiLlama fees overview](https://api.llama.fi/overview/fees); [Stablecoin guide](https://docs.hylo.so/product-guide/stablecoin.md) |

## Sources

- [DefiLlama protocols list](https://api.llama.fi/protocols) : accessed 2026-11-16
- [DefiLlama Hylo API](https://api.llama.fi/protocol/hylo-protocol) : accessed 2026-11-16 (via research agent)
- [DefiLlama stablecoins list](https://stablecoins.llama.fi/stablecoins) : accessed 2026-11-16
- [DefiLlama fees overview](https://api.llama.fi/overview/fees) : accessed 2026-11-16
- [Hylo docs: core mechanism](https://docs.hylo.so/protocol-overview/core-mechanism.md) : accessed 2026-11-16
- [Hylo docs: risk management](https://docs.hylo.so/protocol-overview/risk-management.md) : accessed 2026-11-16
- [Hylo docs: multi-asset architecture](https://docs.hylo.so/protocol-overview/multi-asset-architecture.md) : accessed 2026-11-16
- [Hylo docs: stablecoin guide](https://docs.hylo.so/product-guide/stablecoin.md) : accessed 2026-11-16
- [Hylo docs: audits](https://docs.hylo.so/security/audits) : accessed 2026-11-16 (via research agent)
