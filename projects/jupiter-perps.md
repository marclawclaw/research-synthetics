---
tags: [project, solana, perpetuals, lp-pool]
ecosystem: Solana
category: Perpetuals
website: https://jup.ag/perps
docs: https://hub.jup.ag/guides/perpetual-exchange
launched: 2024
---

# Jupiter Perps

Jupiter Perps is the perpetual futures venue inside Jupiter, Solana's dominant DEX aggregator. It uses a multi-asset liquidity pool (the JLP pool: SOL, ETH, wBTC, USDC, and since mid-2026 JupUSD; USDT is listed with a zero target) as the counterparty to leveraged traders on its SOL, ETH and BTC markets. Pricing comes from an oracle stack with Edge by Chaos Labs as primary and Chainlink and Pyth as verification and fallback, so trade execution has no orderbook slippage. Since 22 September 2026 Jupiter also runs a separate, non-pool orderbook venue (GUM, beta) alongside JLP; see the Update 2026-11 section below.

## Update 2026-11

Snapshot refreshed 2026-11-16 (the May 2026 note was stale). What changed and why it matters:

- **Smaller pool, much lower activity.** JLP pool AUM is $683.5M (Jupiter API) and DefiLlama TVL is $572.9M, against a May 2026 note value of ~$1.3B (which DefiLlama never showed: its series was $677M on 2026-05-19 and $804M to $829M in March to April 2026). Q3 2026 perps volume was $16.7B (down 13% on Q2's $19.1B) and the protocol revenue share was $5.6M (down 18%); open interest is a small fraction of its 2025 level (DefiLlama $274M in January 2025 versus $28M on 2026-11-15). The "2025 volume $263B" figure in the old note could not be reproduced from any primary source in this refresh, and 2026 monthly volumes are far lower than its implied 2025 average of about $22B per month (Pine: $4.5B to $6.4B per month in Q2 and Q3 2026).
- **Drift exploit touched JLP.** On 2026-04-01 the Drift Protocol exploit drained assets that included JLP tokens (see [[projects/drift]]). DefiLlama annotates 2026-04-01 on its Jupiter Perps fees series as "$JLP burnt by drift trade exploiter", and Jupiter Perps TVL on DefiLlama fell from $828.6M (2026-04-01) to $660.1M (2026-04-02). No source reviewed reports a loss to JLP holders, an exploit of Jupiter's own programs, an oracle incident or an outage at Jupiter Perps in 2026 (negative finding: absence of reports, not proof). The quantity of JLP taken (secondary sources say about $155M to $159M) is not confirmed by a primary source here.
- **Competition on Solana has changed.** The "~80% of Solana perp volume" claim is no longer supported. Drift, the former number two, went offline from 2026-04-01 to 2026-09-29 and returned as a much smaller Velocity. Pacifica (orderbook), GMTrade (GMX on Solana), Phoenix and BULK are all taking flow; Pine reports Pacifica's September 2026 trader fees were about 45% of Jupiter's. See the share row in the metrics table.
- **New product alongside JLP.** On 2026-09-22 Jupiter launched GUM beta perps (HYPE, ZEC, JUP, SPCX, SNDK, SKHYNIX; later BNB and NEAR per Jupiter docs) with USDC collateral, an orderbook engine, Chainlink pricing per one report, and no JLP counterparty. Stock perps on SpaceX, SanDisk and SK hynix are the Solana-native synthetic equity angle; they are not backed by the JLP pool.
- **JLP composition and yield changed.** JupUSD became a JLP custody asset (June 2026); USDT now has a zero target and zero balance. JLP APY is 16.74% (APR 15.48%) on 2026-11-14, far below the 30% to 80% "historical range" in the old note. JLP supply fell 13% in Q3 (224M to 194M) with about $109M of net redemptions in August, and is 184.5M on 2026-11-14.
- **Claims corrected:** the oracle description ("three-oracle aggregation") is imprecise, JLP is no longer the "third largest pool" on Solana, and max leverage (250x on JLP markets) was missing. Details below.

## Adoption metrics

All values accessed 2026-11-16 unless stated. "Change since May 2026" shows the value that was in this note in May.

| Metric | Value | Date | Source | Change since May 2026 |
|--------|-------|------|--------|-----------------------|
| JLP pool AUM (Jupiter API) | $683.5M | 2026-11-14 (API timestamp) | [Jupiter perps API : jlp-info](https://perps-api.jup.ag/v1/jlp-info) | Was ~$1.3B (Apr 2026, Eco); about -47% |
| Jupiter Perps TVL (DefiLlama) | $572.9M (plus $119.3M listed as borrowed, not in TVL) | 2026-11-16 | [DefiLlama : Jupiter Perpetual Exchange](https://defillama.com/protocol/jupiter-perpetual-exchange) (data via [api.llama.fi](https://api.llama.fi/protocol/jupiter-perpetual-exchange)) | DefiLlama showed $677.4M on 2026-05-19; about -15% |
| JLP pool AUM at 2026-09-30 | $808.6M (DefiLlama) / $809M (Pine) | 2026-09-30 | [DefiLlama API](https://api.llama.fi/protocol/jupiter-perpetual-exchange); [Pine Analytics Q3 2026](https://pineanalytics.substack.com/p/jupiter-quarterly-report-q3-2026) | Q3 peak; fall since then driven by SOL price and redemptions |
| 24h volume (SOL + ETH + BTC markets) | $318.8M ($190.7M SOL, $75.3M ETH, $52.7M BTC) | 2026-11-16 | [Jupiter perps API : market-stats](https://perps-api.jup.ag/v1/market-stats?mint=So11111111111111111111111111111111111111112) (per mint; sum calculated here) | Old note: no current volume |
| 24h volume (cross-check) | $321.6M | 2026-11-16 03:33 UTC | [bykaranteli.com Solana perps tracker](https://bykaranteli.com/solana-perps) (third party, reads the official Jupiter API) | Agrees with API sum within 1% |
| 7d and 30d volume | [NOT FOUND] (DefiLlama perps volume is paywalled or blocked to this fetch; Jupiter exposes no aggregate endpoint found) | | | |
| Monthly volume | Sep 2026 $6.3B; Q3 $16.7B (Jul $4.5B); Q2 $19.1B (about $6.4B per month) | 2026 Q2 to Q3 | [Pine Analytics Q3 2026](https://pineanalytics.substack.com/p/jupiter-quarterly-report-q3-2026) | Q3 down 13% on Q2 |
| 2025 trading volume | $263B (not re-verified; see note) | 2025 | [Yellow.com Jupiter research](https://yellow.com/research/jupiter-solana-dex-aggregator-market-cap-2026) | Unchanged; unverified (aggregator source; not reproduced in any primary source) |
| Fees (all fees, LP plus protocol) | 24h $599,470; 7d $3.85M; 30d $10.0M; 1y $146.3M | 2026-11-16 | [DefiLlama fees API](https://api.llama.fi/summary/fees/jupiter-perpetual-exchange) | New |
| Protocol revenue (Jupiter's 25% share) | Q3 2026 $5.6M (Q3 2025: $34.8M) | Q3 2026 | [Pine Analytics Q3 2026](https://pineanalytics.substack.com/p/jupiter-quarterly-report-q3-2026) | Was "~$45M quarterly (2025)"; Pine's Q3 2025 figure is $34.8M, so the old figure looks overstated |
| Open interest | $28.0M (DefiLlama); $31.4M one-sided, about $62.8M long plus short (third-party tracker); ~$106M at 2026-09-30 (Pine) | 2026-11-15 / 2026-11-16 / 2026-09-30 | [DefiLlama OI API](https://api.llama.fi/summary/open-interest/jupiter-perpetual-exchange); [bykaranteli.com](https://bykaranteli.com/solana-perps); [Pine](https://pineanalytics.substack.com/p/jupiter-quarterly-report-q3-2026) | Sources conflict (definitions differ); DefiLlama OI was $46.8M on 2026-05-19 and $274M on 2025-01-15 |
| Monthly active traders | ~67,000 (Q3 2026), from ~83,000 (Q2) | Q3 2026 | [Pine Analytics Q3 2026](https://pineanalytics.substack.com/p/jupiter-quarterly-report-q3-2026) | New |
| JLP APY / APR | 16.74% / 15.48% | 2026-11-14 | [Jupiter perps API : jlp-info](https://perps-api.jup.ag/v1/jlp-info) | Was "30-80% APY historical range"; now at the bottom of or below that range |
| JLP supply and price | 184.45M JLP at $3.7055 | 2026-11-14 | [Jupiter perps API : jlp-info](https://perps-api.jup.ag/v1/jlp-info) | Pine: supply 224M to 194M in Q3; price $3.51 to $4.85 in Q3 (SOL rally, since reversed) |
| Share of Solana perp volume | ~80% no longer supported; Jupiter is about 47% of 24h volume across five venues in one tracker (Jupiter $321.6M, Pacifica $238.2M, GM Trade $91.3M, Phoenix $25.1M, Velocity $0.2M; share calculated here, excludes venues not listed) | 2026-11-16 | [bykaranteli.com](https://bykaranteli.com/solana-perps) | Was ~80% (Yellow.com, 2026); a DefiLlama Solana snippet for 2026-11-04 (search extract, unverified) listed GMX Solana at $2.117B 24h volume, above Jupiter at $386M, which conflicts with the tracker above |
| Share of Solana DEX aggregator volume | ~95% (not re-verified) | 2026 | [Yellow.com Jupiter research](https://yellow.com/research/jupiter-solana-dex-aggregator-market-cap-2026) | Not refreshed |
| JLP TVL ranking on Solana | Not third. On DefiLlama's protocol-level list, Jupiter Perps ($573M) sits behind Sanctum ($1.34B), Kamino Lend ($1.13B), Jupiter Lend ($922M), Raydium ($832M), Binance Staked SOL ($748M) and Jito Liquid Staking ($724M), excluding CEX wallets | 2026-11-16 | [DefiLlama protocols API](https://api.llama.fi/protocols) (ranking calculated here) | Was "3rd largest pool (behind Jito, Kamino)" |

## How it works

### User perspective

A trader picks a JLP market (SOL/USD, ETH/USD, wBTC/USD) and opens a long or short with up to 250x leverage (Jupiter docs; minimum $10 collateral). Longs post SOL, wETH or wBTC; shorts post USDC. The execution price is the oracle price. No orderbook means large orders have no orderbook price impact; the cost of size shows up via borrow fees, a 0.06% base open/close fee and a size-dependent price impact fee. Profits are paid from the JLP pool; losses are added to it.

LPs mint JLP by depositing any basket asset. LPs are entitled to 75% of platform fees (open, close, borrow, swap, liquidation, price impact, mint/burn) and earn yield denominated in the basket assets themselves, not a separate reward token. DefiLlama's methodology page and Jupiter's docs both state the 75% LP share and 25% to Jupiter; the 75% claim in the May note remains true.

Since 2026-09-22 a second class of market exists: GUM beta markets (HYPE, ZEC, JUP, BNB, NEAR crypto perps plus SPCX, SNDK, SKHYNIX stock perps) with USDC collateral, hourly funding, maker/taker fees and risk-tiered leverage. They do not use JLP.

### Protocol perspective

- **JLP pool:** multi-asset index pool that backs every JLP-market trade. Composition on 2026-11-14 (target in brackets): SOL 47.2% (47%), USDC 29.9% (30%), wBTC 12.9% (13%), ETH 7.7% (8%), JupUSD 2.2% (2%), USDT 0% (0%); AUM limit $3B. Pool composition matters: when traders are net long, the pool's exposure to BTC/ETH/SOL goes down; net short positions raise pool exposure. Source: [Jupiter perps API](https://perps-api.jup.ag/v1/jlp-info).
- **Oracle stack:** Edge by Chaos Labs is the primary oracle; Chainlink and Pyth are used for verification and fallback. Edge's price is used if it is not stale and is within threshold of both Chainlink and Pyth; otherwise Chainlink and Pyth are compared and the more recent is used if they agree within threshold; if two or more oracles fail no price update occurs. This is primary-with-fallback, not an equal three-way aggregation. A RedStone feed was added for JupUSD (reported June 2026, secondary source). Source: [Jupiter docs : technical reference](https://docs.jup.ag/user-docs/trade/perps/technical-reference).
- **Liquidation rule:** liquidation when effective collateral (after fees) falls to the minimum margin, which the docs formula ties to a 500x maximum leverage (size/500, or 0.2% of position size; the 0.2% is inferred from the formula rather than stated outright). Source: [Jupiter docs : liquidation](https://docs.jup.ag/user-docs/trade/perps/liquidation).
- **Fee model:** 0.06% base open/close fee, linear and additive price impact fees, hourly borrow fee from a jump-rate curve, swap fees weighted by pool share, and a 0.20% base JLP mint/burn fee. DefiLlama also counts SOL staking rewards on pool SOL and, from 2025-12-15, interest from "JLP Loans" as fee items (the JLP Loans programme was not otherwise researched here).

## Key behaviours

- [[patterns/lp-counterparty-risk]] : JLP holders are the counterparty to all traders on the JLP markets.
- [[patterns/oracle-pricing-model]] : Edge-primary oracle with Chainlink/Pyth verification and fallback.
- [[patterns/no-slippage-large-orders]] : oracle pricing means trade size does not move execution price (JLP markets only; GUM markets are orderbook-based).

## Architecture decisions

- **Multi-asset LP pool (not single-side):** depositors get diversified exposure to SOL/ETH/wBTC/USDC/JupUSD, lowering single-asset directional risk.
- **75% fee share to LPs:** high payout to keep the pool well-capitalised; protocol revenue comes from the remaining 25%.
- **Primary oracle with two fallbacks:** redundancy and robustness over a single dominant feed.
- **Tight integration with Jupiter aggregator:** users can swap on Jupiter and open positions in the same UI; routing keeps friction low.
- **Parallel orderbook engine (GUM):** new long-tail and equity perps are listed on a separate engine rather than expanding the JLP basket, which keeps JLP's counterparty risk confined to SOL/ETH/BTC.

## Differentiators

- Largest single perpetual venue on Solana by 24h volume in the November 2026 tracker, but no longer the near-monopoly the May note described.
- Multi-asset LP pool (versus GMX V2's GM single-market pools, or Synthetix's debt pool).
- Very high leverage ceiling (250x) on the three core markets.

## Limitations and criticisms

- LP directional risk: JLP holders take the inverse of net trader positioning. During strong rallies with mostly long traders, JLP NAV can lag. Pine notes the Q3 rise in pool assets (+15%) was driven by SOL's price (+59%), not new deposits.
- Redemption pressure: about $109M of net JLP redemptions in August 2026 (Pine); whale transfers to exchanges in early June ran at 14.7 times the normal pace (single aggregator source, unverified).
- Oracle dependency: even with three feeds, a coordinated oracle failure or manipulation across providers would be catastrophic.
- Concentration risk: JLP is a large Solana DeFi position, and was used as collateral on other venues (it was among the assets drained from Drift on 2026-04-01 and is a borrowable collateral on Jupiter Lend), so contagion paths exist.
- Pool composition slippage: when traders flip net direction sharply, JLP composition shifts and LP exposure changes faster than they can rebalance.
- Declining volume and traders (Q3 2026) while a competing orderbook venue (Pacifica) gained share.
- Jupiter's own new GUM markets may divert long-tail flow away from the JLP pool.

## Sources

Accessed 2026-11-16 unless stated.

- [Jupiter perps API : jlp-info](https://perps-api.jup.ag/v1/jlp-info) : accessed 2026-11-16 (primary; AUM, APY, composition)
- [Jupiter perps API : market-stats (SOL mint)](https://perps-api.jup.ag/v1/market-stats?mint=So11111111111111111111111111111111111111112) : accessed 2026-11-16 (also ETH and wBTC mints)
- [Jupiter docs : Perps overview](https://docs.jup.ag/user-docs/trade/perps) : accessed 2026-11-16
- [Jupiter docs : Perps technical reference](https://docs.jup.ag/user-docs/trade/perps/technical-reference) : accessed 2026-11-16
- [Jupiter docs : Perps liquidation](https://docs.jup.ag/user-docs/trade/perps/liquidation) : accessed 2026-11-16
- [DefiLlama : Jupiter Perpetual Exchange](https://defillama.com/protocol/jupiter-perpetual-exchange) : page returned 403 to fetch; figures read from the DefiLlama public API (api.llama.fi protocol, fees and open-interest endpoints) : accessed 2026-11-16
- [Pine Analytics : Jupiter Quarterly Report Q3 2026](https://pineanalytics.substack.com/p/jupiter-quarterly-report-q3-2026) : published 2026-10-16, accessed 2026-11-16
- [Solana Compass : Jupiter Perps adds six markets via GUM](https://solanacompass.com/news/jupiter-perps-adds-six-markets-including-tokenized-spacex-hype-and-zec-via-gum-orderbook) : published 2026-09-22, accessed 2026-11-16
- [bykaranteli.com : Solana perps tracker](https://bykaranteli.com/solana-perps) : third party, reads venue APIs; accessed 2026-11-16
- [CryptoBriefing : JupUSD added as JLP custody asset](https://cryptobriefing.com/jupiter-jupusd-jlp-custody-asset/) : secondary, from search result only; accessed 2026-11-16
- [Jupiter docs : How Jupiter Perps work](https://hub.jup.ag/guides/perpetual-exchange/how-it-works) : accessed 2026-05-19 (now redirects to developers.jup.ag)
- [Jupiter support : price oracles](https://support.jup.ag/hc/en-us/articles/19355326396700-Price-Oracles) : accessed 2026-05-19 (the 2026-11-16 fetch of the equivalent article returned no oracle content)
- [Jupiter support : fees](https://support.jup.ag/hc/en-us/articles/18735045234588-Fees) : accessed 2026-05-19
- [Eco support : JLP fees, leverage, how it works](https://eco.com/support/en/articles/15083164-jupiter-perps-fees-leverage-how-jlp-works) : accessed 2026-05-19 (superseded for TVL and APY)
- [Yellow.com : Jupiter Solana DEX aggregator 2026](https://yellow.com/research/jupiter-solana-dex-aggregator-market-cap-2026) : accessed 2026-05-19 (aggregator; 2025 volume, revenue and share claims not reproduced elsewhere)
- [CoinMarketCap : JLP token](https://coinmarketcap.com/currencies/jupiter-perps-lp/) : accessed 2026-05-19
