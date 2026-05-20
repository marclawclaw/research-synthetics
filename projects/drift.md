---
tags: [project, solana, perpetuals, hybrid-liquidity]
ecosystem: Solana
category: Perpetuals
website: https://drift.trade
docs: https://docs.drift.trade
launched: 2021 (v1), 2022 (v2), 2023+ (v3)
---

# Drift Protocol

Drift is Solana's dominant decentralised perpetuals exchange, distinguished by a three-tier hybrid liquidity stack: a just-in-time (JIT) auction for market-maker fills, a fully on-chain limit orderbook (DLOB), and an oracle-pegged virtual AMM (DAMM/BAL) as a fallback. Drift also bundles spot trading, lending pools, and unified cross-margining behind a single user account.

## Adoption metrics

| Metric | Value | Date | Source |
|--------|-------|------|--------|
| Open interest | ~$400M | April 2026 | [Drift docs / cryptoadventure review](https://cryptoadventure.com/drift-protocol-review-2026-solana-perps-margin-design-and-real-liquidity-conditions/) |
| TVL range | $150M-$400M | 2026 | [Eco support Drift deep dive](https://eco.com/support/en/articles/15083167-drift-protocol-solana-perpetuals-dex-deep-dive) |
| LP APY range | 10-30% (favourable conditions) | 2026 | [Eco support Drift deep dive](https://eco.com/support/en/articles/15083167-drift-protocol-solana-perpetuals-dex-deep-dive) |
| Order execution latency | <400ms | 2026 | [Eco support Drift deep dive](https://eco.com/support/en/articles/15083167-drift-protocol-solana-perpetuals-dex-deep-dive) |
| Leverage cap | 10x perpetuals | 2026 | [Eco support Drift deep dive](https://eco.com/support/en/articles/15083167-drift-protocol-solana-perpetuals-dex-deep-dive) |

## How it works

### User perspective

A trader funds a Drift account on Solana and places an order. The order routes through three liquidity layers in priority:

1. **JIT auction:** market makers compete to fill the order at an improved price over a short window.
2. **DLOB (on-chain orderbook):** if JIT does not fill, the order can rest on or take from the on-chain limit orderbook.
3. **DAMM/BAL (vAMM with backstop liquidity):** fallback automated liquidity using an oracle-pegged constant product curve.

Drift also offers spot trading and a lending market (borrow/lend pools) under unified cross-margin, so collateral can move across products without bridging.

### Protocol perspective

- **Pyth oracle integration:** Drift uses Pyth as the primary oracle and supports other sources per market.
- **Oracle-pegged AMM:** the vAMM's mid is anchored to the live oracle price, with dynamic spread and peg that update prior to fills.
- **Concentration factor (BAL):** Backstop AMM Liquidity adds an external pool that can be tapped as additional fallback liquidity.
- **Funding rate:** standard 1h interval, derived from long/short skew and the oracle premium.
- **JIT auction:** off-chain market-maker bidding on each order, on-chain settlement.

## Key behaviours

- [[patterns/oracle-pricing-model]] : Pyth-driven mark price, vAMM pegged to oracle.
- [[patterns/jit-auction-routing]] : JIT auction as first-class liquidity tier.
- [[patterns/hybrid-amm-orderbook]] : three-tier routing across JIT, DLOB and vAMM.

## Architecture decisions

- **Solana-native:** built on Solana for sub-second finality and low fees, which is essential for an on-chain orderbook with high update frequency.
- **Hybrid stack:** treats JIT, DLOB and vAMM as complementary rather than substitutes; reduces dependency on any single liquidity source.
- **Unified cross-margin across perps/spot/lending:** treats user as a single account, simplifying capital efficiency.
- **Oracle-anchored vAMM:** prevents the vAMM from drifting from market price during low-volume periods, which was a known failure mode of pure constant-product perps.

## Differentiators

- Most sophisticated on-chain liquidity stack among Solana perp venues.
- Unified margin across perps, spot and lending in a single protocol.
- Open JIT auction model with active third-party market makers, rather than a single protocol-run market maker.

## Limitations and criticisms

- Smaller in absolute size than Hyperliquid or Jupiter Perps despite dominance among on-chain orderbook venues on Solana.
- Pyth oracle dependency: a Pyth outage or stale price could create execution and liquidation issues; mitigations rely on the per-market fallback configuration.
- Hybrid routing complexity makes user-perceived fills less predictable than a pure orderbook venue.
- Historical incident: Drift v1 had a vAMM-related exploit in 2022 which prompted the v2 redesign.

## Sources

- [Drift docs : Understanding Drift](https://docs.drift.trade/protocol/about-v3) : accessed 2026-05-19
- [Drift docs : Drift AMM](https://docs.drift.trade/protocol/about-v3/drift-amm) : accessed 2026-05-19
- [Drift docs : funding rates](https://docs.drift.trade/protocol/trading/perpetuals-trading/funding-rates) : accessed 2026-05-19
- [Eco support : Drift Solana perp deep dive](https://eco.com/support/en/articles/15083167-drift-protocol-solana-perpetuals-dex-deep-dive) : accessed 2026-05-19
- [CryptoAdventure : Drift Protocol review 2026](https://cryptoadventure.com/drift-protocol-review-2026-solana-perps-margin-design-and-real-liquidity-conditions/) : accessed 2026-05-19
- [Bitcoin.com : What is Drift 2026 guide](https://www.bitcoin.com/get-started/what-is-drift-solana/) : accessed 2026-05-19
