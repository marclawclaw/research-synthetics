# Research Summary: Synthetic Assets on ETH, BTC and SOL

## Update 2026-11-16: what changed since the May snapshot

The body of this note below is the May 2026 baseline. Where a May claim is now wrong it is marked "(May 2026; superseded)" with a pointer here; Hyperliquid, dYdX and sBTC were not refreshed in November. Figures are from DefiLlama's API unless stated, and the November findings were agent-researched with spot checks, not yet through a formal fact-check pass.

**Headline: three of the five Ethereum and Solana venues in the May vault suffered an exploit, wind-down or collapse in 2026, and the growth moved to orderbook venues.**

- **Drift (Solana) was exploited for about $285M to $295M on 2026-04-01** (seven weeks before the May snapshot, which did not record it). It was an admin takeover of the 2-of-5 Security Council multisig (zero timelock) using pre-signed durable-nonce transactions and a fake collateral token, not a smart-contract bug. Attribution to a North Korea-aligned group is preliminary. It relaunched as Velocity on 2026-09-29 with negligible TVL and about $0.2M daily volume. Sources and detail: [[projects/drift]].
- **Ostium (Arbitrum) lost $23.75M from the OLP vault on 2026-07-15** through a compromised oracle signer and keeper path (sources disagree on the exact root cause). TVL fell from about $47M to $2.3M. It had already moved to offchain hedging on 2026-04-28, so the "single USDC counterparty vault" description was outdated. See [[projects/ostium]].
- **Synthetix V3 is effectively retired.** The 420 Pool closed on 2026-06-19, SIP-423 retired legacy sUSD at 4 SNX per sUSD, and DefiLlama V3 TVL went to zero on 2026-06-26. Base and Arbitrum were already gone by August 2025; only a small mainnet orderbook exchange remains. See [[projects/synthetix-v3]] and [[projects/synthetix-v2]].
- **Jupiter Perps shrank but is intact:** JLP AUM $683M (Jupiter API) or $573M (DefiLlama), against about $1.3B quoted in May. JLP tokens were among the assets drained from Drift (about $155M to $159M, secondary sources only; I found no report of loss to JLP holders themselves), and JLP supply fell 13% in Q3. The "$263B in 2025" volume and "about 80% of Solana perp volume" claims could not be reproduced and should not be cited. Jupiter's new GUM markets use an orderbook engine with USDC collateral and no JLP pool. See [[projects/jupiter-perps]].
- **GMX V2 is steady but shrinking:** TVL $187M (May $203M), open interest $41M (May $84M), no 2026 exploit found. See [[projects/gmx-v2]].
- **Orderbook perp venues now lead.** Q3 volume shares to 25 September: Hyperliquid 37.3%, Aster 9.6%, Lighter 7.8%, Variational 6.4%, GMX 4.0%. Lighter ([[projects/lighter]]) holds $550M of bridge collateral, about three times GMX. See [[metrics/volume-comparison]].
- **Synthetic dollars are the largest synthetic category by value.** Sky/Maker $3.76B TVL, Ethena USDe $4.89B (down from a $14.82B peak in October 2025), Liquity $201M across V1 and V2. See [[projects/ethena-usde]] and [[patterns/cdp-synthetics-landscape]].
- **CDP synthetics are a graveyard with a few durable survivors.** Of 50 CDP, debt-pool and collateral-split protocols surveyed, 20 were dead, halted, wound down or abandoned by 2026-11-16 (an editorial count). Metronome Synth has about $15.7M of unbacked synths (July 2026) and msUSD at $0.818; Abracadabra's MIM is at $0.0286 after three hacks; Sovryn Zero was halted on 2026-11-07. Survivors share a hard redemption floor, a near-immutable core and non-endogenous collateral. See [[patterns/cdp-synthetics-landscape]].
- **Other 2026 incidents in the same ecosystems:** Resolv USR unbacked mint (about $25M, 2026-03-22), Kelp ($293M, 2026-04-18), Flash Trade wind-down (August), Arcus Perps ($2.23M, 2026-11-10).

