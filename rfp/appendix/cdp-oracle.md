---
tags: [appendix, cdp, lez, oracle, redstone, rfp-019, rfp-020, proxy-oracle]
related: [rfp/appendix/cdp-notion-map, rfp/appendix/cdp-admin-switch, patterns/collateral-debt-units, patterns/oracle-pricing-model, rfp/appendix/cdp-isolation]
---

# CDP on LEZ: the oracle interface and what is out of scope

Working note for the synth CDP program on LEZ (sXMR and sBTC are the examples). It holds the evidence behind Part 6 ("Oracle") of the Notion page "CDP-Backed Synthetics" ([[rfp/appendix/cdp-notion-map]]). The page's decisions are in the first two sections; the rest is evidence: what RFP-019 and RFP-020 say as already recorded in the vault, how existing protocols combine several oracles, and the gaps. Written 2026-11-17. A formal fact-check pass has not been run on this note.

Terms (as on the page): an **oracle provider** (such as RedStone) publishes signed prices; a **feed** is one price, such as XMR/USD. Highlight words: "Recommendation: ...", "Open decision: ..." and "Risk to evaluate: ..." stand for the page's green, yellow and red.

## What the page decides

The CDP reads prices through one interface and does not care where they come from. Which provider supplies them is the deployer's choice. RedStone, delivered by [RFP-020: RedStone Off-Chain Oracle Adaptor for LEZ](https://github.com/logos-co/rfp/blob/master/RFPs/RFP-020-redstone-oracle-adaptor.md) (assumed to be ready by 0.4), is the one provider assumed for now.

What this RFP asks for:

- **An oracle interface.** For each pair it needs, the CDP reads a price account in the standard format from [RFP-019](https://github.com/logos-co/rfp/blob/master/RFPs/RFP-019-twap-oracle.md): base and quote asset, price, timestamp, source identifier, confidence interval. It checks the pair and applies its own maximum age.
- **A role that chooses the source.** The Oracle role pins which price account each pair reads, because one oracle may not support every price. The signer set belongs to the oracle provider, not to this role ([[rfp/appendix/cdp-admin-switch]], roles table).
- **A median proxy.** A separate Proxy Oracle program reads one to n price accounts for the same pair and publishes their median in the same format, so to the CDP it is one more source. Whether it writes an account or computes on read, and how it treats missing or stale inputs, is left to the implementor, who can follow what other CDPs do. A deployer who wants fallbacks or other logic writes their own proxy.

Everything else about the oracle is out of scope for this RFP except that interface and the median Proxy Oracle.

## Out of scope: RedStone and the deployer's choices

What RFP-020 delivers, in short (as on the page; per RFP-020, not confirmed in RedStone's own docs):

- Day-one BTC/USD, ETH/USD, SOL/USD, XMR/USD and ZEC/USD feeds, verified inside a LEZ program (RedStone signatures, M of N signers, 3 of N by default), with no bridge.
- **Push mode:** a relayer submits signed prices to an aggregator program that writes a public price account in the format set by RFP-019. This is the only way a private transaction can read a price.
- **Pull mode, public transactions only:** the consumer carries the signed price and verifies it inline.
- A configurable `maxAge` per feed, with stale packages rejected.
- An admin authority (per RFP-001) that registers feeds and updates signer sets.

Left to RFP-020 or to the deployer who picks a provider:

- Who runs the relayer, and who pays for updates.
- Push or pull for each operation. Operations run from private accounts can only read a push-mode price account.
- Freshness rules on top of the maximum age, such as both legs within one window, and what freezes when a feed stops.
- The depth of the provider's XMR/USD feed. RFP-020 notes RedStone has fewer publishers per feed than Pyth, which has about 80 on XMR/USD; Pyth is planned as a later RFP.
- The admin authority that comes with whichever provider a deployer picks. It can change the signer set of every feed the CDP reads, whatever mode the Oracle role is in ([[rfp/appendix/cdp-admin-switch]]).

## Questions

1. **Risk to evaluate: can a private transaction call the Proxy Oracle and have it read public price accounts, and what happens if a price changes between proof and inclusion?** In the current LEZ dev branch (commit `cb075d5`) a call from a private transaction into a public program looks like a message whose result the proof assumes. This is not confirmed; ask the LEZ team. (LEZ reading in [[rfp/appendix/cdp-admin-switch]], "What LEZ provides"; the privacy question is in [[rfp/appendix/cdp-risk-privacy-market]].)

## Existing designs that combine several oracles

The Proxy Oracle's behaviour on missing or stale inputs is left to implementors "who can follow what other CDPs do". This table collects what the vault already holds about multi-oracle designs. Tags: **chain** and **source** as in the project notes; **docs** = project documentation. It is evidence for implementors, not a recommendation, and none of these designs was re-verified in this pass.

| Pattern | Protocol and design | Source in the vault |
|---|---|---|
| Fallback | **Liquity V1:** Chainlink ETH/USD first, Tellor as fallback; 4 hour timeout per oracle; Chainlink untrusted after a jump of more than 50% between rounds unless Tellor agrees within 5%; return to Chainlink only when both are live and within 5% of each other; if both fail, the last good price is kept and the system keeps running (no shutdown) | [[projects/liquity]] (V1 PriceFeed, chain and source) |
| Fallback | **Drift, before its hack:** Pyth as primary with a configurable fallback (the 2026-04-01 loss bypassed the oracle path through admin takeover) | [[patterns/oracle-pricing-model]]; [[projects/drift]] |
| Fallback | **Lighter:** when external feeds go stale (for example RWA markets outside trading hours, threshold typically 1 minute) pricing switches to an orderbook-derived internal price with a 30 minute EMA; pre-IPO and prelaunch markets use only the internal price | [[projects/lighter]] |
| Cross-check | **Jupiter Perps:** Edge by Chaos Labs is primary; its price is accepted only if fresh and within a threshold of both Chainlink and Pyth; otherwise Chainlink and Pyth are compared and the more recent used if they agree within a threshold; if two or more oracles fail, no price update occurs. Primary-with-fallback, not an equal three-way aggregation | [[projects/jupiter-perps]] (docs) |
| Median | **Lighter mark price:** the median of the orderbook impact price, an index plus an 8 minute EMA premium, and a median of centralised-exchange marks; the index comes from Chainlink, Stork and Pyth | [[projects/lighter]] |
| Median | **Maker:** the ETH/USD source is a Median of 13 of 25 Chronicle signers, read behind a 1 hour delay (the OSM): one provider with many signers, not several providers | [[projects/sky-maker]] (chain) |
| Aggregation | **Synthetix V3 Oracle Manager (retired June 2026):** market creators assemble feeds, for example the minimum of Chainlink, Pyth and a Uniswap TWAP | [[patterns/oracle-pricing-model]]; [[projects/synthetix-v3]] |
| Composite | **Liquity V2, wstETH and rETH:** market price times the token's exchange rate; borrow and liquidation use the lower of market and canonical price, redemption the higher if within 1% (stETH/USD against ETH/USD) or 2% (rETH market against canonical) | [[projects/liquity]]; [[patterns/collateral-debt-units]] section 3.7 |
| Composite | **crvUSD:** a pool EMA clamped to a Chainlink band (plus or minus 1.5%), which the DAO or Emergency DAO can switch on or off; no age check on the Chainlink read | [[projects/crvusd]]; [[patterns/collateral-debt-units]] section 2 |
| Composite | **RAI:** Uniswap RAI/ETH TWAP times an ETH/USD converter feed, for the rate controller only | [[patterns/collateral-debt-units]] section 2 |
| Single provider | **GMX V2** (Chainlink Data Streams); **Synthetix V2** and **Metronome** (Chainlink); **Variational** (in-house, no external cross-check described) | [[projects/gmx-v2]]; [[projects/synthetix-v2]]; [[projects/metronome-synth]]; [[projects/variational]] |

What this says about the Proxy Oracle's open points (inference, for the implementor):

- **Median of one to n accounts.** A median needs at least three inputs to resist one bad input; with one or two it reduces to that input or to a midpoint. The vault's median examples are Lighter (a median of different price kinds) and Maker (a median of signers inside one provider); no surveyed CDP medians several independent providers.
- **Missing inputs.** Seen: no update at all when two or more of three oracles fail (Jupiter); fall back to an internal price (Lighter); keep the last good price and continue (Liquity V1) or shut the branch down (Liquity V2).
- **Stale inputs.** Seen: per-feed limits from 60 seconds (Mirror) to 72 hours (JPY DeFi), 4 hours for Liquity V1, 24 hours for Liquity V2's ETH/USD and 48 hours for rETH/ETH ([[patterns/collateral-debt-units]] section 4, item 3). The page keeps the age check in the CDP itself, so a proxy that writes a fresh timestamp on stale inputs would hide staleness; whether the proxy carries the oldest input's timestamp is an implementor's choice (inference).
- **Write versus compute.** The page leaves open whether the proxy writes an account or computes on read. RFP-020 says a push-mode price account is the only way a private transaction can read a price; whether a private transaction can call a compute-on-read proxy is the open question 1 above.

## Precedent check

Source: the Notion-session precedent audit (agent-reported; it did not check the LEZ repository or the RFP texts), plus this pass.

| Item | Status | Basis |
|---|---|---|
| Oracle role holds the signer set | Novel, and not what the page decided | Sky does not control Chronicle's signers ([[projects/sky-maker]], Oracle row); Synthetix V2's Safe picks the Chainlink feed and stale period only ([[projects/synthetix-v2]]). The page instead pins which price account each pair reads |
| CDP applies its own maximum age on top of the provider's `maxAge` | Backed in practice | Liquity V1 and V2 set their own timeouts per feed; Mirror applies 60 seconds at the consumer ([[patterns/collateral-debt-units]]) |
| A median Proxy Oracle as one more source | Partly seen | Medians exist (Lighter, Maker) but none over several providers; Synthetix V3 Oracle Manager aggregated feeds |
| RedStone publishes XMR/USD | Unclear | A web search found no XMR/USD feed in RedStone's docs; only RFP-020 asserts it |
| Pyth has about 80 publishers on XMR/USD | Not verified | A web search in this pass found no evidence that Pyth publishes an XMR/USD feed at all; the figure is RFP-020's claim |

## Conflicts and gaps

- **RedStone XMR/USD.** RFP-020 asserts day-one XMR/USD and ZEC/USD feeds. Search results on 2026-11-17 did not return a RedStone feed listing; a search-result summary of the RFP's web mirror says RedStone is "the only off-chain provider it surveyed that supports both XMR and ZEC in its public token registry". RedStone's own page says it supports over 1,300 assets, and its MCP server has a `get_supported_assets` tool that could settle the question; not queried. Check this with RedStone, as the page says (Part 2, item 9: [[patterns/collateral-debt-units]]).
- **Pyth XMR/USD.** A web search in this pass found no Pyth XMR/USD feed; RFP-020's "about 80 publishers" was not verified. A search summary of the RFP web mirror adds that Pyth brings higher publisher counts and confidence intervals but is gated on Wormhole integration on LEZ.
- **RFP-019 and RFP-020 texts.** The field list of the price account (base and quote asset, price, timestamp, source identifier, confidence interval) and the RFP-020 facts above are as recorded on the page and in [[rfp/appendix/cdp-admin-switch]] (read from `logos-co/rfp` at `920fad9`, 2026-11-06, in an earlier session). This pass could not re-read them: there is no local clone of `logos-co/rfp`, and the RFP web mirror (logos-co-web.vercel.app) returned 404 when fetched directly. A search summary describes RFP-019 as an on-chain TWAP oracle that records observations from LEZ DEX pools (RFP-004) and exposes geometric-mean prices through a canonical price account standard that adaptors such as RedStone also fill; a LEZ roadmap entry lists the TWAP oracle as in progress (2026 Q2). Repository to read when a clone is available: `logos-co/rfp`, files `RFPs/RFP-019-twap-oracle.md` and `RFPs/RFP-020-redstone-oracle-adaptor.md`.
- **Out of scope by decision:** the provider's feeds, relayer, update funding, push or pull choice and the admin authority that comes with the provider. The only inherited admin the page names is that authority.
- **Not checked:** a RedStone feed for a Logos token with no outside market (see [[patterns/collateral-debt-units]] section 4.1); whether the median proxy needs its own admin authority.

## Sources

- Notion page "CDP-Backed Synthetics", Part 6, as saved 2026-11-17.
- [RFP-019](https://github.com/logos-co/rfp/blob/master/RFPs/RFP-019-twap-oracle.md) and [RFP-020](https://github.com/logos-co/rfp/blob/master/RFPs/RFP-020-redstone-oracle-adaptor.md) : as recorded on the page; read 2026-11-06 at `920fad9` in an earlier session; not re-read
- [RedStone services](https://www.redstone.finance/services/) : surfaced in a search on 2026-11-17, not opened; [Pyth publishers](https://insights.pyth.network/publishers) : surfaced in a search on 2026-11-17, not opened
- [LEZ roadmap: TWAP oracle](https://roadmap.vac.dev/sc/logos/2026q2-twap-oracle) : surfaced in a search on 2026-11-17, not opened
- Vault notes cited in the table: [[projects/liquity]], [[projects/drift]], [[projects/lighter]], [[projects/jupiter-perps]], [[projects/sky-maker]], [[projects/synthetix-v3]], [[projects/crvusd]], [[projects/gmx-v2]], [[projects/synthetix-v2]], [[projects/metronome-synth]], [[projects/variational]], [[patterns/oracle-pricing-model]], [[patterns/collateral-debt-units]]
