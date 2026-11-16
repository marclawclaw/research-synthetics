---
tags: [pattern, oracle, cdp, units, collateral, debt, design]
seen_in: [sky-maker, liquity, synthetix-v2, metronome-synth, mirror-protocol, alchemix, youves, frankencoin, crvusd]
---

# Units of collateral and debt in CDPs and multi-synth protocols

Question: when a CDP takes collateral that is not USD (for example ETH) and/or issues debt that is not USD (a BTC, EUR, JPY or floating synth), how do existing protocols value each side for the collateral ratio (CR) check, and what does that require of the oracle? Context: a CDP minting sXMR and sBTC on a new chain whose oracles publish USD pairs only (XMR/USD, BTC/USD, ETH/USD), plus a planned RAI-style floating stablecoin.

Method: contract source read from shallow clones (commit hashes in Sources), on-chain `eth_call` reads through `ethereum-rpc.publicnode.com` at block 26,426,729 (2026-11-16 09:35 UTC), and project docs where no source was available. All accessed 2026-11-16.

## 1. The arithmetic

Every protocol checked reduces to the same inequality:

`collateral_amount x P(collateral) >= debt_amount x P(debt) x required_ratio`

with both prices in one unit of account. If both prices are quoted in USD, the USD cancels: `P(collateral)/P(debt)` is the cross rate (ETH per XMR, say). So "two USD feeds divided" and "one direct cross pair" are the same number; the designs differ in where the division happens, which feeds go stale together, and whether the debt side is read from an oracle at all.

Observed variants of the debt side:

- **Fixed par:** the debt token is valued at a constant (1 USD, 1 UST, or a governance-set `par`). Only the collateral feed is read.
- **Oracle-priced debt:** each synth has its own USD feed; debt is summed in USD.
- **Internal target price:** the debt is valued at a protocol-computed redemption price that drifts by a controller-set rate; no oracle on the debt side for the CR check.
- **Same-asset debt:** debt and collateral share a denomination, so no external price is needed (except for wrapper exchange rates).
- **Oracle-free:** the borrower declares a price, and third parties challenge it.

## 2. Comparison table

