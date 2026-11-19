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

Status: live and small. Indigo is the main multi-synth CDP on Cardano. Users lock ADA (V2) or ADA and Cardano native tokens (V3) in a collateralised debt position (CDP) and mint "iAssets". The current docs name iUSD (USD), iBTC and iETH (iBTC and iUSD "upon launch", iETH "followed after DAO approval"), with iJPY and iEUR "being introduced through DAO governance" in V3 ([iAssets](https://docs.indigoprotocol.io/readme/iassets.md), read 2026-11-17). iSOL is not named anywhere in the docs (`llms-full.txt`, read 2026-11-17), but the January 2025 tokenomics proposal lists an iSOL Stability Pool receiving 1,000 INDY per epoch ([forum, 2025-01-16](https://forum.indigoprotocol.io/t/proposal-update-of-indigo-tokenomics/14298)), so iSOL was added after launch with its own pool (DAO poll 71, vote ended 2024-11-25; see "INDY token: governance, fees, backstop"); the V3 app config lists an `iSOL/.` Pyth feed, but live minting was not checked. Its design point for this vault is that it keeps a separate Stability Pool (a first-loss pool) for each iAsset, which makes it the only surveyed multi-synth protocol with a pool per synth ([[patterns/cdp-synthetics-landscape]] section 1; [[rfp/appendix/cdp-isolation]], "Indigo: a pool per synth, and what happens when it is too small"). Previously the vault held only one row for it in the landscape note. Written 2026-11-17 from docs, the whitepaper and DefiLlama; no contract source was read (see "Not checked").

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

## INDY token: governance, fees, backstop

Researched 2026-11-19 against the docs (`llms-full.txt`), the whitepaper v2.0.0, the DAO Voting Procedures PDF, the forum, Indigo's analytics API (the indexer the governance app reads) and Koios (on-chain token data). This section covers the three roles a CDP protocol token can play; the fee and Stability Pool items already in "Fees and borrower cost" above are cross-referenced, not repeated. Access date for every source in this section: 2026-11-19.

### Summary by role

| Role | Does INDY play it? | One line of evidence |
|------|--------------------|----------------------|
| Votes and vetoes | Yes for votes; no separate veto found | Staked INDY votes yes or no on on-chain proposals whose payloads (iAssets, collateral, PSM pools, upgrades, protocol parameters, treasury withdrawals) are executed by the governance contracts ([whitepaper](https://indigoprotocol.io/paper.pdf) 2.9.8 to 2.9.10; [polls API](https://analytics.indigoprotocol.io/api/v3/polls)) |
| Fees and rewards | Yes, but discretionary in V3 | Stakers receive protocol revenue that the Treasury converts from iAssets to INDY by market orders "upon DAO determination"; Stability Pool and LP emissions are being wound down to zero ([V3 docs](https://docs.indigoprotocol.io/readme/indigo-protocol-v3.md)) |
| Loss backstop | No | The INDY minting policy is time-locked (one mint of 35M, no burns, no further mint possible after November 2022), and no doc, whitepaper section, on-chain proposal or forum decision makes INDY or the Treasury a bad-debt backstop ([Koios](https://api.koios.rest/api/v1/policy_asset_info?_asset_policy=533bb94a8850ee3ccbe483106489399112b74c905342cb1792a797a0); [whitepaper](https://indigoprotocol.io/paper.pdf) 2.6) |

### 1. Votes and vetoes

**Who votes, and how.**
- Voting power is INDY staked in the protocol at the time of voting; a vote locks that stake until the proposal's Voting Period ends; votes cannot be changed; a voter uses all or none of their stake ([How To Vote](https://docs.indigoprotocol.io/readme/governance.md); [whitepaper](https://indigoprotocol.io/paper.pdf) 2.9.4).
- Only two options: Yes and No ([DAO Voting Procedures](https://forum.indigoprotocol.io/uploads/short-url/iYdEJXNHMCIIfRrV8roqHTtG87G.pdf), B Step 2). No separate veto power, guardian or veto-holder role was found in the docs, whitepaper or procedures. A proposal is stopped only by failing the pass test below. [NOT FOUND]: any veto mechanism comparable to Frankencoin's FPS veto.
- Staked INDY: 11,799,054 INDY ([analytics API, indy-staking](https://analytics.indigoprotocol.io/api/v3/analytics/indy-staking)).

**Who can submit.**
- Anyone holding INDY: an on-chain proposal needs a deposit of 100 INDY, returned if it passes and sent to the DAO Treasury if it fails ([How To Vote](https://docs.indigoprotocol.io/readme/governance.md)). The whitepaper makes the deposit exponential in concurrent proposals, `d = p * 2^a` with `a` the number of active proposals ([whitepaper](https://indigoprotocol.io/paper.pdf) 2.9.3). The proposer must also fund 8 voting shards in ADA ([protocol-params API](https://analytics.indigoprotocol.io/api/v3/protocol-params): `total_shards` 8).
- Off-chain gate: the procedures require a forum Temperature Check and a passing forum poll before the on-chain proposal, and the on-chain proposal "must be materially identical" to it ([DAO Voting Procedures](https://forum.indigoprotocol.io/uploads/short-url/iYdEJXNHMCIIfRrV8roqHTtG87G.pdf), B Steps 1 and 2). This gate is social (moderators), not enforced by the contracts.

**Pass test (quorum and threshold).**
- Hybrid rule: Adaptive Quorum Biasing (AQB) must give `q = yes / sqrt(electorate) - no / sqrt(yes + no) > 0`, and a minimum quorum must be met ([AQB & Minimum Quorum](https://docs.indigoprotocol.io/readme/governance-and-indy-staking/aqb-and-minimum-quorum.md)). At 100% turnout this is a simple majority; at low turnout it approaches unanimity (same page).
- Electorate: in V2.1 it was the circulating supply computed from the distribution schedule (whitepaper 2.9.6); in V3 it is a static parameter, 25,000,000 INDY at launch (AQB page, "V3 Electorate Change"). The polls API shows the electorate rising from 2,413,059 INDY (poll 1, December 2022) to 19,048,477 (poll 118, May 2026), then 25,000,000 from poll 119 onward ([polls API](https://analytics.indigoprotocol.io/api/v3/polls)).
- Current parameter values, from Indigo's indexer of the on-chain protocol-parameters datum (record created 2026-11-03; not read directly from chain): `minimum_quorum` 500,000 INDY, `voting_period` 432,000,000 ms (5 days), `effective_delay` 7,200,000 ms (2 hours), `expiration_period` 604,800,000 ms (7 days), `proposing_period` 7,200,000 ms (2 hours), `proposal_deposit` 100 INDY, `electorate` 25,000,000 INDY ([protocol-params API](https://analytics.indigoprotocol.io/api/v3/protocol-params)).

**Delay before execution.** A passed proposal becomes executable after the Effective Delay and must be executed before the Expiration Period ends, or it expires ([whitepaper](https://indigoprotocol.io/paper.pdf) 2.9.9 and 2.9.10; [DAO Voting Procedures](https://forum.indigoprotocol.io/uploads/short-url/iYdEJXNHMCIIfRrV8roqHTtG87G.pdf), C.6). Current Effective Delay: 2 hours (protocol-params API above). So the timelock between the end of the vote and execution is about 2 hours; the 5-day voting period is the main window.

**What the DAO can change.** Proposal types seen on chain (159 proposals as of 2026-11-19; [polls API](https://analytics.indigoprotocol.io/api/v3/polls)): TEXT 98, MIGRATE_ASSET 22 (update an iAsset's parameters), ADD_COLLATERAL_ASSET 15, PROPOSE_ASSET 6 (new iAsset), UPDATE_COLLATERAL_ASSET 6, MODIFY_STABLESWAP_POOL 4 and PROPOSE_STABLESWAP_POOL 4 (PSM), UPGRADE_PROTOCOL 4.

| Lever | Changeable by INDY vote? | Source |
|-------|--------------------------|--------|
| Add, update or delist an iAsset (oracle, ratios, fees) | Yes | Whitepaper 2.9.8; Voting Procedures B; polls 10, 71, 119, 121, 136 |
| Add or update a collateral asset per iAsset (oracle, LR, MR, RMR, interest oracle, minimum collateral; max 8 non-ADA per iAsset) | Yes | [V3 docs](https://docs.indigoprotocol.io/readme/indigo-protocol-v3.md) section 1; polls 120 to 148 |
| Per-iAsset fees (debt minting, liquidation processing, SP withdrawal, redemption reimbursement and processing) | Yes, in the iAsset proposal payload | V3 docs section 1; payloads of polls 71, 119, 121, 136 |
| Interest | Partly: the DAO "can adjust" rates (interest page); the V3 rate is an algorithmic oracle per pair. Recent interest changes (polls 85, 89, 94, 146) were TEXT proposals, which the whitepaper says are "not executed computationally"; who applies them is [NOT FOUND] | [Interest](https://docs.indigoprotocol.io/readme/interest.md); whitepaper 2.9.8; polls API |
| PSM pairs: add or remove, switch minting or redemption per direction, fees and minimum orders | Yes; the docs also mention an "optional fee manager" for PSM fees | V3 docs section 3; polls 124 to 126, 151 to 154, 157 |
| Oracles | Yes, as part of iAsset and collateral proposals | Whitepaper 2.9.8; V3 docs section 1 |
| Governance and treasury parameters (Effective Delay, quorum, deposit, voting period, treasury spend caps, electorate, redemption signer) | Yes, by an Update Protocol Parameters proposal | Whitepaper 2.9.9; V3 docs sections 10 and 12 |
| Contract upgrades | Yes, by an Upgrade Protocol proposal naming the new scripts; the upgrade is recognised only if the deployed code matches what was approved | Whitepaper 2.9.12; Voting Procedures B "Upgrade Protocol"; polls 33, 63, 118, 158 |
| Treasury withdrawals | Yes, attached to proposals as `treasuryWithdrawValue` and a destination address; per-proposal cap currently 500,000 ADA and 450,000 INDY (`max_treasury_lovelace_spend`, `max_treasury_indy_spend`) | Whitepaper 2.9.9 and 2.9.11; protocol-params API; polls API |

**On chain or carried out by someone else.**
- Typed proposals carry an "executable message"; after the delay "any user can execute it" and the governance contracts apply it ([whitepaper](https://indigoprotocol.io/paper.pdf) 2.9.10). The V3 config lists the governance, execute, poll manager and shard validators and the GOV_NFT and UPGRADE tokens they use ([mainnet-system-params-v3-0-1.json](https://config.indigoprotocol.io/mainnet/mainnet-system-params-v3-0-1.json)). The API does not expose whether each poll was executed; the contract execution step was not traced on chain.
- TEXT proposals record intent only, "not executed computationally" (whitepaper 2.9.8). They are carried out off chain, mostly by the Indigo Foundation and the elected Protocol Working Group (PWG): for example, treasury management v2 (poll 86, vote ended 2025-05-24) says "Buybacks will be managed jointly by the PWG and the Foundation" and "The PWG and Foundation will collaboratively manage emission allocations" ([forum 14395](https://forum.indigoprotocol.io/t/indigo-dao-treasury-management-policies-and-procedures-v2/14395)).
- Treasury withdrawals are made on chain, but to a wallet that then spends the funds off contract: polls 70, 109 and 112 all pay to the same address, which poll 109 calls a "Foundation-operated wallet, acting on behalf of the Indigo DAO" ([polls API](https://analytics.indigoprotocol.io/api/v3/polls), poll 109 content).

**Indigo Foundation powers outside the vote.**
- Whitepaper position: the Foundation (Cayman Islands foundation company) "is established to help implement approved actions of the DAO that cannot otherwise be implemented in an automated or computational manner"; "The Foundation's authority is limited to implementing the votes of the DAO"; it "does not have possession of or control of any Indigo or user funds" ([whitepaper](https://indigoprotocol.io/paper.pdf) 2.9.2).
- Powers the V3 design gives to a signer or multisig rather than to each vote:
  - Loan redemptions: "At V3 launch, the Indigo Foundation controls loan redemptions" through a protocol-configurable redeemer signer ([V3 docs](https://docs.indigoprotocol.io/readme/indigo-protocol-v3.md) section 12; [Redemptions](https://docs.indigoprotocol.io/readme/redemptions.md)). The protocol-params record has `cdp_redemption_signer` set to a key hash (`c8445c31...`); the key's owner is not stated there ([protocol-params API](https://analytics.indigoprotocol.io/api/v3/protocol-params)).
  - Interest distribution: "A distribution wallet must sign transactions to distribute funds"; it "can be rotated by the DAO-designated multi-sig through Protocol Parameter governance actions" (V3 docs section 5). The V3 config names an `INTEREST_ADMIN` multisig NFT in the interest collection parameters ([V3 config](https://config.indigoprotocol.io/mainnet/mainnet-system-params-v3-0-1.json)). The protocol-params API field `foundation_multisig` reads "TODO".
  - PSM: an "Optional fee manager to control minting and redemption fees" (V3 docs section 3). Who holds it: [NOT FOUND].
  - Standing upgrade mandate: poll 158 ("Proposal for Continuous Protocol Enhancements, Upgrades, and Fixes", type UPGRADE_PROTOCOL, vote ended 2026-10-19; yes 4,283,572, no 594,226 INDY) establishes "a standing mandate to implement protocol upgrades, technical improvements, and security fixes without requiring a separate governance vote for every change", with "an expedited path for security fixes", limited to changes that do not "materially alter the protocol's economic mechanism or tokenomics". Its procedure text says such changes need not go through "the process of off-chain discourse"; whether each upgrade still needs its own on-chain UPGRADE_PROTOCOL vote, and who deploys, is not stated ([polls API](https://analytics.indigoprotocol.io/api/v3/polls), poll 158).
- Custody: the DAO approved Foundation-controlled multisig wallets (poll 40, vote ended 2024-05-18; [forum 13990](https://forum.indigoprotocol.io/t/proposal-to-create-indigo-foundation-financial-structures-multisigs/13990)), and poll 149 places 10M INDY in "an Indigo Foundation custodial 3/5 multi-sig". Conflict: this practice differs from the whitepaper's statement that the Foundation holds no Indigo funds.
- The Foundation does not vote: "the Foundation does not participate in governance with Treasury or Foundation held funds" (EC_ATX, forum reply 2025-01-17, [forum 14298](https://forum.indigoprotocol.io/t/proposal-update-of-indigo-tokenomics/14298)). Forum statement only, not verified on chain.

**Precedents: INDY votes that changed live risk parameters or added iAssets.** Tallies are from the polls API (yes and no in INDY, 6 decimals removed); the API does not report pass or execution status, so "executed" is only stated where other evidence exists. Links use the governance app's poll URL format that the forum uses (`/governance/polls/<id>`, which redirects to `/indy-dao/<id>`).

| Poll | Vote ended (UTC) | Type | Change | Tally | Link |
|------|------------------|------|--------|-------|------|
| 10 | 2023-01-05 | PROPOSE_ASSET | iETH listed (asset name hex `69455448`), MCR 110%; poll 9 (ended 2022-12-24) had proposed the same asset with `mcr` 1,100,000, that is 1.1% in the 6-decimal encoding. Docs: "iETH followed after DAO approval" | yes 212,840, no 1,238 | [poll 10](https://app.indigoprotocol.io/governance/polls/10) |
| 11 | 2023-01-16 | MIGRATE_ASSET | iUSD MCR raised to 120% | yes 300,582, no 19,290 | [poll 11](https://app.indigoprotocol.io/governance/polls/11); [forum 12742](https://forum.indigoprotocol.io/t/increasing-iusd-mcr-to-120/12742) |
| 29 | 2023-11-26 | MIGRATE_ASSET | iUSD MCR raised to 150% during a depeg (the forum poll describes 150% "followed by a gradual rollback to 130% over 40 days") | yes 1,601,530, no 461,719 | [poll 29](https://app.indigoprotocol.io/governance/polls/29); [forum 13338](https://forum.indigoprotocol.io/t/strategic-adjustment-of-iusd-mcr/13338) |
| 71 | 2024-11-25 | PROPOSE_ASSET | iSOL listed: LR 110%, MR 115%, RMR 150%, minting fee 0.1%. Poll 70 (same day) withdrew 200,000 ADA and 8,000 INDY from the Treasury to bootstrap it. The V3 config carries an `iSOL/.` Pyth feed, which suggests iSOL is still configured in V3 | yes 5,013,169, no 76,642 | [poll 71](https://app.indigoprotocol.io/governance/polls/71); [forum 14242](https://forum.indigoprotocol.io/t/proposal-to-whitelist-isol/14242); [V3 config](https://config.indigoprotocol.io/mainnet/mainnet-system-params-v3-0-1.json) |
| 119, 121, 136 | 2026-06-02, 2026-06-02, 2026-07-11 | PROPOSE_ASSET | iJPY, iEUR and iADA listed under V3, with ADD_COLLATERAL_ASSET votes for their markets | see API | [polls API](https://analytics.indigoprotocol.io/api/v3/polls) |
| 151 | 2026-08-29 | MODIFY_STABLESWAP_POOL | "Halt USDA Minting Capabilities in the Peg Stability Module": iUSD/USDA PSM minting disabled, redemption kept | yes 1,630,807, no 461,536 (AQB q about 7, a narrow pass on the formula) | [poll 151](https://app.indigoprotocol.io/governance/polls/151) |

### 2. Fees and rewards

What the note already has stands: see "Fees and borrower cost" above for the V2 redemption split to INDY stakers, the 2% Stability Pool ADA reward fee to stakers, the V3 interest split (40% Stability Pool, 20% LPs, 40% Treasury) and the V3 rule that staker revenue is bought as INDY from Treasury iAssets. Updates and additions:

- **V3 staker revenue is discretionary.** "Revenue directed to INDY Stakers can be converted from iAsset(s) in the Treasury to INDY via open market orders for distribution upon DAO determination" ([V3 docs](https://docs.indigoprotocol.io/readme/indigo-protocol-v3.md) section 5). No automatic fee stream to stakers is described for V3.
- **Pre-V3 change to the interest share.** Treasury management v2 (poll 86, vote ended 2025-05-24; yes 5,695,667, no 137,129) set the iUSD interest share for INDY stakers from 30% to 0%, sent 50% of treasury withdrawals to INDY buybacks and raised staking emissions from 5,000 to 10,000 INDY per epoch ([forum 14395](https://forum.indigoprotocol.io/t/indigo-dao-treasury-management-policies-and-procedures-v2/14395); [forum 14430](https://forum.indigoprotocol.io/t/allocate-portion-of-iasset-interest-to-the-indy-buyback-strategy/14430)). Whether the follow-up to set the other iAssets' staker share to 0% reached an on-chain vote: [NOT FOUND] (no matching title in the polls API).
- **Governance participation rewards.** Stakers must vote at least once every 90 days to keep receiving INDY rewards ([Governance & INDY Staking](https://docs.indigoprotocol.io/readme/governance-and-indy-staking.md)).
- **Supply.**
  - On chain: 35,000,000 INDY minted in one transaction on 2022-11-20, mint count 1, burn count 0 ([Koios policy_asset_info](https://api.koios.rest/api/v1/policy_asset_info?_asset_policy=533bb94a8850ee3ccbe483106489399112b74c905342cb1792a797a0)). The policy is a native script requiring a key signature and validity "before" slot 77,422,037, which is 2022-11-20 on mainnet; under that script no later mint or burn can validate (Koios `tx_info` for mint tx `b5c8b52f...`).
  - Docs: "The total supply of INDY is capped at 25,000,000 under Indigo Protocol V3 (previously 35M)"; the same sentence says the monetary policy "disallows future minting and burning" ([Governance & INDY Staking](https://docs.indigoprotocol.io/readme/governance-and-indy-staking.md)). V3 sets the electorate and "max supply" at 25M; about 11.4M INDY of undistributed rewards remain, of which 10M is "reserved for a potential future burn upon DAO approval" and the rest goes to the Treasury ([Tokenomics](https://docs.indigoprotocol.io/readme/tokenomics.md)).
  - Poll 149 (vote ended 2026-08-22; yes 2,608,753, no 0): freeze the 10M INDY for 6 months in a Foundation 3/5 multisig, then burn in four tranches of 2.5M unless the DAO passes an alternative; unspent rewards 11,336,415 INDY as of 2026-07-30; 1,363,676 INDY to the Treasury ([polls API](https://analytics.indigoprotocol.io/api/v3/polls), poll 149).
  - Conflicts: (a) on chain 35M exist and none has been burned, while the docs call 25M the cap; the 25M is a governance figure (electorate) pending the 10M burn. (b) The docs say burning is disallowed yet reserve 10M for a burn; given the expired policy, a "burn" can only mean sending to an unspendable address (reading of the native script, not stated by Indigo). (c) Poll 149's figures do not add up: 11,336,415 minus 10,000,000 is 1,336,415, not 1,363,676.
- **Distribution.**
  - Docs: 54.13% DAO Rewards, 13.87% DAO Treasury, 5% Governance Participation, 1% Protocol Owned Liquidity, 1% Launch Airdrop, 25% Team & Labs Treasury, over a 5-year span ([Tokenomics](https://docs.indigoprotocol.io/readme/tokenomics.md)).
  - Whitepaper (Figure 2): Stability Pool Rewards 40%, iAsset Liquidity Staking 15%, Governance Participation 5%, Team 25%, DAO Treasury 13%, Airdrop 1%, Protocol Owned Liquidity 1%. Initial distribution: 21,000,000 INDY for community rewards (Stability Pools, liquidity, governance), 4,550,000 to the DAO Treasury Reserve, 8,750,000 to Indigo Laboratories (7,875,000 to team members on two-year monthly vesting), 350,000 to DEXs and 350,000 airdrop ([whitepaper](https://indigoprotocol.io/paper.pdf) 2.5.2).
  - Conflict: the docs' rewards plus governance shares (59.13%) and treasury share (13.87%) differ from the whitepaper's 60% and 13%. The docs do not split the 54.13% "DAO Rewards" between Stability Pools and LPs.
- **Stability Pool emissions (actuals).** Before the January 2025 tokenomics update: 21,189.77 INDY per epoch to Stability Pools (iUSD 15,406.67, iBTC 3,606.19, iETH 1,176.91, iSOL 1,000.00) and 10,456.23 to LPs; after: 16,951.82 and 8,364.98, with the 20% difference rerouted to the Treasury ([forum 14298](https://forum.indigoprotocol.io/t/proposal-update-of-indigo-tokenomics/14298)). V3 moves "toward zero INDY emissions" ([V3 docs](https://docs.indigoprotocol.io/readme/indigo-protocol-v3.md)); the date emissions reach zero: [NOT FOUND].
- **Market data.** INDY price $0.0754 (0.4888 ADA) on 2026-11-19 ([analytics API, indy](https://analytics.indigoprotocol.io/api/v3/analytics/indy)).

### 3. Loss backstop

**Finding: INDY has no loss-backstop role, and the Treasury has no defined one.** Evidence and sources checked:

- **INDY cannot be minted.** One mint, no burns, policy time-locked since 2022-11-20 (Koios, above). So the Maker-style "mint MKR to cover bad debt" is not possible with INDY's current policy.
- **No INDY slashing or sale rule.** Staked INDY is locked only for the Voting Period; no slashing or forced sale is described ([How To Vote](https://docs.indigoprotocol.io/readme/governance.md); [whitepaper](https://indigoprotocol.io/paper.pdf) 2.9.4). Unlike Frankencoin's FPS, INDY stakers do not lose first.
- **The shortfall waits in the position, not on the token or Treasury.** Whitepaper 2.6: debt above the pool "remains frozen until more iAsset is deposited into the SP" (see "Shortfall handling" above). The whitepaper's Treasury section (2.9.11) earmarks funds for maintenance, development and Foundation operations, with no loss-covering use. The V3 docs list revenue uses as reinvestment, "INDY buybacks/burns", staker distributions or "other Treasury initiatives" (section 9), with no loss-covering use.
- **No on-chain proposal on bad debt.** A text search of all 159 on-chain proposals for bad debt, backstop, shortfall, recapitalisation, insolvency, deficit, emergency, insurance and similar terms found only a V2 design paragraph about MCR versus RMR (poll 30) ([polls API](https://analytics.indigoprotocol.io/api/v3/polls)).
- **The forum has only community discussion.** Searches for "bad debt", "backstop", "insolvency", "shortfall", "recapitalize" and "insurance fund" return community threads (for example [forum 12737](https://forum.indigoprotocol.io/t/proposal-to-have-a-minting-rate-limitation-to-120-daily-and-position-limitation-to-30-of-stability-pool/12737), January 2023, modelling a 500k iUSD shortfall, and [forum 8492](https://forum.indigoprotocol.io/t/protocol-level-insurance-cover/8492), a 2022 insurance-fund idea), with no adopted mechanism.
- **Never used as a backstop.** No record of INDY or Treasury funds covering bad debt or a Stability Pool shortfall in any source above. Whether any bad debt has ever been left frozen remains unchecked (see "Not checked").
- **Closest to a backstop: the Treasury as a Stability Pool depositor, by vote.** In these cases the Treasury takes first-loss exposure like any other depositor; it does not cover losses beyond the pool:
  - iSOL launch (poll 70, vote ended 2024-11-25): 200,000 ADA from the Treasury opened a DAO-owned CDP at 150% CR, and "The newly minted iSOL will be deposited into the iSOL Stability Pool to ensure sufficient solvency". The funds were afterwards held 30 days in the Foundation multisig as "rainy day funds in case ... the iSOL Stability Pool needs replenishment" ([forum 14242](https://forum.indigoprotocol.io/t/proposal-to-whitelist-isol/14242)).
  - Poll 109 (vote ended 2026-01-31; yes 5,231,500, no 152,769): 500,000 ADA withdrawn to buy iUSD through the Redemption Orderbook and stake it in the iUSD Stability Pool, to "support system stability and generate incentive-based income", executed "via a Foundation-operated wallet" ([polls API](https://analytics.indigoprotocol.io/api/v3/polls), poll 109).
- **Not checked:** the TxPipe V3 audit summary and the contract source, which could show a V3 shortfall rule not in the docs.

### Characterisation for citation

INDY is a fixed-supply governance and fee-sharing token. Stakers vote on chain (yes or no, with no separate veto) on iAssets, collateral, PSM pools, parameters, upgrades and treasury withdrawals; a 2-hour execution delay applies, and redemptions, interest distribution and some upgrades are left to Foundation-held signers or mandates. Staker income now depends on the DAO's decisions about Treasury buybacks. INDY has no loss-backstop role: it cannot be minted, it is not slashed, and losses beyond a Stability Pool stay frozen in the position.

## Not checked

- Contract source for any version (no local clone of the Indigo repositories; contracts not read). If a read is needed: the Indigo on-chain validators for the CDP and Stability Pool contracts, and the TxPipe V3 audit summary attached to the audit page.
- Audits listed on the [audit page](https://docs.indigoprotocol.io/resources/audit.md) (read 2026-11-17): Tweag (October 2021), MLabs (November 2022, March 2024, October 2024 for V2.1, with an informal Sundae Labs review of V2.1) and TxPipe (May 2026, V3 audit summary). None of the reports was read.
- Whether V3 changed the shortfall rule, whether any bad debt has ever been left frozen, and the current size of each Stability Pool (no number found).
- V3 liquidation, maintenance and redemption ratios per pair ([NOT FOUND]).
- The V3 launch is confirmed only by DefiLlama's methodology note (2026-05-28) and a secondary source dated 2026-05-29: [Learn Cardano podcast notes](https://learncardano.io/podcasts/cardano-ada-news-update-strike-perps-indigo-v3-charli3-more-29-may-2026/).
- INDY section: governance parameter values come from Indigo's own indexer (`/protocol-params`), not a direct read of the on-chain datum; pass and execution status of each poll is not exposed by the polls API and the execute transactions were not traced; the owner of `cdp_redemption_signer` and of the PSM fee manager, and how TEXT-type interest changes are applied, were not found; the Indigo DAO Constitution was not read.
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
- [Indigo whitepaper v2.0.0](https://indigoprotocol.io/paper.pdf), sections 2.5 (INDY), 2.9 (governance, Foundation, Treasury, upgrades) : accessed 2026-11-19
- [Indigo docs: llms-full.txt](https://docs.indigoprotocol.io/llms-full.txt), including [Governance & INDY Staking](https://docs.indigoprotocol.io/readme/governance-and-indy-staking.md), [AQB & Minimum Quorum](https://docs.indigoprotocol.io/readme/governance-and-indy-staking/aqb-and-minimum-quorum.md), [How To Vote](https://docs.indigoprotocol.io/readme/governance.md), [Tokenomics](https://docs.indigoprotocol.io/readme/tokenomics.md) and the V3 page : accessed 2026-11-19
- [Indigo DAO Voting Procedures (PDF, linked from How To Vote)](https://forum.indigoprotocol.io/uploads/short-url/iYdEJXNHMCIIfRrV8roqHTtG87G.pdf) : accessed 2026-11-19
- [Indigo analytics API: polls](https://analytics.indigoprotocol.io/api/v3/polls) (all 159 on-chain proposals with type, end time, tallies, electorate, payload and treasury withdrawals), [protocol-params](https://analytics.indigoprotocol.io/api/v3/protocol-params), [indy-staking](https://analytics.indigoprotocol.io/api/v3/analytics/indy-staking), [indy price](https://analytics.indigoprotocol.io/api/v3/analytics/indy) : accessed 2026-11-19 (Indigo's indexer; endpoints found in the governance app bundle at app.indigoprotocol.io)
- [Indigo V3 mainnet system params](https://config.indigoprotocol.io/mainnet/mainnet-system-params-v3-0-1.json) : accessed 2026-11-19
- [Koios: INDY policy asset info](https://api.koios.rest/api/v1/policy_asset_info?_asset_policy=533bb94a8850ee3ccbe483106489399112b74c905342cb1792a797a0) and `tx_info` for mint tx `b5c8b52f91d6e52709cf990d6439adedab2d125ce43348d3a6c7e791a60c02dd` (native script) : accessed 2026-11-19
- Indigo forum topics, all accessed 2026-11-19: [12737](https://forum.indigoprotocol.io/t/proposal-to-have-a-minting-rate-limitation-to-120-daily-and-position-limitation-to-30-of-stability-pool/12737), [12742](https://forum.indigoprotocol.io/t/increasing-iusd-mcr-to-120/12742), [13338](https://forum.indigoprotocol.io/t/strategic-adjustment-of-iusd-mcr/13338), [13990](https://forum.indigoprotocol.io/t/proposal-to-create-indigo-foundation-financial-structures-multisigs/13990), [14242](https://forum.indigoprotocol.io/t/proposal-to-whitelist-isol/14242), [14298](https://forum.indigoprotocol.io/t/proposal-update-of-indigo-tokenomics/14298), [14395](https://forum.indigoprotocol.io/t/indigo-dao-treasury-management-policies-and-procedures-v2/14395), [14430](https://forum.indigoprotocol.io/t/allocate-portion-of-iasset-interest-to-the-indy-buyback-strategy/14430), [8492](https://forum.indigoprotocol.io/t/protocol-level-insurance-cover/8492); plus forum searches for "bad debt", "backstop", "insolvency", "shortfall", "recapitalize" and "insurance fund"
- Governance app poll pages (format used by the forum, redirecting to `/indy-dao/<id>`): [10](https://app.indigoprotocol.io/governance/polls/10), [11](https://app.indigoprotocol.io/governance/polls/11), [29](https://app.indigoprotocol.io/governance/polls/29), [71](https://app.indigoprotocol.io/governance/polls/71), [86](https://app.indigoprotocol.io/governance/polls/86), [151](https://app.indigoprotocol.io/governance/polls/151) : the app is a client-side page; content was read through the polls API, accessed 2026-11-19
- [Learn Cardano: ADA news update, 2026-05-29](https://learncardano.io/podcasts/cardano-ada-news-update-strike-perps-indigo-v3-charli3-more-29-may-2026/) : secondary, accessed in the Notion session
