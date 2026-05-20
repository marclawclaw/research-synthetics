---
tags: [project, solana, perpetuals, lp-pool]
ecosystem: Solana
category: Perpetuals
website: https://jup.ag/perps
docs: https://hub.jup.ag/guides/perpetual-exchange
launched: 2024
---

# Jupiter Perps

Jupiter Perps is the perpetual futures venue inside Jupiter, Solana's dominant DEX aggregator. It uses a multi-asset liquidity pool (the JLP pool: SOL, ETH, wBTC, USDC, USDT) as the counterparty to all leveraged traders. Pricing comes from a multi-oracle aggregation (Edge by Chaos Labs, Chainlink, Pyth) so trade execution has no orderbook slippage.

## Adoption metrics

| Metric | Value | Date | Source |
|--------|-------|------|--------|
| JLP TVL | ~$1.3B | April 2026 | [Eco support : JLP](https://eco.com/support/en/articles/15083164-jupiter-perps-fees-leverage-how-jlp-works) |
| 2025 trading volume | $263B | 2025 | [Yellow.com Jupiter research](https://yellow.com/research/jupiter-solana-dex-aggregator-market-cap-2026) |
| Quarterly revenue | ~$45M | 2025 | [Yellow.com Jupiter research](https://yellow.com/research/jupiter-solana-dex-aggregator-market-cap-2026) |
| Share of Solana perp volume | ~80% | 2026 | [Yellow.com Jupiter research](https://yellow.com/research/jupiter-solana-dex-aggregator-market-cap-2026) |
| Share of Solana DEX aggregator volume | ~95% | 2026 | [Yellow.com Jupiter research](https://yellow.com/research/jupiter-solana-dex-aggregator-market-cap-2026) |
| JLP TVL ranking on Solana | 3rd largest pool (behind Jito, Kamino) | 2026 | [Eco support : JLP](https://eco.com/support/en/articles/15083164-jupiter-perps-fees-leverage-how-jlp-works) |
| LP yield (historical range) | 30-80% APY | 2024-2026 | [Eco support : JLP](https://eco.com/support/en/articles/15083164-jupiter-perps-fees-leverage-how-jlp-works) |

## How it works

### User perspective

A trader picks a market (SOL/USD, ETH/USD, wBTC/USD) and opens a long or short. The execution price is the oracle-aggregated mid. No orderbook means large orders have no price impact; the cost of size shows up via borrow fees and an opening fee. Profits are paid from the JLP pool; losses are added to it.

LPs mint JLP by depositing any of the basket assets (SOL, ETH, wBTC, USDC, USDT). LPs are entitled to 75% of platform fees (open, close, borrow, swap) and earn yield denominated in the basket assets themselves, not a separate reward token.

### Protocol perspective

- **JLP pool:** multi-asset index pool that backs every trade. Pool composition matters: when traders are net long, the pool's exposure to BTC/ETH/SOL goes down; net short positions raise pool exposure.
- **Multi-oracle aggregation:** Edge (Chaos Labs), Chainlink, Pyth. Multiple feeds reduce single-oracle failure risk and let the protocol fall back if any one feed deteriorates.
- **Liquidation rule:** position is liquidated when remaining collateral (less fees and unrealised loss) is below 0.2% of position size.
- **Fee model:** open/close fee, hourly borrow fee, swap fees on JLP minting/redemption.

## Key behaviours

- [[patterns/lp-counterparty-risk]] : JLP holders are the counterparty to all traders.
- [[patterns/oracle-pricing-model]] : multi-oracle aggregated mark price.
- [[patterns/no-slippage-large-orders]] : oracle-mid pricing means trade size does not move execution price.

## Architecture decisions

- **Multi-asset LP pool (not single-side):** depositors get diversified exposure to SOL/ETH/wBTC/USDC/USDT, lowering single-asset directional risk.
- **75% fee share to LPs:** high payout to keep the pool well-capitalised; protocol revenue comes from the remaining 25%.
- **Three-oracle aggregation:** redundancy and robustness over a single dominant feed.
- **Tight integration with Jupiter aggregator:** users can swap on Jupiter and open positions in the same UI; routing keeps friction low.

## Differentiators

- Largest perpetual venue on Solana by volume and TVL.
- Multi-asset LP pool (versus GMX V2's GM single-market pools, or Synthetix's debt pool).
- LP yield is high because Jupiter's aggregator routes huge organic trader flow into the perp pool.

## Limitations and criticisms

- LP directional risk: JLP holders take the inverse of net trader positioning. During strong rallies with mostly long traders, JLP NAV can lag.
- Oracle dependency: even with three feeds, a coordinated oracle failure or manipulation across providers would be catastrophic.
- Concentration risk: Jupiter is the single dominant Solana perp venue, which makes the JLP pool a systemic Solana DeFi position.
- Pool composition slippage: when traders flip net direction sharply, JLP composition shifts and LP exposure changes faster than they can rebalance.

## Sources

- [Jupiter docs : How Jupiter Perps work](https://hub.jup.ag/guides/perpetual-exchange/how-it-works) : accessed 2026-05-19
- [Jupiter support : price oracles](https://support.jup.ag/hc/en-us/articles/19355326396700-Price-Oracles) : accessed 2026-05-19
- [Jupiter support : fees](https://support.jup.ag/hc/en-us/articles/18735045234588-Fees) : accessed 2026-05-19
- [Eco support : JLP fees, leverage, how it works](https://eco.com/support/en/articles/15083164-jupiter-perps-fees-leverage-how-jlp-works) : accessed 2026-05-19
- [Yellow.com : Jupiter Solana DEX aggregator 2026](https://yellow.com/research/jupiter-solana-dex-aggregator-market-cap-2026) : accessed 2026-05-19
- [CoinMarketCap : JLP token](https://coinmarketcap.com/currencies/jupiter-perps-lp/) : accessed 2026-05-19
