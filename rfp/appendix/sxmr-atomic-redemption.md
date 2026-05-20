---
tags: [appendix, synthetic, monero, atomic-swap, pitch, lez]
related: [patterns/synthetic-collateral-models, patterns/oracle-pricing-model, projects/sbtc-stacks, rfp/appendix/lez-vs-dedicated-zone]
---

# sXMR: Oracle-priced Synthetic with Atomic-Swap Redemption

## The pitch

A synthetic Monero token (sXMR) deployed as a program on the canonical Logos Execution Zone (LEZ). Oracle-priced for trading and composability inside LEZ; redeemed to real XMR via peer-to-peer atomic swap with no custodian, no bridge, and no protocol-held reserves.

Deploying on the canonical LEZ (rather than spinning up a separate, sXMR-specific zone) gives sXMR direct composability with every other LEZ program: lending markets can accept sXMR as collateral, DEXes can list it, and the protocol inherits canonical-LEZ consensus security with no separate validator set. See [[rfp/appendix/lez-vs-dedicated-zone]] for the canonical-LEZ vs separate-zone trade-off.

**The wedge:** the only synthetic that terminates in a privacy-preserving asset on a privacy chain. sBTC redeems to public BTC. Every other synthetic redeems to traceable stablecoins. sXMR is the first design where the redemption path itself preserves privacy.

**Honest framing:** this is a synthetic with a soft, market-clearing peg, not a hard-redeemable synthetic. The oracle is the *quoted* price; the *achievable* price is whatever an XMR LP will swap for, when one is willing. Structurally closer to a DEX trading pair than to [[projects/sbtc-stacks]].

The RFP must pick one of two goals before specifying further. They are incompatible:

1. **Non-custodial, mostly-works:** pure atomic-swap design ships. Soft peg, market-dependent exit. The interesting product.
2. **Always real XMR at oracle price:** requires bonded LPs (slashable stable collateral on LEZ) or a protocol XMR reserve. Custody or solvency risk returns. The marketable product.

---

## Goal 1: Non-custodial, mostly-works

### Premise

sXMR is a free-floating claim. The protocol provides three things: an oracle reference price, a Token (issued by the LEZ program) that represents synthetic XMR exposure, and a matching venue where sXMR holders and XMR holders discover each other and execute atomic swaps. The protocol holds no XMR, runs no signer set, and offers no redemption SLA.

### Architecture

```
   ┌────────────────────────┐  oracle (XMR/USD)  ┌───────────────────┐
   │  sXMR LEZ program      │◄───────────────────│  Oracle program   │
   │  (Token + stable vault)│                    │  on LEZ           │
   └───────────┬────────────┘                    └───────────────────┘
               │
       mint    │    burn
               ▼
   ┌────────────────────────┐                   ┌───────────────────┐
   │  Intent gossip         │◄─── match ───────►│  Open XMR LPs     │
   │  (off-chain, via Logos │                   │  (anonymous, free │
   │  Delivery)             │                   │   to enter/exit)  │
   │  sXMR ↔ XMR quotes     │                   │                   │
   └───────────┬────────────┘                   └─────────┬─────────┘
               │                                          │
               │      atomic swap (adaptor-sig)           │
               │      LEZ (LEE) ◄──────────────► Monero L1│
               ▼                                          ▼
       sXMR holder gets XMR                  XMR LP gets sXMR,
       on Monero L1                          burns for stable
```

### Properties

- **Non-custodial.** No vault holds XMR. No signer set. No bridge.
- **Soft peg.** Oracle is a reference; achievable redemption price is whatever an LP quotes. Spread widens under stress without bound.
- **No redemption guarantee.** Counterparty may not exist when you want to exit.
- **Composable on LEZ.** sXMR is a vanilla Token, callable by any other LEZ program (lending, DEX, governance).
- **Private exit.** Successful redemption deposits real XMR on Monero L1, severing the public trail.
- **Open LP set.** Anyone with XMR can quote, no permission required.
- **Regulatory minimalism.** Protocol does not handle XMR; arguably just a price feed plus a matching board.

### When this fits

- Privacy-maximalist user base willing to accept variable redemption.
- Use cases that need sXMR for *trading exposure* rather than guaranteed redemption (DeFi composability, hedging, speculation).
- Builders willing to ship the cryptographically pure version and let the market clear.

### Failure modes

1. **LP exodus:** all XMR holders stop quoting. sXMR trades at an indefinite discount to oracle.
2. **Adverse selection:** LPs only show up when oracle is below true XMR price (free money for them) and vanish when oracle is above (would-be loss). Redemption is asymmetric across regimes.
3. **Demand asymmetry:** easy to mint sXMR (privacy-curious DeFi users want it), hard to source LPs (XMR maximalists may not want a public LP role at all).
4. **UX cost:** Monero atomic swap windows are 30 to 60 minutes; both parties must be online unless an intent layer is built on top.

---

## Cross-cutting design challenge: where the orderbook lives, and what can be enforced

Two questions that the rest of this document treats as settled, but are not:

### The orderbook probably should not be on-chain

