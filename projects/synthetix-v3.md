---
tags: [project, ethereum, base, synthetics, perpetuals]
ecosystem: Ethereum (V3 deployments on Base, Arbitrum and Optimism wound down)
category: Synthetics/Perpetuals
website: https://synthetix.io
docs: https://docs.synthetix.io
launched: 2018 (v1), 2023 (v3)
---

# Synthetix V3

Synthetix is the original on-chain synthetic asset protocol. V1/V2 ran a debt-pool model where SNX stakers minted Synths (sUSD, sBTC, sETH) collateralised by SNX. V3, deployed across Ethereum, Base and Arbitrum, generalised the architecture into a "liquidity layer" that supports perpetuals, spot synths and arbitrary market types, with pluggable oracles and pool-specific collateral.

> **Status as of 2026-11-16 (read the Update section first):** the V3 liquidity layer is no longer an active venue. DefiLlama shows V3 TVL of about $0 since 2026-06-26, following the retirement of the 420 staking pool and legacy sUSD under SIP-423. The only live Synthetix product is a separate order-book style perps exchange on Ethereum mainnet (listed by DefiLlama as "Synthetix V4"), which is small. The May 2026 version of this note described V3 as a Base-focused growth venture; that was already out of date (Base and Arbitrum were dropped in 2025).

## Update 2026-11

**What changed since May 2026**

- **V3 TVL went to zero.** DefiLlama V3 TVL fell from $29.10M on 2026-06-25 to $0 on 2026-06-26 and has stayed at about $0 since (the $0.42 residual sits on Base). It had been $41.19M in May 2026 and about $141M on 2025-10-01 (DefiLlama series). The cause is governance action, not an exploit: see the next bullets.
- **420 Pool closed (2026-06-19).** Per the Synthetix migration docs, the 420 staking system closed on 2026-06-19 and the Debt Jubilee ended the same day. Stakers without debt had staked sUSD returned on 2026-06-19 and staked SNX returned on 2026-06-26. Stakers with debt stay under a 4-year lock from 2026-06-26 followed by 1-year vesting from 2030-06-26, or can exit early by repaying remaining debt (repayable in USDT). The earlier claim that the 420 Pool "lets users stake SNX without managing C-ratio or debt" is now historical.
- **sUSD retired (SIP-423, created 2026-06-12; snapshot 2026-06-26).** Legacy sUSD is suspended and deprecated on Ethereum and Optimism. Holders receive an entitlement of 4 SNX per 1 sUSD (valuing sUSD at $1 and SNX at $0.25), with a maximum mint of 236,464,356 SNX (implied 59,116,089 sUSD entitlement). Lock runs 2026-06-26 to 2027-06-26, claims open on 2027-06-26 and expire worthless after 6 months if unclaimed. sUSD sitting in LP pools, vaults or other contracts is not recovered automatically. If protocol revenue reaches $10M within the lock period, 25% may fund redemption at 1 USDT per sUSD for holders who prefer USDT. At the SNX price of about $0.149 on 2026-11-16 (DefiLlama coins API), 4 SNX is worth roughly $0.60, so the compensation is about 60% of face value (derived calculation, not a sourced figure).
- **SNX staking reform deferred.** SIP-423 marks Phases 3 and 4 (contract builds, staking reform) as deferred and no replacement staking contract is live. SIP-424 (created 2026-08-19, status Vote_Pending on the SIP page) would let all SNX holders, not only stakers, vote in Spartan Council elections, counting unclaimed sUSD entitlements at 4 SNX each.
- **L2 retreat finished earlier than the May note implied.** The 2025-08-16 Synthetix blog says the team is "done with L2s": Arbitrum and Base are already gone, Optimism perps went close-only on 2025-08-18, positions were force-closed on 2025-08-25 and everything ended on 2025-08-31. SIP-416 (created 2025-05-28) covers permanent Arbitrum V3 deprecation. The 2026 roadmap (2026-03-13) describes a single Ethereum mainnet CLOB perp DEX replacing multi-chain AMMs.
- **New exchange is live but small.** The Q2 roadmap update (2026-05-01) says Synthetix Perps on Ethereum mainnet was in invite-only alpha with 10 crypto perps (BTC, ETH, SOL, ZEC, XRP, DOGE, XMR, PEPE, FARTCOIN, PUMP) and reported 4 months of trading with no incidents that lost traders money. ETH multi-collateral margin went live 2026-06-03 and referrals on 2026-06-17 (sidebar dates on the blog). As of 2026-11-16 the synthetix.io homepage shows 24h volume of $2.11M and open interest of $2.68M for the ETH-USDT market only.
- **Buybacks.** The 2026 roadmap set a 50/50 split of exchange revenue between SNX and sUSD buybacks, moving to SNX only once the peg was restored; sUSD has since been retired rather than restored. The homepage states "Buybacks with Revenue". DefiLlama records only $24,716 of all-time fees for the new exchange, so buyback capacity is negligible.
- **Peripheral incident.** On 2026-10-29 the protocol DAO moved 8,405,741 SNX from a wallet linked to a large exchange hot-wallet theft into its Safe, to be returned on request (secondary news reports; the dollar value is disputed across outlets). Not a Synthetix protocol exploit.

