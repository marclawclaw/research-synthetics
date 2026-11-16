---
tags: [project, ethereum, cdp, stablecoin, synthetics, governance]
ecosystem: Ethereum (USDS also on 7 other chains)
category: Synthetics/CDP stablecoin
status: live (rebranded from MakerDAO/DAI to Sky/USDS; DAI still in circulation)
status_date: 2026-11-16
website: https://sky.money
docs: https://developers.skyeco.com
launched: [NOT FOUND in primary sources retrieved]; DefiLlama TVL series starts 2019-01-04
---

# Sky Protocol (formerly MakerDAO): DAI and USDS

Status: live and by far the largest CDP stablecoin. Users lock collateral in a Vault and draw DAI or USDS, a USD-pegged synthetic dollar. It is included as the reference per-position CDP design and as the best-documented failure case for auction liquidations (Black Thursday, March 2020). Sky now also lends to "allocators" and holds real-world assets and stablecoin reserves, so a large share of supply is no longer backed by user CDPs. That share is not quantified in the sources retrieved: [NOT FOUND].

## Adoption metrics

| Metric | Value | Date | Source |
|--------|-------|------|--------|
| TVL, Sky Lending | $3.756B (plus $555.8M in a separate "staking" series) | 2026-11-16 | [DefiLlama Sky Lending](https://api.llama.fi/protocol/sky-lending) |
| TVL peak | $19.82B | 2021-12-01 | [DefiLlama Sky Lending](https://api.llama.fi/protocol/sky-lending) |
| DAI circulating | $4.623B | 2026-11-16 | [DefiLlama stablecoins](https://stablecoins.llama.fi/stablecoins) |
| USDS circulating | $5.351B | 2026-11-16 | [DefiLlama stablecoins](https://stablecoins.llama.fi/stablecoins) |
| Fees: 24h / 30d / 1y / all time | $934,505 / $27.40M / $381.18M / $1.30B | 2026-11-16 | [DefiLlama fees overview](https://api.llama.fi/overview/fees) |
| Revenue (fees minus savings rate paid): 30d / 1y / all time | $13.52M / $189.20M / $784.98M | 2026-11-16 | [DefiLlama revenue overview](https://api.llama.fi/overview/fees?dataType=dailyRevenue) |

Check on units: the $3.756B TVL is far below the $9.97B combined DAI and USDS supply. The gap is consistent with supply backed by non-CDP reserves (PSM, RWA, allocator assets), but the split is [NOT FOUND].

## How it works

### User perspective

A user opens a Vault for a collateral type ("ilk"), deposits collateral, draws DAI or USDS up to the ilk's liquidation ratio, and pays a per-ilk stability fee accruing in the debt. Repaying returns the collateral. Holders of DAI or USDS may deposit into the DSR or SSR savings contracts.

### Protocol perspective

- **Stability fee accounting:** the Vat keeps a cumulative rate per ilk, debt is `art * rate`, and the Jug sets a per-second fee equal to the ilk `duty` plus a global `base`. Fee revenue increases the Vow's surplus. Source: [Sky rates overview](https://developers.skyeco.com/protocol/rates/ovreview/).
- **Liquidation 2.0 (Dutch auction):** `Dog.bark` seizes the whole Vault and calls `Clipper.kick`. Start price is the oracle (OSM) price plus a buffer `buf`; a pluggable calculator lowers the price over time; auctions reset after a `tail` timeout or when price falls below `cusp` times the start price. The penalty is `chop` (example in the docs: 1.13 for a 13% penalty). Keepers receive a flat `tip` and a percentage `chip`. Global and per-ilk `hole` caps limit DAI needed by active auctions. A four level circuit breaker can stop kicks, resets and takes. Bidders can use flash-lent funds. Source: [Sky collateral liquidation](https://developers.skyeco.com/protocol/vaults/collateral-liquidation/).
- **Peg:** DAI debt is repaid at $1 per unit; arbitrage through Vault repayment, the Peg Stability Module (USDC swaps, per DefiLlama's fee methodology) and the savings rate. PSM fee and DSR/SSR current values: [NOT FOUND].
- **Oracle delay:** the OSM price is one to two hours behind the market by design (same Sky docs page), a safety delay against oracle attacks that also creates liquidation lag.

## Key behaviours

- [[patterns/synthetic-collateral-models]] : multi-collateral CDP with governance-set per-collateral risk parameters.
- [[patterns/oracle-pricing-model]] : delayed oracle (OSM) plus Dutch auction; the 2020 stale-price failure is the origin of both.
- [[patterns/cdp-synthetics-landscape]] : per-position CDP reference design.

## Architecture decisions

- **Governance-managed risk parameters per collateral type:** flexible, but requires continuous governance and an off-chain risk function (the Atlas and Stability Scope, not retrieved).
- **Move from English-style auctions (Liquidation 1.0) to Dutch auctions (2.0):** after the March 2020 zero-bid events, with instant settlement and flash-loan-compatible bidding. Quantstamp audited Liquidations 2.0 (link in [DefiLlama Sky Lending](https://api.llama.fi/protocol/sky-lending)).
- **Rebrand to Sky and USDS:** date and mechanics of the DAI to USDS upgrade path are [NOT FOUND] in the primary documents retrieved; DefiLlama now lists both DAI and USDS.

## Differentiators

- Scale: $3.76B TVL and $9.97B combined DAI and USDS supply on 2026-11-16.
- Longest operating history of any decentralised stablecoin; its main failure mode (Black Thursday) is the best-studied of any CDP.
- Revenue: DefiLlama shows $189.2M revenue over the last year.

## Limitations and criticisms

- **Black Thursday, 12 March 2020:** ETH fell roughly half over 12 to 13 March; the Medianizer oracle did not update under congestion and gas costs, and bidders won collateral in auctions for 0 DAI. Quantification conflicts across sources (see below). Debt auctions then minted MKR; 20,980 MKR were sold for 5.3M DAI and Paradigm won 68% of the 106 auctions (The Block). MKR holders voted not to compensate affected Vault owners (Decrypt, The Defiant).
- Centralisation of collateral: USDC and RWAs entered as collateral after 2020 (Cointelegraph notes USDC was added after Black Thursday).
- Third-party keeper hygiene: on 2026-10-06 a dormant third-party ETH-A liquidation keeper bot was drained of about $538k (about 200 WETH) via an unprotected function; this was not a protocol contract failure (DefiLlama hacks list description).
- Governance complexity and rebrand churn; token (MKR to SKY) migration risk: [NOT FOUND] for details.

### Fees and borrower cost

| Item | Value | Date | Source |
|------|-------|------|--------|
| Stability fee per Vault type | Set per ilk as `duty + base`; the current per-ilk percentages are [NOT FOUND] (Sky docs and daistats pages retrieved contained no rates; search snippets gave 5 to 8% on ETH vaults but from low-quality aggregators and are not used) | 2026-11-16 | [Sky rates overview](https://developers.skyeco.com/protocol/rates/ovreview/) |
| Liquidation penalty | `chop` per ilk, docs example 13%; current values [NOT FOUND] | n/a | [Sky collateral liquidation](https://developers.skyeco.com/protocol/vaults/collateral-liquidation/) |
| Keeper incentive | flat `tip` (docs example 100 DAI) plus `chip` (docs example 2% of the target debt) | n/a | same |
| Revenue booked | stability fees, liquidation income, PSM fees minus savings rate paid | 2026-11-16 | [DefiLlama fees methodology](https://api.llama.fi/overview/fees) |
| Fee run rate | $381.18M in fees and $189.20M in revenue over 1 year; $27.40M and $13.52M over 30 days | 2026-11-16 | [DefiLlama fees overview](https://api.llama.fi/overview/fees) |
| Fee destination | savings rate (DSR/SSR) to DAI/USDS depositors; remaining revenue to SKY buybacks and staking rewards, per DefiLlama methodology | 2026-11-16 | [DefiLlama fees overview](https://api.llama.fi/overview/fees) |
| DSR/SSR current rate | [NOT FOUND] | n/a | n/a |

Conflict on Black Thursday bad debt: Cointelegraph (author estimate, 2020-04-29) about 6.65M DAI undercollateralised with 1,461 near-zero auctions and 62,843 ETH; DefiLlama hacks list $8.32M; Decrypt cites 2.5M; Linum Labs 5.67M DAI; a Whiterabbit analysis cited by search results gives $8.325M for 1,462 zero-bid auctions. Not reconciled; the post-mortem on the official Maker blog is no longer served (redirects to sky.money). See [Cointelegraph](https://cointelegraph.com/news/maker-debt-crisis-post-mortem-recommends-new-safeguards).

## Sources

- [DefiLlama Sky Lending API](https://api.llama.fi/protocol/sky-lending) : accessed 2026-11-16
- [DefiLlama stablecoins list](https://stablecoins.llama.fi/stablecoins) : accessed 2026-11-16
- [DefiLlama fees overview](https://api.llama.fi/overview/fees) : accessed 2026-11-16
- [DefiLlama hacks list](https://api.llama.fi/hacks) : accessed 2026-11-16
- [Sky developer docs: collateral liquidation](https://developers.skyeco.com/protocol/vaults/collateral-liquidation/) : accessed 2026-11-16
- [Sky developer docs: rates overview](https://developers.skyeco.com/protocol/rates/ovreview/) : accessed 2026-11-16
- [Cointelegraph: Maker debt crisis post-mortem](https://cointelegraph.com/news/maker-debt-crisis-post-mortem-recommends-new-safeguards) : accessed 2026-11-16
- [The Block: MKR auctions conclude](https://www.theblock.co/post/60447/mkr-auctions-conclude-with-crypto-venture-firm-paradigm-emerging-as-the-biggest-winner) : accessed 2026-11-16 (search summary)
- [Decrypt: Maker votes not to compensate Black Thursday victims](https://decrypt.co/42992/maker-votes-not-compensate-black-thursday-victims) : accessed 2026-11-16 (search summary)
