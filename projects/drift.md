---
tags: [project, solana, perpetuals, hybrid-liquidity]
ecosystem: Solana
category: Perpetuals
website: https://drift.trade
docs: https://docs.drift.trade
launched: 2021 (v1), 2022 (v2), 2023+ (v3)
---

# Drift Protocol

Drift was Solana's most sophisticated decentralised perpetuals exchange, distinguished by a three-tier hybrid liquidity stack: a just-in-time (JIT) auction for market-maker fills, a fully on-chain limit orderbook (DLOB), and an oracle-pegged virtual AMM (DAMM/BAL) as a fallback. It also bundled spot trading, lending pools and unified cross-margining behind a single user account.

**Status as of 2026-11-16:** the original Drift deployment was drained of about $280M to $285M on 2026-04-01 (not a smart contract bug; an admin takeover via the Security Council multisig), was frozen until September, and was replaced by Velocity, a rebuilt fork of Drift v2 that opened to all traders on 2026-09-29. Everything below that describes the three-tier stack and unified margin describes the pre-hack Drift and, where stated, Velocity's current design. The May 2026 version of this note did not mention the incident, which pre-dated it by seven weeks.

## Update 2026-11

What changed and why it matters:

- **Major incident, 2026-04-01.** About $285M (Drift's early figure about $280M; Drift Foundation later puts total user losses at about $295.4M) was drained in 31 withdrawals over roughly 12 minutes. The attacker took over Drift's Security Council admin powers, listed a worthless self-minted token (CarbonVote, CVT) as collateral at an inflated oracle price, removed withdrawal limits and withdrew real assets (USDC, JLP, SOL, cbBTC, wBTC, USDT, wETH, dSOL, JTO and others). Drift states no smart contract vulnerability and no seed phrase compromise was involved. It matters because a sophisticated liquidity stack says nothing about admin-key governance, which was the actual failure point; see [[patterns/oracle-pricing-model]] for the oracle-listing dimension (an attacker-controlled price feed for a newly listed asset).
- **Collapse and relaunch.** DefiLlama's Drift Trade TVL went from $311.4M (2026-04-01) to $23.5M (2026-04-02) and about $0.64M by 2026-06-30. Drift was offline from the exploit until Velocity relaunched on 2026-09-29. Velocity is small: open interest about $193K and 24h volume about $232K on 2026-11-16.
- **Ownership and structure.** Co-founder Cindy Leow stepped down on relaunch day (2026-09-29); a five-person team now leads (David Lu continues recovery work). Tether committed up to $127.5M (announced 2026-04-16, conditions and drawn amount not confirmed) and the settlement stablecoin moved from USDC to USDT; Velocity says it is "Backed by Tether".
- **Compensation.** A transferable recovery token (DFX, 299.5M supply, 1 DFX per 1 USDT of verified loss) opened for claims on 2026-10-01. Per one secondary source citing the Foundation, redemption started near 0.0104 USDT per DFX against a pool of about 3.11M USDT, funded by a 60% to 90% share of daily Velocity net revenue, Tether, partners (up to $20M) and recovered funds.
- **Attribution.** Drift and SEAL 911 assess with medium-high confidence that UNC4736 (AppleJeus, Citrine Sleet; linked to the October 2024 Radiant Capital theft) was responsible after a roughly six-month social-engineering operation; Mandiant had not formally attributed it when last reported. This is preliminary.
- **Claims corrected:** the open interest "~$400M" (April 2026) is not supported by DefiLlama, which showed $62M to $83M in Feb to Mar 2026; the "10x leverage cap" is not in Drift's docs (leverage is set per market, bounded 80x to 1x by the program; Velocity lists 20x on BTC/ETH/SOL and 10x on HYPE); the "dominant Solana perps exchange" framing no longer holds.

### Incident timeline (Drift, 2026)

| Date | Event | Source |
|------|-------|--------|
| 2026-03-23 | Four durable nonce accounts created on-chain (two linked to Security Council members, two to the attacker) | [BlockSec](https://blocksec.com/blog/drift-protocol-incident-multisig-governance-compromise-via-durable-nonce-exploitation) |
| 2026-03-27 | Security Council migrated to a 2-of-5 threshold with zero timelock | [TRM Labs](https://www.trmlabs.com/resources/blog/north-korean-hackers-attack-drift-protocol-in-285-million-heist) |
| 2026-04-01 (~16:05 UTC) | Admin takeover, CVT listed, limits removed, 31 withdrawals in ~12 minutes; BlockSec totals $285,279,417.69 | [BlockSec](https://blocksec.com/blog/drift-protocol-incident-multisig-governance-compromise-via-durable-nonce-exploitation) |
| 2026-04-02 | Drift confirms ~$280M loss and freezes remaining functions; ZachXBT reports over $230M USDC bridged to Ethereum via Circle CCTP | [The Block](https://www.theblock.co/post/396183/drift-280m-exploit-zachxbt-circle) |
| 2026-04-05 | Drift post-mortem: six-month operation by people posing as a quant firm; no contract bug; medium-high confidence DPRK-aligned | [The Block](https://www.theblock.co/post/396361/drift-links-280-million-exploit-to-six-month-social-engineering-op-run-by-suspected-north-korean-actors); [The Record](https://therecord.media/drift-crypto-theft-post-mortem-north-korea) |
| 2026-04-16 | Recovery package announced: up to $127.5M from Tether (proposed), ~$20M from other partners; USDC to USDT switch | [The Block](https://www.theblock.co/post/397735/drift-127-million-tether-recovery-pivots-circle-usdc-usdt-280-million-exploit) |
| 2026-05-05 | Recovery plan for affected users (recovery tokens, lean relaunch) | [DL News](https://dlnews.com/articles/defi/drift-to-issue-recovery-tokens-in-wake-of-295m-hack) |
| 2026-07-01 | Rebrand to Velocity announced (single secondary source) | [Cryptopolitan](https://www.cryptopolitan.com/drift-protocol-relaunches-as-velocity-dex/) |
| 2026-09-17 | OtterSec audit published and source code opened | [Velocity blog](https://velocity.exchange/blog) |
| 2026-09-21 | Beta whitelist removed | [Solana Compass](https://solanacompass.com/news/drift-co-founder-cindy-leow-steps-down-hands-rebuilt-velocity-exchange-to-new-team) |
| 2026-09-29 | Velocity opens to all traders; Cindy Leow steps down; DefiLlama lists Velocity the same day | [Solana Compass](https://solanacompass.com/news/drift-co-founder-cindy-leow-steps-down-hands-rebuilt-velocity-exchange-to-new-team); [DefiLlama API](https://api.llama.fi/protocol/velocity) |
| 2026-10-01 | DFX claims and redemptions open | [Solana Compass](https://solanacompass.com/news/drift-opens-dfx-claims-for-april-exploit-losses-with-redemptions-starting-near-1-cent-per-dollar) |

## Adoption metrics

All values accessed 2026-11-16 unless stated. "Change since May 2026" shows the value in the May note.

| Metric | Value | Date | Source | Change since May 2026 |
|--------|-------|------|--------|-----------------------|
| TVL, Drift Trade (DefiLlama) | $636,392 | 2026-11-16 | [DefiLlama : Drift Trade](https://defillama.com/protocol/drift-trade) (page returned 403; figure read from [api.llama.fi](https://api.llama.fi/protocol/drift-trade)) | May note: "$150M-$400M" range. DefiLlama showed $6.25M on 2026-05-19, $311.4M on 2026-04-01 and $343.0M on 2026-03-01 |
| TVL, Velocity (DefiLlama) | [NOT FOUND] (listed 2026-09-29 with an empty TVL adapter) | 2026-11-16 | [DefiLlama API : Velocity](https://api.llama.fi/protocol/velocity) | New |
| Pre-hack TVL, parent Drift | "Above $550M" (as reported, citing DefiLlama; parent-level scope not verified; the Drift Trade series alone was $311M) | 2026-04-01 | [The Block](https://www.theblock.co/post/396183/drift-280m-exploit-zachxbt-circle) | Sources conflict on scope |
| Open interest, Velocity | $193,333 (DefiLlama); $193.5K one-sided (third-party tracker) | 2026-11-15 / 2026-11-16 | [DefiLlama OI API](https://api.llama.fi/summary/open-interest/velocity); [bykaranteli.com](https://bykaranteli.com/solana-perps) | May note: ~$400M (April 2026) |
| Open interest, pre-hack Drift | $62.1M (2026-04-01); $83.1M (2026-03-15); $182.9M (2025-12-15). DefiLlama series then reads $0 from mid-April | 2026-03 to 2026-04 | [DefiLlama OI API : Drift Trade](https://api.llama.fi/summary/open-interest/drift-trade) | The "$400M OI" is not reproduced; definitions may differ (one-sided versus total), so treat as a range question |
| 24h volume, Velocity | $232.4K | 2026-11-16 03:41 UTC | [bykaranteli.com Solana perps tracker](https://bykaranteli.com/solana-perps) (third party, reads the venue's public API) | May note: no volume. Drift volume was zero while offline (Cryptopolitan, July 2026) |
| 7d and 30d volume | [NOT FOUND] (DefiLlama perps volume is paywalled to this fetch; Velocity publishes no stats endpoint found) | | | |
| Cumulative volume since 2021 | "More than $150B" (founder statement, self-reported) | 2026-09-29 | [Solana Compass](https://solanacompass.com/news/drift-co-founder-cindy-leow-steps-down-hands-rebuilt-velocity-exchange-to-new-team); [TokenPost](https://www.tokenpost.com/news/technology/25591) | New |
| Fees, parent "Drift" (DefiLlama) | 24h $29,167; 7d $189,874; 30d $1.05M | 2026-11-16 | [DefiLlama fees API](https://api.llama.fi/summary/fees/drift) | New; does not reconcile with $232K daily volume (fees would be about 12.6% of volume), so the scope of this series is unclear: do not cite as Velocity fees without checking |
| Fees, Drift Trade (DefiLlama) | 24h none reported; 1y $5.04M; all-time $41.1M | 2026-11-16 | [DefiLlama fees API : Drift Trade](https://api.llama.fi/summary/fees/drift-trade) | New |
| Annualised fee revenue (pre-relaunch) | ~$35M (stated "based on prior activity"; single secondary source); 2025 revenue $19M (DL News) | 2026-07 / 2026-05 | [Cryptopolitan](https://www.cryptopolitan.com/drift-protocol-relaunches-as-velocity-dex/); [DL News](https://dlnews.com/articles/defi/drift-to-issue-recovery-tokens-in-wake-of-295m-hack) | Two sources conflict ($35M versus $19M) |
| Recovery pool | ~3.11M USDT against 299.5M DFX; redemption ~0.0104 USDT per DFX; ~$9.2M frozen; Tether up to $127.5M (drawn amount not reported) | 2026-10-01 | [Solana Compass](https://solanacompass.com/news/drift-opens-dfx-claims-for-april-exploit-losses-with-redemptions-starting-near-1-cent-per-dollar) | New; single secondary source citing the Foundation's 2026-09-30 update |
| Liquidity provider APY | [NOT FOUND] for Velocity (Earn deposits "earn a lending APY"; no rate sourced) | 2026-11-16 | [Velocity site](https://velocity.exchange) | May note: 10-30% (favourable conditions), now stale |
| Order execution latency | <400ms (pre-hack claim, not re-verified) | 2026 (pre-hack) | [Eco support Drift deep dive](https://eco.com/support/en/articles/15083167-drift-protocol-solana-perpetuals-dex-deep-dive) | Not refreshed |
| Leverage cap | Per market (initial margin ratio between 1.25% and 100%, i.e. 80x to 1x program bound). Velocity site: 20x on BTC, ETH, SOL; 10x on HYPE | 2026-11-16 | [Velocity docs : margin](https://docs.velocity.exchange/protocol/trading/margin); [Velocity site](https://velocity.exchange) | May note: "10x perpetuals" (Eco). Not found in Drift docs; the pre-hack Drift docs referred to a High Leverage Mode (beta) for SOL, BTC and ETH |

## How it works

### User perspective

A trader funds an account on Solana and places an order. The Velocity docs (the current successor to Drift's docs) describe three liquidity sources that a single taker order can fill from: resting orders on the decentralised orderbook (DLOB), market makers quoting just in time, and the protocol's own AMM ("Velocity AMM"). Perp market orders run a short Dutch auction. Velocity fees: taker 4 bps at the base tier, falling to 3 bps above $5M and 2 bps above $80M of trailing 30-day volume; maker rebate 0.25 bps (docs; the landing page quotes 6 bps falling to 2 bps, a discrepancy between the two Velocity pages).

The pre-hack Drift routed orders through JIT auction, DLOB and DAMM/BAL in priority; the three-tier description in the May note was accurate for Drift and appears to carry over to Velocity, since Velocity is a fork of Drift v2. Whether the AMM remains an oracle-pegged vAMM with Backstop AMM Liquidity (BAL) was not confirmed in the Velocity docs.

Velocity removed spot trading from its program (spot markets remain only for collateral and borrow/lend). Collateral is wBTC, wETH, SOL and USDT in one cross-margined account, settling in USDT; equity perps (NVDA, TSLA) are advertised as "coming soon" on the site while the FAQ lists them "at launch", a discrepancy not resolved here. The "unified margin across perps, spot and lending" claim is partly true: spot trading is gone but spot assets and lending still back perp margin. Earn products and the mobile app were cut or delayed per the May 2026 recovery plan (DL News).

### Protocol perspective

- **Oracle:** Pre-hack Drift used Pyth as the primary oracle with per-market configuration. Velocity's docs fetched on 2026-11-16 do not name its oracle ([NOT FOUND]). The exploit abused the ability to list a new asset with an oracle the attacker controlled (TRM: the attacker built a price near $1 for CVT via wash trading on Raydium; BlockSec: an attacker-controlled oracle), so oracle listing governance rather than Pyth itself was the weak point.
- **Oracle-pegged AMM:** the vAMM's mid was anchored to the live oracle price, with dynamic spread and peg updates (Drift v2 design).
- **Funding rate:** hourly funding payment between longs and shorts (Velocity docs).
- **JIT auction:** market makers quote just in time; orders are on-chain accounts that keepers fill.
- **Governance and admin controls:** the Drift Security Council was a 2-of-5 multisig with zero timelock from 2026-03-27. The May 2026 plan said the relaunch needs two independent audits, tighter key management, a new multisig structure and enforced delays on critical actions (The Block, 2026-04-16). Velocity reports an OtterSec audit (154 findings, none critical, all high findings fixed), Asymmetric Research operational security review and an ongoing Solana Foundation STRIDE review; the program was not open source at the time of the docs fetch although the blog (2026-09-17) says the source was opened (conflict not resolved).

## Key behaviours

- [[patterns/oracle-pricing-model]] : Pyth-driven mark price, vAMM pegged to oracle (pre-hack; Velocity oracle not confirmed).
- [[patterns/jit-auction-routing]] : JIT auction as first-class liquidity tier.
- [[patterns/hybrid-amm-orderbook]] : three-tier routing across JIT, DLOB and vAMM.

## Architecture decisions

- **Solana-native:** built on Solana for sub-second finality and low fees, which is essential for an on-chain orderbook with high update frequency.
- **Hybrid stack:** treats JIT, DLOB and vAMM as complementary rather than substitutes; reduces dependency on any single liquidity source.
- **Unified cross-margin across perps/spot/lending:** treats user as a single account, simplifying capital efficiency (spot trading removed in Velocity).
- **Oracle-anchored vAMM:** prevents the vAMM from drifting from market price during low-volume periods, which was a known failure mode of pure constant-product perps.
- **Post-hack:** redeployment as a new program at a new address (Velocity), USDT settlement under Tether backing, a leaner product set, and a recovery pool funded from revenue.

## Differentiators

- Most sophisticated on-chain liquidity stack among Solana perp venues (a design claim, not a volume claim; Velocity's current size is a small fraction of its former size).
- Unified margin across perps and lending in a single protocol.
- Open JIT auction model with active third-party market makers, rather than a single protocol-run market maker.

## Limitations and criticisms

- **Admin-key and governance failure:** the 2026-04-01 loss of about $285M came from a compromised Security Council (two of five signers pre-signed malicious transactions using Solana durable nonces; no timelock), not from the trading engine. Any venue with privileged admin keys, listing rights and no timelock carries this risk.
- **Social-engineering exposure:** Drift attributes the intrusion to a six-month operation by DPRK-aligned actors who met contributors in person and likely compromised contributor devices via a shared code repository or a TestFlight app (preliminary).
- **Loss of scale and trust:** TVL fell from about $311M (Drift Trade, 2026-04-01) to under $1M, and Velocity trades about $0.2M per day and holds about $0.2M of open interest. Recovery tokens redeem at about one cent per dollar initially.
- **Stablecoin dependency:** the move from USDC to USDT concentrates settlement on one issuer (Tether), which also funds the recovery and holds a stated backing relationship with the exchange.
- Smaller in absolute size than Hyperliquid or Jupiter Perps; no longer the number two on Solana by usage (Pacifica, GMTrade and Phoenix all show materially higher 24h volume in the 2026-11-16 tracker).
- Hybrid routing complexity makes user-perceived fills less predictable than a pure orderbook venue.
- Historical incident: the May note recorded a Drift v1 vAMM-related exploit in 2022 which prompted the v2 redesign (not re-verified in this refresh).

## Sources

Accessed 2026-11-16 unless stated.

- [DefiLlama : Drift Trade](https://defillama.com/protocol/drift-trade) : page returned 403; figures read from the public API (api.llama.fi protocol, fees and open-interest endpoints for drift-trade, drift and velocity)
- [DefiLlama API : Velocity](https://api.llama.fi/protocol/velocity) : listing date 2026-09-29
- [bykaranteli.com : Solana perps tracker](https://bykaranteli.com/solana-perps) : third party, reads venue APIs
- [Velocity site](https://velocity.exchange) and [Velocity docs](https://docs.velocity.exchange/) : primary; Velocity docs margin page at https://docs.velocity.exchange/protocol/trading/margin
- [Velocity blog](https://velocity.exchange/blog) : audit and source release post dated 2026-09-17; trading challenge post dated 2026-11-05
- [TRM Labs : North Korean hackers attack Drift](https://www.trmlabs.com/resources/blog/north-korean-hackers-attack-drift-protocol-in-285-million-heist) : published 2026-04-02
- [BlockSec : Drift incident, durable nonce exploitation](https://blocksec.com/blog/drift-protocol-incident-multisig-governance-compromise-via-durable-nonce-exploitation) : technical summary
- [The Block : Drift says $280M exploit tied to admin takeover](https://www.theblock.co/post/396183/drift-280m-exploit-zachxbt-circle) : 2026-04-02 (quotes Drift's official X statement)
- [The Block : Drift links exploit to six-month operation](https://www.theblock.co/post/396361/drift-links-280-million-exploit-to-six-month-social-engineering-op-run-by-suspected-north-korean-actors) : 2026-04-05
- [The Record : Drift post-mortem, North Korea](https://therecord.media/drift-crypto-theft-post-mortem-north-korea) : 2026-04-10
- [The Block : Tether recovery package](https://www.theblock.co/post/397735/drift-127-million-tether-recovery-pivots-circle-usdc-usdt-280-million-exploit) : 2026-04-16
- [DL News : Drift to issue recovery tokens](https://dlnews.com/articles/defi/drift-to-issue-recovery-tokens-in-wake-of-295m-hack) : 2026-05-05 plan
- [Cryptopolitan : Drift relaunches as Velocity DEX](https://www.cryptopolitan.com/drift-protocol-relaunches-as-velocity-dex/) : secondary, July 2026
- [Solana Compass : Cindy Leow steps down, Velocity](https://solanacompass.com/news/drift-co-founder-cindy-leow-steps-down-hands-rebuilt-velocity-exchange-to-new-team) : 2026-09-29
- [Solana Compass : DFX claims](https://solanacompass.com/news/drift-opens-dfx-claims-for-april-exploit-losses-with-redemptions-starting-near-1-cent-per-dollar) : 2026-10-01
- [TokenPost : Velocity relaunches after hack](https://www.tokenpost.com/news/technology/25591) : 2026-09-29
- [Drift updates page](https://drift.trade/updates) : lists "Incident Recovery Update, April 16, 2026", "Recovery Plan for Affected Users, May 5, 2026" and "Drift Recovery Update, June 3, 2026" by title only; contents not retrieved
- [Drift docs : Understanding Drift](https://docs.drift.trade/protocol/about-v3) : accessed 2026-05-19 (docs now redirect to docs.velocity.exchange)
- [Drift docs : Drift AMM](https://docs.drift.trade/protocol/about-v3/drift-amm) : accessed 2026-05-19
- [Drift docs : funding rates](https://docs.drift.trade/protocol/trading/perpetuals-trading/funding-rates) : accessed 2026-05-19
- [Eco support : Drift Solana perp deep dive](https://eco.com/support/en/articles/15083167-drift-protocol-solana-perpetuals-dex-deep-dive) : accessed 2026-05-19 (pre-hack; TVL, APY and leverage figures superseded)
- [CryptoAdventure : Drift Protocol review 2026](https://cryptoadventure.com/drift-protocol-review-2026-solana-perps-margin-design-and-real-liquidity-conditions/) : accessed 2026-05-19 (source of the ~$400M open interest; not reproduced by DefiLlama)
- [Bitcoin.com : What is Drift 2026 guide](https://www.bitcoin.com/get-started/what-is-drift-solana/) : accessed 2026-05-19
