---
tags: [project, bitcoin-l2, stacks, btc-synthetic, threshold-signatures]
ecosystem: Bitcoin L2 (Stacks)
category: BTC-backed Synthetic
website: https://stacks.co
docs: https://docs.stacks.co
launched: 2024 (Nakamoto / sBTC)
---

# sBTC on Stacks

sBTC is a BTC-pegged asset issued on the Stacks Bitcoin L2. It is a synthetic representation of BTC backed 1:1 by Bitcoin locked on the Bitcoin L1, with custody held by an open, permissionless set of threshold signers. Unlike WBTC-style custodial wrapped Bitcoin, sBTC's security model is based on a 70%-threshold signature scheme over a rotating set of economically-bonded signers.

## Adoption metrics

| Metric | Value | Date | Source |
|--------|-------|------|--------|
| sBTC TVL (peak Q1 2026) | $545M | Q1 2026 | [Stacks Q1 2026 snapshot](https://www.stacks.co/blog/q1-2026-snapshot) |
| sBTC TVL (end of Q1 2026) | $437M | Q1 2026 | [Yellow.com Stacks Q1 release](https://yellow.com/press-releases/stacks-stx-closes-q1-2026-with-437m-btc-tvl-320-btc-added-to-bitcoin-staking-pilot) |
| Stacks DeFi deployed capital total | $121M | Q1 2026 | [Stacks Q1 2026 snapshot](https://www.stacks.co/blog/q1-2026-snapshot) |
| Zest Protocol TVL (built on sBTC) | $75.9M | Q1 2026 | [Stacks Q1 2026 snapshot](https://www.stacks.co/blog/q1-2026-snapshot) |
| Granite TVL | $26M | Q1 2026 | [Stacks Q1 2026 snapshot](https://www.stacks.co/blog/q1-2026-snapshot) |
| Bitcoin staking pilot AUM | >$100M | since late 2025 | [Stacks Q1 2026 snapshot](https://www.stacks.co/blog/q1-2026-snapshot) |
| Signer count (current) | 15 | 2026 | [BitcoinWrites sBTC security model](https://www.bitcoinwrites.com/p/secure-btc-bridge-understanding-sbtcs-security-model) |
| Signer count (design ceiling) | up to 150 | 2026 | [BitcoinWrites sBTC security model](https://www.bitcoinwrites.com/p/secure-btc-bridge-understanding-sbtcs-security-model) |
| Threshold for signature | 70% | 2026 | [BitcoinWrites sBTC security model](https://www.bitcoinwrites.com/p/secure-btc-bridge-understanding-sbtcs-security-model) |

## How it works

### User perspective

To peg in: send BTC to a sBTC deposit address on the Bitcoin L1 with appropriate metadata. The deposit is detected by signers, who mint an equivalent amount of sBTC on Stacks to the user's Stacks address. Users can then use sBTC in Stacks DeFi (Zest, Granite, StackingDAO, etc.).

To peg out: call `initiate-withdrawal-request` on the sBTC withdrawal contract on Stacks, locking the sBTC plus a max-fee amount. The signers process the request, send BTC on the Bitcoin L1 to the recipient, and the actual fee paid to Bitcoin miners (capped at max-fee) is settled. Withdrawals went live at the end of April 2024.

### Protocol perspective

- **Threshold signature wallet:** a Weighted Schnorr Threshold Signature (WSTS) scheme over the signer set. Currently 15 signers, scaling toward 150. 70% of weight must sign for any valid peg-out.
- **Signer selection:** signers are elected through Bitcoin transactions because Stacks contracts can read Bitcoin state. Signers lock STX as bond.
- **Economic incentives:** signers earn BTC rewards for honest operation; their STX bond is at risk if they misbehave (and economic loss far exceeds short-term theft value while the peg is small enough).
- **Permissionless signer set:** anyone meeting the bond requirement can join or leave, distinguishing sBTC from federated wrappers like WBTC.
- **Contract-bound peg-out:** signers cannot send BTC arbitrarily. They can only co-sign transactions that satisfy the Stacks contract's withdrawal logic (recipient, amount, fee bounds).
- **Nakamoto upgrade (Q4 2024):** added Bitcoin finality for Stacks transactions, faster block times between Bitcoin blocks, and the trustless two-way peg.

## Key behaviours

- [[patterns/threshold-signature-custody]] : 70% weighted threshold over a permissionless signer set.
- [[patterns/btc-anchored-finality]] : Stacks transactions inherit Bitcoin finality post-Nakamoto.
- [[patterns/bond-based-signer-security]] : signers post STX bond, earn BTC yield.

## Architecture decisions

- **WSTS over a multisig federation:** WSTS allows the signer set to scale (up to ~150) and rotate without re-keying. Federated bridges (WBTC, tBTC v1) are simpler but cap decentralisation at a small operator count.
- **STX bond denominated separately from BTC reward:** economic separation prevents signers from cashing out via the same asset they secure.
- **Open signer entry/exit:** prioritises decentralisation over operational control.
- **70% threshold (not 51%):** higher safety margin against collusion at the cost of reduced liveness during high signer churn.
- **Lifted deposit caps in Q1 2026:** signals that the design has cleared initial conservative bounds.

## Differentiators

- Most decentralised BTC-pegged asset by signer-count and signer-bond economics in 2026. Competing wrapped BTCs (WBTC, cbBTC, BitGo-custodied variants) remain federated or single-custodian.
- Native Bitcoin finality through Stacks (post-Nakamoto), not just Stacks-only finality.
- Integrated DeFi stack (Zest lending, Granite, StackingDAO) gives sBTC immediate utility on Stacks.

## Limitations and criticisms

- 15 signers is small relative to design ceiling; until the set scales toward 150, the practical decentralisation is meaningful but not exceptional.
- Threshold signature schemes have non-trivial liveness failure modes: if more than 30% of signer weight goes offline, peg-outs stall.
- STX/BTC economic balance: if sBTC TVL grows large relative to total STX bond value, the attack incentive may exceed the bond, breaking the security model.
- Bridge cap removal in Q1 2026 brought sBTC TVL to $545M (peak) then $437M — large, but still modest versus WBTC ($10B+ historically) and the surface area for an exploit grows with TVL.
- sBTC is one of multiple competing Bitcoin synthetic representations (cbBTC, tBTC, sBTC, BBN-derived assets); fragmentation reduces network effect for any single variant.

## Sources

- [Stacks Q1 2026 snapshot](https://www.stacks.co/blog/q1-2026-snapshot) : accessed 2026-05-19
- [Yellow.com : Stacks Q1 2026 BTC TVL release](https://yellow.com/press-releases/stacks-stx-closes-q1-2026-with-437m-btc-tvl-320-btc-added-to-bitcoin-staking-pilot) : accessed 2026-05-19
- [sBTC whitepaper PDF](https://assets.stacks.co/sbtc.pdf) : accessed 2026-05-19
- [sBTC design : trustless two-way peg](https://stacks-network.github.io/stacks/sbtc.html) : accessed 2026-05-19
- [Stacks docs : sBTC peg-out](https://docs.stacks.co/more-guides/sbtc/bridging-bitcoin/sbtc-to-btc) : accessed 2026-05-19
- [BitcoinWrites : sBTC security model](https://www.bitcoinwrites.com/p/secure-btc-bridge-understanding-sbtcs-security-model) : accessed 2026-05-19
- [GitHub stacks-sbtc/wsts](https://github.com/stacks-sbtc/wsts) : accessed 2026-05-19
- [Stacks blog : Introducing sBTC and Nakamoto](https://stacks.org/sbtc-nakamoto) : accessed 2026-05-19
- [Phemex : Stacks Q1 2026 sBTC TVL](https://phemex.com/news/article/stacks-network-reports-437m-sbtc-tvl-in-q1-2026-76789) : accessed 2026-05-19
