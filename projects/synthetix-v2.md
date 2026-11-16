---
tags: [project, ethereum, optimism, synthetics, debt-pool, cdp, legacy]
ecosystem: Ethereum, Optimism
category: Synthetics/Debt-pool CDP
status: legacy (solo SNX staking phased out April 2025; superseded by Synthetix V3 and the 420 pool)
status_date: 2025-04
website: https://synthetix.io
docs: https://sips.synthetix.io
launched: 2018 (Havven/Synthetix V1); V2 date [NOT FOUND]; SNX inflation started March 2019 (SIP-23)
---

# Synthetix V1/V2 (debt-pool synthetics)

Status: legacy. The original Synthetix design let SNX holders lock SNX at a high collateralisation ratio (C-ratio) and mint sUSD, which they could exchange into other Synths (sBTC, sETH, forex, commodities, equities, inverse Synths). Solo SNX staking was deliberately phased out in April 2025 in favour of the 420 pool on V3 (see [[projects/synthetix-v3]]). This note covers the borrower side (the staker who mints and carries debt), which the V3 note does not.

Key distinction (see [[patterns/synthetic-collateral-models]] and [[patterns/cdp-synthetics-landscape]]): the collateral is staked SNX only. sUSD is the exchange and settlement unit that stakers mint, not collateral. Every staker's debt is a share of one shared global debt pool, so the debt of each staker moves with the aggregate USD value of all outstanding Synths, not with the Synths that staker personally minted.

## Adoption metrics

