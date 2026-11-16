---
tags: [project, ethereum, cdp, stablecoin, synthetics, immutable]
ecosystem: Ethereum
category: Synthetics/CDP stablecoin
status: live (V1 live since April 2021; V2 relaunched 2025-05-19)
status_date: 2026-11-16
website: https://www.liquity.org
docs: https://docs.liquity.org
launched: 2021 (V1, LUSD), 2025 (V2, BOLD)
---

# Liquity V1 and V2 (LUSD and BOLD)

Status: live. Liquity is a pair of immutable, governance-minimal CDP protocols that mint a USD-pegged synthetic stablecoin against ETH (V1: LUSD) and against ETH, wstETH and rETH (V2: BOLD). V1 charges no interest and instead a one-time fee; V2 introduces user-set interest rates. Neither has an admin key or upgradeability, which is the main design contrast with Maker/Sky (see [[projects/sky-maker]]).

## Adoption metrics

| Metric | Value | Date | Source |
|--------|-------|------|--------|
| TVL, V1 | $130.33M | 2026-11-16 | [DefiLlama Liquity V1](https://api.llama.fi/protocol/liquity-v1) |
| TVL peak, V1 | $4.52B | 2021-05-11 | [DefiLlama Liquity V1](https://api.llama.fi/protocol/liquity-v1) |
| LUSD circulating | $26.85M | 2026-11-16 | [DefiLlama stablecoins](https://stablecoins.llama.fi/stablecoins) |
| TVL, V2 | $70.73M | 2026-11-16 | [DefiLlama Liquity V2](https://api.llama.fi/protocol/liquity-v2) |
| TVL peak, V2 | $180.03M | 2025-10-07 | [DefiLlama Liquity V2](https://api.llama.fi/protocol/liquity-v2) |
| BOLD circulating (current; a separate "Legacy BOLD" asset shows $0.20M) | $33.09M | 2026-11-16 | [DefiLlama stablecoins](https://stablecoins.llama.fi/stablecoins) |
| Fees, V1: 30 days / 1 year / all time | $264,765 / $376,610 / $85.11M | 2026-11-16 | [DefiLlama overview fees](https://api.llama.fi/overview/fees) |
| Fees, V2: 30 days / 1 year / all time | $775,501 / $2.49M / $3.89M | 2026-11-16 | [DefiLlama overview fees](https://api.llama.fi/overview/fees) |
| Revenue, V2: 30 days / all time | $30,361 / $696,374 | 2026-11-16 | [DefiLlama overview fees (dailyRevenue)](https://api.llama.fi/overview/fees?dataType=dailyRevenue) |

Cross-check note: DefiLlama V2 "fees" include interest, redemption fees and liquidation gas compensation (its methodology text), and "revenue" is described as 25% of interest, but the 30 day revenue ($30k) is far below 25% of 30 day fees ($194k), so the two series are not reconcilable from the API alone. Use the fee line with caution.

## How it works

### User perspective

A borrower opens a Trove, deposits collateral and mints stablecoin. V1: ETH only, minimum collateral ratio (MCR) 110%, minimum debt 2,000 LUSD plus a 200 LUSD liquidation reserve, a one-time borrowing fee, no interest. V2: three separate borrow markets (ETH, wstETH, rETH), the borrower picks and may change their own annual interest rate, interest accrues into the debt, minimum debt 2,000 BOLD, and the Multiply feature loops collateral through Curve and Uniswap.

### Protocol perspective

- **Liquidation:** Stability Pool first (depositors burn stablecoin and receive the liquidated collateral at a discount). If the pool is empty, the debt and collateral are redistributed across the other Troves. V1 has Recovery Mode (system CR below 150%); V2 removes Recovery Mode. V2 liquidation ratio is 110% for ETH (max LTV 90.91%) and 120% for wstETH and rETH (max LTV 83.33%); the liquidated borrower usually pays a 5% penalty, and up to 10% (ETH) or 20% (LSTs) of debt in a redistribution.
- **Peg:** hard floor by redemption (anyone can swap stablecoin for collateral at $1 face value). V1 redeems against the lowest collateral-ratio Trove; V2 redeems against the lowest-interest-rate Trove first, so borrowers who set low rates carry redemption risk. Ceiling by borrowing demand: V1 through the rising borrowing fee after redemptions; V2 through borrowers cutting rates when BOLD is above $1 and raising them when below.
- **Oracles:** see [[patterns/oracle-pricing-model]]; oracle parameters are not retrieved here: [NOT FOUND].

## Key behaviours

- [[patterns/synthetic-collateral-models]] : multi-position CDP, overcollateralised, immutable; compare with the shared debt pool in [[projects/synthetix-v2]].
- [[patterns/cdp-synthetics-landscape]] : survival correlates (immutability, one or few collateral types, redemption floor).

## Architecture decisions

- **Immutability:** contracts are not upgradeable and V2 governance only directs "Protocol Incentivized Liquidity" (25% of V2 revenue to external initiatives). Source: [V2 general FAQ](https://docs.liquity.org/v2-faq/general.md).
- **Interest-free V1, market-set rates V2:** V1 uses a one-off fee that algorithmically rises with redemptions; V2 hands rate-setting to borrowers (market-driven monetary policy).
- **Separate stability pool per collateral in V2:** keeps risk compartmentalised; redemptions are spread proportional to "outside debt" (debt minus stability pool size) per collateral.
- **No Recovery Mode in V2; delegation of rate setting** to third-party managers.
- **V2 redeployment:** an issue in the Stability Pool code was announced 2025-02-17; users were told to exit the pools. After a five-week Cantina audit contest with 800+ researchers and re-audits, V2 went live on mainnet 2025-05-19. Source: [Liquity blog: V2 is live](https://www.liquity.org/blog/liquity-v2-is-live), [stability pool issue](https://www.liquity.org/blog/stability-pool-issue) (accessed via search summary; bug details [NOT FOUND]).

## Differentiators

- Lowest MCR of the major CDPs (110%) made possible by the Stability Pool and the redemption floor.
- Only ETH and ETH LSTs as collateral: no RWAs, no centralised collateral.
- Fully immutable contracts.

## Limitations and criticisms

- Capital efficiency depends on active redemption risk management by borrowers (V2 rate setting); poorly watched low-rate Troves are redeemed first.
- Narrow collateral set means supply is bounded by ETH-denominated borrow demand: LUSD is $26.85M against a V1 TVL of $130M (accessed 2026-11-16), versus billions at the 2021 peak.
- Bootstrapping stablecoin liquidity needs incentives (PIL).
- V2 pre-launch bug and relaunch cost about three months (February to May 2025).
- Redemptions force a partial close of the borrower's Trove at face value even when the Trove is healthy; borrowers receive the redemption fee as collateral but may lose preferred ETH exposure.

### Fees and borrower cost

| Item | V1 | V2 | Source |
|------|----|----|--------|
| Interest | 0% | User-set per Trove; the current market average is [NOT FOUND] (Liquity publishes it on a Dune dashboard, not retrieved) | [V1 FAQ](https://docs.liquity.org/liquity-v1/faq/borrowing.md); [V2 FAQ](https://docs.liquity.org/v2-faq/borrowing-and-liquidations.md) |
| Opening or drawing fee | One-time borrowing fee, floor 0.5%, rising with recent redemptions up to 5%; 0% during Recovery Mode | Upfront fee equal to 7 days of average interest on that collateral branch | same |
| Rate-change fee | n/a | Premature adjustment fee if the rate is changed within 7 days of the last change | [V2 FAQ](https://docs.liquity.org/v2-faq/borrowing-and-liquidations.md) |
| Redemption fee | (baseRate + 0.5%) times ETH drawn; baseRate half-life 12 hours; paid to LQTY stakers | min(0.5% + baseRate, 100%); baseRate half-life 6 hours; stays in the redeemed Trove as collateral | [V1 FAQ](https://docs.liquity.org/liquity-v1/faq/lusd-redemptions.md); [V2 FAQ](https://docs.liquity.org/v2-faq/redemptions-and-delegation.md) |
| Interest split | n/a | 75% to the collateral market's Stability Pool depositors; 25% to governance-directed Protocol Incentivized Liquidity | [V2 BOLD and Earn FAQ](https://docs.liquity.org/v2-faq/bold-and-earn.md) |
| Liquidation gas compensation | 200 LUSD reserve | 0.0375 WETH refundable deposit plus min(0.5% of collateral, 2 units of WETH or LST) | [V2 FAQ](https://docs.liquity.org/v2-faq/borrowing-and-liquidations.md) |
| Liquidation loss to borrower | about 9.09% of collateral value at 110% MCR | 5% penalty typical; 10% or 20% of debt on redistribution | [V1 FAQ](https://docs.liquity.org/liquity-v1/faq/borrowing.md); [V2 FAQ](https://docs.liquity.org/v2-faq/borrowing-and-liquidations.md) |
| Stability Pool yield source | liquidation gains (LQTY issuance also rewards depositors per the V1 rewards FAQ, not re-fetched) | 75% of interest plus liquidation gains at about 5% discount; no token emissions | [V2 FAQ](https://docs.liquity.org/v2-faq/bold-and-earn.md) |

Audits (V1): Trail of Bits and Coinspect (DefiLlama audit links, [DefiLlama Liquity V1](https://api.llama.fi/protocol/liquity-v1)). Audits (V2): ChainSecurity, Dedaub (28 Aug 2024, 11 Nov 2024, 13 May 2025 fix review) and the Cantina contest, per [DefiLlama Liquity V2](https://api.llama.fi/protocol/liquity-v2) and [search results](https://dedaub.com/audits/liquity/liquity-v2-cantina-fixes-review-may-13-2025/).

## Sources

- [DefiLlama Liquity V1 API](https://api.llama.fi/protocol/liquity-v1) : accessed 2026-11-16
- [DefiLlama Liquity V2 API](https://api.llama.fi/protocol/liquity-v2) : accessed 2026-11-16
- [DefiLlama stablecoins list (LUSD, BOLD)](https://stablecoins.llama.fi/stablecoins) : accessed 2026-11-16
- [DefiLlama fees overview](https://api.llama.fi/overview/fees) : accessed 2026-11-16
- [Liquity V1 borrowing FAQ](https://docs.liquity.org/liquity-v1/faq/borrowing.md) : accessed 2026-11-16
- [Liquity V1 redemptions FAQ](https://docs.liquity.org/liquity-v1/faq/lusd-redemptions.md) : accessed 2026-11-16
- [Liquity V2 general FAQ](https://docs.liquity.org/v2-faq/general.md) : accessed 2026-11-16
- [Liquity V2 borrowing and liquidations FAQ](https://docs.liquity.org/v2-faq/borrowing-and-liquidations.md) : accessed 2026-11-16
- [Liquity V2 BOLD and Earn FAQ](https://docs.liquity.org/v2-faq/bold-and-earn.md) : accessed 2026-11-16
- [Liquity V2 redemptions and delegation FAQ](https://docs.liquity.org/v2-faq/redemptions-and-delegation.md) : accessed 2026-11-16
- [Liquity blog: V2 is live](https://www.liquity.org/blog/liquity-v2-is-live) : accessed 2026-11-16 (search summary)
