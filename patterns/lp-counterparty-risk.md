---
tags: [pattern, liquidity, lp, risk]
seen_in: [synthetix-v3, gmx-v2, jupiter-perps, ostium]
---

# LP-as-counterparty Risk

In oracle-priced perp venues, traders trade against a liquidity pool (or staker debt pool), not against another trader. LPs are the systemic counterparty. When traders are net long during a rally, the pool loses; when traders are net short during a sell-off, the pool loses.

## Why it matters

- LP yield comes from trading fees plus negative PnL from traders.
- LPs are exposed to directional risk regardless of pool composition.
- Pool composition shifts as positions open and close, so LPs' effective exposure can move faster than they can rebalance.

## Implementations

- **[[projects/synthetix-v3]]:** SNX stakers + multi-collateral pools back per-market debt. When trader PnL is positive, stakers' aggregate debt grows. V3 lets stakers choose which pools they back to limit exposure.
- **[[projects/gmx-v2]]:** GM pools (single-market) and GLV vaults (multi-market). LPs deposit basket assets; pool absorbs trader PnL. GLV improves capital efficiency by dynamic allocation.
- **[[projects/jupiter-perps]]:** JLP pool (SOL + ETH + wBTC + USDC + USDT). LPs hold a multi-asset index that takes the inverse of net trader positioning. Gets 75% of platform fees.
- **[[projects/ostium]]:** Single USDC OLP vault is the sole counterparty for all markets (crypto and RWA). Concentrated risk but simple accounting.

## Failure modes

- **One-way trader skew during a strong trend:** if 80% of OI is long during a 30% rally, the pool's net short position bleeds heavily. Funding rates push back but can be insufficient if the move is fast.
- **Correlated liquidations:** when traders are liquidated, the pool keeps the remaining collateral, but if the oracle moves before liquidation can be executed, the pool can be left with negative-equity positions.
- **Oracle failure:** see [[patterns/oracle-pricing-model]]. An oracle attack on an LP-pool venue immediately transfers value from LPs to whoever places the manipulated trade.
- **LP withdrawal stampedes:** if LPs panic-withdraw after a drawdown, the pool's remaining LPs are diluted into the bad positions.

## Mitigations seen across protocols

- **Funding rate** that scales with skew, pushing traders to balance OI.
- **Borrow fees** that LPs collect even on unfilled side, generating baseline yield.
- **Position caps** per market and per asset to bound directional exposure.
- **Insurance funds** that absorb tail losses (HLP on Hyperliquid plays this role explicitly; GMX and Jupiter rely on funding/liquidation buffers).
- **Skew-sensitive borrow rate** (Synthetix V3) so opening a position against the existing skew pays the LP more.

## Relevance to RFP

Any synthetic asset RFP that uses an LP-pool counterparty model must specify:

- LP pool composition (single-asset, multi-asset basket, or staker debt pool).
- Funding rate formula and skew sensitivity.
- Max position size per market and per asset.
- Insurance fund or backstop vault behaviour.
- LP withdrawal lockups or cooldowns to prevent stampedes.
