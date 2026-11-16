---
tags: [project, ethereum, optimism, synthetics, debt-pool, cdp, legacy]
ecosystem: Ethereum, Optimism
category: Synthetics/Debt-pool CDP
status: legacy (solo SNX staking phased out April 2025; superseded by Synthetix V3 and the 420 pool)
status_date: 2025-04
website: https://synthetix.io
docs: https://sips.synthetix.io
launched: 2018 (Havven/Synthetix V1); V2 date [NOT FOUND]; SNX inflation started March 2019 (SIP-23)
---

# Synthetix V1/V2 (debt-pool synthetics)

Status: legacy. The original Synthetix design let SNX holders lock SNX at a high collateralisation ratio (C-ratio) and mint sUSD, which they could exchange into other Synths (sBTC, sETH, forex, commodities, equities, inverse Synths). Solo SNX staking was deliberately phased out in April 2025 in favour of the 420 pool on V3 (see [[projects/synthetix-v3]]). This note covers the borrower side (the staker who mints and carries debt), which the V3 note does not.

Key distinction (see [[patterns/synthetic-collateral-models]] and [[patterns/cdp-synthetics-landscape]]): the collateral is staked SNX only. sUSD is the exchange and settlement unit that stakers mint, not collateral. Every staker's debt is a share of one shared global debt pool, so the debt of each staker moves with the aggregate USD value of all outstanding Synths, not with the Synths that staker personally minted.

## Adoption metrics

