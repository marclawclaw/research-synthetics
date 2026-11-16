---
tags: [project, tezos, cdp, synthetics, stablecoin]
ecosystem: Tezos
category: Synthetics/Multi-asset CDP
status: live (small; uUSD about $1M circulating)
status_date: 2026-11-16
website: https://app.youves.com
docs: https://docs.youves.com
launched: 2021-09 (first DefiLlama TVL datapoint 2021-09-07; mainnet launch date itself [NOT FOUND])
---

# Youves (Tezos: uUSD, uBTC, uXTZ, uDEFI)

Status: live and small. Youves is the main multi-asset synthetic CDP on Tezos. Users open a vault, lock collateral (XTZ, tzBTC, USDt or SIRS, the tez/tzBTC liquidity baking token) and mint "tracker tokens": uUSD (USD), uBTC (BTC priced in USD), uXTZ (XTZ priced in USD) and the now inactive uDEFI (a DeFi index). An uXAU gold tracker was piloted and hit an oracle incident in 2023 (see below). A separate Tezos stablecoin, Kolibri kUSD, is not covered here.

## Adoption metrics

| Metric | Value | Date | Source |
|--------|-------|------|--------|
| TVL (Tezos; plus $1.17M "pool2") | $16.20M | 2026-11-16 | [DefiLlama Youves](https://api.llama.fi/protocol/youves) |
| TVL peak | $43.68M | 2024-03-13 | [DefiLlama Youves](https://api.llama.fi/protocol/youves) |
| uUSD circulating (DefiLlama) | $1.11M; price $1.0101 | 2026-11-16 | [DefiLlama stablecoins](https://stablecoins.llama.fi/stablecoins) |
| uUSD supply, sum of four v3 engines read on chain (XTZ 66,868; tzBTC 35,213; SIRS 274,722; USDt 602,700) | 979,502 uUSD | 2026-11-16 | [TzKT engine storage](https://api.tzkt.io/v1/contracts/KT1JmfujyCYTw5krfu9bSn7YbLYuz2VbNaje/storage) and the three other engine addresses in the docs table |
| uBTC supply (XTZ-collateral engine) | 19.31 uBTC | 2026-11-16 | [TzKT](https://api.tzkt.io/v1/contracts/KT1CP1C8afHqdNfBsSE3ggQhzM2iMHd4cRyt/storage) |
| uXTZ supply (USDt, XTZ, SIRS engines) | 44.35 + 9,380.45 + 0.71 uXTZ | 2026-11-16 | [TzKT](https://api.tzkt.io/v1/contracts/KT1Mf9Nr1KyGC6gUz9pGQnngzWbbZ6thShvc/storage) |
| Fees and revenue (DefiLlama) | [NOT FOUND] (no DefiLlama fee adapter: the endpoint returns an error) | 2026-11-16 | [DefiLlama fees: youves](https://api.llama.fi/summary/fees/youves) |

Cross-check: on-chain engine supply (979,502 uUSD) is within 12% of DefiLlama's 1.11M; the remainder may be legacy v1/v2 engines (not queried). TVL ($16.2M) is much larger than synthetic supply, so TVL includes assets other than vault collateral behind uUSD (for example farm and staking positions); its composition is [NOT FOUND].

## How it works

### User perspective

Connect a Tezos wallet, create and fund a vault, mint a tracker token up to a limit set by the engine, pay a one-time minting fee deducted from the minted amount, and keep the vault's collateral ratio above the engine's emergency ratio. Burning tracker tokens releases collateral. XTZ collateral keeps earning 100% of baking rewards (less any baker fee). Minters also earn YOU governance tokens in proportion to minting volume.

### Protocol perspective

- **Tracker tokens and engines:** each synthetic/collateral pair has its own v3 "engine" contract with its own parameters. All tracker tokens live in one FA2 contract (KT1XRPEPXbZK25r3Htzp2o1x7xdMMmfocKNW, token ids 0 uUSD, 1 uDEFI, 2 uBTC and uXTZ).
- **Collateral ratios (docs, v3 engines):** emergency ratios XTZ 160%, tzBTC 135% (uUSD) or 110% (uXTZ), SIRS 135% (uUSD) or 160% (uXTZ), USDt 110% (uUSD) or 160% (uXTZ vs USDt), frontend target 115% to 200%. v1 and v2 engines used a 300% target and a 200% emergency ratio. The target ratio is a frontend setting only and is not enforced by v3 contracts. Source: [Collateral Management Details](https://docs.youves.com/syntheticAssets/stableTokens/collateralManagement/Collateral-Management-Details).
- **Liquidation ("step-in"):** below the emergency ratio a vault is open to partial liquidation by third parties, who post and burn tracker tokens, bring the vault back to target, and take a bonus from the excess collateral. On chain, the liquidation payout ratio is 112.5% on most engines and 110% on the USDt and XTZ uXTZ engines (see Fees).
- **Last-resort stability mechanism:** YOU staking pool tokens are used to recapitalise vaults below 100% collateral up to the emergency level. Source: [Platform economics](https://docs.youves.com/platformOverview/PlatformOverview-Revenues).
- **Peg mechanism:** an oracle-fed target price plus interest rates that respond to market-to-target deviation. For uUSD the asset interest rate response is exponential and updated weekly (response up to about +5.9% and -5.6% per annum, total asset rate capped at +29.4% and floored at +0.4% per annum); for the newer trackers it is linear, updated every 12 hours (+/-0.4% per annum). Savers who stake tracker tokens in the savings pool earn the asset rate; minters pay a higher liability rate. Holders and minters also have conversion rights ("bailout"), and a flat-curve AMM swaps against the target price. Source: [Interest Rate Response](https://docs.youves.com/syntheticAssets/stableTokens/incentiveFeatures/interestRates/Interest-Rate-Response).
- **Oracle:** an on-chain generic oracle with data transmitters (two outages in 2022 are documented, below). See [[patterns/oracle-pricing-model]].

## Key behaviours

- [[patterns/synthetic-collateral-models]] : multi-collateral, multi-synthetic CDP with interest-rate peg.
- [[patterns/cdp-synthetics-landscape]] : closest in design to Reflexer-style rate controllers, but on Tezos.
- [[patterns/oracle-pricing-model]] : three of ten published post-mortems are oracle failures.

## Architecture decisions

- **Interest rate as the peg control** (no redemption at $1): the platform adjusts rates on tracker tokens to move supply and demand toward the target.
- **Per-engine parameters** instead of one global rule set, so new collateral types (SIRS, tzBTC, USDt) can be added with their own emergency ratios.
- **Unified staking pool with YOU** as the backstop.
- **Introducer (referral) programme in v3:** introducers earn 10% of the new minter's YOU allocation on the non-zero-fee engines. Source: [Introducer fee](https://docs.youves.com/syntheticAssets/stableTokens/StableToken-Introducer-Fee).

## Differentiators

- Only mature on-chain synthetic CDP on Tezos with a ten-incident public post-mortem log.
- Native XTZ collateral keeps earning baking rewards.
- Savings-pool stability design with on-chain interest rate response.

## Limitations and criticisms

- Small: uUSD circulating about $1M against a peak TVL of $43.7M (2024-03-13).
- Security record: 10 public post-mortems on docs.youves.com. Examples: 2022-04-28 SmartPy migration exposed internal entrypoints (no funds at risk); 2022-04-01 and 2022-05-14 oracle denial-of-service events; 2022-11-08 uUSD/tez v3 liquidations above a size threshold failed; 2023-05-22 a compiled-code decimals bug in an unreleased uBTC/tzBTC engine would have allowed minting up to 10^8 times too many tokens (not released, no impact); 2023-09-15 a gold price API error halved the uXAU oracle price for about 3 hours 45 minutes, letting users and bots buy uXAU cheaply (amount [NOT FOUND]); 2023-09-28 an LP token bug locked yyXTZ tokens. Source: [Youves postmortems](https://docs.youves.com/postmortems/postmortem_incident_7) and neighbouring pages.
- Audits: Least Authority (21 September 2021) and Inference AG (12 August 2022); later engine versions audit status [NOT FOUND]. Source: [Security Audit](https://docs.youves.com/security/Security-Audit).
- Liquidity is thin and Tezos-only; no cross-chain use of uUSD beyond limited bridges ([NOT FOUND]).

### Fees and borrower cost

| Item | Value | Date | Source |
|------|-------|------|--------|
| Minting fee, uUSD and uBTC engines (deducted from minted amount, burning is free) | 1.5625% (`minting_fee_ratio` 15625/1,000,000) on every engine checked: uUSD/XTZ, uUSD/tzBTC, uUSD/SIRS, uUSD/USDt, uBTC/XTZ, uBTC/tzBTC | on-chain, 2026-11-16 | [TzKT uUSD/XTZ engine](https://api.tzkt.io/v1/contracts/KT1DHndgk8ah1MLfciDnCV2zPJrVbnnAH9fd/storage), [uUSD/USDt engine](https://api.tzkt.io/v1/contracts/KT1JmfujyCYTw5krfu9bSn7YbLYuz2VbNaje/storage) |
| Minting fee, uXTZ engines | 0.3906% (`minting_fee_ratio` 390625/100,000,000) on all three (USDt, XTZ, SIRS) | on-chain, 2026-11-16 | [TzKT uXTZ/XTZ engine](https://api.tzkt.io/v1/contracts/KT1Mf9Nr1KyGC6gUz9pGQnngzWbbZ6thShvc/storage) |
| Minting fee launch-era figure (consistency check) | 1.5625% on XTZ-collateral uUSD | launch coverage | [XTZ newsletter](https://xtznews.beehiiv.com/p/minting-assets-savings-with-interest-and-you-tokens-the-youves-platform-goes-live-tomorrow) (search result) |
| Docs statement on 0% engines | Engines marked 0% (for example the tzBTC and SIRS 0% engines referenced in the 2023-05 post-mortem) deduct no minting fee | docs | [Minting and burning](https://docs.youves.com/syntheticAssets/stableTokens/lifecycle/Minting-and-Burning) |
| Liability (borrower) interest spread over the asset rate | About 1.00% per annum, claimed by the platform on an ongoing basis (docs: "roughly 1.00% higher"); on chain `spread_rate` = 316 per second at 1e-12 scale = 1.00% per annum compounded | docs and on-chain, 2026-11-16 | [Platform economics](https://docs.youves.com/platformOverview/PlatformOverview-Revenues); TzKT storage above |
| Reference interest rate on chain (`reference_interest_rate`) | uUSD engines 1901 (about 6.18% per annum), uXTZ engines 1700 (5.51%), uBTC engines 196 (0.62%). The mapping of this field to the docs' "asset" versus "liability" rate is my inference; conversion = exp(raw x 1e-12 x 31,557,600) minus 1 | 2026-11-16 | [TzKT](https://api.tzkt.io/v1/contracts/KT1DHndgk8ah1MLfciDnCV2zPJrVbnnAH9fd/storage) |
| Interest rate band (uUSD) | Asset rate capped at +29.4% and floored at +0.4% per annum | docs | [Interest Rate Response](https://docs.youves.com/syntheticAssets/stableTokens/incentiveFeatures/interestRates/Interest-Rate-Response) |
| Liquidation payout | `liquidation_payout_ratio` 1.125 (most engines) or 1.10 (uUSD/USDt and uXTZ/XTZ); settlement payout ratio 0.9375; settlement reward fee ratio 0.125 | on-chain, 2026-11-16 | [TzKT](https://api.tzkt.io/v1/contracts/KT1DHndgk8ah1MLfciDnCV2zPJrVbnnAH9fd/storage) |
| Where fees go | Minting fee and the interest spread are platform revenue; staking rewards go to YOU stakers (revenue in kind: tracker tokens) | docs | [Platform economics](https://docs.youves.com/platformOverview/PlatformOverview-Revenues) |
| Revenue and fees in USD (all-time, 1y) | [NOT FOUND] (no DefiLlama adapter; no revenue dashboard located) | 2026-11-16 | n/a |

Conflicts: the docs collateral table lists the uBTC/tzBTC and uBTC/SIRS engines at the same address and shows an emergency ratio of 110% for uBTC/tzBTC, whereas the contract at that address reads 135%. On-chain values are used throughout.

## Sources

- [DefiLlama Youves API](https://api.llama.fi/protocol/youves) : accessed 2026-11-16
- [DefiLlama stablecoins list](https://stablecoins.llama.fi/stablecoins) : accessed 2026-11-16
- [Youves docs: platform economics](https://docs.youves.com/platformOverview/PlatformOverview-Revenues) : accessed 2026-11-16
- [Youves docs: minting and burning](https://docs.youves.com/syntheticAssets/stableTokens/lifecycle/Minting-and-Burning) : accessed 2026-11-16
- [Youves docs: collateral management details](https://docs.youves.com/syntheticAssets/stableTokens/collateralManagement/Collateral-Management-Details) : accessed 2026-11-16
- [Youves docs: collateral management types (step-in)](https://docs.youves.com/syntheticAssets/stableTokens/collateralManagement/Collateral-Management-Types) : accessed 2026-11-16
- [Youves docs: interest rate response](https://docs.youves.com/syntheticAssets/stableTokens/incentiveFeatures/interestRates/Interest-Rate-Response) : accessed 2026-11-16
- [Youves docs: applied interest rates](https://docs.youves.com/syntheticAssets/stableTokens/incentiveFeatures/interestRates/Interest-Rate-Applied) : accessed 2026-11-16
- [Youves docs: introducer fee](https://docs.youves.com/syntheticAssets/stableTokens/StableToken-Introducer-Fee) : accessed 2026-11-16
- [Youves docs: security audit](https://docs.youves.com/security/Security-Audit) : accessed 2026-11-16
- [Youves docs: postmortems 1 to 9](https://docs.youves.com/postmortems/postmortem_incident_1) : accessed 2026-11-16 (incident 10 page returned a GitHub error page)
- [TzKT contract storage API](https://api.tzkt.io/v1/contracts/KT1JmfujyCYTw5krfu9bSn7YbLYuz2VbNaje/storage) : accessed 2026-11-16
