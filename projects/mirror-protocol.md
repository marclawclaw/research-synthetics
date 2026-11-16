---
tags: [project, terra, cosmos, cdp, synthetics, equities, dead, ust, historical]
ecosystem: Terra (Terra Classic after May 2022)
category: Synthetics/Multi-asset CDP (synthetic equities)
status: dead (collapsed with UST in May 2022; effectively shut down in 2022; DefiLlama TVL zero since 2023-10-12 and flagged dead from 2022-05-26)
status_date: 2022-05 to 2022-12
website: https://mirror.finance (domain now a for-sale placeholder)
docs: [NOT FOUND] (official docs site no longer served; a third-party copy was used)
launched: [NOT FOUND] (first DefiLlama TVL datapoint 2021-04-06)
---

# Mirror Protocol (Terra: mAssets and UST dependence)

Status: dead. Mirror let users lock UST, aUST, LUNA or other approved collateral in CDPs and mint "mAssets": synthetic tokens tracking stocks and other assets, priced by a Band-run oracle (per the docs copy; trading venue [NOT FOUND] in retrieved sources). It is historically instructive for three reasons: it was an at-scale synthetic equity CDP (peak TVL $2.2B), its collateral and its liquidity were both denominated in a single algorithmic stablecoin (UST), and it failed in two stages (UST collapse, then an oracle-mispricing exploit).

## Adoption metrics