| Metric | Value | Date | Source |
|--------|-------|------|--------|
| TVL peak (V1+V2, staked SNX) | $2.967B | 2021-02-10 | [DefiLlama API](https://api.llama.fi/protocol/synthetix-v1+v2) |
| TVL | $154.6M | 2025-01-09 | [DefiLlama API](https://api.llama.fi/protocol/synthetix-v1+v2) |
| TVL (last datapoint, essentially zero after solo-staking phase-out) | $1,789 | 2025-08-08 | [DefiLlama API](https://api.llama.fi/protocol/synthetix-v1+v2) |
| Cumulative exchange fees (V1+V2) | $81.34M | accessed 2026-11-16 | [DefiLlama fees](https://api.llama.fi/summary/fees/synthetix-v1+v2) |
| Fees 2022 / 2023 / 2024 / 2025 | $25.30M / $33.80M / $20.37M / $1.75M | calendar years | [DefiLlama fees](https://api.llama.fi/summary/fees/synthetix-v1+v2) (summed from daily series; series starts Nov 2021, so 2021 shows only $0.10M) |
| sUSD supply, all chains and versions | $69.84M | 2026-11-16 | [DefiLlama stablecoins](https://stablecoins.llama.fi/stablecoins) |
| Active chains | Ethereum, Optimism | 2025 | [DefiLlama API](https://api.llama.fi/protocol/synthetix-v1+v2) |

Caveat: the DefiLlama protocol record flags `deadFrom: 2024-01-01`, but its own TVL series shows $154.6M in January 2025 and the collapse to near zero in April 2025. The April 2025 date matches governance (SCCP-401 and SCCP-403 below). Treat the 2024 flag as a data-labelling error.

## How it works

### User perspective (the borrower / staker)

A staker locks SNX and mints sUSD up to the target C-ratio (initially 750 to 800%, lowered stepwise by governance to 400% and below). The staker may keep, sell or exchange the sUSD. Minting creates a debt denominated in USD that is a share of the global debt pool. To unlock SNX the staker must burn sUSD equal to their current debt, which can be higher or lower than what they originally minted. To claim weekly staking rewards (inflationary SNX plus a share of exchange fees in sUSD) the staker must hold at or above the target C-ratio.

### Protocol perspective

- Synths are priced by oracles (see [[patterns/oracle-pricing-model]]); exchanges between Synths settle at the oracle price with no price slippage, so there is no AMM or order book.
- Exchange fees accrue to the FeePool, which distributes them in sUSD to stakers pro rata to their debt-pool share per fee period.
- Debt pool: when traders exchange sUSD into sETH and ETH rises, the aggregate debt rises and every staker's debt rises pro rata. Traders' profits are therefore paid by stakers (the counterparty design is the same as the LP-pool risk in [[patterns/lp-counterparty-risk]]).
- Peg: sUSD has no direct redemption for $1 of collateral. The structural bid is that stakers must buy and burn sUSD to repay debt or to meet the target C-ratio (SCCP-232 explicitly cites staker sUSD purchases as a peg support and a pressure point). Source: [SCCP-232](https://sips.synthetix.io/sccp/sccp-232/).

## Key behaviours

- [[patterns/synthetic-collateral-models]] : native-token CDP with a shared debt pool.
- [[patterns/oracle-pricing-model]] : the June 2019 sKRW incident (below) is the canonical oracle-misconfiguration case.
- [[patterns/lp-counterparty-risk]] : stakers are the counterparty to all Synth traders.
- [[patterns/cdp-synthetics-landscape]] : comparison to per-position CDPs and delta-neutral designs.

## Architecture decisions

- **Shared debt pool rather than per-position collateral:** lets Synths be exchanged with zero slippage, at the cost of staker exposure to trader PnL.
- **Native token as only collateral (reflexivity):** SNX price down lowers C-ratios and forces stakers to burn sUSD, which pressures the peg.
- **Fee design moved from flat to differential:** flat 30 bps for all Synths, then category rates, then summed pair fees (see Fees section).
- **Liquidation redesign (SIP-148, created 2021-06-11):** flag then liquidate after a delay. Liquidated SNX goes to other stakers through a LiquidatorRewards contract with 12 month escrow, not to the liquidator.
- **Inflation wind-down:** the original schedule halved weekly rewards yearly from March 2019; SIP-23 smoothed it to a 1.25% weekly reduction with a 2.5% terminal rate; SIP-2043 (December 2023) ended SNX inflation.
- **Exit:** SIP-420 (created 2025-01-06) moves debt positions into a protocol-owned delegated pool; SCCP-403 (2025-04-09) raised the solo staking liquidation ratio on a schedule to retire solo staking.

## Differentiators

- First production on-chain synthetic asset protocol, with forex, commodity, equity and inverse Synths priced by oracle.
- Zero-slippage exchange of Synths against a shared pool.
- Exchange fees flow to stakers through the fee pool (DefiLlama methodology: revenue is "awarded to SNX stakers"; the exact treasury share, if any, is [NOT FOUND]).

## Limitations and criticisms

- Reflexive: SNX is both collateral and reward token. Governance repeatedly cut the C-ratio (800% to as low as 350 to 400%) to keep stakers above target after price falls (SCCP-232 cites weakness in SNX price).
- Passive stakers carry unhedged exposure to aggregate trader PnL; hedging required active management.
- sUSD peg was repeatedly stressed; V3 later reused sUSD and needed buybacks (see [[projects/synthetix-v3]]; SIP-405 redirected V3 fees to burn sUSD and reduce V2 stakers' debt).
- Oracle incidents: June 2019 sKRW feed reported Korean Won 1000 times too high; a bot generated about 37M sETH of paper profit within an hour and the funds were recovered through a white-hat negotiation and bounty (DefiLlama hacks record; secondary description, no primary post-mortem retrieved).

### Fees and borrower cost

The borrower (staker) side has no interest rate. The borrower cost is (a) the foregone and variable value of carrying a floating debt, (b) the C-ratio lockup, (c) liquidation penalties, and (d) the fees below on Synth exchanges (which accrue to stakers, not to the protocol treasury, per DefiLlama methodology).

| Item | Value | Date | Source |
|------|-------|------|--------|
| Interest or stability fee on staker debt | None; debt floats with the global debt pool | n/a | [DefiLlama methodology](https://api.llama.fi/summary/fees/synthetix-v1+v2) (describes only exchange fees); no stability fee appears in the SIP/SCCP lists retrieved from [sips.synthetix.io](https://sips.synthetix.io/all-sccp) |
| Fee to mint sUSD against staked SNX | [NOT FOUND] (no explicit statement retrieved; the only issue-type fees in the SCCP list are on wrappers and ETH-backed loans) | accessed 2026-11-16 | [all SCCPs](https://sips.synthetix.io/all-sccp) |
| Exchange fee, flat | 30 bps on every Synth | before 2020-06 | [SCCP-24](https://sips.synthetix.io/sccp/sccp-24) |
| Exchange fee, SCCP-24 categories | forex, commodity, equity 5 bps; crypto and inverse crypto 30 bps | created 2020-06-04 | [SCCP-24](https://sips.synthetix.io/sccp/sccp-24) |
| Exchange fee, SIP-222 (sum of source and destination Synth fee) | L1: sUSD 0, forex 15, sBTC and sETH 25, sLINK 85, sADA, sDOT, sAAVE, sDEFI 100 bps. L2: sUSD 0, forex 15, sETH and sBTC 25, sLINK 30, others 35 bps. Example: sETH to sAAVE on L1 = 125 bps | created 2022-03-27 (Mirach release) | [SIP-222](https://sips.synthetix.io/sips/sip-222/) |
| Exchange fee range quoted by DefiLlama | 10 to 100 bps, usually 30 bps | 2026-11-16 | [DefiLlama fees](https://api.llama.fi/summary/fees/synthetix-v1+v2) |
| Fee recipient | SNX stakers: fees accrue as sUSD to the fee pool and reduce debt on claim (DefiLlama: "automatically burning outstanding debt"). A treasury share is not quantified in the sources retrieved: [NOT FOUND] | 2026-11-16 | [DefiLlama fees](https://api.llama.fi/summary/fees/synthetix-v1+v2); [FeePool contract docs](https://developer.synthetix.io/contracts/source/contracts/FeePool) (search result only, not re-fetched) |
| ETH-backed sUSD loans (Collateral contract), issue fee history | 200 bps (SCCP-103) to 50 bps (SCCP-108) to 25 bps (SCCP-116). SCCP-82 lowered loan C-ratio and liquidation ratio from 150% to 130%. Current value: [NOT FOUND] | 2020 to 2021 | [all SCCPs](https://sips.synthetix.io/all-sccp) |
| Wrapper mint fee history (wETH, DAI, LUSD wrappers) | 200 bps to 100 (SCCP-104) to 50 (109) to 37.5 (111) to 25 (114) to 5 bps (117); DAI wrapper 5 bps (SCCP-147); L2 ETH wrapper 5 bps (SCCP-143) then 250 bps (SCCP-145); ETH wrapper burn fee 5% (SCCP-203). Current values: [NOT FOUND] | 2020 to 2022 | [all SCCPs](https://sips.synthetix.io/all-sccp) |
| Liquidation penalty (SIP-148) | 30% forced; 20% self-liquidation; flag reward 10 SNX (L1) or 1 SNX (L2); liquidate reward 20 SNX (L1) or 2 SNX (L2); 12 hour delay | created 2021-06-11 | [SIP-148](https://sips.synthetix.io/sips/sip-148/) |
| Liquidation ratio | 150% at SIP-148; 145% via SCCP-216; 160% restored via SCCP-401 (created 2025-03-24) | 2021 to 2025 | [SCCP-401](https://sips.synthetix.io/sccp/sccp-401/); [all SCCPs](https://sips.synthetix.io/all-sccp) |
| Target C-ratio path | 750 to 800% (SCCP-2, 14), 700 (30), 600 (41), 500 (59), 450 (127), 400 (128), then SIP-148-era moves (350 in SCCP-204, 400 in SCCP-216 and 240), temporary 350 (SCCP-232, created 2022-09-11) | 2019 to 2022 | [all SCCPs](https://sips.synthetix.io/all-sccp); [SCCP-232](https://sips.synthetix.io/sccp/sccp-232/) |
| SNX inflation reward | 1.44M SNX per week at launch; 2.5% terminal from September 2023; ended by SIP-2043 (December 2023) | 2019 to 2023 | [SIP-23/24 search results](https://sips.synthetix.io/sips/sip-24); [The Block](https://www.theblock.co/news/defi/2023-12-12-synthetix-snx-token-inflation-buybacks-burns-267138) (search result summary, not re-fetched) |
| Revenue per year to stakers (fees) | See adoption table: $25.3M (2022), $33.8M (2023), $20.4M (2024), $1.75M (2025) | 2026-11-16 | [DefiLlama fees](https://api.llama.fi/summary/fees/synthetix-v1+v2) |

## Sources

- [DefiLlama Synthetix v1+v2 API](https://api.llama.fi/protocol/synthetix-v1+v2) : accessed 2026-11-16
- [DefiLlama fees: Synthetix v1+v2](https://api.llama.fi/summary/fees/synthetix-v1+v2) : accessed 2026-11-16
- [DefiLlama stablecoins list (sUSD)](https://stablecoins.llama.fi/stablecoins) : accessed 2026-11-16
- [DefiLlama hacks list](https://api.llama.fi/hacks) : accessed 2026-11-16
- [SCCP-24 differential fee levels](https://sips.synthetix.io/sccp/sccp-24) : accessed 2026-11-16
- [SIP-222 update exchange fee system](https://sips.synthetix.io/sips/sip-222/) : accessed 2026-11-16
- [SIP-148 liquidation mechanism V2](https://sips.synthetix.io/sips/sip-148/) : accessed 2026-11-16
- [SCCP-232 temporary C-ratio 350%](https://sips.synthetix.io/sccp/sccp-232/) : accessed 2026-11-16
- [SCCP-401 unpause staking liquidations at 160%](https://sips.synthetix.io/sccp/sccp-401/) : accessed 2026-11-16
- [SCCP-403 liquidation ratio schedule](https://sips.synthetix.io/sccp/sccp-403/) : accessed 2026-11-16
- [SIP-420](https://sips.synthetix.io/sips/sip-420/) : accessed 2026-11-16
- [SIP-405 temporary distribution of V3 fees to V2 stakers](https://sips.synthetix.io/sips/sip-405) : accessed 2026-11-16
- [All SCCPs index](https://sips.synthetix.io/all-sccp) : accessed 2026-11-16
