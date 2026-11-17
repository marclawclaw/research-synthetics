---
tags: [appendix, cdp, lez, oracle, redstone, pyth, rfp-019, rfp-020, evidence]
related: [rfp/appendix/cdp-notion-map, rfp/appendix/cdp-admin-switch, patterns/collateral-debt-units, patterns/oracle-pricing-model, rfp/appendix/cdp-isolation]
---

# CDP on LEZ: oracle evidence

Evidence for Part 6 ("Oracle") of the Notion page "CDP-Backed Synthetics". The page holds the decisions; this note holds only the facts behind them: what RFP-019 and RFP-020 say, which providers publish XMR/USD, what LEZ does when a private transaction calls a public program, and how existing protocols combine several oracles. Part map: [[rfp/appendix/cdp-notion-map]]. Written 2026-11-17; fact-checked and corrected the same day.

Terms: an **oracle provider** (such as RedStone) publishes signed prices; a **feed** is one price, such as XMR/USD.

## What RFP-019 and RFP-020 say

Read from the local clone of `logos-co/rfp` at `920fad9` (2026-11-06, still `origin/master` on 2026-11-17).

- **Price account standard (RFP-019, Functionality 4, lines 276 to 283).** The struct must include at minimum `base_asset`, `quote_asset`, price, timestamp, source identifier and confidence interval ("where the source provides one; zero otherwise"); the price is base per quote, and consumers "must verify" the asset fields before reading the price.
- **Day-one feeds (RFP-020, lines 48 to 50, 88 to 89, 474 to 475).** BTC/USD, ETH/USD, SOL/USD, XMR/USD and ZEC/USD, registered and exercised on LEZ devnet or testnet. Verification runs in program code inside the RISC-V zkVM, with no bridge and no Wormhole dependency (lines 51 to 53).
- **Signature threshold (RFP-020, line 334 and line 495).** "plain m-of-N secp256k1 ECDSA over keccak256 (typically 3-of-N)"; the threshold is configured per feed, in the tuple `(dataServiceId, feedId, authorised signer set, M-of-N threshold, maxAge)`.
- **Push mode is the only path for private reads (RFP-020, lines 64 to 70 and 183 to 195).** Pull-mode reads from inside a private transaction "are not feasible on consumer hardware" (several minutes of proving per read); private programs "compose with the feed by reading the public price account written by the push aggregator". Pull mode is public only.
- **Staleness (RFP-020, lines 463 to 464).** Packages older than a configurable `maxAge` are rejected.
- **Admin authority (RFP-020, lines 471 to 474; dependency on RFP-001, line 7).** It registers feeds (asset identifier, M-of-N threshold, signer set), updates a feed's signer set when RedStone's roster changes, and deregisters feeds.
- **Why RedStone first (RFP-020, lines 125 to 132).** "RedStone is the only one that combines: support for both XMR and ZEC in its public token registry, a portable connector pattern ..., no cross-chain bridge requirement, and a self-serve deployment path". The same paragraph says "Pyth covers both feeds and adds higher publisher counts and confidence intervals, but is gated on Wormhole integration on LEZ". It does not say RedStone is the only provider with XMR and ZEC.
- **Pyth publisher count (RFP-020, lines 342 to 344).** "roughly 80+ on XMR/USD versus RedStone's smaller per-feed roster".

## Who publishes XMR/USD