| Protocol | Collateral | Debt unit | CR unit of account | Feeds read for CR | How combined | Debt valuation | Staleness and fallback | Source |
|---|---|---|---|---|---|---|---|---|
| Sky/Maker (Vat, Spotter) | Per ilk (ETH-A read here) | DAI/USDS | "ref" (USD) per collateral unit, scaled to DAI by `par` | One per ilk: OSM over a Median; ETH-A OSM `src` Median `wat()` = "ETH/USD", `bar()` = 13 (chain) | `spot = val / par / mat` in `Spotter.poke` (`src/spot.sol`); safe if `art x rate <= ink x spot` (`src/vat.sol`) | Fixed par: `par` "ref per dai", on chain `1.0` (1e27 ray); governance can `file("par")` | OSM delays prices by `hop` = 3,600 s (chain, ETH-A); `peek` returns `has` flag, `spot` = 0 if invalid; OSM `stop`/`void` by auth. ETH-A `mat` = 1.45 (chain) | dss `spot.sol`, `vat.sol`; osm `osm.sol`; chain reads |
| Reflexer RAI (OracleRelayer) | ETH | RAI | USD per ETH, divided by the RAI redemption price | One: ETH-A OSM (`collateralTypes("ETH-A")` orcl `0x105b...4ba3`) | `safetyPrice = price / redemptionPrice() / safetyCRatio` in `updateCollateralPrice` (`src/single/OracleRelayer.sol`) | Internal target price: `redemptionPrice = redemptionRate^(dt) x _redemptionPrice` (`updateRedemptionPrice`); on chain 2.9720 USD, rate exactly 1.0 (zero), last update 2026-02-06 | OSM `updateDelay` 3,600 s (chain); validity flag only, no age check in the relayer (see surprise below). Safety and liquidation CRatio both 1.35 (chain) | geb `OracleRelayer.sol`; geb-fsm `OSM.sol`; chain reads |
| RAI rate setter (controller input, not CR) | n/a | RAI | USD | RAI market price: Uniswap RAI/ETH TWAP times an ETH/USD "converter feed" | `converterComputeAmountOut` in `UniswapConsecutiveSlotsPriceFeedMedianizer.sol`; `PIRateSetter.updateRate` feeds `marketPrice` and `redemptionPrice` to the PI calculator | n/a | Requires `hasValidValue`; rate update gated by `updateRateDelay` | geb-uniswap-median; geb-rrfm-rate-setter `PIRateSetter.sol` |
| Liquity V1 | ETH | LUSD | USD per ETH | Chainlink ETH/USD, Tellor ETH/USD (request ID 1) as fallback | `_computeCR(coll, debt, price) = coll x price / debt` (`LiquityMath.sol`); `PriceFeed.fetchPrice` state machine | Fixed: 1 LUSD = 1 USD (implicit; price is ETH/USD only) | `TIMEOUT` 4 h per oracle; Chainlink "broken" if price jumps over 50% between rounds; switch back needs Chainlink and Tellor within 5%; both untrusted: `lastGoodPrice` | dev `PriceFeed.sol`, `LiquityMath.sol` |
| Liquity V2 (BOLD) WETH branch | WETH | BOLD | USD per ETH | Chainlink ETH/USD | `WETHPriceFeed`; same `_computeCR` | Fixed 1 USD | Staleness threshold per feed (deploy script: 24 h ETH/USD); on failure the branch is shut down and `lastGoodPrice` is used | bold `WETHPriceFeed.sol`, `MainnetPriceFeedBase.sol`, `DeployLiquity2.s.sol` |
| Liquity V2 wstETH branch | wstETH | BOLD | USD per wstETH | Chainlink stETH/USD, Chainlink ETH/USD, `stEthPerToken()` from the wstETH contract | Borrow and liquidation: `stETH/USD x stEthPerWstEth`. Redemption: `max(stETH/USD, ETH/USD) x rate` if the two USD prices are within 1% | Fixed 1 USD | stETH/USD down: shut branch, switch to `ETH/USD x canonical rate` and keep `min(that, lastGoodPrice)`; ETH/USD or rate down: shut branch, `lastGoodPrice`. Deploy script staleness 24 h for both | bold `WSTETHPriceFeed.sol`, `CompositePriceFeed.sol` |
| Liquity V2 rETH branch | rETH | BOLD | USD per rETH | Chainlink ETH/USD, Chainlink rETH/ETH, `getExchangeRate()` from rETH | `market = ETH/USD x rETH/ETH`, `canonical = ETH/USD x rate`; borrow and liquidation use `min(market, canonical)`; redemption uses `max` if within 2% | Fixed 1 USD | As wstETH; deploy script staleness 24 h ETH/USD, 48 h rETH/ETH | bold `RETHPriceFeed.sol` |
| JPY DeFi (JPYdf, Liquity V2 fork) | wstETH (as dwstETH), USDC (dUSDC) | JPYdf | Yen | Per branch collateral/USD feed, shared JPY/USD, stETH/ETH and ETH/USD for redemption, dToken backing ratio | `collateral price in yen = (collateral/USD) / (JPY/USD) x backing ratio`; dwstETH borrow and liquidation `stETH/USD x stETH per wstETH`; redemption `max(stETH/USD, ETH/USD x stETH/ETH)` | Fixed par of 1 yen per JPYdf, via the JPY/USD feed (docs) | Collateral feed fail: branch pause. Redemption needs ETH/USD under 2 h and stETH/ETH under 25 h; JPY/USD 30 h or older stops normal redemption only; JPY/USD under 72 h used as is, then last saved price for 24 h, then pause until governance multisig resumes. Docs flag weekend FX: JPY/USD timestamp advances while price is flat | [JPY DeFi oracle docs](https://jpydf.gitbook.io/jpy-defi/collateral-and-pricing/oracle.md) |
| Synthetix V2 (Issuer, ExchangeRates, DebtCache) | SNX | Debt pool share, denominated in sUSD; any synth (sETH, sBTC, sEUR) | sUSD (= USD); individual CR expressed as debt/collateral in SNX units | One Chainlink aggregator per currency key, SNX/USD included; sUSD hardcoded | `_effectiveValueAndRates`: `value = amount x sourceRate / destinationRate` ("source -> USD -> destination"); `DebtCache._issuedSynthValues`: `sum(totalSupply x rate)`; `Issuer._collateralisationRatio` = debt in SNX / SNX | Oracle-priced per synth; sUSD rate fixed at 1.0 (`_getRateAndUpdatedTime`: "sUSD rate is 1.0") | `rateStalePeriod()` on chain = 90,000 s (25 h); stale, flagged or circuit-broken rates mark `anyRateInvalid`, blocking issuance; sUSD "never stale" | npm `synthetix@2.102.1` `ExchangeRates.sol`, `Issuer.sol`, `BaseDebtCache.sol`; chain read |
| Synthetix inverse synths (historical) | n/a | iETH, iBTC etc. | USD | Underlying USD feed | `_rateOrInverted`: `rate = 2 x entryPoint - price`, clamped to `[lowerLimit, upperLimit]` and frozen at a limit once hit | Oracle-derived | Same as other synths; code absent from 2.102.1 apart from one stale comment | npm `synthetix@2.30.0` `ExchangeRates.sol` |
| Metronome Synth (Pool, MasterOracle) | Multi (USDC, ETH, WBTC, vault tokens) | msUSD, msETH, msBTC | USD | One USD price per token through `MasterOracle.quoteTokenToUsd` (Chainlink per docs) | `Pool.debtPositionOf`: `debtOf` = `sum quoteTokenToUsd(synth, balance)`; `depositOf` = `sum quoteTokenToUsd(underlying, balance) x collateralFactor`; healthy if debt <= limit | msETH and msBTC oracle-priced (chain: 1,795.73 and 64,323.12 USD per unit); msUSD returns exactly 1.0 from its own oracle `0x2789...afb1` (chain) while DefiLlama prices it at $0.818 | [NOT FOUND] in the public repo (MasterOracle source is not in it); docs mention "internal oracle parameters" only | metronome-synth-public `Pool.sol`, `IMasterOracle.sol`; chain reads; [[projects/metronome-synth]] |
| Mirror (Terra, dead) | UST, aUST, LUNA, others | mAssets (mAAPL etc.) | Collateral asset units | mAsset price and collateral price, both quoted in the base asset (UST) | `asset_price_in_collateral_asset = collateral_price / asset_price` (`mirror_mint/src/positions.rs`); `query_price` in `mirror_oracle` returns `base_price / quote_price` | Oracle-priced in UST; the base asset itself is `Decimal::one()` with `last_updated_time = u64::MAX`; delisted assets use a fixed `end_price` | `PRICE_EXPIRE_TIME` = 60 s on asset and collateral queries; operations fail when expired | mirror-contracts `mirror_mint`, `mirror_oracle`, `mirror_collateral_oracle` |
| Youves (Tezos) | XTZ, tzBTC, USDt, SIRS | uUSD, uBTC, uXTZ | USD ("reference currency") | Oracle via Acurast (docs); per-feed detail [NOT FOUND] | Docs: "USD value of available collateral divided by the USD value of outstanding youves tracker tokens" | Oracle target price per tracker; units [NOT FOUND] | [NOT FOUND]; two 2022 oracle DoS post-mortems exist ([[projects/youves]]) | [Collateral management details](https://docs.youves.com/syntheticAssets/stableTokens/collateralManagement/Collateral-Management-Details); [Oracles](https://docs.youves.com/marketsAndData/Oracles) |
| Indigo V3 (Cardano) | ADA and Cardano native tokens | iUSD, iBTC, iETH | Collateral per iAsset (docs) | One "Collateral Asset Price Oracle" per pair, "pricing in Collateral/iAsset (e.g., BTC/iUSD or ADA/iUSD)"; Pyth since V3 | Direct cross pair as far as the protocol sees it; whether the feed is composed from USD legs is [NOT FOUND] | Oracle-priced relative to collateral | [NOT FOUND] | [Indigo V3 docs](https://docs.indigoprotocol.io/readme/indigo-protocol-v3.md) |
| Alchemix V2 | Yield tokens (wstETH, rETH; adapters also exist for Aave, Yearn, Vesper and others) | alETH, alUSD | Debt token (alETH = ETH) | No USD feed; adapter `price()` per yield token. wstETH: Chainlink stETH/ETH times `getStETHByWstETH`; rETH: `getEthValue` only | `_validate`: `totalValue(owner) / debt >= minimumCollateralization`; `totalValue` sums `convertSharesToUnderlyingTokens` then `normalizeUnderlyingTokensToDebt` (decimal `conversionFactor`) | Same asset: 1 alETH = 1 ETH of underlying | wstETH adapter reverts if stETH/ETH older than 86,400 s and caps stETH/ETH at 1.0; `maximumLoss` per yield token blocks actions on sharp value loss | v2-foundry `AlchemistV2.sol`, `adapters/lido/WstETHAdapter.sol`, `adapters/rocket/` |
| Frankencoin | Any ERC-20 per position | ZCHF | ZCHF per collateral unit | None | `_checkCollateral`: `collateral x price >= minted`; `price` "zchf price per unit of the collateral below which challenges succeed" set by the position owner | Fixed: debt counted in ZCHF units | No oracle; a price increase triggers a 3-day minting cooldown (`_preparePriceIncrease`; the 3x cap in this file is not in the deployed `PositionV2`, see [[projects/frankencoin]]); challengers auction collateral if `price` is too high | FrankenCoin `contracts/minting/Position.sol` |
| crvUSD (LLAMMA, Controller) | ETH, wstETH, WBTC etc. | crvUSD | crvUSD (treated as USD) | Curve pool EMA oracles (tricrypto `price_oracle`, crvUSD stableswap pools aggregated) bounded by Chainlink | `CryptoWithStablePriceAndChainlink._raw_price`: `crv_p = p_crypto x p_stable_agg / p_stable`, clamped to `Chainlink x (1 +/- BOUND_SIZE%)`; Controller `_health` = AMM `get_x_down` / debt minus 1 | Fixed: debt in crvUSD units | No staleness check on the Chainlink bound in this oracle (only `latestRoundData().answer` is read); EMA smoothing via `ma_exp_time` | curve-stablecoin `price_oracles/CryptoWithStablePriceAndChainlink.vy`, `controller.vy` |

## 3. Choices observed

### 3.1 Common USD numeraire, one feed per asset

Used by Synthetix V2, Metronome Synth, Mirror (with UST as numeraire), Youves (docs), and JPY DeFi (yen as the CR unit but reached through USD feeds).

Each asset, collateral and synth alike, has a single feed against one quote currency, and cross values are derived by division (`amount x sourceRate / destinationRate` in Synthetix `_effectiveValueAndRates`; `collateral_price / asset_price` in Mirror; `(collateral/USD) / (JPY/USD)` in JPY DeFi). For n assets this needs n feeds, not n x (n-1) pairs. This is the closest match to an sXMR and sBTC CDP against ETH: `CR = (ETH amount x ETH/USD) / (sXMR amount x XMR/USD)`.

In every case the protocol's own unit-of-account token is hardcoded at 1 rather than read from an oracle: sUSD rate 1.0 and "never stale" (Synthetix); base asset `Decimal::one()` with an infinite update time (Mirror); msUSD 1.0 on chain while its market price is $0.818 (Metronome). This means a depeg of the numeraire token itself never reaches the CR check. In Mirror that was UST, the collateral and quote asset of the whole system.

### 3.2 Fixed-par debt: only the collateral feed

Used by Sky/Maker, Liquity V1 and V2, crvUSD and Frankencoin (ZCHF units).

The debt is valued at a constant, so the only oracle read is collateral/USD. Maker keeps the constant as a governance parameter (`par`, "ref per dai", 1.0 on chain), which is the hook RAI later turned into a moving target. JPY DeFi shows the step needed when par is in a currency that is not the feeds' quote currency: one extra shared feed (JPY/USD) for every branch.

### 3.3 Floating internal target price

Used by Reflexer RAI (and Maker's `par` slot structurally allows it).

RAI's `updateCollateralPrice` is Maker's `poke` with `par` replaced by `redemptionPrice()`, which compounds `redemptionRate` per second. So the CR check is `ETH x ETH/USD >= RAI x redemptionPrice x CRatio`: the debt side uses no oracle. A market oracle for RAI (Uniswap RAI/ETH TWAP times ETH/USD) only feeds the PI rate controller, not the solvency check. For a floating stablecoin this splits the oracle into two roles with different failure costs: a collateral feed that drives liquidations, and a market feed that steers the rate (inference from the code layout).

### 3.4 Direct cross pair

Used by Indigo V3 (docs: oracle "pricing in Collateral/iAsset", for example ADA/iUSD); partially by Liquity V2 and JPY DeFi for LSTs (rETH/ETH, stETH/ETH market feeds) and Alchemix (stETH/ETH).

No checked protocol reads a direct collateral/synth market feed for a volatile synth (such as ETH/BTC) from a USD-only provider; Indigo's cross may be composed upstream ([NOT FOUND]). Where cross feeds appear, they are for an asset against its own underlying (LST/ETH), used as a check on the canonical exchange rate.

### 3.5 Same-asset debt

Used by Alchemix V2 (alETH against ETH yield tokens).

`totalValue` converts yield-token shares to underlying and scales by decimals; there is no ETH/USD feed. The residual oracle is the LST/underlying rate, and the wstETH adapter still reads Chainlink stETH/ETH, capped at 1.0 to stop an oracle attack minting at more than the 50% LTV. Same-asset debt removes the price oracle, not every oracle.

### 3.6 Oracle-free, borrower-declared price

Used by Frankencoin.

The owner sets `price` in ZCHF per collateral unit; the CR check is `collateral x price >= minted`. Price increases suspend minting for 3 days (the 3x cap read here is in the top-level `Position.sol`, which is not the deployed V2; the deployed `PositionV2` bounds a raised price only by `price x minimumCollateral <= limit`, see [[projects/frankencoin]]), and anyone can challenge a position whose declared price is too high. Freshness is replaced by challenge periods (see [[patterns/cdp-synthetics-landscape]] section 4b).

### 3.7 LST composite feeds (cross-cutting)

Liquity V2 and JPY DeFi both build LST/USD as `underlying/USD x exchange rate` and choose `min` of market and canonical for borrowing and liquidation, `max` for redemption (within a 1% or 2% band). The asymmetry protects the system in each direction: low price when the borrower benefits from a high one, high price when the redeemer does.

## 4. Requirements this places on an oracle

Derived from the evidence above; items marked (inference) are not stated by any source.

1. **One feed per asset in a common quote currency.** USD-only feeds are sufficient: every multi-asset protocol checked prices each asset against one numeraire and divides. For sXMR and sBTC against ETH: XMR/USD, BTC/USD, ETH/USD. A direct ETH/XMR feed is not required.
2. **The CR depends on the cross rate, so the error budget is per pair** (inference). If ETH/USD and XMR/USD each carry an error or lag, the CR error is roughly the difference of the two relative errors. Deviation thresholds and heartbeats should be set for the cross volatility (ETH vs XMR), not each USD leg alone.
3. **Per-feed freshness limits, checked independently.** Observed limits: Mirror 60 s; JPY DeFi ETH/USD 2 h for redemption; Liquity V1 4 h; Liquity V2 deploy script 24 h (48 h for rETH/ETH); Alchemix stETH/ETH 24 h; Synthetix 25 h (chain); JPY DeFi JPY/USD 30 h, 72 h plus 24 h grace. No protocol checked aligns timestamps across the two legs explicitly; Mirror comes closest by requiring both legs within the same 60 s window. For a two-feed CR, both legs should be checked against the same window (inference).
4. **Validity must encode age, not just a flag.** RAI's ETH-A OSM returned `hasValidValue = true` with a price of 2,637.75 last updated 2025-06-09, while its source read 1,795.73 on 2026-11-16 (chain). A boolean validity bit without an age check let a 17-month-old price stand. crvUSD's Chainlink bound likewise reads `answer` without checking `updatedAt`.
5. **Deviation guards.** Liquity V1 treats a jump over 50% as broken and requires 5% agreement with Tellor to recover; Liquity V2 and JPY DeFi use 1% and 2% bands between market and canonical LST prices; crvUSD clamps the pool EMA to a Chainlink band; Alchemix caps stETH/ETH at 1.0.
6. **A delayed or smoothed price for liquidations.** Maker and RAI read prices through an OSM with a 1 h delay; crvUSD uses EMAs. Metronome's $15.7M shortfall came from oracle latency at an oracle-priced swap, not at liquidation ([[projects/metronome-synth]]). Inference: a delay suits the CR check, but any mint or swap path priced at a lagging oracle is an arbitrage source.
7. **Composite feeds for wrapped collateral.** If ETH collateral arrives as an LST, the oracle needs the exchange rate (read from the LST contract) and ideally a market LST/ETH or LST/USD feed for min and max selection.
8. **A defined fallback.** Observed: secondary oracle (Liquity V1 Tellor); `lastGoodPrice` plus branch shutdown (Liquity V2); pause with governance resume (JPY DeFi); halt all position operations (Mirror). A new chain with one oracle provider gets only the last two (inference).
9. **Shared legs are a common-mode failure.** JPY DeFi's docs warn that JPY/USD is shared by all branches; in an sXMR and sBTC CDP against ETH, ETH/USD is shared by both synths (inference).
10. **Market hours and venue coverage.** JPY DeFi documents that over weekends the JPY/USD timestamp advances while the price is flat, so freshness checks pass on a non-trading price. Inference: an XMR/USD feed needs the same scrutiny of how the publisher handles thin or delisted venues; not checked here.
11. **No oracle on the debt side for a floating stablecoin's CR.** In the RAI design the redemption price is internal; the market price oracle (RAI/ETH TWAP times ETH/USD) is needed only for the rate controller. A floating stablecoin minted against XMR or ETH would therefore need the collateral/USD feed for solvency and a separate market feed for the controller.

## 5. Surprises

- **RAI's collateral price is 17 months old.** The ETH-A OSM's `lastUpdateTime` is 2025-06-09 17:00 UTC, its current value 2,637.75 USD, and `getResultWithValidity` still reports valid; the source medianizer reads 1,795.73 (chain, 2026-11-16). The vault already lists RAI as dormant ([[patterns/cdp-synthetics-landscape]]). The redemption price was last updated on 2026-02-06 at 2.972 USD with a zero rate.
- **Synthetix CR is in SNX, not USD.** `Issuer._collateralisationRatio` converts the account's debt share into SNX and divides by SNX held, so the ratio is debt over collateral (the inverse of the usual CR) and USD only appears in the conversion.
- **Mirror's CR is in collateral units.** Debt is converted into collateral units by dividing two UST-quoted prices, and UST itself is hardcoded at 1 with an infinite update time.
- **Metronome values msUSD at 1.0 on chain while the market price is $0.818.**
- **Liquity V2 shuts a branch down on any single stale feed** rather than falling back to a secondary provider, and JPY DeFi keeps that but adds a governance resume path.

## 6. Not checked

- Maker's LST ilks (wstETH, rETH) oracle composition and the current Median or Scribe implementation behind `0x46ef...7f4e`.
- Liquity V2 staleness thresholds as deployed on mainnet (only the deploy script values were read).
- JPY DeFi contract source; the figures above are from its docs only.
- Metronome MasterOracle source and its staleness or deviation parameters.
- Indigo and Youves contract source; whether Indigo's Pyth cross feeds are composed from USD legs.
- Synthetix V2 GitHub repository: `github.com/Synthetixio/synthetix` returned not found and `synthetix-v2` returned 403, so the npm package was read instead.
- crvUSD oracles other than `CryptoWithStablePriceAndChainlink.vy`, and which oracle each live market uses.
- How XMR/USD is produced by any oracle provider (venues, delisting effects).

## Sources

All accessed 2026-11-16.

- makerdao/dss `src/spot.sol`, `src/vat.sol` at [fa4f663](https://github.com/makerdao/dss/blob/fa4f6630afb0624d04a003e920b0d71a00331d98/src/spot.sol) (the file carries a "FIXME: This contract was altered compared to the production version" note about events)
- makerdao/osm `src/osm.sol` at [e36c874](https://github.com/makerdao/osm/blob/e36c874b4e14fba860e48c0cf99cd600c0c59efa/src/osm.sol)
- Chain reads (Ethereum, publicnode RPC, block 26,426,729): Spotter `0x65C79fcB50Ca1594B025960e539eD7A9a6D434A3` `par()` = 1e27, `ilks("ETH-A")` pip `0x81fe72b5a8d1a857d176c3e7d5bd2679a9b85763`, `mat` 1.45; ETH-A OSM `hop()` 3,600, `zzz()` 2026-11-16 09:00 UTC; Median `wat()` "ETH/USD", `bar()` 13
- reflexer-labs/geb `src/single/OracleRelayer.sol` at [1a59f16](https://github.com/reflexer-labs/geb/blob/1a59f16a377386c49f520006ed0f7fd9d128cb09/src/single/OracleRelayer.sol)
- reflexer-labs/geb-fsm `src/OSM.sol`, `src/DSM.sol` at [ff83ea8](https://github.com/reflexer-labs/geb-fsm/blob/ff83ea8733e0e3e979d2c4884ed1276e234094f6/src/OSM.sol)
- reflexer-labs/geb-uniswap-median `src/UniswapConsecutiveSlotsPriceFeedMedianizer.sol` at [128be0b](https://github.com/reflexer-labs/geb-uniswap-median/blob/128be0b7dc05b6abb1be3ec8d1cce822a435d421/src/UniswapConsecutiveSlotsPriceFeedMedianizer.sol)
- reflexer-labs/geb-rrfm-rate-setter `src/PIRateSetter.sol` at [30bd909](https://github.com/reflexer-labs/geb-rrfm-rate-setter/blob/30bd909aa64d0a63d246052ae13c706d740fa0d4/src/PIRateSetter.sol)
- Chain reads: RAI OracleRelayer `0x4ed9C0dCa0479bC64d8f4EB3007126D5791f7851` `redemptionPrice()` 2.97200 (ray), `redemptionRate()` 1e27, `redemptionPriceUpdateTime()` 2026-02-06 21:29 UTC, `collateralTypes("ETH-A")` OSM `0x105b857583346e250fbd04a57ce0e491eb204ba3`, CRatios 1.35; OSM `getResultWithValidity()` 2,637.747 valid, `lastUpdateTime()` 2025-06-09 17:00 UTC, `updateDelay()` 3,600; source `0xbf26309b0ba639abe651dd1e1042eb3c57c3e100` `read()` 1,795.735
- liquity/dev `packages/contracts/contracts/PriceFeed.sol`, `Dependencies/LiquityMath.sol` at [3e64ee1](https://github.com/liquity/dev/blob/3e64ee1b52c50d51587c64c1cf75e0ba82934979/packages/contracts/contracts/PriceFeed.sol)
- liquity/bold `contracts/src/PriceFeeds/` (`MainnetPriceFeedBase.sol`, `CompositePriceFeed.sol`, `WSTETHPriceFeed.sol`, `RETHPriceFeed.sol`, `WETHPriceFeed.sol`) and `contracts/script/DeployLiquity2.s.sol` at [d262669](https://github.com/liquity/bold/tree/d262669ef3c4d37bf0544df3bb2624c07b8a5ee0/contracts/src/PriceFeeds)
- [JPY DeFi: How Prices Are Determined (Oracle)](https://jpydf.gitbook.io/jpy-defi/collateral-and-pricing/oracle.md); [JPY DeFi docs home](https://jpydf.gitbook.io/jpy-defi)
- Synthetix npm package [synthetix@2.102.1](https://unpkg.com/synthetix@2.102.1/contracts/ExchangeRates.sol) (`ExchangeRates.sol`, `Issuer.sol`, `BaseDebtCache.sol`, `publish/deployed/mainnet/deployment.json`) and [synthetix@2.30.0](https://unpkg.com/synthetix@2.30.0/contracts/ExchangeRates.sol) (inverse pricing)
- Chain read: Synthetix ExchangeRates `0x648280dD2db772CD018A0CEC72fab5bF8B7683AB` `rateStalePeriod()` = 90,000
- autonomoussoftware/metronome-synth-public `contracts/Pool.sol`, `contracts/interfaces/external/IMasterOracle.sol`, `helpers/address.ts` at [1852ce0](https://github.com/autonomoussoftware/metronome-synth-public/blob/1852ce00751499beccf302d324687be3ad9bd27f/contracts/Pool.sol)
- Chain reads: Metronome MasterOracle `0x80704Acdf97723963263c78F861F091ad04F46E2` `quoteTokenToUsd(1e18)`: msUSD `0xab5eB14c09D416F0aC63661E57EDB7AEcDb9BEfA` 1.0, msETH `0x64351fC9810aDAd17A690E4e1717Df5e7e085160` 1,795.73, msBTC `0x8b4F8aD3801B4015Dea6DA1D36f063Cbf4e231c7` 64,323.12; `oracles(msUSD)` `0x278957e88bdba45442acc49e07ac143d893bafb1`
- [Metronome docs: oracle disruptions](https://docs.metronome.io/metronome-synth/risks/oracle-disruptions.md)
- Mirror-Protocol/mirror-contracts `contracts/mirror_mint/src/querier.rs`, `positions.rs`, `contracts/mirror_oracle/src/contract.rs`, `contracts/mirror_collateral_oracle/src/querier.rs` at [56cc694](https://github.com/Mirror-Protocol/mirror-contracts/tree/56cc6946b9457293ede6aa0feb296ee1d16f6974/contracts)
- [Youves: Collateral Management Details](https://docs.youves.com/syntheticAssets/stableTokens/collateralManagement/Collateral-Management-Details); [Youves: Oracles](https://docs.youves.com/marketsAndData/Oracles)
- [Indigo Protocol V3](https://docs.indigoprotocol.io/readme/indigo-protocol-v3.md); [Indigo iAssets](https://docs.indigoprotocol.io/readme/iassets)
- alchemix-finance/v2-foundry `src/AlchemistV2.sol`, `src/adapters/lido/WstETHAdapter.sol`, `src/adapters/rocket/` at [8915ef7](https://github.com/alchemix-finance/v2-foundry/blob/8915ef7b1714c16f9e260a4ef41c5f254d5b7f58/src/AlchemistV2.sol)
- Frankencoin-ZCHF/FrankenCoin `contracts/minting/Position.sol` at [8b4c4ab](https://github.com/Frankencoin-ZCHF/FrankenCoin/blob/8b4c4ab67bb361b91d58c474b87f4608fc4c0566/contracts/minting/Position.sol)
- curvefi/curve-stablecoin `curve_stablecoin/price_oracles/CryptoWithStablePriceAndChainlink.vy`, `curve_stablecoin/controller.vy` at [81b3bea](https://github.com/curvefi/curve-stablecoin/blob/81b3bea490e9b45221c82e78387920ccfeb05b33/curve_stablecoin/price_oracles/CryptoWithStablePriceAndChainlink.vy)
- [[projects/metronome-synth]], [[projects/mirror-protocol]], [[projects/youves]], [[projects/alchemix]], [[patterns/oracle-pricing-model]], [[patterns/cdp-synthetics-landscape]]
