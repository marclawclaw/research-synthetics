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

- **[[projects/synthetix-v3]] (retired June 2026):** market creators can combine Chainlink, Pyth and Uniswap TWAP. Oracle Manager lets per-market aggregation. Commit-reveal settlement reduces front-running.
- **[[projects/gmx-v2]]:** exclusively Chainlink Data Streams (pull-based, signed price reports). Keepers post the signed report alongside order execution; contract verifies signature, then executes.
- **[[projects/drift]]:** Pyth as primary, configurable fallback (pre-exploit; the 2026-04-01 loss came through multisig governance and a fake collateral token listed at an inflated price, so the oracle path was bypassed rather than broken; Velocity's oracle is not named in its docs). The vAMM's mid is pegged to the live oracle, with dynamic spread and peg updates before fills.
- **[[projects/jupiter-perps]]:** Edge by Chaos Labs as primary with Chainlink and Pyth as fallbacks. Multi-feed redundancy reduces single-oracle failure risk.
- **[[projects/ostium]]:** dual oracle stack per its deprecated docs (Chainlink Data Streams for crypto pairs, custom licensed-data aggregator with Stork nodes for RWA pairs). On 2026-07-15 an attacker submitted fake future-dated BTC-USD prices through a compromised signer and keeper path and drained $23.75M from the OLP vault; the exploited path was Ostium-operated, so whether Chainlink was involved is unconfirmed (sources disagree on root cause).
- **[[projects/metronome-synth]]:** a swap module minted and burned synths at oracle price and was exposed to Chainlink update lag, leaving about $15.7M of unbacked synths in July 2026. Oracle-priced minting outside the liquidation path is a failure pattern: see [[patterns/cdp-synthetics-landscape]].

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
