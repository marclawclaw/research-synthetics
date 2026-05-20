---
tags: [project, arbitrum, avalanche, perpetuals, synthetics]
ecosystem: Ethereum L2 (Arbitrum), Avalanche, MegaETH, Botanix
category: Perpetuals
website: https://gmx.io
docs: https://docs.gmx.io
launched: 2021 (v1), 2023 (v2)
---

# GMX V2

GMX is a permissionless perpetual exchange that produces synthetic price exposure to BTC, ETH and other tracked assets through liquidity pools (GM and GLV) priced by Chainlink Data Streams oracles, not an orderbook. Traders take leveraged positions against the pool; LPs are the counterparty.

## Adoption metrics

| Metric | Value | Date | Source |
|--------|-------|------|--------|
| TVL (V2 perps total) | $203.47M | May 2026 | [DefiLlama GMX V2 Perps](https://defillama.com/protocol/gmx-v2-perps) |
| TVL on Arbitrum | $191.43M | May 2026 | [DefiLlama GMX V2 Perps](https://defillama.com/protocol/gmx-v2-perps) |
| TVL on Avalanche | $10.99M | May 2026 | [DefiLlama GMX V2 Perps](https://defillama.com/protocol/gmx-v2-perps) |
| TVL on MegaETH | $1.03M | May 2026 | [DefiLlama GMX V2 Perps](https://defillama.com/protocol/gmx-v2-perps) |
| 24h volume | $106.46M | May 2026 | [DefiLlama GMX V2 Perps](https://defillama.com/protocol/gmx-v2-perps) |
| 7d fees | $456,470 | May 2026 | [DefiLlama GMX V2 Perps](https://defillama.com/protocol/gmx-v2-perps) |
| Cumulative trading volume (all chains, since launch) | >$345B across >735K traders | 2026 | [Coinbureau GMX review](https://coinbureau.com/review/gmx-review/) |

## How it works

### User perspective

A trader deposits supported collateral on Arbitrum, Avalanche, MegaETH or Botanix, then opens a long or short on a market (e.g. ETH/USD). Execution price is the Chainlink Data Streams index price at the moment a keeper picks up the request; the trader pays a borrow fee while the position is open and pays/receives funding based on long/short skew. No orderbook, no slippage based on order size: pricing is the oracle mid, and the cost of size is captured through funding and price impact.

LPs mint GM (single-market) or GLV (multi-market vault) tokens by depositing the pool's basket assets. They earn 60-70% of fees but absorb the pool's PnL versus traders.

### Protocol perspective

- **GM pools:** isolated single-market pools. Each backs one market (e.g. ETH/USD-WETH-USDC).
- **GLV vaults:** yield vaults that dynamically allocate liquidity across multiple GM pools based on utilisation.
- **Chainlink Data Streams:** pull-based, low-latency oracle. Keepers fetch a signed price report and post it with the order; the contract verifies the signature and uses the report price for execution.
- **Two-step keeper execution:** order requests are submitted, then a keeper executes against the next valid oracle update. This blocks oracle front-running because the executor cannot choose a stale price.

## Key behaviours

- [[patterns/lp-counterparty-risk]] : GM/GLV LPs are direct counterparty to traders.
- [[patterns/oracle-pricing-model]] : Chainlink Data Streams pull oracle as the canonical execution price.
- [[patterns/keeper-based-execution]] : keepers submit oracle updates and execute orders.

## Architecture decisions

- **Oracle-mid pricing:** zero slippage on execution because price comes from oracle, not from an orderbook depth curve. Size cost is captured via dynamic funding and price impact, not bid/ask depth.
- **Permissionless market listing in V2:** markets can be added with minimal governance, expanding beyond crypto majors.
- **Multi-chain expansion:** beyond Arbitrum and Avalanche, GMX has deployed to MegaETH and Botanix (a BTC-anchored EVM), plus a separate GMX-Solana deployment using Chainlink Data Streams on Solana.
- **GLV vault for capital efficiency:** allocates LP capital across GM pools by demand, lifting average capital utilisation.

## Differentiators

- One of the largest oracle-priced (non-orderbook) perp DEXes by cumulative volume.
- Strong real-world asset perp expansion: 24/7 oil, gold, silver, gas perps with 100x leverage launched 2026 ($1.18B volume in one week).
- Multi-chain native presence including a BTC L2 (Botanix) and a Solana deployment.

## Limitations and criticisms

- LPs (GM/GLV) carry directional risk: when traders are skewed long during a rally, the pool pays out. V1 LPs (GLP) saw drawdowns during sharp moves.
- Oracle dependence is total: a malicious or stale Chainlink Data Streams update would mis-price liquidations directly. Keeper enforcement of "next valid update" is the main mitigation.
- Funding-rate model can be aggressive in skewed conditions, pushing traders out of positions.
- TVL has declined materially relative to 2024 peaks as Hyperliquid and orderbook competitors took share.

## Sources

- [DefiLlama GMX V2 Perps page](https://defillama.com/protocol/gmx-v2-perps) : accessed 2026-05-19
- [GMX docs](https://docs.gmx.io/docs/intro/) : accessed 2026-05-19
- [GMX-Solana adopts Chainlink Data Streams](https://gmxio.substack.com/p/gmx-solana-adopts-chainlink-data) : accessed 2026-05-19
- [GMX V2 a quick guide (blocmates)](https://www.blocmates.com/articles/gmx-v2-a-quick-guide-to-the-upgrade) : accessed 2026-05-19
- [Coinbureau GMX review 2026](https://coinbureau.com/review/gmx-review/) : accessed 2026-05-19
- [GMX live on Ethereum mainnet](https://gmxio.substack.com/p/gmx-is-live-on-ethereum) : accessed 2026-05-19