| Metric | Value | Date | Source |
|--------|-------|------|--------|
| TVL peak (V1+V2, staked SNX) | $2.967B | 2021-02-10 | [DefiLlama API](https://api.llama.fi/protocol/synthetix-v1+v2) |
| TVL | $154.6M | 2025-01-09 | [DefiLlama API](https://api.llama.fi/protocol/synthetix-v1+v2) |
| TVL (last datapoint, essentially zero after solo-staking phase-out) | $1,789 | 2025-08-08 | [DefiLlama API](https://api.llama.fi/protocol/synthetix-v1+v2) |
| Cumulative exchange fees (V1+V2) | $81.34M | accessed 2026-11-16 | [DefiLlama fees](https://api.llama.fi/summary/fees/synthetix-v1+v2) |
| Fees 2022 / 2023 / 2024 / 2025 | $25.30M / $33.80M / $20.37M / $1.75M | calendar years | [DefiLlama fees](https://api.llama.fi/summary/fees/synthetix-v1+v2) (summed from daily series; series starts Nov 2021, so 2021 shows only $0.10M) |
| sUSD supply, all chains and versions | $69.84M | 2026-11-16 | [DefiLlama stablecoins](https://stablecoins.llama.fi/stablecoins) |
| Active chains | Ethereum, Optimism | 2025 | [DefiLlama API](https://api.llama.fi/protocol/synthetix-v1+v2) |

Caveat: the DefiLlama protocol record flags `deadFrom: 2024-01-01`, but its own TVL series shows $154.6M in January 2025 and the collapse to near zero in April 2025. The April 2025 date matches governance (SCCP-401 and SCCP-403 below). Treat the 2024 flag as a data-labelling error.

## How it works

### User perspective (the borrower / staker)

A staker locks SNX and mints sUSD up to the target C-ratio (initially 750 to 800%, lowered stepwise by governance to 400% and below). The staker may keep, sell or exchange the sUSD. Minting creates a debt denominated in USD that is a share of the global debt pool. To unlock SNX the staker must burn sUSD equal to their current debt, which can be higher or lower than what they originally minted. To claim weekly staking rewards (inflationary SNX plus a share of exchange fees in sUSD) the staker must hold at or above the target C-ratio.

### Protocol perspective

- Synths are priced by oracles (see [[patterns/oracle-pricing-model]]); exchanges between Synths settle at the oracle price with no price slippage, so there is no AMM or order book.
- Exchange fees accrue to the FeePool, which distributes them in sUSD to stakers pro rata to their debt-pool share per fee period.
- Debt pool: when traders exchange sUSD into sETH and ETH rises, the aggregate debt rises and every staker's debt rises pro rata. Traders' profits are therefore paid by stakers (the counterparty design is the same as the LP-pool risk in [[patterns/lp-counterparty-risk]]).
- Peg: sUSD has no direct redemption for $1 of collateral. The structural bid is that stakers must buy and burn sUSD to repay debt or to meet the target C-ratio (SCCP-232 explicitly cites staker sUSD purchases as a peg support and a pressure point). Source: [SCCP-232](https://sips.synthetix.io/sccp/sccp-232/).

## Key behaviours

- [[patterns/synthetic-collateral-models]] : native-token CDP with a shared debt pool.
- [[patterns/oracle-pricing-model]] : the June 2019 sKRW incident (below) is the canonical oracle-misconfiguration case.
- [[patterns/lp-counterparty-risk]] : stakers are the counterparty to all Synth traders.
- [[patterns/cdp-synthetics-landscape]] : comparison to per-position CDPs and delta-neutral designs.

## Architecture decisions

- **Shared debt pool rather than per-position collateral:** lets Synths be exchanged with zero slippage, at the cost of staker exposure to trader PnL.
- **Native token as only collateral (reflexivity):** SNX price down lowers C-ratios and forces stakers to burn sUSD, which pressures the peg.
- **Fee design moved from flat to differential:** flat 30 bps for all Synths, then category rates, then summed pair fees (see Fees section).
- **Liquidation redesign (SIP-148, created 2021-06-11):** flag then liquidate after a delay. Liquidated SNX goes to other stakers through a LiquidatorRewards contract with 12 month escrow, not to the liquidator.
- **Inflation wind-down:** the original schedule halved weekly rewards yearly from March 2019; SIP-23 smoothed it to a 1.25% weekly reduction with a 2.5% terminal rate; SIP-2043 (December 2023) ended SNX inflation.
- **Exit:** SIP-420 (created 2025-01-06) moves debt positions into a protocol-owned delegated pool; SCCP-403 (2025-04-09) raised the solo staking liquidation ratio on a schedule to retire solo staking.

## Differentiators

- First production on-chain synthetic asset protocol, with forex, commodity, equity and inverse Synths priced by oracle.
- Zero-slippage exchange of Synths against a shared pool.
- Exchange fees flow to stakers through the fee pool (DefiLlama methodology: revenue is "awarded to SNX stakers"; the exact treasury share, if any, is [NOT FOUND]).

## Limitations and criticisms

- Reflexive: SNX is both collateral and reward token. Governance repeatedly cut the C-ratio (800% to as low as 350 to 400%) to keep stakers above target after price falls (SCCP-232 cites weakness in SNX price).
- Passive stakers carry unhedged exposure to aggregate trader PnL; hedging required active management.
- sUSD peg was repeatedly stressed; V3 later reused sUSD and needed buybacks (see [[projects/synthetix-v3]]; SIP-405 redirected V3 fees to burn sUSD and reduce V2 stakers' debt).
- Oracle incidents: June 2019 sKRW feed reported Korean Won 1000 times too high; a bot generated about 37M sETH of paper profit within an hour and the funds were recovered through a white-hat negotiation and bounty (DefiLlama hacks record; secondary description, no primary post-mortem retrieved).

### Fees and borrower cost

The borrower (staker) side has no interest rate. The borrower cost is (a) the foregone and variable value of carrying a floating debt, (b) the C-ratio lockup, (c) liquidation penalties, and (d) the fees below on Synth exchanges (which accrue to stakers, not to the protocol treasury, per DefiLlama methodology).

| Item | Value | Date | Source |
|------|-------|------|--------|
| Interest or stability fee on staker debt | None; debt floats with the global debt pool | n/a | [DefiLlama methodology](https://api.llama.fi/summary/fees/synthetix-v1+v2) (describes only exchange fees); no stability fee appears in the SIP/SCCP lists retrieved from [sips.synthetix.io](https://sips.synthetix.io/all-sccp) |
| Fee to mint sUSD against staked SNX | [NOT FOUND] (no explicit statement retrieved; the only issue-type fees in the SCCP list are on wrappers and ETH-backed loans) | accessed 2026-11-16 | [all SCCPs](https://sips.synthetix.io/all-sccp) |
| Exchange fee, flat | 30 bps on every Synth | before 2020-06 | [SCCP-24](https://sips.synthetix.io/sccp/sccp-24) |
| Exchange fee, SCCP-24 categories | forex, commodity, equity 5 bps; crypto and inverse crypto 30 bps | created 2020-06-04 | [SCCP-24](https://sips.synthetix.io/sccp/sccp-24) |
| Exchange fee, SIP-222 (sum of source and destination Synth fee) | L1: sUSD 0, forex 15, sBTC and sETH 25, sLINK 85, sADA, sDOT, sAAVE, sDEFI 100 bps. L2: sUSD 0, forex 15, sETH and sBTC 25, sLINK 30, others 35 bps. Example: sETH to sAAVE on L1 = 125 bps | created 2022-03-27 (Mirach release) | [SIP-222](https://sips.synthetix.io/sips/sip-222/) |
| Exchange fee range quoted by DefiLlama | 10 to 100 bps, usually 30 bps | 2026-11-16 | [DefiLlama fees](https://api.llama.fi/summary/fees/synthetix-v1+v2) |
| Fee recipient | SNX stakers: fees accrue as sUSD to the fee pool and reduce debt on claim (DefiLlama: "automatically burning outstanding debt"). A treasury share is not quantified in the sources retrieved: [NOT FOUND] | 2026-11-16 | [DefiLlama fees](https://api.llama.fi/summary/fees/synthetix-v1+v2); [FeePool contract docs](https://developer.synthetix.io/contracts/source/contracts/FeePool) (search result only, not re-fetched) |
| ETH-backed sUSD loans (Collateral contract), issue fee history | 200 bps (SCCP-103) to 50 bps (SCCP-108) to 25 bps (SCCP-116). SCCP-82 lowered loan C-ratio and liquidation ratio from 150% to 130%. Current value: [NOT FOUND] | 2020 to 2021 | [all SCCPs](https://sips.synthetix.io/all-sccp) |
| Wrapper mint fee history (wETH, DAI, LUSD wrappers) | 200 bps to 100 (SCCP-104) to 50 (109) to 37.5 (111) to 25 (114) to 5 bps (117); DAI wrapper 5 bps (SCCP-147); L2 ETH wrapper 5 bps (SCCP-143) then 250 bps (SCCP-145); ETH wrapper burn fee 5% (SCCP-203). Current values: [NOT FOUND] | 2020 to 2022 | [all SCCPs](https://sips.synthetix.io/all-sccp) |
| Liquidation penalty (SIP-148) | 30% forced; 20% self-liquidation; flag reward 10 SNX (L1) or 1 SNX (L2); liquidate reward 20 SNX (L1) or 2 SNX (L2); 12 hour delay | created 2021-06-11 | [SIP-148](https://sips.synthetix.io/sips/sip-148/) |
| Liquidation ratio | 150% at SIP-148; 145% via SCCP-216; 160% restored via SCCP-401 (created 2025-03-24) | 2021 to 2025 | [SCCP-401](https://sips.synthetix.io/sccp/sccp-401/); [all SCCPs](https://sips.synthetix.io/all-sccp) |
| Target C-ratio path | 750 to 800% (SCCP-2, 14), 700 (30), 600 (41), 500 (59), 450 (127), 400 (128), then SIP-148-era moves (350 in SCCP-204, 400 in SCCP-216 and 240), temporary 350 (SCCP-232, created 2022-09-11) | 2019 to 2022 | [all SCCPs](https://sips.synthetix.io/all-sccp); [SCCP-232](https://sips.synthetix.io/sccp/sccp-232/) |
| SNX inflation reward | 1.44M SNX per week at launch; 2.5% terminal from September 2023; ended by SIP-2043 (December 2023) | 2019 to 2023 | [SIP-23/24 search results](https://sips.synthetix.io/sips/sip-24); [The Block](https://www.theblock.co/news/defi/2023-12-12-synthetix-snx-token-inflation-buybacks-burns-267138) (search result summary, not re-fetched) |
| Revenue per year to stakers (fees) | See adoption table: $25.3M (2022), $33.8M (2023), $20.4M (2024), $1.75M (2025) | 2026-11-16 | [DefiLlama fees](https://api.llama.fi/summary/fees/synthetix-v1+v2) |

## Admin control: who can change the rules (read on chain 2026-11-16)

Verified by `eth_call` and `eth_getStorageAt` against Ethereum mainnet through `ethereum-rpc.publicnode.com` (selectors computed locally; reproducible by anyone with an RPC endpoint).

| Check | Result |
|-------|--------|
| `owner()` of the SNX proxy (`0xC011a73ee8576Fb46F5E1c5751cA3B9Fe0af2a6F`) | `0xEb3107117FEAd7de89Cd14D463D340A2E6917769` |
| `owner()` of AddressResolver (`0x823bE81bbF96BEc0e25CA13170F5AaCb5B79ba83`) | same address |
| `owner()` of Issuer, ExchangeRates, SystemSettings, SystemStatus, Exchanger, CollateralManager, DebtCache, FeePool (resolved through the AddressResolver) | same address for all eight |
| Owner contract type | Safe (Gnosis Safe) v1.3.0 proxy; singleton `0xd9Db270c1B5E3Bd161E8c8503c55cEABeE709552` |
| Safe threshold and signers | 4 of 7; nonce 3,370 (actively used) |
| Safe guard and modules | none set (guard storage slot zero; module list empty) |
| `nominatedOwner()` on SNX proxy and AddressResolver | zero address (no pending transfer) |
| Upgrade functions present in bytecode | `setTarget(address)` on the proxy (repoints the implementation); `importAddresses(bytes32[],address[])` on the AddressResolver (repoints which contract every other contract resolves) |

What this establishes: one 4-of-7 multisig owns the whole V2 contract set, and there is no on-chain timelock, guard or module between a signed Safe transaction and a change of implementation or of the resolver map. The protocol is governed, with an admin-key trust model on the Aave end of the spectrum, not the Morpho end.

What it does not establish:
- **Signer identities** are not established (seven addresses, unlabelled here).
- **Function-level permissions** were first checked only as the presence of `setTarget` and `importAddresses` in bytecode. The follow-up below reads the source: both are `onlyOwner`.
- **Voting layer:** per Synthetix's SIP text (SIP-124, SIP-172, SIP-401, read through a page summariser, not the contract source), stakers elect the Spartan Council, which votes on SIPs and SCCPs; execution is by multisig. I found no on-chain mechanism binding the multisig to a vote result, so the link from stakers to contract changes rests on the signers following the vote. Treat as unverified.
- **Documentation conflicts:** [SIP-124](https://sips.synthetix.io/sips/sip-124) (status Implemented) describes a protocolDAO threshold of at least five signatures with membership expected to reach nine, members bonded with 10k SNX and validated by the Spartan Council; the chain shows 4 of 7. A search-result summary of the Synthetix developer docs names this address as the pDAO Gnosis multisig, but I could not open that page (HTTP 404), and an older Synthetix post is summarised as 4 of 8. Either the configuration changed or the SIPs lag.
- **Unexecuted handover:** a search summary of SIP-172 says SystemSettings ownership would move from this address to a Spartan Council multisig. The chain shows SystemSettings still owned by this Safe, so that handover is not reflected on chain; I could not read the SIP-172 text itself ([SIP-172 page](https://sips.synthetix.io/sips/sip-172) rendered different content). [SIP-401](https://sips.synthetix.io/sips/sip-401) (a Risk Council replacing the pDAO) is marked rejected (abandoned).

### Lever by lever: who, how fast, within what bounds (read 2026-11-16)

Method. Chain reads as above, at block 26,426,688 (2026-11-16 09:27 UTC), with every contract address resolved through `AddressResolver.getAddress` on chain. A clone of the Synthetix GitHub repository was refused by this session's network policy, so source was read from the npm release [`synthetix@2.102.1`](https://www.npmjs.com/package/synthetix/v/2.102.1) (published 2025-03-31, gitHead `e71a798dd784b0f54bb8b1327f3409f1a7e6786e`). That release ships `contracts/` and `publish/deployed/mainnet/deployment.json`. It is Synthetix's published source, not Etherscan-verified source, and I did not compare deployed bytecode with it. Four `SystemSettingsLib` constants read on chain at `0x4a39AEf2281ac0D192a9c4783604833bA8F31174` match it: `MAX_ISSUANCE_RATIO` 1e18, `MAX_EXCHANGE_FEE_RATE` 0.1e18, `MAX_LIQUIDATION_PENALTY` 0.9e18 and `MIN_LIQUIDATION_DELAY` 300. Tags: **chain** (read on chain), **source** (npm 2.102.1 source), **docs** (SIP or SCCP text).

"Safe" below is the 4-of-7 owner `0xEb31...7769`, which has no guard and no modules, so every Safe change takes effect as soon as four signers execute it.

| Lever | Who | Speed | Bounds | Evidence |
|-------|-----|-------|--------|----------|
| Collateral list | Safe. Staking collateral is SNX only, by design. Extra collateral for loans and shorts goes through `CollateralManager.addCollaterals`, and wrappers through `WrapperFactory.createWrapper`; both are `onlyOwner` | immediate | none in source | source. Current loan and wrapper collateral not read |
| Synth list | Safe: `Issuer.addSynth(s)` and `removeSynth(s)` are `onlyOwner`. Removing a synth that still has supply mints sUSD to SynthRedeemer at the oracle rate | immediate | removal needs a non-zero rate when supply exists; no limit on adding | source; chain: `Issuer.availableCurrencyKeys` lists 10 synths (sUSD, sEUR, sJPY, sAUD, sGBP, sCHF, sKRW, sBTC, sETH, sETHBTC) |
| Risk parameters | Safe: all 40 `SystemSettings` setters are `onlyOwner` | immediate | issuance ratio at most 1 (C-ratio at least 100%); liquidation ratio at least 2 times the issuance ratio and at most 1/(1 + SNX penalty); penalties at most 90%; liquidation delay 300 s to 3 days; minimum stake time at most 1 week; target threshold at most 50%; fee period 1 to 60 days. **Unbounded:** liquidation escrow duration, flag and liquidate rewards, rate stale period, price deviation threshold, waiting period, debt snapshot stale time | source; 4 constants chain |
| Oracle | Safe: `ExchangeRates.addAggregator` and `removeAggregator`, plus `setRateStalePeriod`, `setPriceDeviationThresholdFactor` and `setAggregatorWarningFlags` on SystemSettings, are `onlyOwner`. `CircuitBreaker` trips automatically (only Issuer and ExchangeRates can probe it); only the owner can reset it (`resetLastValue`) | immediate | `addAggregator` checks only that the feed answers `latestRound()` and has at most 27 decimals. Stale period and deviation threshold unbounded. sUSD is hard-coded at 1 USD in `ExchangeRates` | source and chain |
| Fees and rates | Safe: `setExchangeFeeRateForSynths`, `setExchangeMaxDynamicFee`, `setAtomicExchangeFeeRate` and the wrapper fee setters are `onlyOwner` | immediate | exchange fee per synth at most 10%; maximum dynamic fee non-zero and at most 10%; wrapper fees within plus or minus 100%. There is no interest rate on staker debt | source; current values chain |
| Pause | Any account holding a `SystemStatus.accessControl(section, account)` entry. Only the owner can grant entries (`updateAccessControl` is `onlyOwner`). The owner gets no implicit right; it needs its own entry, and holds one | immediate, one transaction by the grantee | none: any section can stay suspended indefinitely | source and chain |
| Code upgrades | Safe: `Proxy.setTarget` and `AddressResolver.importAddresses` are `onlyOwner`. Ownership moves in two steps (`nominateNewOwner` by the owner, then `acceptOwnership` by the nominee) | immediate; no timelock | none | source; chain: `nominatedOwner()` is zero on all 16 contracts checked |
| Shutdown | Safe, by combining the synth list, risk parameter, pause and upgrade levers. Source has no dedicated shutdown or global settlement function | immediate | none | chain (current state below); docs (SIP-423) |

**The bounds do not bind the owner.** `FlexibleStorage`, which holds every setting, accepts writes from whatever address the AddressResolver maps to the name "SystemSettings" (`ContractStorage.onlyContract`), and `importAddresses` is `onlyOwner`. So the Safe can bypass every bound above by importing a different SystemSettings contract. The bounds stop a mistyped value, not a determined owner (source).

**Pause: who holds suspend rights (chain).** I read `accessControl` for 17 accounts across the six sections (System, Issuance, Exchange, Futures, SynthExchange, Synth):
- Owner Safe: can suspend and resume all six.
- Issuer (`0x7A1b...83Fd`): can suspend Issuance only. ExchangeCircuitBreaker (`0xeAca...5437`, marked deprecated in its source): can suspend Synth only. Neither contract calls a SystemStatus suspend function in the 2.102.1 source, so these entries look like leftovers from earlier versions (not proven).
- No rights: the seven Safe signers; the deployer EOA `0xEde8a407913A874Dd7e3d5B731AFcA135D30375E`; the `marketClosure` account `0xC105Ea57Eb434Fbe44690d7Dec2702e4a2FBFCf7`; the `oracle` account `0xaC1ED4Fabbd5204E02950D68b6FC8c446AC95362` (these three are named in the package's `getUsers` list); CircuitBreaker, Exchanger, FuturesMarketManager and the V3 LegacyMarket.
- Not established: whether some other account, such as a historical pDAO EOA, still holds an entry. Entries can be listed only from `AccessControlUpdated` events, and the endpoint refused wide `eth_getLogs`. Among the accounts checked, none other than the Safe can suspend.

**Current suspension state (chain).**
- `systemSuspension`, `issuanceSuspension` and `futuresSuspension` are not set. SNX transfers still work (simulated `transfer` returned true).
- `exchangeSuspension` is set with reason code 2,059, which matches [SIP-2059](https://sips.synthetix.io/sips/sip-2059/) (legacy spot synth migration). Docs and chain differ in method: SIP-2059 describes per-synth `suspendSynthsExchange`, but every per-synth exchange flag reads zero and the global exchange flag is set instead.
- `synthSuspension` is set with reason 1 (`SUSPENSION_REASON_UPGRADE`) for sUSD, sETH, sBTC, sEUR, sJPY, sAUD, sGBP, sCHF, sKRW and sETHBTC. This flag blocks transfers (`Synth._ensureCanTransfer` calls `requireSynthActive`). sLINK, which is no longer listed, is not flagged.

**Shutdown: what is in place on chain.**
- **sUSD retired by upgrade.** The sUSD proxy `0x57Ab1ec28D129707052df4dF418D58a2D46d5f51` now targets `0xeb05c1de97296ae3dca16f99cd0bdf5a7910943c`. That is a 1,092-byte contract named "Synth sUSD (retired)", with no `owner()` and `totalSupply()` 0. A simulated transfer reverts with "sUSD retired". The old implementation (SynthsUSD `0x10A5...275f`) still records 43,600,114 sUSD of supply, and `Issuer.totalIssuedSynths(sUSD)` returns $43.60M. This matches [SIP-423](https://sips.synthetix.io/sips/sip-423/) (created 2026-06-12, status Implemented). SIP-423 has the pDAO suspend sUSD transfers, minting and burning on Ethereum and Optimism, with a holder snapshot at Ethereum block 25,473,065 (2026-07-06 11:02 UTC per the chain timestamp).
- **Minting off by parameter.** `issuanceRatio` is 1 wei (1e-18), which makes the maximum issuable sUSD effectively zero. The issuance suspension flag itself is not set.
- **Liquidation settings.** `liquidationRatio` is 0.625, a 160% C-ratio, as [SCCP-401](https://sips.synthetix.io/sccp/sccp-401/) set. With the 60% SNX penalty this is the highest ratio the source allows. Other values: `snxLiquidationPenalty` 60%, `selfLiquidationPenalty` 50%, `liquidationEscrowDuration` 15,552,000 s (180 days), `liquidationDelay` 28,800 s (8 hours), flag reward 10 SNX, liquidate reward 20 SNX. [SCCP-403](https://sips.synthetix.io/sccp/sccp-403/) gives a schedule of 250%, then 500%, then "high enough to deprecate", but its text says it applies to "v3 accounts". The V2 SystemSettings value is still 160%.
- **All V2 debt now sits in V3.** `SynthetixDebtShare.totalSupply` is 47,015,845, and the V3 LegacyMarket `0x3acf...b2f5` holds all of it. LegacyMarket is owned by the same Safe, and its `pauseStablecoinConversion()` and `pauseMigration()` both return true. `DebtMigratorOnEthereum.initiationActive` is false.
- **Synth redemption disabled.** `DynamicSynthRedeemer.redemptionActive` is true, but `discountRate` is 0. The source requires a non-zero redemption rate, so `redeem` should revert with "Synth not redeemable" (inferred from source, not simulated).
- **Remaining synth supply** read through the proxies: 11,579.9 sETH and 200.37 sBTC. Holders not identified; protocol-held balances may be part of this.

**How a remaining holder exits today.**
- **sUSD:** there is no on-chain route. Per SIP-423, a LegacySUSDReceipt contract on mainnet will let snapshot holders claim 4 SNX per sUSD, burning the legacy sUSD, after a 1 year lock and then a 1 year vest. The claim window opens at T+365 days, the build is marked deferred, and unclaimed receipts expire after 6 months. sUSD held in LP pools or vaults goes through a Treasury claims process reviewed case by case. SIP-423 also describes 25% of revenue redeeming sUSD claims at 1 USDT once revenue passes $10M inside the 2 year lock (docs). I did not find a LegacySUSDReceipt address.
- **sETH, sBTC and the other synths:** transfers and exchanges are suspended, and DynamicSynthRedeemer pays zero (and would pay in retired sUSD). SIP-423 does not mention these synths. Exit route: [NOT FOUND].

**Current parameter values (chain), for reference:** `rateStalePeriod` 90,000 s (25 hours); `priceDeviationThresholdFactor` 100e18, so the circuit breaker trips only on a 100-fold price move or a zero price (in effect off); exchange fee 35 bps for sETH and sBTC and 15 bps for sEUR (moot while exchange is suspended); `exchangeMaxDynamicFee` 5%; `minimumStakeTime` 1 s. Of the 23 Chainlink aggregators in `ExchangeRates.aggregatorKeys`, DOT and ADA revert (their rates read invalid); the other 21 return fresh rates (for example SNX/USD 0.14745).

**Conflicts with the rest of this note:**
- Fees table: SIP-222 lists sETH and sBTC on L1 at 25 bps; the chain now shows 35 bps (a later change, not dated here).
- Liquidation: SIP-148 lists a 30% forced and 20% self-liquidation penalty with 12 month escrow; the chain shows 60%, 50% and 180 days.
- Architecture decisions: that section says SCCP-403 raised the *solo staking* liquidation ratio. SCCP-403's text targets V3 accounts, and the V2 setting is unchanged at 160%.
- Adoption table: DefiLlama gives sUSD supply of $69.84M across all chains and versions, but the L1 V2 sUSD proxy now reports a total supply of 0. SIP-423 caps the entitlement at 59,116,089 sUSD across Ethereum and Optimism.
- Status line: since 2026-07-06 the protocol is not only legacy but shut down for sUSD, which is retired.

**Not checked:** event history (who received or lost access-control entries, and when the discount rate and fees changed); top holders of sETH and sBTC; the Optimism deployment; CollateralManager and wrapper state; LegacyMarket source (only its getters were called); a bytecode comparison between the deployed contracts and the npm source.

## Sources

- [DefiLlama Synthetix v1+v2 API](https://api.llama.fi/protocol/synthetix-v1+v2) : accessed 2026-11-16
- [DefiLlama fees: Synthetix v1+v2](https://api.llama.fi/summary/fees/synthetix-v1+v2) : accessed 2026-11-16
- [DefiLlama stablecoins list (sUSD)](https://stablecoins.llama.fi/stablecoins) : accessed 2026-11-16
- [DefiLlama hacks list](https://api.llama.fi/hacks) : accessed 2026-11-16
- [SCCP-24 differential fee levels](https://sips.synthetix.io/sccp/sccp-24) : accessed 2026-11-16
- [SIP-222 update exchange fee system](https://sips.synthetix.io/sips/sip-222/) : accessed 2026-11-16
- [SIP-148 liquidation mechanism V2](https://sips.synthetix.io/sips/sip-148/) : accessed 2026-11-16
- [SCCP-232 temporary C-ratio 350%](https://sips.synthetix.io/sccp/sccp-232/) : accessed 2026-11-16
- [SCCP-401 unpause staking liquidations at 160%](https://sips.synthetix.io/sccp/sccp-401/) : accessed 2026-11-16
- [SCCP-403 liquidation ratio schedule](https://sips.synthetix.io/sccp/sccp-403/) : accessed 2026-11-16
- [SIP-420](https://sips.synthetix.io/sips/sip-420/) : accessed 2026-11-16
- [SIP-405 temporary distribution of V3 fees to V2 stakers](https://sips.synthetix.io/sips/sip-405) : accessed 2026-11-16
- [All SCCPs index](https://sips.synthetix.io/all-sccp) : accessed 2026-11-16
- [SIP-423 sUSD retirement and staking reform](https://sips.synthetix.io/sips/sip-423/) : accessed 2026-11-16
- [SIP-2059 legacy spot synth migration](https://sips.synthetix.io/sips/sip-2059/) : accessed 2026-11-16
- [synthetix npm package 2.102.1](https://www.npmjs.com/package/synthetix/v/2.102.1) ([tarball](https://registry.npmjs.org/synthetix/-/synthetix-2.102.1.tgz); gitHead `e71a798dd784b0f54bb8b1327f3409f1a7e6786e`; files read: `contracts/SystemStatus.sol`, `Owned.sol`, `Proxy.sol`, `AddressResolver.sol`, `ExchangeRates.sol`, `CircuitBreaker.sol`, `ExchangeCircuitBreaker.sol`, `SystemSettings.sol`, `SystemSettingsLib.sol`, `FlexibleStorage.sol`, `ContractStorage.sol`, `Issuer.sol`, `Synth.sol`, `BaseSynthetix.sol`, `DynamicSynthRedeemer.sol`, `CollateralManager.sol`, `WrapperFactory.sol`, `index.js`, `publish/deployed/mainnet/deployment.json`) : accessed 2026-11-16
- Ethereum mainnet `eth_call`, `eth_getCode` and `eth_getBlockByNumber` via [publicnode](https://ethereum-rpc.publicnode.com) at block 26,426,688 (SystemStatus `0x696c905F8F8c006cA46e9808fE7e00049507798F`, SystemSettings `0x202ae40Bed1640b09e2AF7aC5719D129A498B7C8`, ExchangeRates `0x648280dD2db772CD018A0CEC72fab5bF8B7683AB`, CircuitBreaker `0x9f231dBE53D460f359B2B8CC47574493caA5B7Bf`, Issuer `0x7A1b3DB73E5B8c58EDC8A821890005064f2B83Fd`, DynamicSynthRedeemer `0x38c8A4d93757D3D2E0110Cfb5f18B9cC293e0fdA`, SynthetixDebtShare `0x89FCb32F29e509cc42d0C8b6f058C993013A843F`, LegacyMarket `0x3acf163b9e6a384d539e10dac7e11213c638b2f5`, sUSD proxy `0x57Ab1ec28D129707052df4dF418D58a2D46d5f51`) : accessed 2026-11-16
