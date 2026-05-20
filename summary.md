# Research Summary: Synthetic Assets on ETH, BTC and SOL

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

**Pure synthetics (price-tracking tokens):** Synthetix-style sBTC/sETH/sUSD synthetic tokens have declined sharply in importance. Synthetix V3 TVL at $41M is roughly 1% of category peaks in 2021. The "synthetic stocks" experiments of 2020-2021 (Mirror Protocol, Synthetix sFTSE, etc.) largely failed. The surviving pure-synthetic primitive at scale is sBTC: a 1:1 BTC-backed token on Stacks, $437M TVL end of Q1 2026.

**Synthetic exposure via perpetuals:** by 2026 the dominant route to on-chain synthetic exposure is via perpetuals platforms. Hyperliquid alone processes $176B in 30-day perp volume; Jupiter Perps did $263B in 2025; GMX V2, dYdX v4, Drift and Ostium add tens of billions more. Perpetuals deliver synthetic price exposure without minting a synthetic-asset token; the position itself is the exposure.

The category total exceeds $400B in annual on-chain perp volume and approximately $7-8B in productive TVL across all venues researched.

## Project rankings

| Rank | Project | Ecosystem | Primary metric | Value | Source |
|------|---------|-----------|---------------|-------|--------|
| 1 | [[projects/hyperliquid]] | Custom L1 | 30d perp volume | $176.06B | [DefiLlama](https://defillama.com/protocol/hyperliquid) |
| 2 | [[projects/jupiter-perps]] | Solana | JLP TVL / 2025 volume | $1.3B / $263B | [Yellow.com](https://yellow.com/research/jupiter-solana-dex-aggregator-market-cap-2026) |
| 3 | [[projects/sbtc-stacks]] | Bitcoin L2 | sBTC TVL | $437M (peak $545M Q1 2026) | [Stacks blog](https://www.stacks.co/blog/q1-2026-snapshot) |
| 4 | [[projects/drift]] | Solana | Open interest | ~$400M | [CryptoAdventure](https://cryptoadventure.com/drift-protocol-review-2026-solana-perps-margin-design-and-real-liquidity-conditions/) |
| 5 | [[projects/gmx-v2]] | Arbitrum + others | TVL | $203.47M | [DefiLlama](https://defillama.com/protocol/gmx-v2-perps) |
| 6 | [[projects/ostium]] | Arbitrum | Monthly volume | ~$6B | [MEXC](https://blog.mexc.com/ostium-airdrop-2026-how-to-farm-points-on-the-rwa-perp-dex-bringing-gold-oil-and-stocks-on-chain/) |
| 7 | [[projects/dydx-v4]] | Cosmos | Daily volume | ~$200M avg | [Coinbureau](https://coinbureau.com/review/dydx) |
| 8 | [[projects/synthetix-v3]] | Ethereum + Base | V3 TVL | $41.19M | [DefiLlama](https://defillama.com/protocol/synthetix-v3) |

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

- **Ethereum L1:** still the home of value but expensive for high-frequency perp updates. Synthetix V3 returned here in December 2025 for composability, GMX added an Ethereum deployment.
- **Ethereum L2s (Arbitrum, Base):** the dominant L2 home of synthetics. GMX V2 majority TVL is on Arbitrum ($191M of $203M). Synthetix V3 active deployments are on Base. Ostium is Arbitrum-native.
- **Solana:** deeply consolidated. Jupiter Perps holds ~80% of Solana perp volume; Drift holds most of the remainder. JLP is the third-largest TVL pool on Solana behind Jito and Kamino.
- **Custom L1 (Hyperliquid):** purpose-built for perps. Custom HyperBFT consensus, integrated matching engine in the validator binary, HyperEVM smart-contract layer added on top.
- **Cosmos appchain (dYdX Chain):** sovereign chain with in-memory orderbook in every validator. CometBFT consensus.
- **Bitcoin L2 (Stacks):** post-Nakamoto Stacks adds Bitcoin finality and the sBTC two-way peg. The Bitcoin-anchored finality is unique among L2s.

### Oracle infrastructure

Almost every venue researched depends on Chainlink Data Streams, Pyth, or both:

- **Chainlink Data Streams:** pull-based, signed price reports. Used by GMX V2 (exclusively), Synthetix V3 (as one input), Ostium (for crypto pairs), and GMX-Solana.
- **Pyth:** sub-second pull oracle native to Solana. Used by Drift (primary), Jupiter Perps (one of three feeds), Synthetix V3 (one input).
- **Edge by Chaos Labs:** newer feed used by Jupiter Perps for aggregation.
- **Custom RWA oracle:** Ostium runs its own licensed-data aggregator for gold, oil, forex and equities.

Cross-cutting note: [[patterns/oracle-pricing-model]].

### Bridge / custody infrastructure

- **USDC bridges:** Hyperliquid's Arbitrum bridge is effectively a 3-of-4 multisig with a 200-second dispute window. dYdX Chain avoids a bridge entirely by holding USDC natively via Noble + IBC.
- **BTC custody:** sBTC uses a 70%-threshold Weighted Schnorr Threshold Signature over a permissionless signer set (currently 15, target 150). Signers post STX bond. Contract-bound peg-outs prevent arbitrary fund movement.
- **Cross-chain messaging:** Synthetix V3 has prepared a Chainlink CCIP integration for cross-chain stablecoin transfers; not in critical path yet.

Cross-cutting note: [[patterns/bridge-trust-assumptions]].

## Trust components

Trust assumptions across the eight protocols cluster into five categories.

### Oracle trust

Five of eight projects (Synthetix, GMX, Jupiter, Drift, Ostium) place execution-price authority in an oracle. A coordinated oracle failure or manipulation in any of these immediately mis-prices every trade and liquidation. Mitigations include multi-feed aggregation (Jupiter uses three), pull-based signed reports (Chainlink Data Streams) and commit-reveal settlement (Synthetix). None eliminate the dependency.

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
- **Synthetix V3:** mature audit history, modular architecture.
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
- **Drift:** most sophisticated hybrid liquidity stack (JIT + DLOB + vAMM) on Solana.
- **GMX:** earliest large-scale oracle-priced perp DEX; broadest multi-chain footprint including Botanix (BTC L2).
- **Ostium:** only on-chain perp venue with credible RWA volume; custom oracle stack for traditional markets.
- **Synthetix:** longest-running synthetic asset protocol; V3 generalised the primitive but has lost market share.

## Problem data

Issues that surfaced repeatedly during this research, which any RFP touching synthetic assets should address:

- **Oracle dependence is the dominant systemic risk.** Five of eight protocols are fully dependent on a single or small set of oracles for execution. Mitigations exist but no protocol eliminates the dependency.
- **Bridge centralisation is the dominant trust risk.** Hyperliquid's small validator set holds billions in USDC. Most BTC-pegged assets remain federated or single-custodian; sBTC is the leading decentralised exception.
- **LP-pool venues concentrate risk.** Jupiter's JLP and Ostium's OLP are single pools backing all markets. A tail loss in one market hits every LP.
- **Synthetic tokens have not scaled.** Pure synthetic-asset designs (Synthetix-style) have stagnated at $40M-$500M while perp venues exceed billions. Markets prefer perpetual-position exposure over minted synthetic tokens, with sBTC the notable exception (because it represents a real 1:1 backed BTC, not a price-tracking derivative).
- **Decentralisation versus speed trade-off is unresolved.** Hyperliquid's dominance comes from a centralised-feeling 4-node bridge with a 200-second dispute window. dYdX's more decentralised orderbook has 1/40th the daily volume.

## Gaps and open questions

- **Drift TVL/volume current figures:** sources cite open interest (~$400M) but exact TVL and 24h volume need verification from Drift's official analytics for any RFP citation.
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
