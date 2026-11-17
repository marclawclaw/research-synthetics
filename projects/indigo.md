---
tags: [project, cardano, cdp, stablecoin, synthetics, multi-synth, stability-pool]
ecosystem: Cardano
category: Synthetics/Multi-synth CDP (iAssets)
status: live (V3 live since 2026-05-28 per DefiLlama's fee methodology; V3 contracts not read)
status_date: 2026-11-17
website: https://indigoprotocol.io
docs: https://docs.indigoprotocol.io
launched: November 2022 (DefiLlama `listedAt` and first TVL datapoint 2022-11-24; iBTC and iUSD "upon launch" per the iAssets docs page; exact mainnet date [NOT FOUND] in a primary source)
---

# Indigo Protocol (Cardano: iUSD, iBTC, iETH and more)

Status: live and small. Indigo is the main multi-synth CDP on Cardano. Users lock ADA (V2) or ADA and Cardano native tokens (V3) in a collateralised debt position (CDP) and mint "iAssets". The current docs name iUSD (USD), iBTC and iETH (iBTC and iUSD "upon launch", iETH "followed after DAO approval"), with iJPY and iEUR "being introduced through DAO governance" in V3 ([iAssets](https://docs.indigoprotocol.io/readme/iassets.md), read 2026-11-17). iSOL is not named anywhere in the docs (`llms-full.txt`, read 2026-11-17), but the January 2025 tokenomics proposal lists an iSOL Stability Pool receiving 1,000 INDY per epoch ([forum, 2025-01-16](https://forum.indigoprotocol.io/t/proposal-update-of-indigo-tokenomics/14298)), so iSOL was added after launch with its own pool; whether it is still live is unverified. Its design point for this vault is that it keeps a separate Stability Pool (a first-loss pool) for each iAsset, which makes it the only surveyed multi-synth protocol with a pool per synth ([[patterns/cdp-synthetics-landscape]] section 1; [[rfp/appendix/cdp-isolation]], "Indigo: a pool per synth, and what happens when it is too small"). Previously the vault held only one row for it in the landscape note. Written 2026-11-17 from docs, the whitepaper and DefiLlama; no contract source was read (see "Not checked").

## Adoption metrics

| Metric | Value | Date | Source |
|--------|-------|------|--------|
| TVL | $2.31M ($2.28M on 2026-11-16 in the landscape note) | 2026-11-17 | [DefiLlama Indigo](https://api.llama.fi/protocol/indigo) |
| TVL peak | $125.56M | 2024-03-12 | [DefiLlama Indigo](https://api.llama.fi/protocol/indigo) |
| TVL, first datapoint | $6.58M | 2022-11-24 | [DefiLlama Indigo](https://api.llama.fi/protocol/indigo) |
| iUSD circulating and price (name and chain matched on DefiLlama's list, not otherwise verified) | $1.15M at $1.0013 (an earlier reading the same day: $1.14M at $0.9888; the price moves intraday) | 2026-11-17 | [DefiLlama stablecoins](https://stablecoins.llama.fi/stablecoins) |
| Fees: 30 days / 1 year / all time | $24,895 / $2.86M / $9.40M (the 1 year figure is lumpy: two days, 2026-01-30 at $705k and 2026-06-05 at $518k, make up about 43% of it) | 2026-11-17 | [DefiLlama fees: Indigo](https://api.llama.fi/summary/fees/indigo) |

Fee caveat: DefiLlama defines Indigo fees as CDP fees paid by users. Since V3 (2026-05-28) that is borrow interest added to the interest-collection contract plus debt minting, redemption, liquidation processing and stableswap fees paid into the treasury; before V3 it is inflows to the V2 treasury and the INDY staker collector, taken from Indigo's analytics API. It excludes liquidated collateral, which goes to Stability Pool depositors, and Minswap fee-sharing income. Source: [DefiLlama fees: Indigo](https://api.llama.fi/summary/fees/indigo), methodology field.

## How it works

### User perspective

A borrower opens a CDP with a minimum of 10 ADA (whitepaper v2.0.0), mints an iAsset up to the iAsset's maintenance ratio, and keeps the position above the iAsset's liquidation ratio (LR). Indigo supports liquid staking of the ADA collateral, so staking rewards keep accruing to the borrower's wallet (whitepaper). In V3 collateral is chosen per pair: "Each loan position is single-collateral-type from creation to closure" and each iAsset can have up to 8 non-ADA collateral assets (NIGHT, USDM/A, USDCx and xBTC are named) ([V3 docs](https://docs.indigoprotocol.io/readme/indigo-protocol-v3.md)). Holders of an iAsset can deposit it in that iAsset's Stability Pool ("Stability Providers") to earn liquidation collateral and a share of interest.

### Protocol perspective

- **One pool per synth.** "Each iAsset, such as iBTC, has its dedicated Stability Pool." Depositors are any holders of the iAsset, whether minted or bought on a DEX. On a liquidation "iAssets in the corresponding Stability Pool are burned to repay the CDP's debt", and providers lose a share of their iAsset deposit and gain a share of the position's collateral (V3: ADA or a native token). Source: [Stability pools and liquidations](https://docs.indigoprotocol.io/readme/stability-pools-and-liquidations.md), re-read 2026-11-17.
- **Freeze, then liquidate.** When a CDP's collateral ratio (CR) falls below the iAsset's LR, any user can submit a transaction to freeze it, which takes it away from its owner ("The former owner loses all access and rights to the CDP"); once frozen, any user can liquidate it against the Stability Pool (whitepaper v2.0.0, section 2.4.2). Docs: "Liquidations in the Indigo Protocol are automated by bots" ([Liquidations](https://docs.indigoprotocol.io/readme/liquidations.md)). No liquidator reward is documented; the pool depositors take the collateral, which "may exceed the value of the canceled debt".
- **Shortfall handling (when the pool cannot cover a liquidation).** The whitepaper v2.0.0 draft (March 2024), section 2.6, says that if a CDP's debt is higher than all iAsset in the related pool, "the protocol attempts to cancel as much debt as possible with the iAsset supply available. Any remaining non-liquidated collateral and debt of the CDP remains frozen until more iAsset is deposited into the SP and another liquidation is initiated." That is a partial liquidation: the pool burns everything it holds, the position's debt falls by that amount, the same share of its collateral goes to stakers, and the rest stays frozen in the position. Frozen CDPs can be merged so several can be liquidated at once (whitepaper). No redistribution to other positions, no auction, no treasury or reserve backstop is described in the whitepaper or in any docs page read. Bad debt is not mentioned in any docs page read. The V3 docs pages read (stability pools, liquidations, V3 overview, redemptions, interest) are silent on this case. Whether V3 changed it is not confirmed; the [audit page](https://docs.indigoprotocol.io/resources/audit.md) lists a TxPipe V3 audit summary dated May 2026 as an attached file, which was not opened.
- **Redemptions.** V2 introduced redemption: anyone can redeem iAsset for the collateral of a CDP whose CR is below the iAsset's redemption margin ratio (RMR), at the oracle price, only down to the RMR (whitepaper section 2.4.4). V2 RMR: "185% for $iUSD, 150% for $iETH, and $iBTC", that is 150% for both iETH and iBTC ([V2 docs](https://docs.indigoprotocol.io/readme/indigo-protocol-v2.md), read 2026-11-17). In V3 each collateral and iAsset pair has its own LR, MR and RMR, the OrderBook is bi-directional, and "At launch, the Indigo Foundation controls loan redemptions" through an optional redeemer signer ([Redemptions](https://docs.indigoprotocol.io/readme/redemptions.md)). No treasury backstop is mentioned.
- **Peg.** The whitepaper (section 2.8) names four tools: the liquidation ratio (a low LR such as 110% caps the premium at about 10% by arbitrage, a higher one pushes owners to close), the maintenance ratio, the interest rate (the DAO can raise or lower it) and the redemption margin ratio. V3 adds a Peg Stabilization Module (PSM): 1:1 swaps between iAssets and a paired stable collateral such as iUSD/USDCx, each direction switchable by the DAO per pair. See [[patterns/cdp-peg-mechanisms]].
- **Interest.** V2.1: borrowers paid interest implicitly in ADA. V3: "each collateral type/iAsset pair has its own algorithmic interest rate oracle", depending on the iAsset's total collateral ratio and on redeemable iAsset liquidity in the OrderBook; interest is minted as new iAssets and added to the borrower's debt ([Interest](https://docs.indigoprotocol.io/readme/interest.md)). The docs give no numeric rate range.
- **Oracle.** V3: "all existing oracle infrastructure migrates to Pyth", with one price oracle per collateral and iAsset pair ("pricing in Collateral/iAsset"); whether Pyth builds the cross from USD legs was not found ([[patterns/collateral-debt-units]] section 2). The whitepaper's launch design updated price feeds once per hour.

## Key behaviours

- [[patterns/cdp-synthetics-landscape]] : the multi-synth case with a Stability Pool per synth; losses beyond the pool wait in the position instead of being spread.
- [[rfp/appendix/cdp-isolation]] : "Indigo: a pool per synth, and what happens when it is too small"; the shortfall waits in the position.
- [[patterns/cdp-vs-morpho-trust]] : one fungible token per iAsset with a first-loss pool.

## Architecture decisions

- **A Stability Pool per iAsset**, so the pool's synth matches the debt it repays and the first loss stays within one synth.
- **Freeze then liquidate** rather than a one-step liquidation, so a position that is too large for the pool can be partly liquidated and the rest wait.
- **Redemption by margin ratio** (a limit on which CDPs are redeemable) plus interest and PSM as peg tools.

## Differentiators

- The only surveyed multi-synth product with a pool per synth.
- Cardano-native, with liquid staking of the collateral.

## Limitations and criticisms

- Small: TVL $2.31M against a peak of $125.56M (2024-03-12).
- The shortfall rule means a position larger than the pool is not closed: debt and collateral stay frozen until depositors return, and the system carries that unresolved position meanwhile (reading of the whitepaper; not tested).
- V3's behaviour in that case is not documented in the pages read.
- Foundation control of redemptions at V3 launch is a trust point (docs).

### Fees and borrower cost

| Item | Value | Source |
|------|-------|--------|
| Liquidation processing fee, V2 | 2% on the liquidated iAsset, replacing the earlier collateral fee | [V2 docs](https://docs.indigoprotocol.io/readme/indigo-protocol-v2.md) |
| Liquidation processing fee, V3 | A percentage of the collateral that would go to the Stability Pool, paid to the Treasury and capped so pool providers do not incur a net loss (ADA network fees aside) | [Liquidations](https://docs.indigoprotocol.io/readme/liquidations.md) |
| Debt minting fee | V3 docs example 0.5% on 100 iUSD, paid to the Treasury; DefiLlama API shows 0.1% (conflict C11 in [[patterns/cdp-synthetics-landscape]]) | [V3 docs](https://docs.indigoprotocol.io/readme/indigo-protocol-v3.md) |
| Redemption fee, V2 | 2%, half to the redeemed CDP and half to INDY stakers (docs); the API shows 1% plus 1% (conflict C11) | [V2 docs](https://docs.indigoprotocol.io/readme/indigo-protocol-v2.md) |
| Stability Pool fees | 5 ADA initial deposit fee, 0.5% withdrawal fee (loyalty bonus to remaining depositors) and 1 ADA per transaction, all redistributed to providers; a 2% fee on ADA rewards goes to INDY stakers (the DAO can adjust it) | [Stability pools and liquidations](https://docs.indigoprotocol.io/readme/stability-pools-and-liquidations.md) |
| Interest split, V3 | 40% to Stability Pool depositors, 20% to Liquidity Providers, 40% to the Treasury; for the Stability Pool, "15-minute snapshots are created and distributed daily to pool users from day 31–60"; Treasury distributions every 30 days | [V3 docs](https://docs.indigoprotocol.io/readme/indigo-protocol-v3.md) |
| INDY staker revenue, V3 | The Treasury receives fees; V3 sends no fees to stakers directly (DefiLlama methodology); INDY staker revenue is converted from iAssets to INDY by market orders (V3 docs) | [V3 docs](https://docs.indigoprotocol.io/readme/indigo-protocol-v3.md); [DefiLlama fees: Indigo](https://api.llama.fi/summary/fees/indigo) |

## V2 versus V3

| Item | V2 | V3 |
|------|----|----|
| Collateral | ADA | ADA and Cardano native tokens, set per collateral and iAsset pair; a position is single-collateral from creation to closure |
| Oracle | Feeds updated about hourly in the whitepaper design; provider not stated in the pages read | Pyth, one oracle per pair |
| Redemption | Margin-ratio redemption by any user; 2% fee | OrderBook bi-directional; Foundation controls loan redemptions at launch; PSM for stable pairs |
| Interest | Applied to all CDPs; paid implicitly in ADA | Algorithmic rate oracle per pair; minted as iAssets and added to debt |
| Stability Pool | Collateral gains; INDY rewards; 0.5% withdrawal fee | Multi-asset rewards; 40% of interest; account adjustment fee removed, cooldown added |
| Shortfall beyond the pool | Frozen in the position (whitepaper) | Not documented in the pages read |

## Not checked

- Contract source for any version (no local clone of the Indigo repositories; contracts not read). If a read is needed: the Indigo on-chain validators for the CDP and Stability Pool contracts, and the TxPipe V3 audit summary attached to the audit page.
- Audits listed on the [audit page](https://docs.indigoprotocol.io/resources/audit.md) (read 2026-11-17): Tweag (October 2021), MLabs (November 2022, March 2024, October 2024 for V2.1, with an informal Sundae Labs review of V2.1) and TxPipe (May 2026, V3 audit summary). None of the reports was read.
- Whether V3 changed the shortfall rule, whether any bad debt has ever been left frozen, and the current size of each Stability Pool (no number found).
- V3 liquidation, maintenance and redemption ratios per pair ([NOT FOUND]).
- The V3 launch is confirmed only by DefiLlama's methodology note (2026-05-28) and a secondary source dated 2026-05-29: [Learn Cardano podcast notes](https://learncardano.io/podcasts/cardano-ada-news-update-strike-perps-indigo-v3-charli3-more-29-may-2026/).
- Shortfall-handling findings came from a research agent in the Notion session (agent-reported); the whitepaper passage, the stability-pools page and the V3, V2, redemption, interest and liquidation pages were re-read in this pass.

## Sources

- [Indigo whitepaper v2.0.0 (March 2024 draft)](https://indigoprotocol.io/paper.pdf), sections 2.4, 2.6 and 2.8 : accessed 2026-11-17
- [Indigo docs: Stability pools and liquidations](https://docs.indigoprotocol.io/readme/stability-pools-and-liquidations.md) : accessed 2026-11-17
- [Indigo docs: Liquidations](https://docs.indigoprotocol.io/readme/liquidations.md) : accessed 2026-11-17
- [Indigo docs: Indigo Protocol V3](https://docs.indigoprotocol.io/readme/indigo-protocol-v3.md) : accessed 2026-11-17
- [Indigo docs: Indigo Protocol V2](https://docs.indigoprotocol.io/readme/indigo-protocol-v2.md) : accessed 2026-11-17
- [Indigo docs: Redemptions](https://docs.indigoprotocol.io/readme/redemptions.md) : accessed 2026-11-17
- [Indigo docs: Interest](https://docs.indigoprotocol.io/readme/interest.md) : accessed 2026-11-17
- [Indigo docs: iAssets](https://docs.indigoprotocol.io/readme/iassets) : accessed 2026-11-17
- [Indigo docs: Audit](https://docs.indigoprotocol.io/resources/audit.md) : page read 2026-11-17; the attached reports were not opened
- [Indigo forum: Proposal: Update of Indigo Tokenomics](https://forum.indigoprotocol.io/t/proposal-update-of-indigo-tokenomics/14298) : posted 2025-01-16, read 2026-11-17 (iSOL Stability Pool emissions)
- [DefiLlama Indigo](https://api.llama.fi/protocol/indigo), [fees: Indigo](https://api.llama.fi/summary/fees/indigo), [stablecoins list](https://stablecoins.llama.fi/stablecoins) : accessed 2026-11-17
- [Learn Cardano: ADA news update, 2026-05-29](https://learncardano.io/podcasts/cardano-ada-news-update-strike-perps-indigo-v3-charli3-more-29-may-2026/) : secondary, accessed in the Notion session