| Metric | Value | Date | Source |
|--------|-------|------|--------|
| TVL peak | $2.207B | 2021-05-12 | [DefiLlama Mirror](https://api.llama.fi/protocol/mirror) |
| TVL, last datapoint | $0 | 2023-10-12 | [DefiLlama Mirror](https://api.llama.fi/protocol/mirror) |
| DefiLlama hallmarks | "UST depeg" 2022-05-07; "LUNC mispricing exploit" 2022-05-28 | 2022 | [DefiLlama Mirror](https://api.llama.fi/protocol/mirror) |
| DefiLlama dead flag | `deadFrom` 2022-05-26 | accessed 2026-11-16 | [DefiLlama protocols](https://api.llama.fi/protocols) |
| Fees and revenue | [NOT FOUND] (DefiLlama has no fee adapter for Mirror; the fees endpoint returns an error) | 2026-11-16 | [DefiLlama fees: mirror](https://api.llama.fi/summary/fees/mirror) |
| mAsset supply history | [NOT FOUND] (no DefiLlama stablecoin record; synthetic equities are not stablecoins) | n/a | n/a |

## How it works

### User perspective

A minter opens a CDP with approved collateral, picks an mAsset and a collateral ratio at or above that mAsset's minimum, and receives the mAsset. The minter keeps the CDP healthy by adding collateral or burning mAssets. Opening a CDP could also directly short an mAsset and mint short-LP (sLP) tokens, which are staked for MIR inflation rewards and burned when the mAsset is returned (docs copy).

### Protocol perspective (third-party copy of the Mirror docs)

- **Minimum collateral ratio:** set per mAsset (docs example 150%). Collateral multipliers raise the effective ratio for riskier collateral: LUNA, MIR, ANC and LUNA X 1.3333334 (about 200% effective at a 150% base), aUST 1. Source: [Everest GitBook copy of Mirror docs](https://getthemlols-organization.gitbook.io/everest/protocol/mirrored-assets-massets) (authority unverified; the official docs are no longer served).
- **Liquidation:** undercollateralised CDPs are sold in a discounted auction (discount capped by `auction_discount` or the minimum collateral ratio minus one); any unsold collateral returns to the CDP owner.
- **Oracle:** a dedicated Oracle Feeder per mAsset (genesis feeders owned by Band Protocol); prices valid for 60 seconds, after which mint, burn, deposit and withdraw are disabled until the feed resumes. See [[patterns/oracle-pricing-model]].
- **Peg:** no redemption at fair value outside a CDP close; the peg relies on arbitrage between the mAsset market price and the oracle price plus the CDP burn route. The docs copy says the feeder can shift rewards between LP and sLP tokens according to the mAsset's price premium.

## Key behaviours

- [[patterns/synthetic-collateral-models]] : multi-collateral CDP for synthetic equities.
- [[patterns/oracle-pricing-model]] : the May 2022 failure is an oracle-feed failure caused by a chain fork.
- [[patterns/cdp-synthetics-landscape]] : the clearest case of collateral and peg liquidity sharing one failure domain.

## Architecture decisions

- **Collateral in UST and aUST (Anchor yield):** attractive yield-bearing collateral, but it tied solvency to Terra's stablecoin.
- **Short farming:** incentivised short positions (sLP) to hold mAsset prices close to the oracle.
- **Fee to MIR stakers:** the 1.5% protocol fee was converted to MIR and distributed to stakers (see Fees).
- **60-second oracle validity:** safe against stale prices but halts CDP operations when feeders stop.

## Differentiators

- Synthetic stocks and other assets as on-chain tokens without custody of the underlying.
- Native integration with Terra's Anchor yield (aUST collateral).

## Limitations and criticisms

- **Collapse with UST (May 2022):** UST depegged on or around 2022-05-07 (DefiLlama hallmark); Mirror's collateral and its pools were mainly UST-denominated. DefiLlama marks the protocol dead from 2022-05-26.
- **Oracle mispricing exploit (2022-05-30):** after the chain fork, some Terra Classic validators kept reporting the LUNA price (about $10) instead of LUNC (about $0.0001); CDPs could be opened against LUNC valued at LUNA prices. CertiK's example: 100,000 LUNC valued at $10 backed a loan of 30,275 DOT (about $300,000). Amount stolen: about $2M (attributed to an on-chain analyst on Twitter by CertiK; The Block reports more than $2M). CertiK stated the root cause was validators failing to update, not a contract flaw, and outside its audit scope. Sources: [CertiK](https://www.certik.com/blog/mirror-protocol-exploited-due-to-incorrect-oracle-price), [The Block](https://www.theblock.co/news/regulation/2022-05-30-mirror-protocol-suffers-new-exploit-and-could-be-drained-in-hours-149361) (search result).
- **Shutdown date conflict:** Band support withdrawn August 2022 (one source) versus all operations ceased December 2022 (another); neither confirmed from a primary source.
- **DefiLlama hacks list entry of $90M (2021-10-08 to 2022-05-17, "duplicate ID" vulnerability):** a single-source entry that no other retrieved source corroborates; treated as unverified. See [DefiLlama hacks](https://api.llama.fi/hacks).
- Audit: a DocSend link is listed by DefiLlama ([API](https://api.llama.fi/protocol/mirror)); the auditor is [NOT FOUND].

### Fees and borrower cost

| Item | Value | Date | Source |
|------|-------|------|--------|
| Protocol fee | 1.5% of the value of mAsset burned, charged whenever collateral is withdrawn from a CDP (position closure, partial withdrawal and liquidation auctions included); collected, converted to MIR and distributed to MIR stakers | pre-2022 | [Everest copy of Mirror docs](https://getthemlols-organization.gitbook.io/everest/protocol/mirrored-assets-massets) (authority unverified) |
| Stability fee or interest | None found in the docs copy: [NOT FOUND] | n/a | same |
| Minimum collateral ratio | per-mAsset (150% in the docs example); times collateral multiplier (1.3333334 for LUNA, MIR, ANC; 1 for aUST) | pre-2022 | same |
| Liquidation penalty | Auction discount, lower of (minimum collateral ratio minus 1) or `auction_discount`; docs example 20% | pre-2022 | same |
| Trading fee on Terraswap | [NOT FOUND] | n/a | n/a |
| Borrower yield offset | MIR inflation for staked LP and sLP tokens | pre-2022 | same |

Fee revenue in USD: [NOT FOUND].

## Sources

- [DefiLlama Mirror API](https://api.llama.fi/protocol/mirror) : accessed 2026-11-16
- [DefiLlama protocols list (deadFrom)](https://api.llama.fi/protocols) : accessed 2026-11-16
- [DefiLlama hacks list](https://api.llama.fi/hacks) : accessed 2026-11-16
- [Everest GitBook: mirrored assets (copy of Mirror docs)](https://getthemlols-organization.gitbook.io/everest/protocol/mirrored-assets-massets) : accessed 2026-11-16
- [CertiK: Mirror Protocol exploited due to incorrect oracle price](https://www.certik.com/blog/mirror-protocol-exploited-due-to-incorrect-oracle-price) : accessed 2026-11-16
- [The Block: Mirror Protocol suffers new exploit](https://www.theblock.co/news/regulation/2022-05-30-mirror-protocol-suffers-new-exploit-and-could-be-drained-in-hours-149361) : accessed 2026-11-16 (search result only)
- [Bitcoinist: how Mirror was attacked](https://bitcoinist.com/mirror-protocol-attacked-no-one-turned-rescue-it) : accessed 2026-11-16 (search result only)
