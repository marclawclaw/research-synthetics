---
tags: [project, cosmos, appchain, perpetuals, orderbook]
ecosystem: dYdX Chain (Cosmos SDK appchain)
category: Perpetuals
website: https://dydx.exchange
docs: https://docs.dydx.exchange
launched: 2019 (v1), 2023 (v4 / dYdX Chain)
---

# dYdX v4

dYdX v4 is the migration of dYdX from a StarkEx-based Ethereum L2 to a sovereign Cosmos SDK appchain ("dYdX Chain"). It runs a fully decentralised orderbook by having each validator maintain an in-memory orderbook off-chain; only matched trades are committed on-chain each block. This achieves the speed of off-chain matching while keeping settlement, custody and matching logic open and verifiable.

## Adoption metrics

| Metric | Value | Date | Source |
|--------|-------|------|--------|
| Average daily trading volume | ~$200M | 2026 | [Coinbureau dYdX 2026 review](https://coinbureau.com/review/dydx) |
| Peak daily volume (April 2026) | $500M | April 2026 | [Coinbureau dYdX 2026 review](https://coinbureau.com/review/dydx) |
| Open interest | ~$175M | 2026 | [Coinbureau dYdX 2026 review](https://coinbureau.com/review/dydx) |
| Active markets | 182+ | 2026 | [Coinbureau dYdX 2026 review](https://coinbureau.com/review/dydx) |
| Validator set (initial cap) | up to 60 validators | 2024+ | [Chorus One dYdX Chain 101](https://chorus.one/articles/network-101-a-comprehensive-guide-to-dydx-v4) |

## How it works

### User perspective

A trader bridges USDC into the dYdX Chain, then places limit, market or conditional orders against the orderbook for the chosen perp market. Orders propagate through the validator gossip network, matches occur in validator memory in real time, and the settled trades are written into the next on-chain block. The user experience approximates a centralised exchange.

### Protocol perspective

- **Cosmos SDK + CometBFT:** delegated proof-of-stake L1 with a validator set responsible for both consensus and exchange logic.
- **In-memory orderbook:** each validator runs an identical matching engine; orderbook state is eventually consistent across validators and is not committed to consensus.
- **On-chain settlement:** the result of matching (trades, position updates, funding payments) is committed each block.
- **Native USDC:** custody is native to the dYdX Chain; no external bridge in the trading critical path.
- **DYDX token:** stakers secure the chain and earn fees; governance controls protocol parameters and validator set rules.

## Key behaviours

- [[patterns/on-chain-clob]] : matching is off-chain in validator memory but settlement is on-chain.
- [[patterns/appchain-validator-set]] : trust assumption is that 2/3 of the validator set is honest.
- [[patterns/no-bridge-architecture]] : USDC lives natively on dYdX Chain, removing bridge risk.

## Architecture decisions

- **Custom appchain over rollup:** chose the Cosmos SDK to control the matching engine and avoid Ethereum L1 settlement costs. Trade-off: weaker security than Ethereum L1, no Ethereum composability.
- **In-memory orderbook:** keeps matching speed comparable to centralised exchanges while preserving open-source matcher logic that every validator runs identically.
- **Validator-side exchange logic:** all matching, funding and risk-engine logic ships as part of the validator binary, ensuring consensus on every trade.
- **2026 roadmap:** ecosystem grants, real-world asset markets, token economic updates focused on value accrual.

## Differentiators

- One of the few perp DEXes where the matching engine is genuinely decentralised (every validator runs it; no central operator).
- Long operational track record predating the migration: dYdX v1-v3 ran since 2019.
- Mature risk engine (cross-margining, isolated margin, liquidation auctions).

## Limitations and criticisms

- Volume has plateaued versus Hyperliquid: $200M daily versus Hyperliquid's $8.5B daily.
- Appchain isolation: no easy composability with Ethereum or Solana DeFi.
- USDC liquidity must be bridged in via Noble or Skip; while not in the trading path, it still introduces a non-trivial onramp.
- Initial validator set was small and dominated by professional validators; geographic and operator diversity has improved but is still narrower than mature L1s.

## Sources

- [Chorus One : dYdX Chain network 101](https://chorus.one/articles/network-101-a-comprehensive-guide-to-dydx-v4) : accessed 2026-05-19
- [dYdX v4-chain repo](https://github.com/dydxprotocol/v4-chain) : accessed 2026-05-19
- [Coinbureau dYdX 2026 review](https://coinbureau.com/review/dydx) : accessed 2026-05-19
- [Medium : Decentralized order book design in dYdX v4](https://medium.com/@gwrx2005/decentralized-order-book-design-in-dydx-v4-625ac0152e80) : accessed 2026-05-19
- [Medium : dYdX v4 architectural evolution](https://medium.com/@gwrx2005/dydx-v4-architectural-and-protocol-evolution-from-v3-6c312f51f7b7) : accessed 2026-05-19
- [Datawallet : DYDX V4 explained](https://www.datawallet.com/crypto/what-is-dydx) : accessed 2026-05-19
