---
tags: [project, ethereum, cdp, stablecoin, synthetics, immutable]
ecosystem: Ethereum
category: Synthetics/CDP stablecoin
status: live (V1 live since April 2021; V2 relaunched 2025-05-19)
status_date: 2026-11-16
website: https://www.liquity.org
docs: https://docs.liquity.org
launched: 2021 (V1, LUSD), 2025 (V2, BOLD)
---

# Liquity V1 and V2 (LUSD and BOLD)

Status: live. Liquity is a pair of immutable, governance-minimal CDP protocols that mint a USD-pegged synthetic stablecoin against ETH (V1: LUSD) and against ETH, wstETH and rETH (V2: BOLD). V1 charges no interest and instead a one-time fee; V2 introduces user-set interest rates. Neither has an admin key or upgradeability, which is the main design contrast with Maker/Sky (see [[projects/sky-maker]]).

## Adoption metrics

| Metric | Value | Date | Source |
|--------|-------|------|--------|
| TVL, V1 | $130.33M | 2026-11-16 | [DefiLlama Liquity V1](https://api.llama.fi/protocol/liquity-v1) |
| TVL peak, V1 | $4.52B | 2021-05-11 | [DefiLlama Liquity V1](https://api.llama.fi/protocol/liquity-v1) |
| LUSD circulating | $26.85M | 2026-11-16 | [DefiLlama stablecoins](https://stablecoins.llama.fi/stablecoins) |
| TVL, V2 | $70.73M | 2026-11-16 | [DefiLlama Liquity V2](https://api.llama.fi/protocol/liquity-v2) |
| TVL peak, V2 | $180.03M | 2025-10-07 | [DefiLlama Liquity V2](https://api.llama.fi/protocol/liquity-v2) |
| BOLD circulating (current; a separate "Legacy BOLD" asset shows $0.20M) | $33.09M | 2026-11-16 | [DefiLlama stablecoins](https://stablecoins.llama.fi/stablecoins) |
| Fees, V1: 30 days / 1 year / all time | $264,765 / $376,610 / $85.11M | 2026-11-16 | [DefiLlama overview fees](https://api.llama.fi/overview/fees) |
| Fees, V2: 30 days / 1 year / all time | $775,501 / $2.49M / $3.89M | 2026-11-16 | [DefiLlama overview fees](https://api.llama.fi/overview/fees) |
| Revenue, V2: 30 days / all time | $30,361 / $696,374 | 2026-11-16 | [DefiLlama overview fees (dailyRevenue)](https://api.llama.fi/overview/fees?dataType=dailyRevenue) |

Cross-check note: DefiLlama V2 "fees" include interest, redemption fees and liquidation gas compensation (its methodology text), and "revenue" is described as 25% of interest, but the 30 day revenue ($30k) is far below 25% of 30 day fees ($194k), so the two series are not reconcilable from the API alone. Use the fee line with caution.

## How it works

### User perspective

A borrower opens a Trove, deposits collateral and mints stablecoin. V1: ETH only, minimum collateral ratio (MCR) 110%, minimum total debt 2,000 LUSD (1,800 LUSD net plus a 200 LUSD liquidation reserve; chain, see the admin control section), a one-time borrowing fee, no interest. V2: three separate borrow markets (ETH, wstETH, rETH), the borrower picks and may change their own annual interest rate, interest accrues into the debt, minimum debt 2,000 BOLD, and the Multiply feature loops collateral through Curve and Uniswap.

### Protocol perspective

- **Liquidation:** Stability Pool first (depositors burn stablecoin and receive the liquidated collateral at a discount). If the pool is empty, the debt and collateral are redistributed across the other Troves. V1 has Recovery Mode (system CR below 150%); V2 removes Recovery Mode. V2 liquidation ratio is 110% for ETH (max LTV 90.91%) and 120% for wstETH and rETH (max LTV 83.33%); the liquidated borrower usually pays a 5% penalty, and up to 10% (ETH) or 20% (LSTs) of debt in a redistribution.
- **Peg:** hard floor by redemption (anyone can swap stablecoin for collateral at $1 face value). V1 redeems against the lowest collateral-ratio Trove; V2 redeems against the lowest-interest-rate Trove first, so borrowers who set low rates carry redemption risk. Ceiling by borrowing demand: V1 through the rising borrowing fee after redemptions; V2 through borrowers cutting rates when BOLD is above $1 and raising them when below.
- **Oracles:** see [[patterns/oracle-pricing-model]]; feeds, staleness thresholds and failure behaviour are in "Admin control" below.

## Key behaviours

- [[patterns/synthetic-collateral-models]] : multi-position CDP, overcollateralised, immutable; compare with the shared debt pool in [[projects/synthetix-v2]].
- [[patterns/cdp-synthetics-landscape]] : survival correlates (immutability, one or few collateral types, redemption floor).

## Architecture decisions

- **Immutability:** contracts are not upgradeable and V2 governance only directs "Protocol Incentivized Liquidity" (25% of V2 revenue to external initiatives). Source: [V2 general FAQ](https://docs.liquity.org/v2-faq/general.md).
- **Interest-free V1, market-set rates V2:** V1 uses a one-off fee that algorithmically rises with redemptions; V2 hands rate-setting to borrowers (market-driven monetary policy).
- **Separate stability pool per collateral in V2:** keeps risk compartmentalised; redemptions are spread proportional to "outside debt" (debt minus stability pool size) per collateral.
- **No Recovery Mode in V2; delegation of rate setting** to third-party managers.
- **V2 redeployment:** an issue in the Stability Pool code was announced 2025-02-17; users were told to exit the pools. After a five-week Cantina audit contest with 800+ researchers and re-audits, V2 went live on mainnet 2025-05-19. Source: [Liquity blog: V2 is live](https://www.liquity.org/blog/liquity-v2-is-live), [stability pool issue](https://www.liquity.org/blog/stability-pool-issue) (accessed via search summary; bug details [NOT FOUND]).

## Differentiators

- Lowest MCR of the major CDPs (110%) made possible by the Stability Pool and the redemption floor.
- Only ETH and ETH LSTs as collateral: no RWAs, no centralised collateral.
- Fully immutable contracts.

## Limitations and criticisms

- Capital efficiency depends on active redemption risk management by borrowers (V2 rate setting); poorly watched low-rate Troves are redeemed first.
- Narrow collateral set means supply is bounded by ETH-denominated borrow demand: LUSD is $26.85M against a V1 TVL of $130M (accessed 2026-11-16), versus billions at the 2021 peak.
- Bootstrapping stablecoin liquidity needs incentives (PIL).
- V2 pre-launch bug and relaunch cost about three months (February to May 2025).
- Redemptions force a partial close of the borrower's Trove at face value even when the Trove is healthy; borrowers receive the redemption fee as collateral but may lose preferred ETH exposure.

### Fees and borrower cost

| Item | V1 | V2 | Source |
|------|----|----|--------|
| Interest | 0% | User-set per Trove; the current market average is [NOT FOUND] (Liquity publishes it on a Dune dashboard, not retrieved) | [V1 FAQ](https://docs.liquity.org/liquity-v1/faq/borrowing.md); [V2 FAQ](https://docs.liquity.org/v2-faq/borrowing-and-liquidations.md) |
| Opening or drawing fee | One-time borrowing fee, floor 0.5%, rising with recent redemptions up to 5%; 0% during Recovery Mode | Upfront fee equal to 7 days of average interest on that collateral branch | same |
| Rate-change fee | n/a | Premature adjustment fee if the rate is changed within 7 days of the last change | [V2 FAQ](https://docs.liquity.org/v2-faq/borrowing-and-liquidations.md) |
| Redemption fee | (baseRate + 0.5%) times ETH drawn; baseRate half-life 12 hours; paid to LQTY stakers | min(0.5% + baseRate, 100%); baseRate half-life 6 hours; stays in the redeemed Trove as collateral | [V1 FAQ](https://docs.liquity.org/liquity-v1/faq/lusd-redemptions.md); [V2 FAQ](https://docs.liquity.org/v2-faq/redemptions-and-delegation.md) |
| Interest split | n/a | 75% to the collateral market's Stability Pool depositors; 25% to governance-directed Protocol Incentivized Liquidity | [V2 BOLD and Earn FAQ](https://docs.liquity.org/v2-faq/bold-and-earn.md) |
| Liquidation gas compensation | 200 LUSD reserve | 0.0375 WETH refundable deposit plus min(0.5% of collateral, 2 units of WETH or LST) | [V2 FAQ](https://docs.liquity.org/v2-faq/borrowing-and-liquidations.md) |
| Liquidation loss to borrower | about 9.09% of collateral value at 110% MCR | 5% penalty typical; 10% or 20% of debt on redistribution | [V1 FAQ](https://docs.liquity.org/liquity-v1/faq/borrowing.md); [V2 FAQ](https://docs.liquity.org/v2-faq/borrowing-and-liquidations.md) |
| Stability Pool yield source | liquidation gains (LQTY issuance also rewards depositors per the V1 rewards FAQ, not re-fetched) | 75% of interest plus liquidation gains at about 5% discount; no token emissions | [V2 FAQ](https://docs.liquity.org/v2-faq/bold-and-earn.md) |

Audits (V1): Trail of Bits and Coinspect (DefiLlama audit links, [DefiLlama Liquity V1](https://api.llama.fi/protocol/liquity-v1)). Audits (V2): ChainSecurity, Dedaub (28 Aug 2024, 11 Nov 2024, 13 May 2025 fix review) and the Cantina contest, per [DefiLlama Liquity V2](https://api.llama.fi/protocol/liquity-v2) and [search results](https://dedaub.com/audits/liquity/liquity-v2-cantina-fixes-review-may-13-2025/).

## Admin control: who can change the rules (read 2026-11-16)

Tags: **chain** = read from Ethereum mainnet (block about 26,426,729, 2026-11-16 09:35 UTC); **source** = read in the verified contract source; **docs** = project docs or README. "Nobody" means no setter exists in the deployed source and every owner slot read returns the zero address; it is not a claim about bytecode I did not decompile.

| Lever | V1 (LUSD): who | V1: speed | V1: bounds | V2 (BOLD): who | V2: speed | V2: bounds |
|---|---|---|---|---|---|---|
| Collateral list | Nobody: ETH only, hard-wired (source); TroveManager, BorrowerOperations, StabilityPool `owner()` = zero (chain) | not applicable | fixed at deployment | Nobody: CollateralRegistry stores up to 10 tokens as `immutable`, no add function (source); `totalCollaterals()` = 3 (chain) | not applicable | WETH, wstETH, rETH, fixed at deployment |
| Synth list | Nobody: LUSD minters are `immutable` constructor addresses (source); LUSDToken has no `owner()` (chain, call reverts) | not applicable | LUSD only | Nobody: BoldToken `setBranchAddresses` was `onlyOwner` until `setCollateralRegistry` renounced it (source); BoldToken `owner()` = zero (chain) | not applicable | BOLD only |
| Risk parameters | Nobody: `constant` in LiquityBase (source); MCR 110%, CCR 150%, min net debt 1,800 LUSD, gas reserve 200 LUSD (chain) | not applicable | fixed | Nobody: `immutable` in each AddressesRegistry, owner renounced in `setAddresses` (source); all three registry `owner()` = zero (chain). Values in the note below (chain) | not applicable | fixed; constructor bounds were CCR, MCR, SCR in (100%, 200%), SP penalty at least 5%, redistribution penalty at most 20% (source) |
| Oracle | Nobody inside Liquity: PriceFeed `owner()` = zero, feed addresses set once in `setAddresses` (chain, source). Outside Liquity: the Chainlink ETH/USD proxy's `owner()` is a contract answering Safe getters, threshold 4 of 9 owners (chain), which can repoint the aggregator; Tellor is a third-party contract (not checked) | Liquity side not applicable; Chainlink side not checked (no timelock read) | Chainlink primary, Tellor fallback; constants below (chain, source) | Nobody inside Liquity: no setter in the three PriceFeed contracts, no `owner()` (source, chain). Outside: the same Chainlink owner contract owns the stETH/USD and rETH/ETH proxies (chain); the LST exchange rates come from Lido's wstETH and Rocket Pool's rETH contracts (not checked) | Liquity side not applicable | per-branch feeds and staleness below (chain, source) |
| Fees and rates | Nobody: borrowing fee (baseRate + 0.5%, cap 5%) and redemption fee (floor 0.5%) are algorithmic constants (source, chain) | not applicable | 0.5% to 5% borrowing; redemption floor 0.5% | Borrowers set their own rate; nobody sets protocol-level fees: rate bounds, upfront fee period, redemption floor and 75%/25% interest split are `constant` (source) | borrower rate change is free once per 7 days, else a premature adjustment fee (source) | rate 0.5% to 250% a year; redemption floor 0.5%; SP share 75% (source) |
| Pause | Nobody: no pause or freeze function in the deployed source (source, grep of PriceFeed, TroveManager, BorrowerOperations, StabilityPool, LUSDToken) | not applicable | none exists; Recovery Mode (TCR below 150%) restricts operations automatically (source) | Nobody: no pause function (source); the only freeze is branch shutdown | not applicable | none exists |
| Code upgrades | Nobody: no proxies, EIP-1967 implementation slot empty on TroveManager, BorrowerOperations, StabilityPool, LUSDToken, PriceFeed (chain) | not applicable | none | Nobody: EIP-1967 slot empty on BoldToken, the three BorrowerOperations, three PriceFeeds and Governance (chain); no `delegatecall` in core `src/` outside Zappers (source) | not applicable | the only route is a new deployment that users migrate to by choice, as happened with "Legacy BOLD" in 2025 (docs) |
| Shutdown | Nobody: no shutdown function exists (source) | not applicable | none; on oracle failure the system keeps running at `lastGoodPrice` | Anyone, or automatic: `BorrowerOperations.shutdown()` is permissionless when branch TCR < SCR; `shutdownFromOracleFailure()` fires from any `fetchPrice` that sees a failed feed (source) | immediate, one transaction, irreversible (source, docs) | per branch only; SCR 110% (WETH) or 120% (wstETH, rETH) (chain); no branch is shut down today (chain) |

**V2 risk parameters per branch (chain, AddressesRegistry getters; matching `Constants.sol`, source).**

| Branch | CCR | MCR | SCR | BCR | Penalty (Stability Pool) | Penalty (redistribution) | Min debt | Current TCR (chain, at `lastGoodPrice`) |
|---|---|---|---|---|---|---|---|---|
| WETH | 150% | 110% | 110% | 10% | 5% | 10% | 2,000 BOLD | 155.8% |
| wstETH | 160% | 120% | 120% | 10% | 5% | 20% | 2,000 BOLD | 225.8% |
| rETH | 160% | 120% | 120% | 10% | 5% | 20% | 2,000 BOLD | 296.5% |

The TCR column is my calculation from `getEntireBranchColl()`, `getEntireBranchDebt()` and the feed's stored `lastGoodPrice` (not a live `fetchPrice`), so treat it as approximate. The WETH branch sits 45.8 points above its 110% shutdown line. BCR (an extra margin required of batch managers) and the min debt are `constant` in source.

**Oracle detail.**

- V1 PriceFeed `0x4c517D4e2C851CA76d7eC94B805269Df0f2201De` (chain): `priceAggregator()` = Chainlink ETH/USD proxy `0x5f4e...8419`; `tellorCaller()` = `0xad43...1112`, whose `tellor()` is `0x88df...78a0` ("Tellor Tributes"). Constants: `TIMEOUT` 14,400 s (4 hours) for both oracles; a Chainlink move of more than 50% between consecutive rounds makes the feed untrusted unless Tellor agrees within 5%; the system returns to Chainlink only when both oracles are live and within 5% of each other (source, chain). If Chainlink fails it uses Tellor; if both fail it freezes at `lastGoodPrice` and keeps operating at that price, with no shutdown (source). At read time `status()` = 0 (Chainlink working) and `lastGoodPrice` = $1,809.35; Tellor's latest value was $1,811.84 timestamped 05:14 UTC, about 4.2 hours old, so by V1's own rule Tellor was momentarily "frozen" (chain).
- V2 feeds (chain, source). WETH `0xcc5f...c24f`: Chainlink ETH/USD, staleness 86,400 s (24 hours). wstETH `0xe7aa...314e`: Chainlink stETH/USD `0xCfE5...b4a8` (24 hours) times `stEthPerToken()` from wstETH; ETH/USD (24 hours) also read. rETH `0x34f1...7b2b`: Chainlink rETH/ETH `0x5362...C2d0` (172,800 s, 48 hours) times ETH/USD, and the price is the lower of that market price and ETH/USD times rETH `getExchangeRate()`. For redemptions only, the higher of the two is used when they are within 1% (stETH/USD versus ETH/USD) or 2% (rETH market versus canonical), to blunt redemption arbitrage. A feed counts as failed if the call reverts, the answer is zero or negative, or its age is at or above the threshold (source). All three `priceSource()` = 0 (primary) at read time (chain).
- V2 on failure (source): ETH/USD or exchange-rate failure shuts the branch down and freezes it at `lastGoodPrice`; an LST market feed failure shuts it down and switches to ETH/USD times the canonical rate, capped at `lastGoodPrice`; a later ETH/USD failure drops to `lastGoodPrice`. The thresholds are constructor arguments stored in storage with no setter, so they cannot change.

**V2 shutdown: what freezes and how people exit (source, README).** After shutdown: no new Troves, no debt or collateral adjustment, no rate changes, no normal redemptions against the branch (CollateralRegistry skips branches that are shut down or below SCR), and interest stops accruing from `shutdownTime`. Still allowed: `closeTrove` (including the last Trove), liquidations, Stability Pool deposits and withdrawals, `claimCollateral`, and `urgentRedemption` on that branch's TroveManager. Urgent redemptions charge no fee, pay the redeemer collateral worth 102% of the BOLD burned (`URGENT_REDEMPTION_BONUS` 2%), let the redeemer pick any Troves, and take a `_minCollateral` slippage bound. A borrower exits by repaying and closing; a BOLD holder exits by urgent redemption on the shut branch, normal redemption on the healthy branches, or the market. The README names bad debt on a shut branch as a known risk and lists remedies (redirecting redemption fees, Stability Pool haircuts, governance redirecting interest) as possible solutions; I found no such mechanism in the deployed core, so bad debt left on a shut branch has no on-chain clearing path as far as this read goes.

**V2 governance (source, chain, docs).** Governance `0x807d...eee1` is also every branch's `interestRouter`: ActivePool mints the non-Stability-Pool 25% of interest to it, and that is the only link from core to governance (source). It holds no role in any core contract. Its owner registered the three initial initiatives and renounced (`owner()` = zero, chain). Voters are LQTY holders who deposit through a per-user proxy that stakes in V1 LQTYStaking, with voting power growing with LQTY amount times stake age (source, docs). On chain: epoch 604,800 s (7 days), voting cutoff 518,400 s (after day 6 only vetoes), registration fee 100 BOLD, registration threshold 0.01% of votes, qualifying threshold 2% of votes, unregistration after 4 epochs. Registering an initiative needs the caller's own voting power to be at least 0.01% of the previous epoch's total, plus the 100 BOLD fee (source); unregistering a failed initiative is open to anyone once its conditions are met (docs, source). It can decide which addresses receive that 25% stream; it cannot change collateral, parameters, oracles, fees, the 75%/25% split, or code (source; docs say the same in [V2 general FAQ](https://docs.liquity.org/v2-faq/general.md)). A side note: the deploy script adds LUSD rewards to a Curve gauge with a "FUNDS_SAFE" `0xF060...45f8` as distributor; that touches a Curve gauge, not Liquity core (source, not read on chain).

**Method.** Repos cloned at liquity/dev `3e64ee1` (2025-11-25), liquity/bold `d262669` (2026-09-26) and liquity/V2-gov `8f4bff3` (2026-10-17); addresses from `packages/lib-ethers/deployments/default/mainnet.json` (V1) and `contracts/addresses/1.json` (V2). I compared the repo files with the Sourcify-verified sources of the deployed contracts. V2 BorrowerOperations, TroveManager, the three PriceFeeds and Governance match the repo except for comment and identifier typo fixes; V1 PriceFeed, BorrowerOperations and LUSDToken match exactly, while V1 TroveManager and StabilityPool differ from later repo commits, so for those I grepped the deployed source. On chain: `eth_call` of getters listed above, `eth_getStorageAt` for the EIP-1967 implementation slot, `eth_getCode` for size, through `ethereum-rpc.publicnode.com`.

**Not checked.** Event history (`eth_getLogs` refused over wide ranges), so I did not confirm from logs that ownership renouncements happened in the deployment transactions, only that the owners are zero now; V1 SortedTroves, ActivePool, DefaultPool, CollSurplusPool and LQTYStaking owners (source shows the same `_renounceOwnership()` pattern); V2 StabilityPool, ActivePool and SortedTroves owners or proxy slots; the Chainlink owner contract's signers, modules and any delay; Tellor's upgradeability; Lido and Rocket Pool control over the exchange-rate contracts; the Zapper and exchange helper contracts (periphery; not read).

**Conflicts with the rest of this note.** The "User perspective" section gave V1 minimum debt as "2,000 LUSD plus a 200 LUSD liquidation reserve" (now corrected); on chain `MIN_NET_DEBT()` is 1,800 LUSD and `LUSD_GAS_COMPENSATION()` 200 LUSD, so the minimum total debt is 2,000 LUSD including the reserve. "Neither has an admin key" holds for Liquity's own contracts, but both versions inherit Chainlink's admin over the feed proxies, and V2 additionally Lido's and Rocket Pool's over the exchange rates.

## Sources

- [DefiLlama Liquity V1 API](https://api.llama.fi/protocol/liquity-v1) : accessed 2026-11-16
- [DefiLlama Liquity V2 API](https://api.llama.fi/protocol/liquity-v2) : accessed 2026-11-16
- [DefiLlama stablecoins list (LUSD, BOLD)](https://stablecoins.llama.fi/stablecoins) : accessed 2026-11-16
- [DefiLlama fees overview](https://api.llama.fi/overview/fees) : accessed 2026-11-16
- [Liquity V1 borrowing FAQ](https://docs.liquity.org/liquity-v1/faq/borrowing.md) : accessed 2026-11-16
- [Liquity V1 redemptions FAQ](https://docs.liquity.org/liquity-v1/faq/lusd-redemptions.md) : accessed 2026-11-16
- [Liquity V2 general FAQ](https://docs.liquity.org/v2-faq/general.md) : accessed 2026-11-16
- [Liquity V2 borrowing and liquidations FAQ](https://docs.liquity.org/v2-faq/borrowing-and-liquidations.md) : accessed 2026-11-16
- [Liquity V2 BOLD and Earn FAQ](https://docs.liquity.org/v2-faq/bold-and-earn.md) : accessed 2026-11-16
- [Liquity V2 redemptions and delegation FAQ](https://docs.liquity.org/v2-faq/redemptions-and-delegation.md) : accessed 2026-11-16
- [Liquity blog: V2 is live](https://www.liquity.org/blog/liquity-v2-is-live) : accessed 2026-11-16 (search summary)
- [liquity/dev repository (V1 source and mainnet deployment manifest), commit 3e64ee1](https://github.com/liquity/dev) : accessed 2026-11-16
- [liquity/bold repository (V2 source, `contracts/addresses/1.json`, README sections "Collateral branch shutdown" and "Oracles in Liquity v2"), commit d262669](https://github.com/liquity/bold) : accessed 2026-11-16
- [liquity/V2-gov repository (Governance source), commit 8f4bff3](https://github.com/liquity/V2-gov) : accessed 2026-11-16
- [Sourcify verified sources, e.g. V2 WETH BorrowerOperations](https://sourcify.dev/server/v2/contract/1/0x372abd1810eaf23cb9d941bbe7596dfb2c46bc65) and [V1 PriceFeed](https://sourcify.dev/server/v2/contract/1/0x4c517D4e2C851CA76d7eC94B805269Df0f2201De) : accessed 2026-11-16
- [Ethereum mainnet JSON-RPC (publicnode)](https://ethereum-rpc.publicnode.com) : `eth_call`, `eth_getStorageAt`, `eth_getCode` read 2026-11-16
- [Liquity V2 LQTY staking and voting FAQ](https://docs.liquity.org/v2-faq/lqty-staking.md) : accessed 2026-11-16
- [Liquity V2 legacy deployment addresses](https://docs.liquity.org/v2-documentation/technical-resources.md) : accessed 2026-11-16