An on-chain orderbook (whether on the canonical LEZ or anywhere else) is expensive, leaks intent publicly (which undermines the privacy story), and provides no security benefit: the atomic swap is what makes settlement trustless, not the matching layer. A better split:

- **Off-chain matching via Logos Delivery.** LPs broadcast quotes; redeemers broadcast intents; parties pair up bilaterally. Quotes and intents never hit chain state.
- **On-chain settlement only.** A minimal LEZ program that verifies the atomic-swap primitive (lock, reveal, refund) on the LEZ side. It knows nothing about prices, identities, or who matched with whom.

This also aligns better with Goal 1's privacy proposition: an on-chain orderbook would be a public registry of "everyone trying to acquire real XMR right now."

The same reasoning applies, more strongly, to the second question.

### Atomic-swap execution is hard to enforce

Atomic swaps are deliberately symmetric: either party can refuse the next message at any stage, and both sides refund at timeout. On-chain evidence cannot distinguish:

- An LP maliciously refusing to proceed,
- An LP whose node went down or lost connectivity,
- A redeemer never locking their side, making the LP's non-lock correct behaviour,
- A redeemer locking and then refusing to reveal, blaming the LP.

This is the whole point of an atomic swap: nobody can be forced to complete, and nobody loses funds if they walk away.

Consequently, **a clean "LP defaulted → slash the bond" rule is not enforceable from on-chain state alone.** Any slashing or "punishment" mechanism needs an off-chain attestation of who refused, which means one of:

- A trusted attestor or committee deciding default, which re-introduces centralised trust.
- A reputation system, which is only useful at scale and cannot enforce against first-time defection.
- A multi-attestor oracle quorum watching both chains, which adds its own trust assumption and liveness requirement.

Without one of those, the on-chain program can only do one thing: deny future LP slots, queue priority, or fee tiers. It cannot slash collateral with cryptographic certainty.

This is the same architectural reason the orderbook does not benefit from being on-chain: the chain cannot adjudicate participation, only finalise settlement when both parties cooperate.

### Implication for the goals below

- **Goal 1** is unaffected. It never claimed enforceable LP commitments.
- **Goal 2a (bonded LPs)** is structurally weaker than its description below suggests. The bond cannot be slashed on a simple "LP refused" condition; any real slashing requires a reputation or attestor system layered on top. Read the design as best-effort with a reputation layer, not as cryptographically guaranteed redemption.
- **Goal 2b (protocol reserve)** is unaffected by the symmetry problem; trust simply lives in the signer set instead.

---

## Goal 2: Always real XMR at oracle price

### Premise

sXMR must be redeemable for real XMR at (or very near) the oracle price, on demand. The atomic swap is still the settlement primitive, but counterparty availability is no longer left to the open market. The protocol either commits LPs to honour redemption or holds an XMR reserve itself.

Two sub-designs, each restoring some trust that Goal 1 deliberately avoided.

### Sub-design 2a: Bonded LP set

LPs join a registered set. Each LP posts stable collateral on LEZ equal to (or some multiple of) their XMR commitment. When a redemption request is routed to an LP, they must complete the atomic swap within a window. If they default, their bond is slashed and paid to the redeemer. LPs may leave the set, but only after a notice period that exceeds the redemption SLA.

**Enforceability caveat (see the cross-cutting section above):** "LP defaulted" is not a verdict an on-chain program can render from atomic-swap state alone. Any slashing rule needs an off-chain attestor or a reputation system to attribute fault. Without one, the bond can be used to gate participation (priority, fee tiers, future-slot access) but not slashed with cryptographic certainty on a single failed swap.

```
                          ┌─────────────────────────┐
                          │  sXMR LEZ program       │
                          │  + LP registry          │
                          │  + slashing logic       │
                          └────┬──────────┬─────────┘
                               │          │
              ┌────────────────┘          └──────────────────┐
              │ redemption request                    LP bond│
              ▼                                              ▼
   ┌─────────────────────┐                        ┌─────────────────────┐
   │  sXMR holder        │                        │  Bonded LP          │
   │  burns sXMR,        │       atomic swap      │  posts stable bond, │
   │  receives XMR       │◄──────────────────────►│  delivers XMR or    │
   │                     │   (adaptor-sig, SLA)   │  forfeits bond      │
   └─────────────────────┘                        └─────────────────────┘
              ▲                                              ▲
              │ if LP defaults: bond is paid out as          │
              └─ compensation, oracle attests non-delivery ──┘
```

### Sub-design 2b: Protocol XMR reserve

The protocol accumulates an XMR reserve from mint fees, a yield programme, or a one-time treasury seed. Reserve is held in a threshold-signer multisig or analogous custody arrangement on Monero. Redemption draws from the reserve directly, with the atomic swap acting as the settlement rail between the reserve custodian and the redeemer.

```
                          ┌─────────────────────────┐
                          │  sXMR LEZ program       │
                          │  + reserve accounting   │
                          └───────────┬─────────────┘
                                      │
                       burn sXMR      ▼     trigger swap
                                  ┌───────────────┐
                                  │ Reserve module│
                                  │ (LEZ program) │
                                  └───────┬───────┘
                                          │
                       atomic swap        ▼
                  ┌───────────────────────────────────────┐
                  │  Threshold-signer reserve on Monero   │
                  │  (n-of-m, bonded signers)             │
                  └───────────────────────────────────────┘
```

