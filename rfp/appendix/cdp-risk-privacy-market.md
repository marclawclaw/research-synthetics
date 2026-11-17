---
tags: [appendix, cdp, lez, liquidation, privacy, market, evidence]
related: [rfp/appendix/cdp-notion-map, rfp/appendix/cdp-isolation, rfp/appendix/cdp-oracle, rfp/appendix/sxmr-atomic-redemption, projects/monero-zcash-synthetics-community-history]
---

# CDP on LEZ: evidence on who carries the risk, privacy and market

Evidence for three parts of the Notion page "CDP-Backed Synthetics": Part 5 (who carries the risk), Part 7 (privacy) and Part 8 (market). The page holds the questions and any decisions; this note holds only the protocol evidence the vault has on each topic. Part map: [[rfp/appendix/cdp-notion-map]]. Written 2026-11-17; fact-checked and corrected the same day.

## Part 5: who carries the risk

### Who mints a synth, and why

- **No source gives sXMR minters' reasons.** The vault has no data on who minted the one existing sXMR (Synthetix, ERC-20 `0x5299d6F7472DCc137D7f3C4BcfBBB514BaBF341A`, "low volume", price-tracked with Chainlink, minted against SNX staked into the debt pool; [[projects/monero-zcash-synthetics-community-history]]). [DATA NEEDED]: minter addresses or counts; not collected.
- **The short is structural in oracle-priced designs.** In Synthetix V2 a staker's debt is a share of the global pool, so stakers are together short every synth and the counterparty to every synth holder; their incentive is exchange fees in proportion to their share of the pool plus staking rewards ([[projects/synthetix-v2]]). In Metronome, "synth debt tracks the reference asset price, so debt in USD terms rises if the user mints msETH or msBTC and the reference asset rallies" ([[projects/metronome-synth]]).
- **Fee income as a motive.** Synthetix V2 stakers earn exchange fees and rewards; in the last fee regime (SIP-222, 2022) each synth carried a fee, summed for the two synths of a trade: on L1 from 0 (sUSD) and 15 bps (forex) up to 100 bps, on L2 up to 35 bps ([[projects/synthetix-v2]], Fees). Youves minters earn YOU governance tokens in proportion to minting volume; Mirror incentivised short positions with sLP tokens ([[projects/youves]], [[projects/mirror-protocol]]). These are incentives set by each protocol, not measured minter motives.
- **The asymmetry from the supply side.** The earlier sXMR pitch: "easy to mint sXMR (privacy-curious DeFi users want it), hard to source LPs" ([[rfp/appendix/sxmr-atomic-redemption]], failure mode 3).
- **Collateral-split designs have no borrower.** Hylo's leveraged-token holders play the counterparty role instead and pay a borrow cost ([[projects/hylo]]).

### The loss path

The protocol comparison and the first-loss table are in [[rfp/appendix/cdp-isolation]]. In short:

| Protocol | Loss path (vault evidence) |
|---|---|
| Liquity V2 | The branch's Stability Pool absorbs first; what it cannot absorb, with its collateral, is redistributed to the other positions on the same collateral ([[projects/liquity]], "Stability Pool and redistribution") |
| Indigo | The pool burns what it holds; the rest of the debt and collateral stay frozen in the position until the pool is refilled ([[projects/indigo]]) |
| Frankencoin | FPS equity first, then the borrowers' reserve, then new unbacked ZCHF (dilution of all holders) ([[projects/frankencoin]]) |
| Sky/Maker | Today: Dutch auctions (Liquidation 2.0), then the surplus buffer. On Black Thursday (March 2020) liquidations ran on the earlier English-style "flip" auctions (Liquidation 1.0); the shortfall was covered by debt auctions that minted MKR (20,980 MKR sold for 5.3M DAI) ([[projects/sky-maker]]) |
| Hylo | Leveraged-token holders first; below 100% backing, staked hyUSD in the Earn Pool is burned ([[projects/hylo]]) |
| Synthetix V2 | No separate pool: the borrowers carry everything through the global debt ([[projects/synthetix-v2]]) |
| Metronome | The July 2026 disclosure left about 6,367 msETH (31% of supply) and 4.57M msUSD (16%) unbacked, about $15.7M to $16M; the source does not say who bears the loss, and cites $34M of defensive positions and $6.5M of protocol-owned liquidity, with recapitalisation or supply reduction as possible paths ([Crypto Briefing](https://cryptobriefing.com/metronome-dao-16m-oracle-shortfall/), 2026-07-31). msUSD at $0.816 on 2026-11-17 ([CoinGecko](https://www.coingecko.com/en/coins/metronome-synth-usd)) ([[projects/metronome-synth]]) |

### Who liquidates

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

LEZ support for liquidators: [DATA NEEDED]. The vault holds no LEZ evidence on keepers, bots, permissionless liquidation calls, fee payment for liquidation transactions, an on-chain auction or private-transaction liquidation. What it does hold is in [[rfp/appendix/cdp-admin-switch]] ("What LEZ provides": chained calls, program-derived accounts, a multisig program) and [[rfp/appendix/cdp-oracle]] (private transactions read prices only through push-mode price accounts).

## Part 7: privacy

Terms: to shield is to move tokens from a public LEZ account to a private one; to unshield is to move them back.

- **No surveyed protocol has shielded positions.** The survey in [[patterns/cdp-synthetics-landscape]] did not look for private positions, and no note in the vault records one. [NOT FOUND] for a precedent.
- **Why privacy matters to the product.** The Monero and Zcash community record finds the same objection in 2019, 2021 and 2026: "a synthetic on a transparent chain ... does not inherit the privacy properties of native XMR/ZEC, and the wrap/unwrap step is itself a deanonymisation event" ([[projects/monero-zcash-synthetics-community-history]], "The privacy objection is the universal blocker"). The earlier sXMR pitch makes privacy the wedge ("the only synthetic that terminates in a privacy-preserving asset on a privacy chain") and warns that an on-chain orderbook "leaks intent publicly (which undermines the privacy story)" ([[rfp/appendix/sxmr-atomic-redemption]]).
- **The oracle limits what a private transaction can read.** RFP-020 (lines 64 to 70 and 183 to 195, `logos-co/rfp` at `920fad9`): a push-mode price account is the only way a private transaction can read a price; pull mode is public only ([[rfp/appendix/cdp-oracle]]).
- **A private transaction calling a public program (LEZ `dev` at `cb075d5`).** The proof carries predicted messages between private and public actors and assumptions on the public context; at inclusion the public execution must match them, or the transaction fails with `BoundaryMismatch` or `CrossMessageMismatch` (`lee/state_machine/core/src/execution_state.rs` lines 103 to 108 and 280 to 285; `circuit_io.rs` line 188). Inference: a public state change between proving and inclusion (such as a new price) makes the transaction fail. Not tested for a CDP or price read; confirm with the LEZ team. Detail in [[rfp/appendix/cdp-oracle]].

## Part 8: market

- **Minted price-exposure tokens have not scaled; perps did.** The summary note: "Pure synthetic-asset designs (Synthetix-style) stagnated at $40M-$500M ... while perp venues exceed billions; synthetic dollars (Sky, Ethena, Liquity) are the exception" ([[summary]], "Problem data"). The $40M to $500M range is from the 2026-05-20 baseline and was not refreshed in November, so it is about six months old. Of 50 CDP, debt-pool and collateral-split protocols surveyed, 20 were dead, halted, wound down or abandoned by 2026-11-16 (an editorial count, [[patterns/cdp-synthetics-landscape]]).
- **XMR already trades as a perp on several venues.** Read from each venue's public API on 2026-11-17 (about 05:35 UTC; XMR about $500):

| Venue | Market | 24h volume | Open interest | Source |
|---|---|---|---|---|
| Hyperliquid | `XMR` perp, up to 5x | $49.98M | 182,529 XMR (about $91.4M at the $501.0 mark) | [Hyperliquid info API](https://api.hyperliquid.xyz/info) (`metaAndAssetCtxs`) |
| Lighter | `XMR` perp, market 77, created 2025-09-08 | $6.03M | about $9.67M | [Lighter API](https://mainnet.zklighter.elliot.ai/api/v1/orderBookDetails?market_id=77) |
| Aster | `XMRUSDT` perp | $0.93M | 1,270 XMR (about $0.64M) | [Aster ticker](https://fapi.asterdex.com/fapi/v1/ticker/24hr?symbol=XMRUSDT), [open interest](https://fapi.asterdex.com/fapi/v1/openInterest?symbol=XMRUSDT) |

- **Hyperliquid's first XMR listing.** Yellow.com (2026-01-16) reports Monero perpetuals "through Felix Protocol's HIP-3 deployment", up to 5x leverage, as XMR reached an all-time high of $797 on 2026-01-14 ([Yellow.com](https://yellow.com/news/hyperliquid-lists-monero-perpetuals-as-xmr-hits-dollar797-all-time-high)). The article gives no listing date; 2026-01-15 comes from search summaries only. The vault's Hyperliquid note dates from May 2026 and has no XMR data ([[projects/hyperliquid]]).
- **The community has preferred native swaps to synthetics.** Every well-received XMR or ZEC cross-chain integration in the last three years was a native swap; the oracle-priced sXMR on Synthetix has low volume and no recorded Monero-side sentiment ([[projects/monero-zcash-synthetics-community-history]]).
- **The earlier sXMR notes raise the same viability question from the supply side.** A synth with a soft, market-clearing peg is "structurally closer to a DEX trading pair"; the demand asymmetry (easy to mint, hard to source XMR LPs) is failure mode 3 ([[rfp/appendix/sxmr-atomic-redemption]]).
- **No holder data.** [NOT FOUND]: who holds the existing sXMR or any minted privacy-coin synth; holder counts were not collected.

## Conflicts and gaps

- **Not re-verified in this pass:** the project-note facts in the loss-path and liquidation tables (2026-11-16 reads).
- **Stale inputs:** the summary's $40M to $500M range (May 2026); the Hyperliquid, dYdX and sBTC project notes (May 2026).
- **Open evidence requests:** LEZ support for keepers and auctions; a tested private-to-public call on LEZ; sXMR holders and minters.

## Sources

- Notion page "CDP-Backed Synthetics", Parts 5, 7 and 8, as saved 2026-11-17.
- [Crypto Briefing: MetronomeDAO 16M shortfall](https://cryptobriefing.com/metronome-dao-16m-oracle-shortfall/) (2026-07-31); [CoinGecko msUSD](https://www.coingecko.com/en/coins/metronome-synth-usd) : fetched 2026-11-17
- [Yellow.com: Hyperliquid lists Monero perpetuals as XMR hits $797 all-time high](https://yellow.com/news/hyperliquid-lists-monero-perpetuals-as-xmr-hits-dollar797-all-time-high) : opened 2026-11-17, dated 2026-01-16
- Venue APIs (Hyperliquid, Lighter, Aster) : fetched 2026-11-17, links in the table
- `logos-co/rfp` at `920fad9` (RFP-020); `logos-execution-zone` `dev` at `cb075d5` : local clones, read 2026-11-17
- Vault notes cited in the tables: [[projects/synthetix-v2]], [[projects/metronome-synth]], [[projects/youves]], [[projects/mirror-protocol]], [[projects/hylo]], [[projects/liquity]], [[projects/indigo]], [[projects/frankencoin]], [[projects/sky-maker]], [[projects/crvusd]], [[patterns/cdp-synthetics-landscape]], [[projects/monero-zcash-synthetics-community-history]], [[rfp/appendix/sxmr-atomic-redemption]], [[summary]]