**Lessons the exploits reinforce for any RFP:** the oracle and key-management paths are the failure points, not the pricing model. Drift's loss was multisig governance (no timelock); Ostium's was a signer/keeper credential path; Metronome's was oracle lag in a swap module. Single-pool LP designs (OLP, JLP) transmitted the loss directly to LPs.

**Addendum 2026-11-17: notes behind the Notion page "CDP-Backed Synthetics" (CDP on LEZ).** The CDP notes were aligned with that page, which is now the source of truth for their structure, terms and decisions, and extended with the evidence the page summarises. New or reworked: [[rfp/appendix/cdp-notion-map]] (part map, terms, where the vault and the page disagreed), [[rfp/appendix/cdp-admin-switch]] (rules, admin switch, precedent check), [[patterns/collateral-debt-units]] (units and oracle requirements; keeps ETH as the worked example), [[patterns/cdp-peg-mechanisms]] (new), [[rfp/appendix/cdp-isolation]] (retitled "how a deployment shares risk"; five-protocol first-loss table), [[rfp/appendix/cdp-oracle]] (new), [[rfp/appendix/cdp-risk-privacy-market]] (new), [[projects/indigo]] and [[projects/monolith]] (new), with additions to [[projects/liquity]], [[projects/hylo]] and [[projects/alchemix]], and terminology and link updates in [[patterns/cdp-vs-morpho-trust]] and [[patterns/cdp-synthetics-landscape]]. What these notes show, as recorded there and not fact-checked: Liquity V1 and V2 have no rule anyone can change, Sky/Maker and crvUSD put most rules behind a vote with faster bounded paths, Synthetix V2 put them behind one multisig with no delay; of the surveyed first-loss pools, only Indigo is per synth and its shortfall rule (V2-era whitepaper) leaves the rest of a position frozen until the pool is refilled. The precedent checks mark the page's recommendations as backed, partly backed or novel. Open items: whether RedStone publishes XMR/USD (only RFP-020 asserts it), how Indigo V3 handles a shortfall, and whether LEZ can support keepers and private-account interaction.

## Methodology