At this point the design has reinvented [[projects/sbtc-stacks]] with an oracle bolted on. The atomic swap is just the wire format; trust lives in the signer set.

### Properties

| Property | 2a Bonded LPs | 2b Protocol reserve |
|----------|---------------|---------------------|
| Custodian | None (LPs custody their own XMR) | Yes (signer set) |
| Redemption guarantee | Up to total bonded capacity | Up to reserve size |
| Slashing surface | Yes (bond slashed on default) | No (reserve is the slashing) |
| Oracle role | Pricing + default attestation | Pricing only |
| LP economics | Yield from spreads plus protocol incentives, less bond opportunity cost | N/A (no third-party LPs) |
| Decentralisation | High (anyone can be a bonded LP if they post the bond) | Low (signer set is gated) |
| Censorship resistance | Medium (LP set is registered) | Low (signers are known) |
| Best-case redemption speed | Atomic swap (30 to 60 min) | Atomic swap (30 to 60 min) |
| Failure mode | Bond runs out under coordinated default | Signer collusion or custody breach |

### When this fits

- Audiences that need a redemption SLA (institutions, market makers, structured products).
- Use cases where sXMR is collateral inside other protocols and a stable peg matters more than purist non-custody.
- Regulatory contexts where "guaranteed redemption" is a feature, not a liability.

### Failure modes

- **Bonded LPs (2a):** coordinated default exceeds bonded capacity; the slashing oracle for default attestation is itself a trusted component; bond opportunity cost limits LP supply.
- **Protocol reserve (2b):** signer set is a target; if reserve is undercollateralised, peg breaks; effectively recreates [[projects/sbtc-stacks]] custody risk.

---

## Property matrix across goals

| Property | Goal 1 (pure) | Goal 2a (bonded LPs) | Goal 2b (reserve) | sBTC (reference) |
|----------|---------------|----------------------|-------------------|------------------|
| Custodian | None | None (LPs self-custody) | Yes | Yes |
| Reserve | None | None | Yes | Yes |
| Redemption guarantee | None | Bounded by bonds | Bounded by reserve | Bounded by reserve |
| Peg type | Soft | Hard within capacity | Hard within capacity | Hard 1:1 |
| Oracle dependency | Pricing | Pricing + default attestation | Pricing | None for peg |
| LP role | Optional, open | Registered, bonded | None | None |
| Privacy on redemption | Yes (XMR L1) | Yes (XMR L1) | Yes (XMR L1) | No (BTC L1) |
| Worst-case failure | Indefinite discount, no exit | Bond depleted, partial exit | Signer compromise, full loss | Signer compromise, full loss |
| Closest existing analogue | DEX trading pair | Bonded relay (no direct analogue) | [[projects/sbtc-stacks]] | [[projects/sbtc-stacks]] |

---

## Decision tree for the RFP

```
                Does the RFP need a redemption SLA?
                              │
                ┌─────────────┴─────────────┐
               no                          yes
                │                            │
                ▼                            ▼
         Goal 1 (pure)        Does the protocol accept custody risk?
                                             │
                              ┌──────────────┴───────────────┐
                             no                              yes
                              │                                │
                              ▼                                ▼
                   Goal 2a (bonded LPs)              Goal 2b (reserve)
                                                  (== sBTC with oracle)
```

## Pre-spec validation

Before committing to either goal:

1. **Atomic swap UX with Monero today.** Live protocols (unstoppableswap, COMIT) take 30 to 60 minutes and require both parties online. Confirm async / intent-based variants are production-grade before designing UX around them.
2. **LP supply.** Will XMR holders actually LP? They self-select for privacy maximalism and may not want a public on-chain LP role. The LP side is the bottleneck; validate before designing the rest.
3. **Bond economics (Goal 2a only).** Required bond size as a multiple of XMR notional; opportunity cost of locked stable collateral on LEZ; expected APY needed to attract bonded LPs.
4. **Signer set sourcing (Goal 2b only).** Same problem space as [[projects/sbtc-stacks]]; revisit that project's trust assumptions before reinventing them.
5. **Regulatory.** A synthetic that terminates in a privacy coin will draw scrutiny under any of the three designs. Goal 1 has the cleanest defence (protocol is a price feed and a matching board); Goal 2b has the weakest (protocol custodies XMR).

## Bottom line

- **Goal 1** is the most interesting design and the one with the strongest privacy story. It will not satisfy users who expect a redemption SLA.
- **Goal 2a** is the most novel of the SLA-bearing designs: non-custodial, atomic-swap-settled, but with bonded LPs underwriting redemption capacity. No direct analogue exists in the vault.
- **Goal 2b** is a real product but, structurally, an oracle-priced sBTC for Monero. The atomic swap is cosmetic; the trust assumption is the signer set.

Pick the goal before writing the spec. The three designs have different threat models, different LP economics, and different regulatory exposures.