| Provider | XMR/USD evidence | Checked |
|---|---|---|
| Pyth | Live feed `Crypto.XMR/USD` ("MONERO / US DOLLAR", id `46b8cc93...c321d`) returned by the [Hermes API](https://hermes.pyth.network/v2/price_feeds?query=XMR&asset_type=crypto); also listed in `logos-co/rfp` `appendix/oracle-ecosystem.md` line 1361. Publisher count: RFP-020 says about 80; not read live | 2026-11-17 |
| RedStone | Listed as `XMR` (line 45) and `ZEC` (line 26) in the [`ALL_SUPPORTED_TOKENS.md` registry](https://github.com/redstone-finance/redstone-api/blob/main/docs/ALL_SUPPORTED_TOKENS.md), last changed 2021-07-31 (GitHub commit history); cited by `oracle-ecosystem.md` lines 1362 and 1529 to 1530. A 2021 list does not show a live data package today; no live RedStone XMR feed was found by search, and RedStone's gateway was not queried. RedStone's [price feeds page](https://www.redstone.finance/price-feeds/) says it supports 1,300+ assets | 2026-11-17 |
| Others | Per `oracle-ecosystem.md` (lines 1355 to 1365, not re-checked): Chainlink on Optimism and Polygon (permissioned), Chainlink Data Streams (paid), DIA Lumina (permissionless, per-chain deployment), Supra (team engagement), Chronicle (not in its public catalogue) | Not re-checked |

## LEZ: a private transaction that calls a public program

Read in `logos-execution-zone` on branch `dev` at `cb075d5` (2026-11-13). Not confirmed with the LEZ team.

- A private or hybrid transaction's proof carries "predicted messages between private and public actors" (`WholeExecutionOutcome.predicted_cross_messages`, `lee/state_machine/core/src/execution_state.rs` lines 103 to 108 and 141) and "assumptions on the public environment context for the proof" (`public_context`, `lee/state_machine/core/src/circuit_io.rs` line 188).
- When the transaction is included, public execution must match those predictions, or it fails with `BoundaryMismatch` ("Boundary step {index} does not match the execution") or `CrossMessageMismatch` ("The cross message at boundary step {index} does not match the executed delivery") (`execution_state.rs` lines 280 to 285).
- Inference: if a public price changes between proving and inclusion, the transaction fails rather than settling at the old price.

## Existing designs that combine several oracles

Tags: **chain** and **source** as in the project notes; **docs** = project documentation.

| Pattern | Protocol and design | Source in the vault |
|---|---|---|
| Fallback | **Liquity V1:** Chainlink ETH/USD first, Tellor as fallback; 4 hour timeout per oracle; Chainlink untrusted after a jump of more than 50% between rounds unless Tellor agrees within 5%; return to Chainlink only when both are live and within 5% of each other; if both fail, the last good price is kept and the system keeps running (no shutdown) | [[projects/liquity]] (V1 PriceFeed, chain and source) |
| Single primary | **Drift, before its hack:** Pyth as primary with per-market configuration; a configured fallback is stated only in [[patterns/oracle-pricing-model]], not in the project note. The 2026-04-01 loss bypassed the oracle path (an attacker-controlled oracle for a newly listed asset) | [[projects/drift]]; [[patterns/oracle-pricing-model]] |
| Fallback | **Lighter:** when external feeds go stale (for example RWA markets outside trading hours, threshold typically 1 minute) pricing switches to an orderbook-derived internal price with a 30 minute EMA; pre-IPO and prelaunch markets use only the internal price | [[projects/lighter]] |
| Cross-check | **Jupiter Perps:** Edge by Chaos Labs is primary; its price is accepted only if fresh and within a threshold of both Chainlink and Pyth; otherwise Chainlink and Pyth are compared and the more recent used if they agree within a threshold; if two or more oracles fail, no price update occurs | [[projects/jupiter-perps]] (docs) |
| Median | **Lighter mark price:** the median of the orderbook impact price, an index plus an 8 minute EMA premium, and a median of centralised-exchange marks; the index comes from Chainlink, Stork and Pyth | [[projects/lighter]] |
| Co-signed value | **Maker:** the ETH/USD source is a Chronicle Scribe feed: 13 of 25 signers co-sign one value (`bar()` = 13), with an optimistic challenge period of 600 s; no on-chain median. Read behind a 1 hour delay (the OSM). One provider with many signers, not several providers | [[projects/sky-maker]] (chain); [Scribe docs](https://github.com/chronicleprotocol/scribe/blob/main/docs/Scribe.md) |
| Aggregation | **Synthetix V3 Oracle Manager (retired June 2026):** market creators assemble feeds, for example the minimum of Chainlink, Pyth and a Uniswap TWAP | [[patterns/oracle-pricing-model]]; [[projects/synthetix-v3]] |
| Composite | **Liquity V2, wstETH and rETH:** market price times the token's exchange rate; borrow and liquidation use the lower of market and canonical price, redemption the higher if within 1% (stETH/USD against ETH/USD) or 2% (rETH market against canonical) | [[projects/liquity]]; [[patterns/collateral-debt-units]] section 3.7 |
| Composite | **crvUSD:** a pool EMA clamped to a Chainlink band (plus or minus 1.5%, `BOUND_SIZE` 0.015) that the DAO or Emergency DAO can switch on or off; the switch reads off (`use_chainlink` false) on the WBTC and wstETH oracles, so the clamp is not applied there; no age check on the Chainlink read | [[projects/crvusd]] (chain); [[patterns/collateral-debt-units]] section 2 |
| Composite | **RAI:** Uniswap RAI/ETH TWAP times an ETH/USD converter feed, for the rate controller only | [[patterns/collateral-debt-units]] section 2 |
| Single provider | **GMX V2** (Chainlink Data Streams); **Synthetix V2** and **Metronome** (Chainlink); **Variational** (in-house, no external cross-check described) | [[projects/gmx-v2]]; [[projects/synthetix-v2]]; [[projects/metronome-synth]]; [[projects/variational]] |

What the table shows:

- **Medians.** The vault's median examples are Lighter (a median of different price kinds). Maker's Chronicle feed is a threshold of co-signers on one value, not a median. No surveyed CDP takes a median over several independent providers. A median needs at least three inputs to resist one bad input.
- **Missing inputs.** No update at all when two or more of three oracles fail (Jupiter); fall back to an internal price (Lighter); keep the last good price and continue (Liquity V1) or shut the branch down (Liquity V2).
- **Stale inputs.** Per-feed limits from 60 seconds (Mirror) to 72 hours (JPY DeFi), 4 hours for Liquity V1, 24 hours for Liquity V2's ETH/USD and 48 hours for rETH/ETH ([[patterns/collateral-debt-units]] section 4, item 3).

## Precedents

| Fact | Basis |
|---|---|
| CDP governance does not hold the provider's signer set | Sky does not control Chronicle's signers ([[projects/sky-maker]], Oracle row); Synthetix V2's Safe picks the Chainlink feed and stale period only ([[projects/synthetix-v2]]) |
| Consumers apply their own maximum age on top of the provider's | Liquity V1 and V2 set their own timeouts per feed; Mirror applies 60 seconds at the consumer ([[patterns/collateral-debt-units]]) |
| Aggregating several providers | Synthetix V3 Oracle Manager aggregated feeds; Jupiter cross-checks three; no surveyed CDP takes a median over providers |
| The provider's admin is an inherited admin | RFP-020's admin authority updates signer sets for every registered feed (lines 471 to 474); Liquity inherits Chainlink's 4 of 9 multisig ([[rfp/appendix/cdp-admin-switch]]) |

## Conflicts and gaps

- **RedStone XMR/USD live status.** The only evidence is the 2021 registry; whether a live XMR data package exists today was not established. RedStone's MCP server has a `get_supported_assets` tool that could settle it; not queried.
- **Pyth XMR/USD publisher count.** The feed exists; RFP-020's "roughly 80+" publishers was not read live ([Pyth publishers](https://insights.pyth.network/publishers) not opened).
- **RFP-019 status.** A [LEZ roadmap entry](https://roadmap.vac.dev/sc/logos/2026q2-twap-oracle) was reported by search as "in progress (2026 Q2)"; the page returned 502 and then failed to connect on 2026-11-17, so the status is unverified and, at over four months old, stale.
- **LEZ private calls.** The reading above is from the `dev` branch and is not tested end to end for a price read; confirm with the LEZ team.
- **Not checked:** a RedStone feed for a Logos token with no outside market (see [[patterns/collateral-debt-units]] section 4.1).

## Sources

- Notion page "CDP-Backed Synthetics", Part 6, as saved 2026-11-17.
- `logos-co/rfp` at `920fad9`: `RFPs/RFP-019-twap-oracle.md`, `RFPs/RFP-020-redstone-oracle-adaptor.md`, `appendix/oracle-ecosystem.md` (local clone, read 2026-11-17). Web: [RFP-019](https://github.com/logos-co/rfp/blob/master/RFPs/RFP-019-twap-oracle.md), [RFP-020](https://github.com/logos-co/rfp/blob/master/RFPs/RFP-020-redstone-oracle-adaptor.md)
- [Pyth Hermes price feeds, XMR](https://hermes.pyth.network/v2/price_feeds?query=XMR&asset_type=crypto) : fetched 2026-11-17
- [RedStone ALL_SUPPORTED_TOKENS.md](https://github.com/redstone-finance/redstone-api/blob/main/docs/ALL_SUPPORTED_TOKENS.md) : fetched 2026-11-17, last commit 2021-07-31
- [RedStone price feeds](https://www.redstone.finance/price-feeds/) : fetched 2026-11-17 ("1,300+ assets")
- [Chronicle Scribe docs](https://github.com/chronicleprotocol/scribe/blob/main/docs/Scribe.md)
- `logos-execution-zone` `dev` at `cb075d5`: `lee/state_machine/core/src/execution_state.rs`, `lee/state_machine/core/src/circuit_io.rs` (local clone, read 2026-11-17)
- Vault notes cited in the table: [[projects/liquity]], [[projects/drift]], [[projects/lighter]], [[projects/jupiter-perps]], [[projects/sky-maker]], [[projects/synthetix-v3]], [[projects/crvusd]], [[projects/gmx-v2]], [[projects/synthetix-v2]], [[projects/metronome-synth]], [[projects/variational]], [[patterns/oracle-pricing-model]], [[patterns/collateral-debt-units]]
