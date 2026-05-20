---
tags: [appendix, synthetic, monero, atomic-swap, pitch]
related: [patterns/synthetic-collateral-models, patterns/oracle-pricing-model, projects/sbtc-stacks]
---

# sXMR — Oracle-priced Synthetic with Atomic-Swap Redemption

## The pitch

A synthetic Monero token (sXMR) on EVM. Oracle-priced for trading and composability; redeemed to real XMR via peer-to-peer atomic swap with no custodian, no bridge, and no protocol-held reserves.

**The wedge:** the only synthetic that terminates in a privacy-preserving asset on a privacy chain. sBTC redeems to public BTC. Every other synthetic redeems to traceable stablecoins. sXMR is the first design where the redemption path itself preserves privacy.

**Honest framing:** this is a synthetic with a soft, market-clearing peg — not a hard-redeemable synthetic. The oracle is the *quoted* price; the *achievable* price is whatever an XMR LP will swap for, when one is willing. Structurally closer to a DEX trading pair than to [[projects/sbtc-stacks]].

The RFP must pick one of two goals before specifying further — they are incompatible:

- **Non-custodial-and-mostly-works** → pure atomic-swap design ships. Soft peg, market-dependent exit. This is the interesting product.
- **Always-real-XMR-at-oracle-price** → requires bonded LPs (slashable USDC collateral) or a protocol XMR reserve. Custody or solvency risk returns. This is the marketable product.
