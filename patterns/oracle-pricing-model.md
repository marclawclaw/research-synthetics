---
tags: [pattern, oracle, pricing, execution]
seen_in: [synthetix-v3, gmx-v2, drift, jupiter-perps, ostium]
---

# Oracle-priced Execution

Almost every non-orderbook synthetic asset venue uses an oracle as the canonical execution price. This is the architectural alternative to a bid/ask orderbook: rather than matching opposing orders, the venue routes every order against a liquidity pool at the oracle mid-price.

## Why it matters

- **No size-based slippage at execution:** a $10M trade and a $100 trade execute at the same oracle price. The cost of size shows up via funding, borrow rate, or "price impact" rather than orderbook depth.
- **Total dependency on oracle integrity:** a stale, manipulated or mis-published price leads directly to mis-priced liquidations and trader/LP losses.
- **Liquidations are oracle-triggered:** every liquidation, funding update and PnL calculation runs through the oracle.

## Implementations

- **[[projects/synthetix-v3]]:** market creators can combine Chainlink, Pyth and Uniswap TWAP. Oracle Manager lets per-market aggregation. Commit-reveal settlement reduces front-running.
- **[[projects/gmx-v2]]:** exclusively Chainlink Data Streams (pull-based, signed price reports). Keepers post the signed report alongside order execution; contract verifies signature, then executes.
- **[[projects/drift]]:** Pyth as primary, configurable fallback. The vAMM's mid is pegged to the live oracle, with dynamic spread and peg updates before fills.
- **[[projects/jupiter-perps]]:** three-oracle aggregation (Edge by Chaos Labs, Chainlink, Pyth). Multi-feed redundancy reduces single-oracle failure risk.
- **[[projects/ostium]]:** dual oracle stack. Chainlink Data Streams for crypto pairs, custom licensed-data aggregator for RWA pairs.

## Trade-offs versus orderbook (CLOB) pricing

| Dimension | Oracle-priced | Orderbook |
|-----------|---------------|-----------|
| Slippage at execution | None (oracle mid) | Yes (depth-dependent) |
| Cost of size | Funding, borrow, price impact | Bid/ask spread, depth |
| Reliance on external feed | Total | Minimal (some venues use oracle for liquidations only) |
| Front-running surface | Oracle update timing | Order placement timing |
| Failure mode | Bad oracle = bad fills + bad liquidations | Liquidity drying up = wider spreads |

## Relevance to RFP

If an RFP requires synthetic exposure to BTC, ETH or SOL with predictable execution and no orderbook to bootstrap, oracle-priced pricing is the default. Requirements should specify:

- Oracle sources (Chainlink Data Streams, Pyth, or specific multi-feed aggregation).
- Liveness guarantees (max oracle staleness).
- Front-running mitigation (pull-based reports + commit-reveal settlement).
- Fallback behaviour when the primary oracle is unavailable.

See also [[patterns/lp-counterparty-risk]] for the LP side of this design.
