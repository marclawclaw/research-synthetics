---
tags: [project, ethereum, cdp, stablecoin, synthetics, governance]
ecosystem: Ethereum
category: Synthetics/CDP stablecoin
status: live (nine mint markets, seven with a non-zero debt ceiling)
status_date: 2026-11-16
website: https://www.curve.finance
docs: https://docs.curve.finance/developer/crvusd/overview
launched: 2023 (first market sfrxETH; launch date not re-sourced here, [NOT FOUND])
---

# crvUSD (Curve stablecoin)

Status: live. crvUSD is Curve's per-position CDP stablecoin. Borrowers lock collateral in one of nine DAO-created markets and mint crvUSD. Collateral sits in a per-market AMM (LLAMMA) that converts it to crvUSD gradually as the price falls ("soft liquidation") rather than in one auction. The peg is held by PegKeepers and an interest-rate policy, not by redemption. Everything is governed by the Curve DAO (veCRV Aragon votes). Since October 2026 a 5-of-9 Emergency DAO Safe can also act at once on a bounded set of levers (see the admin section).

## Adoption metrics

| Metric | Value | Date | Source |
|--------|-------|------|--------|
| TVL | $73.6M | 2026-11-16 | [DefiLlama crvUSD](https://api.llama.fi/protocol/crvusd) (copied from [[patterns/cdp-synthetics-landscape]] section 2.2) |
| Borrow rate, main markets | 6.48% to 7.92% APR | 2026-11-16 03:56 UTC | [Curve markets API](https://prices.curve.finance/v1/crvusd/markets?fetch_on_chain=false&page=1&per_page=50) (via the landscape table) |
| Debt in the WBTC market | 17,491,116 crvUSD, 164 loans | 2026-11-16 | chain: `total_debt()`, `n_loans()` on controller `0x4e59541306910aD6dC1daC0AC9dFB29bD9F15c67` |
| Debt ceilings (per market) | wstETH 150M; WBTC 200M; WETH 200M; sfrxETH (v2) 50M; tBTC 50M; weETH 20M; cbBTC 50M; sfrxETH (v1) 0; LBTC 0 | 2026-11-16 | chain: `ControllerFactory.debt_ceiling(controller)` |

`crvUSD.totalSupply()` (2.13B) is not a circulation figure: the factory mints each market's whole debt ceiling into the controller up front, so supply includes unborrowed headroom.

## How it works

### User perspective

A borrower picks a market (WBTC, wstETH, WETH, sfrxETH, tBTC, weETH, cbBTC; LBTC and the first sfrxETH market are at a zero debt ceiling), deposits collateral and chooses a number of bands N (4 to 50) over which the collateral is spread. Maximum LTV comes from the market's `loan_discount` (WBTC: 9%, so roughly 91% before band effects; landscape table gives 84% to 91% across markets). The interest rate is variable and set per market by the monetary policy contract. There is no redemption: crvUSD holders cannot claim collateral at $1. Source: [crvUSD docs overview](https://docs.curve.finance/developer/crvusd/overview); chain reads below.

### Protocol perspective

- **Markets** are created by `ControllerFactory.add_market` from blueprint contracts; each market is a Controller plus an AMM, neither behind a proxy (source; chain).
- **Soft liquidation:** the AMM sells collateral for crvUSD band by band as the oracle price enters the borrower's bands, and buys it back as the price recovers. Hard liquidation is possible once health, computed with the borrower's stored `liquidation_discount` (WBTC: 6%), falls below zero (source).
- **Supply:** crvUSD's only minter is the factory. The factory mints when the admin raises a debt ceiling and burns when it is lowered and the crvUSD is idle (source).
- **Peg:** PegKeepers hold crvUSD minted to them through their own debt ceilings. Above $1 they deposit it into crvUSD/stablecoin pools; below $1 they withdraw what they deposited. The monetary policy lowers rates as PegKeeper debt rises and raises them when crvUSD trades below $1 (source; [PegKeeper docs](https://docs.curve.finance/developer/crvusd/pegkeepers/overview)).
- **Oracles:** per-market contracts built from Curve pool EMA prices, with an optional Chainlink clamp; detail in the oracle row below. See also [[patterns/oracle-pricing-model]].

## Key behaviours

- [[patterns/cdp-synthetics-landscape]] : per-position CDP without a redemption floor; sections 4a and 4b place crvUSD on the governed (Aave) end of axes 2 and 3.
- [[patterns/cdp-vs-morpho-trust]] : comparison of CDP admin models with Morpho.
- [[patterns/synthetic-collateral-models]] : overcollateralised, multi-collateral, isolated per market.

## Admin control: who can change the rules (read 2026-11-16)

Method. **chain**: `eth_call`, `eth_getStorageAt` and `eth_getCode` against Ethereum mainnet through `ethereum-rpc.publicnode.com`, latest block about 26,426,730 (2026-11-16 09:35 UTC); selectors computed locally, and "selector present" means the 4-byte selector occurs in the runtime bytecode. **source**: the [curve-stablecoin](https://github.com/curvefi/curve-stablecoin) repository at tag `v1` (commit `edbb5ef`, 2025-12-04), which holds the mint-market contracts (`master` has since been refactored into a new layout), plus Etherscan-verified source for two deployed contracts: the factory admin [Lending Owner Proxy](https://etherscan.io/address/0xeDA020b067872717901916a3B8C724C21f6080d1#code) (Vyper 0.4.3) and the [wstETH Controller](https://etherscan.io/address/0x100dAa78fC509Db39Ef7D04DE0c1ABD299f4C6CE#code) (Vyper 0.3.7). **docs**: Curve developer docs. I did not compare deployed bytecode with the `v1` source; the selector checks below are consistent with it.

### The admin chain

- `crvUSD.minter()` is the ControllerFactory `0xC9332fdCB1C491Dcc683bAe86Fe3cb70360738BC` (chain). crvUSD's `set_minter` is callable only by the minter, and the factory has no function that calls it, so the minter is fixed to the factory in practice (source).
- `factory.admin()` is `0xeDA020b067872717901916a3B8C724C21f6080d1`, Etherscan name **Lending Owner Proxy**, created 2026-10-22 01:21 UTC (block 26,244,830, via the CreateX deployer). It is an access-control forwarder, not an upgradeable proxy (source; chain). Its constructor arguments and getters read `dao()` = Curve Ownership Agent `0x40907540d8a6C65c637785e8f8B742ae6b0b9968`, `emergency_dao()` = `0x467947EE34aF926cF1DCac093870f613C96B1E0c`, and `CONFIGURATOR()` = `0x6065858d0eF0AA240DFdf6f1A0B2ae34B41f49bC` (the LlamaLend V2 configurator, outside crvUSD minting) (chain).
- The Ownership Agent acts only after a vote in the Ownership Voting app `0xE478de485ad2fe566d49342Cbd03E49ed7DB3356`: `voteTime()` 604,800 s (7 days), `supportRequiredPct()` 51%, `minAcceptQuorumPct()` 30%, `minBalance()` 2,500 veCRV to create a vote (chain; addresses per [Curve governance docs](https://docs.curve.finance/developer/curve-dao/governance/overview.md)). No timelock sits between a passed vote and execution in the path read.
- The Emergency DAO `0x467947...1E0c` is a Safe v1.3.0 (singleton `0xd9Db...9552`), **5 of 9** owners, no guard and no modules (chain). Docs call it the "Emergency Agent" and also list a separate `emergency-dao-multisig` `0x6d447e544D01a59cb0774763bf15526574CffFeD` ([deployments](https://docs.curve.finance/developer/deployments.md)), which I did not read.
- **Lending Owner Proxy, all 22 external functions (source):**
  - DAO only: `execute(target, calldata)` (arbitrary call, so the DAO keeps every factory and controller admin power) and `set_emergency_dao`.
  - DAO or Emergency DAO, with no bound in the proxy: `set_monetary_policy(controller, policy)`, `set_callback(controller, cb)`, `set_use_chainlink(oracle, bool)`, `set_killed(gauge)`, `set_gauge_manager(gauge, manager)`, `pause(target)`, `remove_emergency()`.
  - DAO or Emergency DAO, one direction only: `reduce_debt_ceiling(factory, to, amount)` (amount must not exceed the current ceiling) and `increase_loan_discount(controller, new)` (must exceed the current value; keeps the liquidation discount).
  - LlamaLend V2 only (through the configurator, not used by crvUSD mint markets): `set_v2_monetary_policy`, `increase_v2_loan_discount`, `set_v2_price_oracle` (only when the current oracle fails and the new one works), `set_v2_callback`, `remove_v2_custom_admin`, `set_v2_borrow_cap_zero`, `set_v2_max_supply_zero`.
  - Anyone: `accept_transfer_ownership(factory)`. Getters: `dao`, `emergency_dao`, `CONFIGURATOR`.
  - The contract header says: "The DAO can revoke the emergency DAO and return every admin role through execute."
- Other admins (chain): the monetary policy `0x07491D124ddB3Ef59a8938fCB3EE50F9FA0b9251`, the crvUSD price aggregator `0x18672b1b0c623a30089A280Ed9256379fb0E4E62`, all five PegKeepers and the PegKeeperRegulator `0x36a04CAffc681fa179558B2Aaba30395CDdd855f` have `admin()` = the Ownership Agent directly, so only a DAO vote changes them. The regulator's `emergency_admin()` is the Emergency DAO Safe.

### Lever by lever

"DAO" means an Ownership vote (7 days, then execution through the Agent and, for factory or controller calls, `LendingOwnerProxy.execute`). "eDAO" means the 5-of-9 Emergency DAO Safe acting through the Lending Owner Proxy or the regulator, effective in one transaction.

| Lever | Who | Speed | Bounds | Evidence |
|-------|-----|-------|--------|----------|
| Collateral list | DAO: `add_market` requires `msg.sender == self.admin`, and the proxy forwards arbitrary calls only for the DAO. eDAO cannot add a market. No remove function: a market is retired by cutting its debt ceiling (DAO, or eDAO through `reduce_debt_ceiling`) | add: 7 day vote. Retire: eDAO immediate | At creation: A between 2 and 10,000; AMM fee between 1e-12 and 10%; admin fee below 100%; liquidation discount at least 1%; loan discount at most 50% and above the liquidation discount; the oracle must return a non-zero `price()` equal to `price_w()`; the monetary policy must answer `rate_write()` | source; chain (`add_market` selector present; nine markets; two at ceiling 0) |
| Synth list | Only crvUSD; minter fixed to the factory. **Supply:** the factory admin can mint crvUSD to any address with `set_debt_ceiling(to, amount)` (used for markets and PegKeepers). DAO can raise or lower; eDAO can only lower | DAO 7 days; eDAO lowering immediate | No cap on `set_debt_ceiling` amount or recipient in source. PegKeeper ceilings now: USDC 135M, USDT 135M, pyUSD 45M, frxUSD 9M, GHO 0 | source; chain |
| Risk parameters | `set_borrowing_discounts` (loan and liquidation discount) and `set_amm_fee` on each controller: factory admin, so DAO. eDAO can only raise `loan_discount`. Older controllers (sfrxETH v1, wstETH, WBTC) also have `set_amm_admin_fee`; the LBTC controller does not. Debt ceiling as above. **Fixed at creation:** A (band width 1/A; WBTC A = 100), collateral token, oracle contract, AMM admin fee on newer markets | DAO 7 days; eDAO immediate for loan discount increases and ceiling cuts | Liquidation discount at least 1%, loan discount at most 50% and above liquidation discount. AMM fee 1e-12 to 10% (deployed wstETH controller), or to min(4/A, 10%) in the `v1` source. Admin fee at most 100% (deployed wstETH controller). Each borrower's liquidation discount is stored when they last change their own loan, so a new value does not apply to existing loans retroactively | source; chain (selectors on four controllers; WBTC values: loan discount 9%, liquidation discount 6%, fee 1.9%, admin fee 1 wei) |
| Oracle | No one can replace a market's oracle: `price_oracle_contract` is immutable in the AMM, and no `set_price_oracle` selector exists on the four controllers and AMMs checked. Inside the oracle, `set_use_chainlink` is factory admin only, so DAO or eDAO. Oracle inputs: the crvUSD price aggregator's pool list (`add_price_pair`, `remove_price_pair`) is DAO only | Chainlink toggle: eDAO immediate. Aggregator pool list: DAO 7 days | WBTC worked example `0xBe83fD842DB4937C0C3d15B2aBA6AF7E854f8dcb`: TVL-weighted EMA of two Curve tricrypto pools (`0x7F86...829B`, `0xf5f5...e2B4`) times the aggregator price. Chainlink clamp ±1.5% (`BOUND_SIZE` 0.015) applies only when `use_chainlink` is true; it reads **false** on the WBTC and wstETH oracles, so the clamp is off. The AMM caps how far the oracle price it uses can move relative to the price two minutes earlier to a factor of 1.25 (`MAX_P_O_CHG`, `PREV_P_O_DELAY`) | source; chain |
| Fees and rates | Monetary policy parameters (`set_rate`, `set_sigma`, `set_target_debt_fraction`, `set_extra_const`, `add_peg_keeper`, `remove_peg_keeper`): policy admin = Ownership Agent, so DAO only. **Swapping a market's policy contract:** DAO or eDAO (`set_monetary_policy`). Fee receiver: DAO | parameters: 7 days. Policy swap: eDAO immediate | rate0 at most `MAX_RATE`; sigma 1e14 to 1e18; target debt fraction at most 1e18; extra constant at most `MAX_RATE`. The controller caps any policy's output at `MAX_RATE` = 43,959,106,799 per second, which is ln 4 per year (debt can at most quadruple in a year; source comments call it 300% or 400% APY). A swapped-in policy is otherwise unchecked beyond answering `rate_write()` once. Current values: rate0 1,585,489,600 per second (5.0% a year), sigma 0.008, target debt fraction 1.0, extra constant 0; WBTC market rate 6.76% a year | source; chain |
| Pause | No pause function on the factory, controllers, AMMs, monetary policy, regulator or PegKeepers (no `pause()` selector; the proxy's `pause(target)` has no crvUSD mint-market target). Brakes that exist: cut a debt ceiling (no new borrowing), raise `loan_discount`, and `PegKeeperRegulator.set_killed` (stops PegKeeper deposits, withdrawals or both; admin or `emergency_admin`) | eDAO immediate; DAO 7 days | Repayment, collateral changes and liquidation cannot be switched off by any designed lever. Inference from source, not tested: because every loan action calls the policy's `rate_write()`, a swapped-in policy that later reverts would block those actions; this is a side effect of the policy lever, not a pause | source; chain (regulator `is_killed()` = 0) |
| Code upgrades | Controllers, AMMs and the factory are not upgradeable (no EIP-1967 slot on the factory; controllers per section 4a of the landscape note). `set_implementations` changes the blueprints for **future** markets only (DAO). Swappable modules on live markets: monetary policy and liquidity-mining callback (DAO or eDAO), Chainlink toggle (DAO or eDAO). Factory admin can be moved by `set_admin` through `execute` (DAO, one step, no delay) | DAO 7 days; eDAO immediate for policy and callback swaps | Callback: any address; in the `v1` AMM source callbacks are called with `revert_on_failure=False`, so a bad callback should not block the AMM (older deployed AMMs not checked). eDAO can be removed by the DAO (`set_emergency_dao`) or by itself (`remove_emergency`) | source; chain |
| Shutdown | No global settlement or redemption. Per market: debt ceiling to 0 (eDAO or DAO); anyone can then call `rug_debt_ceiling` to burn idle crvUSD in the controller; the DAO or eDAO can swap in a higher-rate policy to push borrowers to repay. Existing loans stay open until repaid or liquidated | first steps immediate (eDAO); loans run off at borrowers' pace | The policy's ceiling-fill multiplier applies only when the ceiling is above 0, so a zero ceiling does not raise the rate by itself (source). Worked example, sfrxETH v1 market (controller `0x8472A9A7632b173c8Cf3a86D3afec50c35548e76`): ceiling 0, its own policy `0xc684432FD6322c6D58b6bC5d28B18569aA0AD0A1` with rate0 4,756,468,797 per second (15.0% a year) and a current rate of 20.2% a year, and still 4 loans with 21,631 crvUSD of debt. LBTC: ceiling 0, 2 loans, 1.11 crvUSD of debt | chain; source |

**How a crvUSD holder exits.** There is no on-chain claim on collateral. Holders sell crvUSD in Curve pools (the five PegKeeper pools pair crvUSD with USDC, USDT, pyUSD, frxUSD and GHO) or to borrowers who need crvUSD to repay. Below $1, PegKeepers may withdraw only crvUSD they previously deposited (source). All five read `debt()` = 0 on 2026-11-16, so they had no downside capacity at that moment (chain).

**Reading.** On paper crvUSD is governed by a 7 day DAO vote with no timelock. Since October 2026, though, a 5-of-9 Safe can do several things at once without a vote. It can swap any market's interest-rate contract, so a rate anywhere from 0 to the `MAX_RATE` cap applies immediately. It can also cut debt ceilings, tighten max LTV, toggle the Chainlink clamp on oracles and kill PegKeepers. Its powers stop short of minting crvUSD, adding markets, replacing oracles or changing policy parameters directly. The protocol has no pause that freezes repayment or liquidation.

**Conflicts with the vault and docs.**
- Landscape sections 4a and 4b call the factory admin a "22-function proxy contract". The count is right, but the contract is the Lending Owner Proxy, a forwarder with an emergency role, not an upgradeable proxy. The 4b reading "who can change the rules: Curve DAO vote (7 days)" needs a qualifier: the Emergency DAO acts immediately on the levers above. The 4b item "not checked: whether any off-chain guardian can pause" is now answered. The Emergency DAO is tied to crvUSD through the factory admin and the PegKeeperRegulator, though not with a pause as such.
- [Factory docs](https://docs.curve.finance/developer/crvusd/factory.md) say the factory `admin` is the CurveOwnershipAgent; on chain it is the Lending Owner Proxy, which was created 2026-10-22. The date of the admin transfer is [NOT FOUND] (it needs event logs, which the endpoint refuses over wide ranges).
- The landscape row lists LBTC among the collaterals without noting that its debt ceiling is 0 (chain). The first sfrxETH market is also at 0.
- [Deployments docs](https://docs.curve.finance/developer/deployments.md) list four PegKeepers (USDC, USDT, pyUSD, frxUSD). The chain has five in both the monetary policy and the regulator; the fifth, on the GHO pool, has a zero debt ceiling.

**Not checked:** veCRV voting concentration; the Aragon Agent's own upgradeability; the Emergency DAO signer identities and the separate `emergency-dao-multisig` address in the docs; bytecode equality between deployed contracts and the `v1` source, other than the wstETH controller and the proxy, whose verified source was read; callback failure behaviour in the older deployed AMMs; the oracle contracts of markets other than WBTC and wstETH; and the LM callback contract `0x7a91...5f7d` on the WBTC AMM.

## Sources

- [DefiLlama crvUSD API](https://api.llama.fi/protocol/crvusd) : accessed 2026-11-16 (TVL, via [[patterns/cdp-synthetics-landscape]])
- [Curve crvUSD markets API](https://prices.curve.finance/v1/crvusd/markets?fetch_on_chain=false&page=1&per_page=50) : accessed 2026-11-16 (via the landscape table)
- [curve-stablecoin GitHub repository, tag v1](https://github.com/curvefi/curve-stablecoin/tree/v1/contracts): `ControllerFactory.vy`, `Controller.vy`, `AMM.vy`, `Stablecoin.vy`, `mpolicies/AggMonetaryPolicy3.vy` and `AggMonetaryPolicy3c.vy`, `stabilizer/PegKeeperV2.vy` and `PegKeeperRegulator.vy`, `price_oracles/CryptoWithStablePriceWBTC.vy` and `AggregateStablePrice3.vy` : accessed 2026-11-16
- [Etherscan: Lending Owner Proxy 0xeDA0...80d1, verified source and constructor arguments](https://etherscan.io/address/0xeDA020b067872717901916a3B8C724C21f6080d1#code) : accessed 2026-11-16
- [Etherscan: crvUSD wstETH Controller 0x100d...C6CE, verified source](https://etherscan.io/address/0x100dAa78fC509Db39Ef7D04DE0c1ABD299f4C6CE#code) : accessed 2026-11-16
- [Curve docs: crvUSD overview](https://docs.curve.finance/developer/crvusd/overview) : accessed 2026-11-16
- [Curve docs: crvUSD factory](https://docs.curve.finance/developer/crvusd/factory.md) : accessed 2026-11-16
- [Curve docs: PegKeepers overview](https://docs.curve.finance/developer/crvusd/pegkeepers/overview) : accessed 2026-11-16
- [Curve docs: governance and voting](https://docs.curve.finance/developer/curve-dao/governance/overview.md) : accessed 2026-11-16
- [Curve docs: deployments](https://docs.curve.finance/developer/deployments.md) : accessed 2026-11-16
- Ethereum mainnet JSON-RPC (`https://ethereum-rpc.publicnode.com`): `eth_call`, `eth_getStorageAt`, `eth_getCode`, `eth_getTransactionByHash` against the addresses named above : accessed 2026-11-16