Projects were discovered by searching DefiLlama, DappRadar, project documentation, and reputable news/analytics sources (Coin Metrics, Messari, The Defiant, Yellow.com, Bankless, CoinGecko) in May 2026. Headline TVL and volume figures were verified by direct fetch from DefiLlama where possible, since LLM-summarised numbers from search results contained at least one extraction error (an inflated GMX TVL figure quoted in search results was corrected to $203.47M via DefiLlama's protocol page).

Eight projects were selected for atomic notes to cover the design space across collateral models, trust assumptions and host ecosystems:

- **Ethereum L2s:** [[projects/synthetix-v3]], [[projects/gmx-v2]], [[projects/ostium]]
- **Solana:** [[projects/jupiter-perps]], [[projects/drift]]
- **Custom L1:** [[projects/hyperliquid]]
- **Cosmos appchain:** [[projects/dydx-v4]]
- **Bitcoin L2:** [[projects/sbtc-stacks]]

## Ecosystem landscape

The synthetic asset category has bifurcated since 2024 into two distinct sub-markets.

**Pure synthetics (price-tracking tokens):** Synthetix-style sBTC/sETH/sUSD synthetic tokens have declined sharply in importance. Synthetix V3 TVL was $41M in May 2026 (roughly 1% of category peaks in 2021) and is now $0 (superseded, see Update 2026-11). The "synthetic stocks" experiments of 2020-2021 (Mirror Protocol, Synthetix sFTSE, etc.) largely failed. The surviving pure-synthetic primitive at scale is sBTC: a 1:1 BTC-backed token on Stacks, $437M TVL end of Q1 2026.

**Synthetic exposure via perpetuals:** by 2026 the dominant route to on-chain synthetic exposure is via perpetuals platforms. Hyperliquid alone processes $176B in 30-day perp volume; Jupiter Perps was reported at $263B in 2025 (May 2026; superseded, unverified); GMX V2, dYdX v4, Drift and Ostium add tens of billions more. Perpetuals deliver synthetic price exposure without minting a synthetic-asset token; the position itself is the exposure.

The category total exceeds $400B in annual on-chain perp volume and approximately $7-8B in productive TVL across all venues researched. (May 2026; superseded: category perp volume was about $1.8T in Q2 2026 and about $1.6T in Q3 to 25 September per DefiLlama via press, so the $400B annual figure was an understatement or a different scope; the May TVL total no longer holds after the Drift, Ostium and Synthetix losses.)

## Project rankings

| Rank | Project | Ecosystem | Primary metric | Value | Source |
|------|---------|-----------|---------------|-------|--------|
| 1 | [[projects/hyperliquid]] | Custom L1 | 30d perp volume | $176.06B | [DefiLlama](https://defillama.com/protocol/hyperliquid) |
| 2 | [[projects/jupiter-perps]] | Solana | JLP TVL / 2025 volume | $1.3B / $263B (May 2026; superseded: $573M to $683M, volume unverified) | [Yellow.com](https://yellow.com/research/jupiter-solana-dex-aggregator-market-cap-2026) |
| 3 | [[projects/sbtc-stacks]] | Bitcoin L2 | sBTC TVL | $437M (peak $545M Q1 2026) | [Stacks blog](https://www.stacks.co/blog/q1-2026-snapshot) |
| 4 | [[projects/drift]] | Solana | Open interest | ~$400M (May 2026; superseded: exploited 2026-04-01, now $0.2M OI) | [CryptoAdventure](https://cryptoadventure.com/drift-protocol-review-2026-solana-perps-margin-design-and-real-liquidity-conditions/) |
| 5 | [[projects/gmx-v2]] | Arbitrum + others | TVL | $203.47M (May 2026; now $187.05M) | [DefiLlama](https://defillama.com/protocol/gmx-v2-perps) |
| 6 | [[projects/ostium]] | Arbitrum | Monthly volume | ~$6B (May 2026; superseded: exploited 2026-07-15, 30d volume ~$47M) | [MEXC](https://blog.mexc.com/ostium-airdrop-2026-how-to-farm-points-on-the-rwa-perp-dex-bringing-gold-oil-and-stocks-on-chain/) |
| 7 | [[projects/dydx-v4]] | Cosmos | Daily volume | ~$200M avg | [Coinbureau](https://coinbureau.com/review/dydx) |
| 8 | [[projects/synthetix-v3]] | Ethereum + Base | V3 TVL | $41.19M (May 2026; superseded: $0, Base was already gone) | [DefiLlama](https://defillama.com/protocol/synthetix-v3) |

See also [[metrics/tvl-comparison]] and [[metrics/volume-comparison]] for cross-protocol metric tables.

## Technology stack: how the assets actually work

Three architectural archetypes cover the entire landscape.

### 1. On-chain CLOB (central limit orderbook)

- **Used by:** [[projects/hyperliquid]], [[projects/dydx-v4]]
- **Mechanism:** validators run a matching engine; trades clear against another trader's resting order. PnL is settled in USDC.
- **Synthetic dimension:** the position itself is the synthetic exposure. No token is minted.
- **Strengths:** capital-efficient, low oracle dependence (oracle used only for liquidations and funding, not execution), familiar UX.
- **Weaknesses:** requires deep market-maker liquidity; not viable for long-tail assets unless someone makes the market.

### 2. Oracle-priced LP pool perpetuals

- **Used by:** [[projects/gmx-v2]], [[projects/jupiter-perps]], [[projects/ostium]], [[projects/drift]] (partial, vAMM fallback)
- **Mechanism:** an LP pool of basket assets backs every position. Pricing comes from an oracle (Chainlink Data Streams, Pyth, or multi-feed aggregation), not an orderbook.
- **Synthetic dimension:** trader holds a perp position; LPs hold the inverse exposure as part of their pool share.
- **Strengths:** no slippage at execution, easy to bootstrap, accommodates long-tail and RWA markets where market makers do not exist on-chain.
- **Weaknesses:** total dependence on oracle integrity; LP directional risk; see [[patterns/oracle-pricing-model]] and [[patterns/lp-counterparty-risk]].

### 3. CDP-backed synthetic tokens

- **Used by:** [[projects/synthetix-v3]], [[projects/sbtc-stacks]]
- **Mechanism:** collateral is locked, a synthetic token is minted at a target ratio (debt-pool in Synthetix, 1:1 peg in sBTC). The token trades anywhere and tracks the underlying.
- **Synthetic dimension:** the token is the synthetic asset.
- **Strengths:** composable (the synthetic token is a regular ERC-20/SIP-010 asset), good for spot exposure use cases like collateral in lending.
- **Weaknesses:** complex incentives (CDP designs), bridge or custody assumptions (1:1 pegs); historically lower capital efficiency.

A complementary axis is the [[patterns/synthetic-collateral-models]] note, which classifies all five collateral models seen.

## Infrastructure

### Host chains

- **Ethereum L1:** still the home of value but expensive for high-frequency perp updates. Synthetix V3 returned here in December 2025 for composability (May 2026 claim; superseded, V3 was retired in June 2026), GMX added an Ethereum deployment.
- **Ethereum L2s (Arbitrum, Base):** the dominant L2 home of synthetics. GMX V2 majority TVL is on Arbitrum ($180M of $187M in November; $191M of $203M in May). Synthetix V3 deployments are gone (Base was already shut by August 2025; May claim superseded). Ostium is Arbitrum-native.
- **Solana:** deeply consolidated. Jupiter Perps was described in May as holding ~80% of Solana perp volume with Drift holding most of the remainder, and JLP as the third-largest TVL pool on Solana (May 2026; superseded: the share is unverified, Drift was exploited, and Pacifica, GMTrade and Phoenix have grown; one tracker put Jupiter at about 47% of 24h volume across five venues).
- **Custom L1 (Hyperliquid):** purpose-built for perps. Custom HyperBFT consensus, integrated matching engine in the validator binary, HyperEVM smart-contract layer added on top.
- **Cosmos appchain (dYdX Chain):** sovereign chain with in-memory orderbook in every validator. CometBFT consensus.
- **Bitcoin L2 (Stacks):** post-Nakamoto Stacks adds Bitcoin finality and the sBTC two-way peg. The Bitcoin-anchored finality is unique among L2s.

### Oracle infrastructure

Almost every venue researched depends on Chainlink Data Streams, Pyth, or both:

- **Chainlink Data Streams:** pull-based, signed price reports. Used by GMX V2 (exclusively), Synthetix V3 (as one input; retired), Ostium (for crypto pairs), and GMX-Solana.
- **Pyth:** sub-second pull oracle native to Solana. Used by Drift (primary before the exploit; Velocity's oracle is not named in its docs), Jupiter Perps (fallback behind Edge as primary), Synthetix V3 (one input; retired).
- **Edge by Chaos Labs:** newer feed used by Jupiter Perps for aggregation.
- **Custom RWA oracle:** Ostium ran its own licensed-data aggregator (Stork-operated nodes per its deprecated docs) for gold, oil, forex and equities; the July 2026 exploit went through an Ostium-operated signer path.

Cross-cutting note: [[patterns/oracle-pricing-model]].

### Bridge / custody infrastructure

- **USDC bridges:** Hyperliquid's Arbitrum bridge is effectively a 3-of-4 multisig with a 200-second dispute window. dYdX Chain avoids a bridge entirely by holding USDC natively via Noble + IBC.
- **BTC custody:** sBTC uses a 70%-threshold Weighted Schnorr Threshold Signature over a permissionless signer set (currently 15, target 150). Signers post STX bond. Contract-bound peg-outs prevent arbitrary fund movement.
- **Cross-chain messaging:** Synthetix V3 had prepared a Chainlink CCIP integration for cross-chain stablecoin transfers (May 2026 claim; moot after V3's retirement).

Cross-cutting note: [[patterns/bridge-trust-assumptions]].

## Trust components

Trust assumptions across the eight protocols cluster into five categories.

### Oracle trust

Five of eight projects (Synthetix, GMX, Jupiter, Drift, Ostium) placed execution-price authority in an oracle in May; two of those have since been exploited through oracle or key paths (Ostium) or governance keys (Drift). A coordinated oracle failure or manipulation in any of these immediately mis-prices every trade and liquidation. Mitigations include multi-feed aggregation (Jupiter uses three), pull-based signed reports (Chainlink Data Streams) and commit-reveal settlement (Synthetix). None eliminate the dependency.

### Validator / signer set trust

Hyperliquid, dYdX, and sBTC each rely on a small-to-medium validator or signer set:

- **Hyperliquid:** ~4 validator nodes effectively running a 3-of-4 multisig for the USDC bridge; same team controls hot and cold keys per independent review.
- **dYdX Chain:** up to 60 validators initially, delegated proof-of-stake.
- **sBTC:** 15 bonded signers currently, design target 150, 70% weighted threshold; permissionless entry.

sBTC's bond-based permissionless signer model is the most decentralised of the three. Hyperliquid's is the most centralised and is the most acknowledged risk in independent reviews.

### LP-pool trust

LPs in GMX, Jupiter and Ostium trust:

- The oracle pricing the trades they back.
- The protocol's risk-engine and liquidation logic.
- The protocol's funding-rate model to balance skew.
- The smart-contract correctness of the pool itself.

A failure on any axis transfers value from LPs to traders or to attackers. Concentration in single-pool designs (Ostium's OLP, Jupiter's JLP) increases blast radius of any failure.

### Bridge trust

See [[patterns/bridge-trust-assumptions]]. The two notable bridge designs in this set are Hyperliquid's USDC bridge (validator multisig + dispute window) and sBTC's BTC peg (threshold signatures + STX bond). They sit at opposite ends of the bridge-trust spectrum.

### Smart-contract trust

All protocols rely on the correctness of their on-chain contracts. Public audit coverage varies:

- **Hyperliquid bridge:** audited by Cyfrin. The full HyperCore/HyperBFT stack does not have public top-tier audits matching its volume scale.
- **GMX V2:** multiple audits, mature codebase.
- **Synthetix V3:** mature audit history, modular architecture (retired June 2026).
- **sBTC:** has had audits but the WSTS scheme is relatively new in production.
- **Others:** broadly audited but coverage depth varies.

## Common patterns

- [[patterns/oracle-pricing-model]] : oracle is the canonical execution price in LP-pool venues.
- [[patterns/lp-counterparty-risk]] : LPs absorb trader PnL in pool-based venues.
- [[patterns/bridge-trust-assumptions]] : custody layer trust dominates synthetic-token designs.
- [[patterns/synthetic-collateral-models]] : five distinct collateral models across the landscape.

## Key differentiators

- **Hyperliquid:** dominance in raw on-chain perp volume; integrated HyperCore + HyperEVM architecture.
- **Jupiter:** vertical integration with the dominant Solana aggregator delivers organic flow into JLP.
- **sBTC:** most decentralised BTC representation by signer-bond economics in 2026.
- **dYdX:** most decentralised orderbook (every validator runs the matcher).
- **Drift:** most sophisticated hybrid liquidity stack (JIT + DLOB + vAMM) on Solana (May 2026; the venue was exploited on 2026-04-01 and relaunched as Velocity with spot trading removed).
- **GMX:** earliest large-scale oracle-priced perp DEX; broadest multi-chain footprint (Botanix TVL has been about $0 since July 2026).
- **Ostium:** was the leading RWA perp venue in May; since the July 2026 exploit its volume is about 1% of April levels. RWA perp volume has moved to trade.xyz and Variational (about 90% of $123B to $148B a month, per Castle Labs).
- **Synthetix:** longest-running synthetic asset protocol; V3 generalised the primitive but has lost market share.

## Problem data

Issues that surfaced repeatedly during this research, which any RFP touching synthetic assets should address:

- **Oracle dependence is the dominant systemic risk (now realised: Ostium, Metronome).** Five of eight protocols are fully dependent on a single or small set of oracles for execution. Mitigations exist but no protocol eliminates the dependency.
- **Bridge centralisation is the dominant trust risk.** Hyperliquid's small validator set holds billions in USDC. Most BTC-pegged assets remain federated or single-custodian; sBTC is the leading decentralised exception.
- **LP-pool venues concentrate risk.** Jupiter's JLP and Ostium's OLP were single pools backing all markets (Ostium has since added offchain hedging; Jupiter's GUM markets do not use JLP). A tail loss in one market hits every LP; the Ostium exploit realised this.
- **Synthetic tokens have not scaled.** Pure synthetic-asset designs (Synthetix-style) stagnated at $40M-$500M (Synthetix V3 now $0) while perp venues exceed billions; synthetic dollars (Sky, Ethena, Liquity) are the exception at the multi-billion scale. Markets prefer perpetual-position exposure over minted synthetic tokens, with sBTC the notable exception (because it represents a real 1:1 backed BTC, not a price-tracking derivative).
- **Decentralisation versus speed trade-off is unresolved.** Hyperliquid's dominance comes from a centralised-feeling 4-node bridge with a 200-second dispute window. dYdX's more decentralised orderbook has 1/40th the daily volume.

## Gaps and open questions

- **Drift (resolved 2026-11):** the TVL and open-interest figures in May were not reproducible; DefiLlama shows Drift Trade at $311M TVL pre-exploit and $0.64M now. Open: Velocity TVL, 7d and 30d volume, Drift's pre-hack leverage cap, and the full set of exploit losses (about $285M to $295M depending on source).
- **Ostium custom-oracle decentralisation:** the licensed-data aggregator is novel but the source list, licensing terms and node operators are not public. An RFP grounding any RWA design here would need to push for more transparency.
- **Hyperliquid validator set evolution:** announcements suggest validator-set decentralisation is on the roadmap, but the practical timeline and the path to publicly auditable validator-key custody is unclear.
- **dYdX TVL data:** current TVL figures not directly captured; OI proxy used.
- **sBTC signer growth path:** the gap between 15 current signers and the 150 design target is material; the rate of growth and the bond-to-TVL economic ratio over time both warrant ongoing monitoring.
- **Cross-chain synthetic ETH and SOL on Bitcoin L2s:** sBTC exists, but synthetic ETH or SOL on Bitcoin L2s are absent from this snapshot and worth a follow-up survey.

## Recommended next steps for an RFP

If an RFP downstream of this research targets synthetic asset infrastructure, the most informative architectural questions are:

1. **Token model or position model?** Synthetic-asset token (Synthetix, sBTC style) or perpetual position (Hyperliquid, GMX, Jupiter style)? This drives every other decision.
2. **Oracle or orderbook?** If oracle, specify feed, liveness budget, multi-feed aggregation, and front-running mitigation. If orderbook, specify matching architecture (in-validator like dYdX, integrated like Hyperliquid).
3. **Custody and bridge model.** If BTC or external assets are involved, specify signer set, threshold, bond, and dispute window.
4. **LP model.** If pool-based, specify pool composition, fee share, and skew-sensitivity of funding.
5. **Insurance and backstop.** Every venue researched has one (HLP, OLP, JLP, debt pool, signer bond); pick a model and parameterise it.
