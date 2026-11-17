---
tags: [appendix, cdp, lez, liquidation, privacy, market, questions]
related: [rfp/appendix/cdp-notion-map, rfp/appendix/cdp-isolation, rfp/appendix/cdp-oracle, rfp/appendix/sxmr-atomic-redemption, projects/monero-zcash-synthetics-community-history]
---

# CDP on LEZ: who carries the risk, privacy and market

Working note for the synth CDP program on LEZ (sXMR and sBTC are the examples). It backs three short parts of the Notion page "CDP-Backed Synthetics" that are only questions: Part 5 (who carries the risk), Part 7 (privacy) and Part 8 (market). Each question is kept as the page words it, with the protocol evidence the vault already holds. Written 2026-11-17. A formal fact-check pass has not been run. Part map: [[rfp/appendix/cdp-notion-map]]. Highlight words: "Open decision: ..." (the page's yellow, a decision with no recommendation yet) and "Risk to evaluate: ..." (red).

## Part 5: Who carries the risk

### Questions (from the page)

1. **Open decision: who mints sXMR, and why?** A borrower who mints sXMR is short XMR. What is their reason: hedging, a spread bet against their collateral, or fee income?
2. **Open decision: what is the loss path when a position goes underwater?** Liquidators first, then a first-loss pool, then every holder pro rata? Whether the pool is per branch or shared is Part 4.
3. **Open decision: who liquidates, and can LEZ support them?** That means keepers, bots and an on-chain auction or a first-loss pool.

### Evidence for question 1: who mints a synth, and why

- **No source gives sXMR minters' reasons.** The vault has no data on who minted the one existing sXMR (Synthetix, ERC-20 `0x5299d6F7472DCc137D7f3C4BcfBBB514BaBF341A`, "low volume", price-tracked with Chainlink, minted against SNX staked into the debt pool; [[projects/monero-zcash-synthetics-community-history]]). [DATA NEEDED]: minter addresses or counts; not collected.
- **The short is structural in oracle-priced designs.** In Synthetix V2 a staker's debt is a share of the global pool, so stakers are together short every synth and the counterparty to every synth holder; their incentive is exchange fees in proportion to their share of the pool plus staking rewards ([[projects/synthetix-v2]]). In Metronome, "synth debt tracks the reference asset price, so debt in USD terms rises if the user mints msETH or msBTC and the reference asset rallies" ([[projects/metronome-synth]]).
- **Fee income as a motive.** Synthetix V2 stakers earn exchange fees (5 to 100 bps by category at the end) and rewards; Youves minters earn YOU governance tokens in proportion to minting volume; Mirror incentivised short positions with sLP tokens ([[projects/synthetix-v2]], [[projects/youves]], [[projects/mirror-protocol]]). These are incentives set by each protocol, not measured minter motives.
- **The structure the page assumes.** For sXMR the page treats the minter as the short side. The earlier sXMR pitch notes the same asymmetry from the other direction: "easy to mint sXMR (privacy-curious DeFi users want it), hard to source LPs" ([[rfp/appendix/sxmr-atomic-redemption]], failure mode 3).
- **Collateral-split designs have no borrower.** Hylo's leveraged-token holders play the counterparty role instead and pay a borrow cost ([[projects/hylo]]).

### Evidence for question 2: the loss path

The vault holds the answer protocol by protocol; the comparison, the five-step order the page proposes and the precedent check are in [[rfp/appendix/cdp-isolation]] (angle 5 and "Precedent check"). In short:

| Protocol | Loss path (vault evidence) |
|---|---|
| Liquity V2 | The branch's Stability Pool absorbs first; what it cannot absorb, with its collateral, is redistributed to the other positions on the same collateral ([[projects/liquity]], "Stability Pool and redistribution") |
| Indigo | The pool burns what it holds; the rest of the debt and collateral stay frozen in the position until the pool is refilled ([[projects/indigo]]) |
| Frankencoin | FPS equity first, then the borrowers' reserve, then new unbacked ZCHF (dilution of all holders) ([[projects/frankencoin]]) |
| Sky/Maker | Dutch auctions, then the surplus buffer; Black Thursday ended with debt auctions that minted MKR (20,980 MKR sold for 5.3M DAI) ([[projects/sky-maker]]) |
| Hylo | Leveraged-token holders first; below 100% backing, staked hyUSD in the Earn Pool is burned ([[projects/hylo]]) |
| Synthetix V2 | No separate pool: the borrowers carry everything through the global debt ([[projects/synthetix-v2]]) |
| Metronome | The July 2026 shortfall of about $15.7M to $16M fell on msETH and msUSD holders; msUSD at $0.818 on 2026-11-16 ([[projects/metronome-synth]]) |

### Evidence for question 3: who liquidates

| Protocol | Who liquidates and what they are paid | Source |
|---|---|---|
| Sky/Maker | Keepers bid in Dutch auctions; flat `tip` (docs example 100 DAI) plus `chip` (docs example 2%); bidders can use flash loans. A dormant third-party keeper bot was drained of about $538k (about 200 WETH) on 2026-10-06 through an unprotected function (not a protocol contract failure) | [[projects/sky-maker]] |
| Liquity | The Stability Pool absorbs the debt; V2 pays gas compensation (0.0375 WETH refundable deposit plus min(0.5% of collateral, 2 units of WETH or LST)), V1 a 200 LUSD reserve | [[projects/liquity]] |
| Indigo | Any user can freeze an undercollateralised position and then liquidate it; "automated by bots"; no liquidator reward documented | [[projects/indigo]] |
| Frankencoin | Anyone can challenge a position by posting collateral; the challenger earns 2%; bidders buy at a falling price | [[projects/frankencoin]] |
| crvUSD | LLAMMA soft liquidation by arbitrage through the AMM, then hard liquidation | [[projects/crvusd]] |
| Synthetix V2 | Flag, 12 hour delay, then liquidate; the liquidated SNX goes to other stakers, not the liquidator | [[projects/synthetix-v2]] |
| Metronome | 18% extra premium, of which 10% to the liquidator and 8% to Metronome | [[projects/metronome-synth]] |
| Alchemix V3 | 1.5% liquidator fee | [[patterns/cdp-synthetics-landscape]] |
| Hylo | No liquidation; fee curves restore the ratio | [[projects/hylo]] |

Can LEZ support them? [DATA NEEDED]. The vault holds no LEZ evidence on keepers, bots, permissionless liquidation calls, fee payment for liquidation transactions, an on-chain auction or private-transaction liquidation. What it does hold is in [[rfp/appendix/cdp-admin-switch]] ("What LEZ provides": chained calls, program-derived accounts, a multisig program) and [[rfp/appendix/cdp-oracle]] (private transactions read prices only through push-mode price accounts).

## Part 7: Privacy

Positions are public, because the protocol must be able to check that each one is healthy. That rules out shielded positions and a shielded pool. Working assumption, not taken from a surveyed protocol: holders keep synths in private accounts and move them in and out. Terms: to shield is to move tokens from a public LEZ account to a private one; to unshield is to move them back.

### Question (from the page)

1. **Open decision: can a user interact with the CDP directly from a private account, or must they first move tokens to a public account?** Mint, repay, liquidation and collateral deposit are probably public.

### Evidence

- **No surveyed protocol has shielded positions.** The survey in [[patterns/cdp-synthetics-landscape]] did not look for private positions, and no note in the vault records one. [NOT FOUND] for a precedent; the working assumption is the page's.
- **Why privacy matters to the product.** The Monero and Zcash community record finds the same objection in 2019, 2021 and 2026: "a synthetic on a transparent chain ... does not inherit the privacy properties of native XMR/ZEC, and the wrap/unwrap step is itself a deanonymisation event" ([[projects/monero-zcash-synthetics-community-history]], "The privacy objection is the universal blocker"). The earlier sXMR pitch makes privacy the wedge ("the only synthetic that terminates in a privacy-preserving asset on a privacy chain") and warns that an on-chain orderbook "leaks intent publicly (which undermines the privacy story)" ([[rfp/appendix/sxmr-atomic-redemption]]). The page's assumption (holders keep synths privately, positions are public) is a middle point between those; the vault does not evaluate it.
- **The oracle limits what a private account can read.** Per RFP-020 as recorded on the page, a push-mode price account is the only way a private transaction can read a price; pull mode is public only. Whether a private transaction can call a program that reads public price accounts, and what happens if a price changes between proof and inclusion, is Part 6's open question ([[rfp/appendix/cdp-oracle]], question 1: "Risk to evaluate").
- **LEZ facts on private and public accounts** are not in the vault beyond those two items. [DATA NEEDED]: how a call from a private transaction into a public program works on the LEZ dev branch (commit `cb075d5`); ask the LEZ team.

## Part 8: Market

### Question (from the page)

1. **Risk to evaluate: who holds sXMR before step 2, and why instead of a perp elsewhere?** This is the viability test for price exposure first.

### Evidence

- **Minted price-exposure tokens have not scaled; perps did.** The summary note states: "Synthetic tokens have not scaled. Pure synthetic-asset designs (Synthetix-style) stagnated at $40M-$500M ... while perp venues exceed billions; synthetic dollars (Sky, Ethena, Liquity) are the exception ... Markets prefer perpetual-position exposure over minted synthetic tokens, with sBTC the notable exception" ([[summary]], "Problem data"). Of 50 CDP, debt-pool and collateral-split protocols surveyed, 20 were dead, halted, wound down or abandoned by 2026-11-16 (an editorial count, [[summary]]).
- **XMR already has a perp.** A search on 2026-11-17 found headlines that Hyperliquid listed Monero perpetuals on 2026-01-15, as XMR reached an all-time high of $797 on 2026-01-14 ([Yellow.com](https://yellow.com/news/hyperliquid-lists-monero-perpetuals-as-xmr-hits-dollar797-all-time-high); search summary, page not opened). The same search found no XMR listing on Lighter or Aster; undated volume and open-interest figures from aggregators were not used. The vault's Hyperliquid note was not refreshed in November and has no XMR data ([[projects/hyperliquid]]). [DATA NEEDED]: XMR perp volume and open interest on dated sources, to compare with any sXMR demand.
- **The community has preferred native swaps to synthetics.** Every well-received XMR or ZEC cross-chain integration in the last three years was a native swap; the oracle-priced sXMR on Synthetix has low volume and no recorded Monero-side sentiment ([[projects/monero-zcash-synthetics-community-history]]).
- **The earlier sXMR notes say the same viability question from the supply side.** A synth with a soft, market-clearing peg is "structurally closer to a DEX trading pair"; the demand asymmetry (easy to mint, hard to source XMR LPs) is the first-listed failure mode ([[rfp/appendix/sxmr-atomic-redemption]]).
- **No holder data.** [NOT FOUND]: who holds the existing sXMR or any minted privacy-coin synth; holder counts were not collected.

## Conflicts and gaps

- **Questions only.** These three parts contain no recommendation on the page; this note makes none.
- **Not re-verified in this pass:** the project-note facts in the tables (2026-11-16 reads). The Hyperliquid XMR listing is from search headlines only.
- **Open evidence requests:** LEZ support for keepers and auctions; private-account behaviour; sXMR holders; dated XMR perp volume.

## Sources

- Notion page "CDP-Backed Synthetics", Parts 5, 7 and 8, as saved 2026-11-17.
- [Yellow.com: Hyperliquid lists Monero perpetuals as XMR hits $797 all-time high](https://yellow.com/news/hyperliquid-lists-monero-perpetuals-as-xmr-hits-dollar797-all-time-high) : surfaced in a search on 2026-11-17, not opened
- Vault notes cited in the tables: [[projects/synthetix-v2]], [[projects/metronome-synth]], [[projects/youves]], [[projects/mirror-protocol]], [[projects/hylo]], [[projects/liquity]], [[projects/indigo]], [[projects/frankencoin]], [[projects/sky-maker]], [[projects/crvusd]], [[patterns/cdp-synthetics-landscape]], [[projects/monero-zcash-synthetics-community-history]], [[rfp/appendix/sxmr-atomic-redemption]], [[summary]]
