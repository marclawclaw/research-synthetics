---
tags: [project, custom-l1, perpetuals, orderbook]
ecosystem: Hyperliquid L1 (USDC bridge on Arbitrum)
category: Perpetuals
website: https://hyperliquid.xyz
docs: https://hyperliquid.gitbook.io/hyperliquid-docs
launched: 2023
---

# Hyperliquid

Hyperliquid is a custom Layer-1 blockchain purpose-built for an on-chain central limit orderbook (CLOB) perpetuals exchange. By 2026 it has captured roughly 70% of all on-chain perpetuals volume. Its dual-layer architecture splits execution between HyperCore (orderbook + matching) and HyperEVM (general-purpose EVM smart contracts sharing state with HyperCore).

## Adoption metrics

| Metric | Value | Date | Source |
|--------|-------|------|--------|
| TVL | $5.111B | May 2026 | [DefiLlama Hyperliquid](https://defillama.com/protocol/hyperliquid) |
| 24h perp volume | $8.516B | May 2026 | [DefiLlama Hyperliquid](https://defillama.com/protocol/hyperliquid) |
| 30d perp volume | $176.06B | May 2026 | [DefiLlama Hyperliquid](https://defillama.com/protocol/hyperliquid) |
| Open interest | $8.86B | May 2026 | [DefiLlama Hyperliquid](https://defillama.com/protocol/hyperliquid) |
| Share of on-chain perps volume | ~70% | 2026 | [Yellow.com research](https://yellow.com/research/hyperliquid-perp-volume-dominance-how-2026) |
| Transactions per second (claimed) | 200,000+ | 2026 | [Yellow.com research](https://yellow.com/research/hyperliquid-perp-volume-dominance-how-2026) |
| HyperEVM development teams deployed | 175+ | early 2026 | [HypeWatch guide](https://www.hypewatch.io/blog/hyperliquid-complete-guide-2026) |
| Validator nodes controlling USDC bridge | 4 | 2026 | [QuillAudits Hyperliquid review](https://www.quillaudits.com/blog/blockchain/hyperliquid-security-beyond-orderbooks) |

## How it works

### User perspective

USDC is bridged from Arbitrum to Hyperliquid via the bridge contract. The user funds an HLP account, then places limit, market or trigger orders on perps (BTC, ETH, SOL, hundreds of altcoin markets). Matching happens in HyperCore, which is the on-chain CLOB; finality is ~1 second with HyperBFT consensus.

For developers, HyperEVM exposes the same state to general-purpose EVM contracts, so a DeFi app can read positions or compose with the orderbook.

### Protocol perspective

- **HyperCore:** the custom L1 module running the CLOB matching engine and perp accounting natively in consensus.
- **HyperEVM:** an EVM execution environment colocated with HyperCore, sharing state.
- **HyperBFT consensus:** custom BFT consensus across the validator set.
- **HLP (Hyperliquidity Provider) vault:** protocol vault that supplies market-making liquidity, performs liquidations and earns a share of fees. HLP is exempt from some position restrictions to maintain a continuous quote.
- **Bridge:** custodial USDC bridge on Arbitrum secured by validator signatures (3-of-4 multisig effectively, with a ~200 second dispute window).

## Key behaviours

- [[patterns/on-chain-clob]] : full orderbook lives in consensus, not off-chain.
- [[patterns/bridge-trust-assumptions]] : USDC custody on Arbitrum is held by a small validator multisig.
- [[patterns/protocol-vault-market-maker]] : HLP is the platform's house market maker.

## Architecture decisions

- **Custom L1 instead of L2 or appchain on Cosmos:** maximum control over throughput, finality and fee model; tightly couples consensus with the matching engine.
- **HyperEVM for composability:** added later to enable an open developer ecosystem without compromising HyperCore performance.
- **Tight validator set for the bridge:** prioritises low-latency withdrawals (~200s dispute) at the cost of decentralisation. Independent analysis characterises it as effectively a 3-of-4 multisig.
- **HLP as house maker:** protocol-owned market making capacity (and a yield product for depositors), but it operates under looser limits than regular accounts.

## Differentiators

- Largest on-chain perp venue by volume by a wide margin: 30d perp volume of $176B is more than all other on-chain derivatives combined.
- Flipped Coinbase notional volume for 2025.
- On-chain CLOB with sub-second finality, not vAMM or LP-pool model.

## Limitations and criticisms

- **Bridge centralisation:** USDC custody bridge is held by ~4 validator nodes; the same team controls both hot and cold validator keys per independent analysis.
- **200-second dispute window:** much shorter than Arbitrum/Optimism's ~7-day fraud-proof windows, and disputes can only be raised by validators, not the public.
- **Closed validator set:** until validators are externally distributed, the chain is effectively a permissioned high-performance system marketed as decentralised.
- **HLP risk concentration:** as the house market maker, HLP losses in a tail event would propagate to all depositors.
- **Auditor coverage:** bridge contracts audited by Cyfrin, but the full stack (HyperCore matching, HyperBFT) has not had public top-tier audits matching its scale.

## Sources

- [DefiLlama Hyperliquid](https://defillama.com/protocol/hyperliquid) : accessed 2026-05-19
- [Hyperliquid docs : bridge](https://hyperliquid.gitbook.io/hyperliquid-docs/hypercore/bridge) : accessed 2026-05-19
- [Hyperliquid docs : protocol vaults](https://hyperliquid.gitbook.io/hyperliquid-docs/hypercore/vaults/protocol-vaults) : accessed 2026-05-19
- [QuillAudits : Hyperliquid security architecture](https://www.quillaudits.com/blog/blockchain/hyperliquid-security-beyond-orderbooks) : accessed 2026-05-19
- [OneKey : Hyperliquid bridge risk](https://onekey.so/blog/ecosystem/hyperliquid-bridge-risk/) : accessed 2026-05-19
- [Yellow.com : Hyperliquid perp dominance 2026](https://yellow.com/research/hyperliquid-perp-volume-dominance-how-2026) : accessed 2026-05-19
- [HypeWatch : Hyperliquid 2026 complete guide](https://www.hypewatch.io/blog/hyperliquid-complete-guide-2026) : accessed 2026-05-19
- [The Market Periodical : Hyperliquid flipped Coinbase notional 2025](https://themarketperiodical.com/2026/02/11/hyperliquid-flipped-coinbases-notional-volume-for-2025/) : accessed 2026-05-19
