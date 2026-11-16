---
tags: [synthetics, monero, zcash, privacy, community-sentiment, history]
category: ecosystem-history
launched: n/a
---

# Monero and Zcash Synthetics: Community History

Survey of whether synthetic assets backed by or representing Monero (XMR) and Zcash (ZEC) have been discussed, proposed, or built. The short answer: yes, repeatedly, but almost every concrete instance has been a custodial wrapper (WZEC, WXMR, renZEC, sXMR via Chainlink price feed) rather than a non-custodial synthetic backed by native XMR/ZEC. The community reaction in both ecosystems is consistently sceptical of the privacy and trust tradeoffs.

## Summary findings

| Instrument | Type | Status | Launched | Source |
|------------|------|--------|----------|--------|
| Synthetix sXMR | Price-tracked synth (Chainlink oracle; backed by the Synthetix debt pool, i.e. staked SNX collateral) | Live as ERC-20, low volume; SIP not located | 2020 | [Etherscan](https://etherscan.io/token/0x5299d6F7472DCc137D7f3C4BcfBBB514BaBF341A) |
| Synthetix sZEC (SIP-119) | Price-tracked synth proposal | Rejected | Feb 2021 (proposed) | [SIP-119](https://sips.synthetix.io/sips/sip-119) |
| WZEC (Tokensoft / Anchorage) | Custodial wrapped ERC-20 | Live | 2019/2020 | [Decrypt](https://decrypt.co/45899/wrapped-zcash-is-coming-to-ethereums-defi-ecosystem) |
| WXMR (BTSE) | Custodial wrapped ERC-20 | Live | Jan 2021 | [Cointelegraph](https://cointelegraph.com/news/wrapped-monero-launches-as-exchanges-crack-down-on-privacy-coins) |
| wXMR (bonedaddy) | M-of-N multisig wrapper | Experimental / partial MVP | 2020-2021 | [GitHub](https://github.com/bonedaddy/wXMR) |
| renZEC | Custodial-ish (RenVM darknodes) bridge token | Sunset Nov 2022 | 2020 | [Zcash forum](https://forum.zcashcommunity.com/t/ren-1-0-being-sunset-bridge-renzec-to-native-zec-as-soon-as-possible/43393) |
| WZEC (Cyfrin grant, 2026) | BitGo-custodied wrapper with Chainlink PoR | Grant rejected Mar 2026 | n/a | [ZCG forum](https://forum.zcashcommunity.com/t/grant-application-wrapped-zcash-wzec/54575) |
| Privacy Bridge (NEAR nZEC) | Custodial bridge + lending market | Grant rejected Jan 2026 | n/a | [Zcash forum](https://forum.zcashcommunity.com/t/proposal-for-a-grant-to-build-privacy-bridge-a-defi-platform-for-zcash-users/54032) |
| NEAR Intents ZEC | Custodial/intent-based bridge (Omnibridge) | Live | 2025 | [Zcash forum](https://forum.zcashcommunity.com/t/near-intents/54588) |
| ETH-XMR atomic swap (Athanor Labs) | Native swap, not synthetic | Live beta | 2022 onward | [CCS](https://ccs.getmonero.org/proposals/noot-eth-xmr-atomic-swap.html) |
| Serai DEX | Native cross-chain LP DEX, not synthetic | In development | n/a | [Serai](https://serai.exchange/) |
| Maya / THORChain XMR-ZEC native swap | Native, not synthetic | Live (Zcash on Maya) | 2024-2025 | [Maya blog](https://www.mayaprotocol.com/blog-maya-academy/zcash-integrates-maya) |
| Monero-Starknet atomic swap (DLEQ) | Native swap, not synthetic | Alpha | Dec 2025 | [Starknet forum](https://community.starknet.io/t/trustless-monero-starknet-atomic-swaps-using-dleq-proofs/116084) |

## Zcash community: documented history

### 2019: original "Wrapped ZEC" thread

A thread titled "Wrapped ZEC" was opened on the Zcash Community Forum on November 7, 2019 by tromer, reacting to a CoinDesk piece. The thread distinguishes two design archetypes that recur in every later debate:

1. **WZEC**: a central custodian model (the same custody design that BTSE later used for WXMR and that BitGo proposes in 2026).
2. **TZEC**: an "economic system on a remote chain" (closer to a non-custodial synthetic).

Contributors prestwich, mhluongo and nathan-at-least raised the recurring privacy objection: remote wrapped ZEC cannot share Zcash's anonymity set, and wrap/unwrap operations themselves leak. The Electric Coin Company position at the time was supporting infrastructure (MMR / ZIP-221) rather than building a wrapped product directly.

Source: [forum.zcashcommunity.com/t/wrapped-zec/35331](https://forum.zcashcommunity.com/t/wrapped-zec/35331).

### 2020: WZEC ships (custodial)

Tokensoft and Anchorage launched WZEC as an ERC-1404 token, 1:1 backed by ZEC under Anchorage custody. Decrypt covered the launch in May 2020. WZEC plus RenVM's renZEC became the standard route for ZEC into Maker, Compound and Uniswap. Aave governance discussed adding WZEC and renZEC as collateral in late 2020.

### Jan 2021: "Please help me understand WZEC" thread

Forum thread posted January 11, 2021 captures the canonical user-side framing of the trust model:

> "none of the current wrapped offerings are both completely decentralized and trustless. You ultimately need to trust the ZEC custodian to allow you to convert your WZEC back to ZEC when you ask."

> "The downside is that there are fees involved and the wrapped versions of Zcash have no privacy."

Source: [forum.zcashcommunity.com/t/please-help-me-understand-wzec/38223](https://forum.zcashcommunity.com/t/please-help-me-understand-wzec/38223).

### Feb 2021: SIP-119 proposes sZEC, rejected

Maximilian Roszko submitted SIP-119 to Synthetix on February 13, 2021, proposing sZEC as a Chainlink-fed synth that would be pooled with renZEC on Curve for cross-pool swaps. The economic argument cited "on average a billion dollars worth of trading volume per day, all of that on centralized exchanges." Status: **Rejected**. No equivalent sXMR SIP appears in the Synthetix repo; the existing sXMR ERC-20 at `0x5299d6F7...F341A` predates the public SIP process.

Source: [SIP-119](https://sips.synthetix.io/sips/sip-119).

### Nov 2022: renZEC sunset

The Ren team announced the sunset of Ren 1.0 after the Alameda bankruptcy. Zcash forum users were instructed to burn renZEC back to native ZEC before mid-December 2022. Mint was already disabled. This was, in practice, the death of the largest synthetic ZEC-on-Ethereum venue at the time.

Source: [forum.zcashcommunity.com/t/ren-1-0-being-sunset-bridge-renzec-to-native-zec-as-soon-as-possible/43393](https://forum.zcashcommunity.com/t/ren-1-0-being-sunset-bridge-renzec-to-native-zec-as-soon-as-possible/43393).

### 2025-2026: NEAR Intents and Privacy Bridge

NEAR Intents went live in 2025, bridging ZEC to Solana and other chains via Omnibridge. Reported as the first time a privacy coin had "significant DEX-connected liquidity" (community quote on the NEAR Intents thread). The grant application posted February 6, 2026 was withdrawn February 20, 2026 (community pushback on grant size, not on the bridge itself).

Privacy Bridge (a ZEC-collateralised lending market on NEAR using wrapped nZEC) was rejected by ZCG on January 5, 2026.

Source: [NEAR Intents thread](https://forum.zcashcommunity.com/t/near-intents/54588), [Privacy Bridge thread](https://forum.zcashcommunity.com/t/proposal-for-a-grant-to-build-privacy-bridge-a-defi-platform-for-zcash-users/54032).

### Feb-Apr 2026: Cyfrin WZEC grant

Mark Scrine (CSO Cyfrin), Patrick Collins and Hans Friese applied for an $80K ZCG grant on February 5, 2026 for a trust-minimised WZEC using BitGo custody plus Chainlink Proof-of-Reserve. Community reaction mixed:

- Supportive: decentrathai called "BitGo + Chainlink PoR is about as trust-minimized as you can get."
- Critical: outgoing.doze said "the custodial architecture is the weak link" and asked whether funds should go to trustless alternatives like red·bridge instead.
- ZCG rejected the grant on March 20, 2026 as "out of scope," with artkor clarifying the real objection was the $70K liquidity-bootstrapping component, not the wrapper itself.

Source: [forum.zcashcommunity.com/t/grant-application-wrapped-zcash-wzec/54575](https://forum.zcashcommunity.com/t/grant-application-wrapped-zcash-wzec/54575).

## Monero community: documented history

### sXMR on Synthetix (pre-SIP era)

Synth sXMR exists as an ERC-20 at `0x5299d6F7472DCc137D7f3C4BcfBBB514BaBF341A`, listing Monero among Synthetix's earliest crypto synths. It uses Chainlink price feeds. Like all V1/V2 synths it is minted against SNX staked by the debt pool (sUSD is the exchange and settlement unit, not the collateral); it does not touch the Monero chain at all. There is no recorded community sentiment thread on the Monero side about sXMR; it predates the public SIP repo, and the Monero forum and r/Monero appear not to have discussed it materially.

A Secret Network forum thread ("XMR Rewards Plan") references incentivising sXMR pools on SecretSwap when the Monero bridge went live, indicating modest cross-ecosystem awareness but no Monero-side endorsement.

### Jan 2021: BTSE launches WXMR; Monero leads publicly reject

BTSE (Jonathan Leong) launched WXMR on Ethereum on January 6, 2021. Coverage in Cointelegraph and CryptoBriefing carries the canonical Monero-community quote, from then-community lead Justin Ehrenhofer:

> "Wrapped XMR has different (worse) privacy than XMR... Ethereum is highly transparent. Thus people can see when you move this wrapped asset around."

Riccardo Spagni (former Monero lead maintainer) posted satirical commentary on Twitter, sarcastically saying exchanges that delisted XMR for being untraceable should be happy to list the wrapped, traceable version.

Sources: [Cointelegraph](https://cointelegraph.com/news/wrapped-monero-launches-as-exchanges-crack-down-on-privacy-coins), [CryptoBriefing](https://cryptobriefing.com/monero-uniswap-btse-wrapped-ethereum/).

This is the strongest public-record Monero-community position on synthetic / wrapped XMR: the privacy properties don't survive the wrapping, so wrapping defeats the asset's reason to exist.

### bonedaddy/wXMR (experimental multisig)

A 2020-era GitHub project by bonedaddy attempted a non-custodial alternative: an M-of-N multisig wallet holding native XMR, with periodic reserve proofs published on-chain and burn-redemption from the reserve. Repo shows 36 commits, 9 stars, partial MVP, no recent activity. No prominent forum thread documents the project being adopted or endorsed.

Source: [github.com/bonedaddy/wXMR](https://github.com/bonedaddy/wXMR).

### Direction the Monero community actually backed

CCS (Monero's community crowdfunding) funded the **ETH-XMR atomic swap** (noot / Athanor Labs) from January through June 2022. The framing is explicitly about trustless **native** swaps, not synthetics. The CCS proposal contains zero references to creating a synthetic XMR.

Source: [ccs.getmonero.org/proposals/noot-eth-xmr-atomic-swap.html](https://ccs.getmonero.org/proposals/noot-eth-xmr-atomic-swap.html).

Follow-on work (Athanor Labs ETH-XMR repo, the Starknet DLEQ proposal in December 2025) extends the same atomic-swap line of thinking to new chains. Serai DEX, Haveno, Maya Protocol's Zcash integration, and THORChain's Monero/Zcash integration plans all explicitly market themselves as **no wrapping, no synthetics**.

> "no more wrapped tokens, just pure, native assets." — Maya Protocol marketing copy

The general 2026 community framing on r/Monero (per search aggregators; direct Reddit threads were not directly indexable) is to "not trust any wrapped XMR on BNB Chain unless you personally audit the bridge/reserves," with existing wrapped XMR tokens categorised as "custodial bridges with high trust risk, low-liquidity experimental tokens, and outright scams."

## Patterns observed

### Both ecosystems prefer native swaps over synthetics

Every well-received cross-chain integration in either community over the last three years is a **native swap** (atomic swaps, vault-based AMMs like THORChain/Maya, intent networks). Every synthetic or wrapped representation that has been proposed or shipped has either:

1. Been a centralised custodial wrapper (WXMR/BTSE, WZEC/Anchorage, WZEC/BitGo proposal), and met with public scepticism from community leadership; or
2. Been a price-tracked oracle synth with no actual XMR/ZEC backing (sXMR on Synthetix, SIP-119 sZEC proposal), and either rejected or low-engagement; or
3. Been an experimental multisig (bonedaddy wXMR) that never reached production.

### The privacy objection is the universal blocker

The objection cited in 2019 (tromer thread), 2021 (Ehrenhofer on WXMR, "please help me understand WZEC" thread) and 2026 (Cyfrin grant thread) is the same: a synthetic on a transparent chain (Ethereum, NEAR, Solana) does not inherit the privacy properties of native XMR/ZEC, and the wrap/unwrap step is itself a deanonymisation event. This is the load-bearing objection to every synthetic XMR/ZEC design we found.

### Trust-minimisation is the only direction with traction

The 2026 Cyfrin WZEC proposal explicitly frames itself as "trust-minimised" (BitGo custody plus Chainlink PoR), reflecting that the prior generation (Anchorage WZEC, BTSE WXMR, RenVM) is treated as unacceptable. Even so, community feedback ("the custodial architecture is the weak link") suggests the bar has shifted from "custodial with audit" toward "non-custodial or don't bother."

A non-custodial synthetic, in which XMR/ZEC are locked or escrowed by atomic-swap-like protocols rather than custodians, has been hinted at (the 2019 "TZEC" thread, the bonedaddy multisig design) but no funded production project exists.

## Implications for sXMR-as-LEZ RFP

The RFP context (Logos sXMR as a program on the canonical Logos Execution Zone (LEZ) / privacy synthetic) lands in a gap that has been recognised but not filled: every funded effort has been either a custodial wrapper (rejected by community) or a native atomic swap (accepted, but does not produce a synthetic asset usable as collateral). A non-custodial, atomic-swap-backed synthetic XMR is something the Monero community has *not* explicitly debated (search did not surface a forum thread on this design specifically). The closest analogues are the 2019 TZEC concept on the Zcash side and the bonedaddy wXMR multisig design on the Monero side.

Practical takeaway for the RFP: the privacy-objection precedent ([[patterns/synthetic-collateral-models]], [[patterns/bridge-trust-assumptions]]) is the central argument any sXMR proposal must rebut. Whatever escrow/atomic-swap mechanism is chosen must:

1. Avoid named custodians (BitGo, Anchorage, exchanges).
2. Not require the synthetic to live on a chain whose privacy is weaker than XMR's, or accept that the synthetic is for collateral-utility only and is not a privacy product.
3. Frame the wrap/unwrap event itself as not leaking more than an atomic swap already leaks.

## Sources

- [Synthetix Proxy sXMR Token (Etherscan)](https://etherscan.io/token/0x5299d6F7472DCc137D7f3C4BcfBBB514BaBF341A) — accessed 2026-05-20 — [archived](../sources/2026-05-20-etherscan-sxmr.html)
- [SIP-119: Add ZEC crypto Synth (Synthetix)](https://sips.synthetix.io/sips/sip-119) — accessed 2026-05-20 — [archived](../sources/2026-05-20-sips-synthetix-io-sip-119.html)
- [Wrapped Zcash thread (Zcash forum, 2019)](https://forum.zcashcommunity.com/t/wrapped-zec/35331) — accessed 2026-05-20 — [archived](../sources/2026-05-20-forum-zcashcommunity-wrapped-zcash-2019.html)
- [Please help me understand WZEC (Zcash forum, 2021)](https://forum.zcashcommunity.com/t/please-help-me-understand-wzec/38223) — accessed 2026-05-20 — [archived](../sources/2026-05-20-forum-zcashcommunity-wzec-understand.html)
- [Ren 1.0 being sunset (Zcash forum, 2022)](https://forum.zcashcommunity.com/t/ren-1-0-being-sunset-bridge-renzec-to-native-zec-as-soon-as-possible/43393) — accessed 2026-05-20 — [archived](../sources/2026-05-20-forum-zcashcommunity-ren-sunset.html)
- [Grant Application: Wrapped Zcash WZEC (Zcash forum, 2026)](https://forum.zcashcommunity.com/t/grant-application-wrapped-zcash-wzec/54575) — accessed 2026-05-20 — [archived](../sources/2026-05-20-forum-zcashcommunity-wzec-grant-2026.html)
- [NEAR Intents (Zcash forum, 2026)](https://forum.zcashcommunity.com/t/near-intents/54588) — accessed 2026-05-20 — [archived](../sources/2026-05-20-forum-zcashcommunity-near-intents.html)
- [Privacy Bridge grant proposal (Zcash forum, 2026)](https://forum.zcashcommunity.com/t/proposal-for-a-grant-to-build-privacy-bridge-a-defi-platform-for-zcash-users/54032) — accessed 2026-05-20 — [archived](../sources/2026-05-20-forum-zcashcommunity-privacy-bridge.html)
- [bonedaddy/wXMR (GitHub)](https://github.com/bonedaddy/wXMR) — accessed 2026-05-20 — [archived](../sources/2026-05-20-github-bonedaddy-wxmr.html)
- [ETH-XMR Atomic Swap Development (Monero CCS)](https://ccs.getmonero.org/proposals/noot-eth-xmr-atomic-swap.html) — accessed 2026-05-20 — [archived](../sources/2026-05-20-ccs-getmonero-org-noot-eth-xmr-atomic-swap.html)
- [BTSE Wrapped Monero on Ethereum (CryptoBriefing, Jan 2021)](https://cryptobriefing.com/monero-uniswap-btse-wrapped-ethereum/) — accessed 2026-05-20 — [archived](../sources/2026-05-20-cryptobriefing-monero-uniswap-btse-wrapped.html)
- [Wrapped Monero launches as exchanges crack down on privacy coins (Cointelegraph, Jan 2021)](https://cointelegraph.com/news/wrapped-monero-launches-as-exchanges-crack-down-on-privacy-coins) — accessed 2026-05-20 — [archived](../sources/2026-05-20-cointelegraph-wrapped-monero-launches.html)
- [Trustless Monero-Starknet Atomic Swaps Using DLEQ Proofs (Starknet forum, Dec 2025)](https://community.starknet.io/t/trustless-monero-starknet-atomic-swaps-using-dleq-proofs/116084) — accessed 2026-05-20 — [archived](../sources/2026-05-20-community-starknet-monero-atomic-swaps.html)
- [Wrapped Zcash is Coming to Ethereum's DeFi Ecosystem (Decrypt, 2019)](https://decrypt.co/45899/wrapped-zcash-is-coming-to-ethereums-defi-ecosystem) — accessed 2026-05-20 — unarchived (Decrypt blocks scrape)
- [Wrapped Monero Appears in the Age of DeFi (Decrypt, 2021)](https://decrypt.co/53402/wrapped-monero-appears-in-the-age-of-defi) — accessed 2026-05-20 — unarchived (Decrypt blocks scrape)
- [Zcash x Maya integration (Maya Protocol blog)](https://www.mayaprotocol.com/blog-maya-academy/zcash-integrates-maya) — accessed 2026-05-20 — unarchived