**Why it matters:** the "debt-pool / SNX-collateralised synthetic" model, and the V3 multi-collateral pool model, are no longer operating as a venue. The surviving Synthetix business is an Ethereum mainnet perp exchange whose design differs from V3 (order-book style rather than oracle-priced debt pool, per the roadmap). Any pattern note that treats Synthetix V3 as a live example of staker-debt LP counterparty risk should describe it as historical.

## Adoption metrics

| Metric | Value | Date | Source | Change since May 2026 (May value) |
|--------|-------|------|--------|-----------------------------------|
| TVL (V3) | $0.42 (Ethereum $0, Arbitrum $0, Base $0.42), effectively zero | 2026-11-16 | [DefiLlama API, synthetix-v3](https://api.llama.fi/protocol/synthetix-v3) | Down from $41.19M (May 2026); last non-zero reading $29.10M on 2026-06-25 |
| TVL (Synthetix parent, now Ethereum exchange only) | $0.887M | 2026-11-16 | [DefiLlama API, synthetix](https://api.llama.fi/protocol/synthetix) and [synthetix-v4](https://api.llama.fi/protocol/synthetix-v4) | n/a (V3 was $41M of the parent total in May) |
| Active deployment chains | Ethereum mainnet only (exchange); V3 on Base and Arbitrum drained | 2026-11-16 | [Synthetix blog, 2025-08-16](https://blog.synthetix.io/mainnet-is-where-the-heart-is/); [DefiLlama API](https://api.llama.fi/protocol/synthetix-v3) | Corrects May row "Ethereum, Base, Arbitrum (winding down)" |
| V3 24h / 7d / 30d fees and open interest | No data (DefiLlama adapters return null) | 2026-11-16 | [DefiLlama fees API](https://api.llama.fi/summary/fees/synthetix-v3) | 24h perp volume was $0 in May |
| V3 cumulative fees | $2,519,644 | 2026-11-16 | [DefiLlama fees API](https://api.llama.fi/summary/fees/synthetix-v3) | Not captured in May |
| V3 cumulative perp volume | [NOT FOUND] (DefiLlama volume endpoints are paywalled and the protocol page returned HTTP 403 on 2026-11-16) | 2026-11-16 | n/a | May value $6.962B; no V3 volume since wind-down, so unlikely to have changed materially |
| Exchange (Synthetix V4 listing) fees, 24h / 30d / all-time | $136 / $3,718 / $24,716 | 2026-11-16 | [DefiLlama fees API, synthetix-v4](https://api.llama.fi/summary/fees/synthetix-v4) | New since May |
| Exchange open interest (DefiLlama) | $296,705 (latest daily value) | 2026-11-15 | [DefiLlama OI API, synthetix-v4](https://api.llama.fi/summary/open-interest/synthetix-v4) | New since May; conflicts with the homepage figure below |
| Exchange 24h volume and OI, ETH-USDT market only | $2,113,728 volume; $2,679,751 open interest | 2026-11-16 | [synthetix.io homepage](https://synthetix.io/) | New since May; protocol-wide totals [NOT FOUND] |
| SNX price | $0.149 | 2026-11-16 | [DefiLlama coins API](https://coins.llama.fi/prices/current/ethereum:0xC011a73ee8576Fb46F5E1c5751cA3B9Fe0af2a6F) | [NOT FOUND] for May |
| sUSD circulating supply | $69.84M (DefiLlama stablecoins; may be stale given the contract is suspended) | 2026-11-16 | [DefiLlama stablecoins API](https://stablecoins.llama.fi/stablecoins?includePrices=true) | Conflicts: SIP-423 implies a 59.1M sUSD entitlement; one news report says about 40M |
| Daily average trading volume (CMC summary) | ~$42M | 2026 | [CoinMarketCap Synthetix](https://coinmarketcap.com/cmc-ai/synthetix/price-prediction/) | Stale, from May note; not re-verified and inconsistent with the volumes above. Treat as unreliable |
| Cumulative perp volume (CMC summary, all versions) | $62.72B | 2026 | [CoinMarketCap Synthetix](https://coinmarketcap.com/cmc-ai/synthetix/price-prediction/) | Stale, from May note; not re-verified |

Note: the DefiLlama protocol web pages returned HTTP 403 to automated fetches on 2026-11-16, so the figures above come from DefiLlama's public API endpoints that back those pages.

## How it works

The sections below describe the V3 design as it existed. They are historical: the V3 deployments are drained and the live exchange has a different architecture.

### User perspective

A trader deposits accepted collateral (USDC, sUSD, ETH derivatives depending on market) into a Synthetix V3 perp account, then opens a leveraged position on BTC, ETH or supported altcoins. Pricing is fetched from a configured oracle aggregator (Chainlink Data Streams plus Pyth pull oracles) at execution time. Settlement is two-step to mitigate front-running: commit, then settle after an oracle update window.

LPs deposit collateral into specific debt pools, which in turn back specific markets. LP returns come from a share of trading fees and from absorbing trader PnL.

### Protocol perspective

V3 separates the protocol into three layers:

- **Core (Liquidity Layer):** account registry, collateral management, pool accounting, market registry.
- **Markets:** independent modules (perps, spot synths, options via Derive) that pull liquidity credit from one or more pools.
- **Oracles:** Oracle Manager allows market creators to assemble price feeds (e.g. min of Chainlink, Pyth and a Uniswap TWAP).

The differentiated debt pools let stakers pick risk exposure (which markets back which pool) instead of socialising all market risk across one global SNX pool.

### Current exchange (Ethereum mainnet)

Per the roadmap and Q2 update, the live product is an Ethereum mainnet perp exchange described as a CLOB, with cross-margin, TP/SL orders, REST and WebSocket APIs, a Python SDK, multi-collateral margin (ETH first, then cbBTC and a wider basket planned) and an SLP market-making vault (private, about 20% APY in the 2026-05-01 update, public launch targeted Q2 with a goal of at least $15M of sUSD by end of Q3; given sUSD retirement the status of this target is [NOT FOUND]). Its oracle design is [NOT FOUND] in the sources reviewed, so the Chainlink Data Streams plus Pyth description above must not be assumed to carry over.

## Key behaviours

- [[patterns/synthetic-collateral-models]] : V3 moved from SNX-only collateral to multi-collateral pools. As of 2026-11 the SNX-staked debt model has been retired (legacy sUSD and 420 Pool closed).
- [[patterns/oracle-pricing-model]] : V3 used an oracle-aggregated mark price with on-chain commit-reveal settlement; the current exchange's oracle design is [NOT FOUND].
- [[patterns/lp-counterparty-risk]] : in V3, LPs were the counterparty and debt grew when traders won. The current exchange's SLP vault plays a market-making role.

## Architecture decisions

- **Liquidity Layer abstraction:** V3 is a primitive for any synthetic market, not just Synths. Markets plug in.
- **Retreat from L2s to Ethereum mainnet:** Optimism perps were shut down in August 2025 and Arbitrum and Base V3 deployments were dropped; the 2026 roadmap consolidates on a single Ethereum mainnet exchange. (Earlier note text said Base was the growth target and the pivot happened in December 2025; the mainnet perps launch is reported as 2025-12-19 by a CoinMarketCap summary, secondary source.)
- **420 Pool simplified staking (closed):** introduced via SIP-420, closed 2026-06-19 under SIP-423.
- **Fee buyback:** 2026 roadmap committed exchange revenue to SNX and sUSD buybacks (50/50, then SNX only after peg restoration). Revenue scale is very small (see metrics).
- **SnaXchain governance L2:** Optimism Superchain rollup launched late Q3 2024 as the home of Synthetix governance. Its current status was not re-verified; SIP-423 and SIP-424 are described as Ethereum and Optimism proposals.

## Differentiators

- Pluggable per-market oracle aggregation (Chainlink + Pyth + TWAP min), for V3 as designed.
- Debt pool model: LP credit, not orderbook (historical).
- Native support for spot synths (sUSD, sBTC, sETH) alongside perps (sUSD now retired).
- 7+ years of production iteration on synthetic asset economics.

## Limitations and criticisms

- TVL has collapsed: V3 TVL went from about $141M (2025-10-01) to $41M (May 2026) to about $0 (2026-06-26 onwards), against a historical Synthetix peak of ~$3B+ across V1/V2 during 2021.
- Debt pool model is conceptually complex: stakers' debt changes with aggregate trader PnL, which historically deterred passive LPs.
- sUSD depegged for an extended period and was ultimately retired, with holders compensated in vested SNX at a ratio worth roughly 60% of face at the 2026-11-16 SNX price (derived). The founder's public thread attributed the depeg to treasury mismanagement and said a replacement basis-vault-backed stablecoin would run on the new exchange (The Defiant, undated).
- Strategic flux (Optimism to Arbitrum to Base to Ethereum mainnet only) created user friction and fragmented liquidity, and the current exchange has very low volume and fees.

## Sources

- [DefiLlama Synthetix V3 protocol page](https://defillama.com/protocol/synthetix-v3) : accessed 2026-05-19 (HTTP 403 to automated fetch on 2026-11-16)
- [DefiLlama API, synthetix-v3](https://api.llama.fi/protocol/synthetix-v3) : accessed 2026-11-16
- [DefiLlama API, synthetix (parent)](https://api.llama.fi/protocol/synthetix) : accessed 2026-11-16
- [DefiLlama API, synthetix-v4](https://api.llama.fi/protocol/synthetix-v4) : accessed 2026-11-16
- [DefiLlama fees API, synthetix-v3](https://api.llama.fi/summary/fees/synthetix-v3) : accessed 2026-11-16
- [DefiLlama fees API, synthetix-v4](https://api.llama.fi/summary/fees/synthetix-v4) : accessed 2026-11-16
- [DefiLlama OI API, synthetix-v4](https://api.llama.fi/summary/open-interest/synthetix-v4) : accessed 2026-11-16
- [DefiLlama stablecoins API](https://stablecoins.llama.fi/stablecoins?includePrices=true) : accessed 2026-11-16
- [DefiLlama coins API (SNX price)](https://coins.llama.fi/prices/current/ethereum:0xC011a73ee8576Fb46F5E1c5751cA3B9Fe0af2a6F) : accessed 2026-11-16
- [DefiLlama Synthetix v1+v2 page](https://defillama.com/protocol/synthetix-v1+v2) : accessed 2026-05-19
- [SIP-423: sUSD Retirement and Staking Reform](https://sips.synthetix.io/sips/sip-423/) : accessed 2026-11-16
- [SIP-424: Unstaked SNX voting for governance elections](https://sips.synthetix.io/sips/sip-424) : accessed 2026-11-16
- [SIP-416: Arbitrum V3 Deprecation](https://sips.synthetix.io/sips/sip-416) : accessed 2026-11-16
- [Synthetix docs : 420 Legacy Staking](https://docs.synthetix.io/legacy-migration/420-legacy-staking) : accessed 2026-11-16
- [Synthetix blog : Mainnet is where the heart is (2025-08-16)](https://blog.synthetix.io/mainnet-is-where-the-heart-is/) : accessed 2026-11-16
- [Synthetix blog : 2026 Roadmap (2026-03-13)](https://blog.synthetix.io/2026-roadmap/) : accessed 2026-11-16
- [Synthetix blog : Q2 Roadmap Update (2026-05-01)](https://blog.synthetix.io/2026-roadmap-update-q2/) : accessed 2026-11-16
- [Synthetix homepage](https://synthetix.io/) : accessed 2026-11-16
- [The Defiant : Warwick personal blame, sUSD mismanagement, basis vault replacement](https://thedefiant.io/news/defi/warwick-personal-blame-susd-mismanagement-basis-vault-replacement) : accessed 2026-11-16
- [Crypto Briefing : Synthetix retires sUSD (2026-06-22)](https://cryptobriefing.com/synthetix-retires-susd-snx-conversion/) : accessed 2026-11-16
- [Value the Markets : Synthetix retires legacy stablecoin (2026-06-23)](https://www.valuethemarkets.com/cryptocurrency/news/synthetix-retires-legacy-stablecoin-and-introduces-snx-compensation) : accessed 2026-11-16
- [CoinMarketCap CMC AI : Synthetix latest updates](https://coinmarketcap.com/cmc-ai/synthetix/latest-updates/) : accessed 2026-11-16 (previously 2026-05-19)
- [Odaily : SNX recovered from exchange hot-wallet exploiter (2026-10)](https://www.odaily.news/en/newsflash/529181) : listed in search results on 2026-11-16, not opened
- [The Defiant : Synthetix to pivot V3 perps from Arbitrum to Base](https://thedefiant.io/news/defi/synthetix-to-pivot-v3-perps-from-arbitrum-to-base) : accessed 2026-05-19 (superseded by the 2025-08-16 post)
- [Synthetix V3 blog overview](https://blog.synthetix.io/what-is-synthetix-v3/) : accessed 2026-05-19
- [Synthetix V3 oracle docs](https://docs.synthetix.io/synthetix-protocol/the-synthetix-protocol/oracles) : accessed 2026-05-19
- [Zipmex : Synthetix 2026 guide](https://zipmex.com/blog/synthetix-snx-guide-2026/) : accessed 2026-05-19
