---
tags: [pattern, cdp, peg, rate-mechanism, redemption, pegkeeper, stablecoin, synthetics]
seen_in: [liquity, sky-maker, crvusd, youves, synthetix-v2, synthetix-v3, metronome-synth, indigo, monolith, alchemix, hylo, mirror-protocol, frankencoin]
---

# Peg mechanisms in CDP synthetics

Question: what keeps a CDP synth's market price near its reference price, and which of those mechanisms could the LEZ CDP use? This note backs Part 3 ("Peg mechanics") of the Notion page "CDP-Backed Synthetics" ([[rfp/appendix/cdp-notion-map]]). A peg mechanism is anything that keeps a synth's market price near its reference price. A rate mechanism is the subset where the protocol changes a borrowing rate or fee to move supply and demand. ("Lever" is Part 1's word for admin rules, not for these; see [[rfp/appendix/cdp-admin-switch]].) Written 2026-11-17 from the vault's project notes, with docs and whitepaper re-reads for crvUSD, Indigo and Monolith; no contract source was read in this pass. Fact-checked on 2026-11-17 and corrected (see "Gaps and conflicts", evidence status).

Highlight words (the Notion page's colours, written in words): "Open decision: ..." is a decision with no recommendation yet; "Risk to evaluate: ..." is a risk that needs evaluating.

## The page's table

| Peg mechanism | Who uses it |
|---|---|
| Repay or close at the oracle price | Every CDP |
| Rate or fee changes that move supply and demand | RAI, Liquity V2, crvUSD, Sky/Maker, Youves |
| Protocol-owned liquidity in pools (PegKeepers) | crvUSD |
| Nothing; only a shared debt pool | Synthetix V2 |

The reflexive stablecoin (RFP-013) is a separate product and is not a peg mechanism here; the page's Terms note that "a redemption price, as in RAI, is something else: an internal target price", not Liquity's redemption at face value.

## Questions (from the page)

1. **Open decision: which of these does the CDP use for its synths?** Repay at oracle is the baseline. Do we add a rate mechanism, where the protocol changes a borrowing rate or fee to move demand for the synth?
2. **Risk to evaluate: when someone repays or swaps at the oracle price, what stops a stale feed being exploited?** Metronome's swap module minted synths at a lagging oracle price and ended up about $15.7M unbacked.

## 1. Repay or close at the oracle price (every CDP)

Mechanism. A borrower can always return the synth and take the collateral back, so a synth trading below its oracle value is bought and burned, and one trading above is minted and sold, subject to the collateral ratio. This is the baseline of every CDP in the survey; what differs is what the synth is valued at.

| Protocol | How repay or close is priced | Evidence |
|---|---|---|
| Sky/Maker | DAI debt repaid at $1 per unit; `par` = 1.0 on chain | [[projects/sky-maker]]; [[patterns/collateral-debt-units]] row Sky/Maker |
| Liquity V1 and V2 | 1 LUSD or BOLD = 1 USD, implicit; only ETH, WETH, wstETH or rETH is priced | [[projects/liquity]] |
| Synthetix V2 | Debt is a share of the global pool; to unlock SNX the staker must burn sUSD equal to current debt, which can be higher or lower than what was minted | [[projects/synthetix-v2]] |
| Metronome | msETH and msBTC valued by Chainlink; msUSD valued at exactly $1.00 inside the protocol | [[projects/metronome-synth]] |
| Mirror (dead) | No redemption outside a CDP close; the peg relies on arbitrage between the mAsset market price and the oracle price plus the CDP burn route (the vault's summary, not a docs quote); prices valid 60 s | [[projects/mirror-protocol]] |
| Indigo | Repay burns the iAsset; whitepaper 2.8.1: if an iAsset falls below its peg, CDP owners can buy it "to repay their loan at a discount", and a low liquidation ratio (launch design 110%) bounds the premium at about 10% by arbitrage | [[projects/indigo]] |
| Frankencoin | No oracle: debt counted in ZCHF units; challenges replace price feeds | [[projects/frankencoin]] |

Risk to evaluate (question 2): the baseline is only as good as the feed. Failures in the vault:

- **Metronome, 2026-07-30.** The DAO disclosed about 6,367 msETH (about 31% of msETH supply) and 4.57M msUSD (about 16% of msUSD supply) created without adequate collateral, about $15.7M to $16M. The cause was the swap module, which mints and burns synths at oracle price: Chainlink ETH/USD latency on Base (a median latency breach of about 54 seconds and up to 5 minutes 37 seconds in extreme cases, per Crypto Briefing) let swaps mint at a lagging price. It did not come from repays or liquidations. The treasury says it staged over $34M of defensive looped positions and injected $6.5M of protocol-owned liquidity; swaps are paused; msUSD traded at $0.818 on 2026-11-16 (supply $41.95M). The primary Metronome post-mortem was not retrieved; headline and summary sources only ([[projects/metronome-synth]]).
- **RAI, ongoing.** The ETH-A feed still reports valid at 2,637.75, a price last updated 2025-06-09, while its source reads 1,795.73 (chain, 2026-11-16): a validity flag with no age check ([[patterns/collateral-debt-units]] section 5).
- **Sky/Maker, 2020.** Black Thursday: the Medianizer oracle did not update under congestion and gas costs and bidders won collateral for 0 DAI ([[projects/sky-maker]]). The delayed price module (Oracle Security Module, one hour) was already live: release 1.0.0 of Multi-Collateral DAI (2019-11-18) lists the same OSM `0x81fe...5763` as `PIP_ETH` that ETH-A reads today ([changelog](https://changelog.makerdao.com/releases/mainnet/1.0.0/contracts.json), read 2026-11-17). So the OSM predates the failure and was part of the price lag, not the fix; secondary coverage names the lag among the causes (search results only). The redesign that followed was Liquidations 2.0 (Dutch auctions). The vault's Sky note says the 2020 failure is the origin of both, which conflicts with the changelog.
- **Mirror, May 2022.** The vault describes the failure as an oracle-feed failure caused by a chain fork, alongside the UST depeg ([[projects/mirror-protocol]]).

What mitigates it, from the vault (see also [[rfp/appendix/cdp-oracle]]): per-feed freshness limits from 60 seconds (Mirror) to 72 hours (JPY DeFi, which may then use the last saved price for up to 24 more hours before the branch pauses, so up to about 96 hours; [docs](https://jpydf.gitbook.io/jpy-defi/collateral-and-pricing/oracle.md), re-read 2026-11-17); a delayed or smoothed price for liquidations; an age check on validity; a defined fallback; and keeping any mint or swap path off a lagging oracle (inference, [[patterns/collateral-debt-units]] section 4).

## 2. Rate or fee changes that move supply and demand (the "rate mechanism")

Mechanism. The protocol or the borrowers change what it costs to hold debt in the synth, so minting becomes less attractive when the synth is below its reference and more attractive when it is above.

| Protocol | Who sets the rate and how | Numbers and dates |
|---|---|---|
| RAI (Reflexer) | A PI controller (the code's `PIRateSetter` and PI calculator; Reflexer's FAQ calls it a PID controller) sets a redemption rate from the gap between RAI's market price (Uniswap RAI/ETH TWAP times ETH/USD as read from the code; which market feed the live deployment uses was not checked on chain; Reflexer posts describe a 2021 Chainlink RAI/USD integration and a proposed Uniswap V3 migration, seen in search results only) and its internal redemption price, so the target itself floats. The redemption price enters the collateral check (Maker's `par` slot) | Redemption price 2.9720 USD, rate exactly 1.0 (zero), last updated 2026-02-06; TVL $1.89M (peak $389.4M on 2021-03-15); live-small, dormant ([[patterns/collateral-debt-units]] section 2; [[patterns/cdp-synthetics-landscape]] section 2.2) |
| Liquity V2 | Each borrower sets their own annual interest rate; redemptions take the lowest-rate positions first, so a low rate carries redemption risk. The ceiling comes from borrowing demand: borrowers cut rates when BOLD is above $1 and raise them when below. The 75% to Stability Pool, 25% to governance split is hard-coded | Rate 0.5% to 250% a year; a premature adjustment fee if the rate changes within 7 days ([[projects/liquity]]); BOLD $0.9985 on 2026-11-16 ([[patterns/cdp-synthetics-landscape]] section 3.2) |
| Liquity V1 (for contrast) | One-time borrowing fee that rises with redemptions, no interest | Borrowing fee 0.5% rising to 5%; baseRate half-life 12 hours ([[projects/liquity]]); LUSD $1.003 on 2026-11-16 ([[patterns/cdp-synthetics-landscape]] section 3.2) |
| crvUSD | A monetary policy contract sets each market's rate: it lowers the rate as PegKeeper debt rises (DebtFraction = PegKeeper debt / total debt) and raises it when crvUSD trades below $1. The DAO changes settings (7 day vote); the DAO or Emergency DAO can swap a market's rate contract | Borrow rate 6.48% to 7.92% APR on 2026-11-16 03:56 UTC; hard cap at which debt at most quadruples in a year ([[projects/crvusd]]; [PegKeepers docs](https://docs.curve.finance/developer/crvusd/pegkeepers/overview), re-read 2026-11-17) |
| Sky/Maker | Stability fee per collateral type (`duty` plus `base`), plus the DSR and SSR savings rates. A spell sets them (48 hours), or SP-BEAM, a bounded setter run by a 2 of 3 Safe | SP-BEAM: ETH-A and the savings rates between 2% and 30% (DSR 0% to 30%), at most 4 points per change, one batch per 16 hours; today ETH-A 10.50%, DSR 1.25%, SSR 3.80% (chain, 2026-11-16, [[projects/sky-maker]]) |
| Youves | Interest rates respond to the market-to-target deviation. uUSD: exponential response updated weekly, up to about +5.9% and -5.6% a year, total asset rate capped at +29.4% and floored at +0.4%. Newer trackers: linear, updated about every 12 hours, each response at most plus or minus 0.4% a year, so at most +5.6% and -5.6% a year per week of 14 responses (docs, re-read 2026-11-17). Savers earn the asset rate, minters pay a higher liability rate | uUSD $1.0101, circulating $1.11M on 2026-11-16 ([[projects/youves]]; [Interest Rate Response docs](https://docs.youves.com/syntheticAssets/stableTokens/incentiveFeatures/interestRates/Interest-Rate-Response)) |

Seen in the vault but not in the page's rate row:

- **Indigo.** The whitepaper (section 2.8.3) says the DAO can raise or lower the base interest to incentivise or disincentivise minting or buying iUSD, with a heightened rate when redemption occurs; V3 has an "algorithmic interest rate oracle" per collateral and iAsset pair that depends on the total collateral ratio and on redeemable iAsset liquidity ([[projects/indigo]]; [docs](https://docs.indigoprotocol.io/readme/interest.md), re-read 2026-11-17).
- **Monolith.** A "fully autonomous interest rate controller": when the free-debt share (redeemable debt) is below target the borrow rate rises over time, and when plentiful it decays toward a low floor ([[projects/monolith]]).
- **Alchemix.** No interest; the yield repays the loan, so no rate lever ([[projects/alchemix]]).

Reading. The vault records no measured effect of a rate mechanism on a peg. The only outcome data are the day's market prices ([[patterns/cdp-synthetics-landscape]] section 3.2), and those also depend on the other anchors each protocol has.

## 3. Protocol-owned liquidity in pools (PegKeepers): crvUSD

Mechanism ([PegKeeper docs](https://docs.curve.finance/developer/crvusd/pegkeepers/overview), re-read 2026-11-17; [[projects/crvusd]]):

- PegKeepers hold crvUSD minted to them through their own debt ceilings. When crvUSD trades above 1.0 they deposit it into their linked crvUSD/stablecoin pool and receive LP tokens, which pushes the price down. When it falls below 1.0 they burn LP tokens and withdraw crvUSD; "the contract must have LP tokens to burn", so a PegKeeper can only withdraw what it deposited earlier.
- `update` is callable by any account; the caller gets a share as an incentive. PegKeeperV2 acts only if the AMM price is within a set deviation of the oracle EMA price, and the PegKeeperRegulator blocks deposits if the largest price falls below a threshold. The regulator's admin or emergency admin can pause and unpause deposits and withdrawals; ratio limits stop one PegKeeper taking all the debt.
- Debt ceilings on chain, 2026-11-16: USDC 135M, USDT 135M, pyUSD 45M, frxUSD 9M, GHO 0. The chain has five PegKeepers in both the monetary policy and the regulator; the docs list four (conflict recorded in [[projects/crvusd]]).
- Risk to evaluate: all five PegKeepers read `debt()` = 0 on 2026-11-16, so they had no capacity to support the peg below $1 at that moment ([[projects/crvusd]]). The page's Shutdown cell says the same.
- Also seen: Metronome's treasury-owned liquidity (AMOs and protocol-owned liquidity) "supports peg and generates fee income" and $6.5M was injected after the 2026 shortfall ([[projects/metronome-synth]]); not on the page's table.

## 4. Nothing; only a shared debt pool: Synthetix V2

Mechanism ([[projects/synthetix-v2]]): sUSD has no direct redemption for $1 of collateral. The structural bid is that stakers must buy and burn sUSD to repay debt or to meet the target collateral ratio; SCCP-232 (2022-09-11) names those purchases as pressure on the peg: lowering the target ratio would "prevent some pressure on the sUSD peg which would be caused by stakers buying additional sUSD on the market to pay their debt" ([SCCP-232](https://sips.synthetix.io/sccp/sccp-232/), re-read 2026-11-17). Calling them a peg support is the vault's inference, not the SCCP's. Because SNX is the only collateral, a falling SNX price lowers collateral ratios and forces stakers to burn sUSD, which pressures the peg (reflexivity). sUSD's peg was repeatedly stressed; V3 reused sUSD and needed buybacks; sUSD was then retired under SIP-423 with an entitlement of 4 SNX per sUSD ([[projects/synthetix-v3]]), and the V3 note derives that holders were compensated in vested SNX worth roughly 60% of face at the 2026-11-16 SNX price (derived).

## Seen in the survey, not in the page's table

The vault holds these as evidence. Whether to add them to Part 3 is a call for the page's owner; this note does not change the page's table.

| Mechanism | Who | Evidence |
|---|---|---|
| Redemption of the synth for collateral at face value | Liquity V1 and V2 (against the lowest collateral ratio, or lowest rate, positions); Indigo (CDPs below the redemption margin ratio); Monolith ("free" debt is redeemable) | The landscape note's survival section: hard redemption pegs held tighter than soft anchors on 2026-11-16 (LUSD $1.003, BOLD $0.9985, Mezo MUSD $0.9929, Thala MOD $0.998, against Alchemix's queue $0.9666, MIM $0.0286, Metronome $0.818), with one exception (Gravita GRAI $0.4057 despite redemption) ([[patterns/cdp-synthetics-landscape]] section 3.2; DefiLlama prices) |
| A redemption queue (Transmuter) | Alchemix: alAsset holders stake into the Transmuter and receive the underlying 1:1 as yield is released; V3 earmarks debt as deposits vest. A soft peg: slow redemption lets the alAsset trade below the underlying while the queue is long | alUSD $0.9666 on 2026-11-16; alETH traded below a 0.991 ETH soft target after the 2023 Curve incident (Alchemix Q4 2023 report, secondary snippet; unverified). V3 charges the borrower 0.25% (0.10% on Base) on each amount of earmarked debt settled, including through a Transmuter redemption; the Transmuter fee itself is 0.00% ([V3 FAQ](https://alchemix-v3-docs.vercel.app/user/faq), re-read 2026-11-17; [[projects/alchemix]]) |
| A peg stability module (PSM) | Sky (LitePSM, `tin` and `tout` both 0 on 2026-11-16); Indigo V3 (1:1 swaps between an iAsset and a paired stable collateral, each direction switchable by the DAO); Monolith (optional per instance; buys only before the immutability deadline, fee ramp 0 to 100 bps) | [[projects/sky-maker]]; [[projects/indigo]]; [[projects/monolith]] |
| A reserve-ratio band | Hylo and Djed (minting the stable side stops below a reserve-ratio floor; fee curves by ratio); no per-user liquidation. Mint and redeem rules differ per protocol | Hylo: pools target about 150% with zones from 175% down to below 100%; hyUSD mints that would push a pool below 150% are blocked; the docs do not say whether hyUSD redemption stays open below that ([risk management](https://docs.hylo.so/protocol-overview/risk-management.md), re-read 2026-11-17; [[projects/hylo]]). Djed: below its 400% floor since mid-2023 (derived ratio about 289%, DJED $1.013 on 2026-11-16, [[patterns/cdp-synthetics-landscape]] section 3.3); DJED minting stops below 400%. Whether DJED redemption also stops is in conflict: crypto.news (2023-07-05) says holders could no longer unmint, while COTI posts (via a search summary, not opened) say DJED burns stay open at any ratio. The Djed paper's rule was not read (ePrint returned 403) |

## Gaps and conflicts

- **Page versus vault.** The page's table says Liquity V2 uses a rate mechanism and says nothing of redemption at face value, although the vault's Liquity note calls redemption the hard floor ("Peg: hard floor by redemption"). Followed the page for the table and recorded redemption under "seen in the survey".
- **Questions unchanged.** Question 1 stays an open decision; the vault makes no recommendation.
- **Not found.** PSM fee and DSR/SSR numbers: the Sky note's older fee table says [NOT FOUND] while its admin section has the chain values used here; Liquity V2 market-average rate [NOT FOUND]; Youves peg-quality data beyond the 2026-11-16 price; a measured effect of any rate mechanism on a peg.
- **Evidence status.** crvUSD PegKeeper mechanics, Indigo interest and Monolith rate-controller wording were re-read from docs in this pass (2026-11-17). A fact-check on 2026-11-17 re-read SCCP-232, the Youves rate docs, the Alchemix V3 FAQ, the JPY DeFi oracle page, the Hylo risk page and the Maker 1.0.0 changelog; the other cells come from the project notes (2026-11-16 reads).

## Sources

- Notion page "CDP-Backed Synthetics", Part 3, as saved 2026-11-17.
- [Curve docs: PegKeepers overview](https://docs.curve.finance/developer/crvusd/pegkeepers/overview) : accessed 2026-11-17
- [Maker changelog, mainnet release 1.0.0 (2019-11-18)](https://changelog.makerdao.com/releases/mainnet/1.0.0/contracts.json) : accessed 2026-11-17
- [JPY DeFi: oracle](https://jpydf.gitbook.io/jpy-defi/collateral-and-pricing/oracle.md); [Hylo: risk management](https://docs.hylo.so/protocol-overview/risk-management.md); [Alchemix V3 FAQ](https://alchemix-v3-docs.vercel.app/user/faq) : accessed 2026-11-17
- [crypto.news on Djed, 2023-07-05](https://crypto.news/cardanos-djed-stablecoin-faces-unminting-and-depegging-challenges-amid-reserve-ratio-decline/) : accessed 2026-11-17
- [Indigo whitepaper v2.0.0](https://indigoprotocol.io/paper.pdf), section 2.8 : accessed 2026-11-17; [Indigo docs: Interest](https://docs.indigoprotocol.io/readme/interest.md) : accessed 2026-11-17
- [Monolith docs: redemptions](https://docs.monolith.market/protocol/redemptions.md), [PSM](https://docs.monolith.market/protocol/peg-stability-module) : accessed 2026-11-17
- [Youves: Interest Rate Response](https://docs.youves.com/syntheticAssets/stableTokens/incentiveFeatures/interestRates/Interest-Rate-Response) : accessed 2026-11-16, re-read 2026-11-17
- [SCCP-232](https://sips.synthetix.io/sccp/sccp-232/) : accessed 2026-11-16, re-read 2026-11-17
- [Crypto Briefing: Metronome DAO shortfall](https://cryptobriefing.com/metronome-dao-16m-oracle-shortfall/) and [The Defiant](https://thedefiant.io/news/defi/metronome-discloses-usd15-7-million-synth-shortfall-blames-oracle-lag-in-swap-module) : cited via [[projects/metronome-synth]], headline and summary only
- Vault notes: [[projects/liquity]], [[projects/sky-maker]], [[projects/crvusd]], [[projects/youves]], [[projects/synthetix-v2]], [[projects/synthetix-v3]], [[projects/metronome-synth]], [[projects/mirror-protocol]], [[projects/alchemix]], [[projects/hylo]], [[projects/indigo]], [[projects/monolith]], [[patterns/cdp-synthetics-landscape]], [[patterns/collateral-debt-units]]
