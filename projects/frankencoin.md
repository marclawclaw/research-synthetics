---
tags: [project, ethereum, cdp, stablecoin, synthetics, oracle-free, veto-governance]
ecosystem: Ethereum (plus CCIP-bridged deployments on Polygon, Arbitrum, Optimism, Base, Avalanche, Gnosis and Sonic)
category: Synthetics/CDP stablecoin (CHF)
status: live (ZCHF and FPS since 2023-10-28; MintingHub V2 since 2024-12-11; FCS governance wrapper since about 2026-08-28)
status_date: 2026-11-16
website: https://frankencoin.com
docs: https://docs.frankencoin.com
launched: 2023
---

# Frankencoin (ZCHF)

Status: live. Frankencoin is a Swiss franc stablecoin (ZCHF) minted against arbitrary ERC-20 collateral in per-position CDPs whose terms the borrower proposes, with no price oracle in liquidation. Rule changes are not made by an admin key: anyone may propose a new position or a new minting module, and holders of the reserve share (FPS, now mostly wrapped in FCS) may veto during a waiting period. Once a minting module is approved it cannot be removed. See [[patterns/cdp-synthetics-landscape]] sections 4a and 4b and [[patterns/cdp-vs-morpho-trust]].

## Adoption metrics

| Metric | Value | Date | Source |
|--------|-------|------|--------|
| TVL | $48.3M (peak $88.3M on 2026-09-22) | 2026-11-16 | [DefiLlama Frankencoin](https://api.llama.fi/protocol/frankencoin) (via [[patterns/cdp-synthetics-landscape]]) |
| ZCHF supply, Ethereum (chain: `totalSupply()`) | 27,771,913 ZCHF | 2026-11-16 | `eth_call` on ZCHF `0xB58E61C3098d85632Df34EecfB899A1Ed80921cB` |
| Reserve held by FPS contract (chain: `balanceOf(FPS)`) | 11,070,613 ZCHF, of which equity 3,553,638 (`equity()`) and borrowers' minter reserve 7,516,975 (`minterReserve()`) | 2026-11-16 | same |
| FPS price (chain: `price()`) | 1,251.10 ZCHF | 2026-11-16 | `eth_call` on FPS `0x1bA26788dfDe592fec8bcB0Eaff472a42BE341B2` |
| Savings deposits in the main savings module | 10.55M ZCHF | 2026-11-16 | [Frankencoin API savings/core/info](https://api.frankencoin.com/savings/core/info) |

## How it works

### User perspective

A borrower opens a position by choosing the collateral token, liquidation price, minting limit, term, challenge period, risk premium and reserve share, paying a 1,000 ZCHF opening fee and waiting through an initialisation period of at least three days (source: `MintingHubV2.OPENING_FEE`, `PositionV2` constructor `require(_initPeriod >= 3 days)`). Others can clone an approved position, which needs no new fee or veto window. Interest is charged upfront for the term: (global lead rate plus the position's fixed risk premium) times time to expiry, capped at 100% (source: `PositionV2.calculateFee`). Part of each mint (the reserve share, at least the 2% challenger reward) is locked in the reserve pool and returned on repayment.

ZCHF holders can deposit into a savings module that pays the savings lead rate; the interest is drawn from the reserve via `coverLoss` (source: `AbstractSavings.refresh`).

### Protocol perspective

- **Liquidation (oracle-free):** anyone can challenge a position by posting collateral of the same type. Bidders buy at the liquidation price during the first phase (which averts the challenge), then at a price declining linearly to zero over the challenge period; the challenger earns 2% (source: `MintingHubV2._calculatePrice`, `CHALLENGER_REWARD = 20000` ppm). Expired positions are sold through `buyExpiredCollateral`, starting at 10 times the liquidation price and falling to zero over two challenge periods (`EXPIRED_PRICE_FACTOR = 10`).
- **Loss waterfall:** a loss is paid first from FPS equity, then from the borrowers' minter reserve (which shrinks pro rata, `calculateAssignedReserve`), and once the reserve contract is empty `Frankencoin.coverLoss` mints new unbacked ZCHF (source: `coverLoss`). There is no haircut or redemption for ZCHF holders; the loss becomes dilution of all ZCHF.
- **Reserve share (FPS):** anyone can mint FPS by depositing ZCHF; redemption requires a 90 day average holding period. Voting power is FPS balance times holding time.
- **FCS wrapper:** FCS (`0xdb861830d9ae2d1fcf99fa0cfd3973de382b0b5b`) wraps FPS 1:1, removes the 90 day lock in favour of a redemption discount that deepens with recent redemptions, and exercises the wrapped FPS votes through three governance modules (below).

## Admin control: who can change the rules (read 2026-11-16)

Evidence tags: **chain** = `eth_call` against Ethereum mainnet through `ethereum-rpc.publicnode.com`; **source** = contract source, either the Sourcify-verified source of the deployed address or the [FrankenCoin repository](https://github.com/Frankencoin-ZCHF/FrankenCoin) at commit `8b4c4ab` (2026-09-07); **docs** = documentation or the project's indexer API.

### The governance path

There is no owner, admin or proxy in the core (chain, from earlier reads in [[patterns/cdp-synthetics-landscape]] section 4a; source confirms no pause, owner or upgrade function in `Frankencoin.sol`, `Equity.sol`, `MintingHubV2` or `PositionV2`). Every rule change is a qualification check against vote weight:

| Layer | Contract | Threshold | Evidence |
|-------|----------|-----------|----------|
| FPS1 (the deployed reserve, 2023) | FPS `0x1bA2...41B2` | `QUORUM = 200` basis points: at least 2% of FPS time-weighted votes, own plus delegated (`_votes * 10000 < QUORUM * totalVotes()` reverts) | source (Sourcify, deployed address) |
| FCS (2026) | MainnetVotes `0x3bc9575c1E41322De656ef93eF5cAd9BE9e90DcD` | `QUORUM = 100`, strictly more than 1% of FCS votes (`<=` reverts); holding time capped at 365 days by anyone calling `cap()` | source (Sourcify); chain `HOLDING_DURATION_CAP()` = 365 days |
| Bridge from FCS to FPS1 | FPS1 delegation chain FCS to InterestGovernance (`0x6f2e...bd93`) to MinterGovernance (`0x4655...e73c`) to CCIPGovernance (`0x3b5f...db52`) | an FCS action passes only if the FCS contract's FPS1 votes clear the 2% FPS1 quorum. FCS holds 35.07% of FPS1 votes (5,600 of 8,521 FPS), so it clears it; `isBinding()` (more than two thirds) is false | chain: `delegates()`, `relativeVotes(FCS)`, `balanceOf(FCS)`, `isBinding()` |

So two groups can act today: any FPS1 holder group with at least 2% of FPS1 votes acting directly, and any FCS group with more than 1% of FCS votes acting through the modules. Voting power is balance times holding duration in both (source: `Equity.votes`, `AccumulatingVotesToken`), delegation is non-subtractive and transitive, and votes can be destroyed by sacrificing one's own (`kamikaze` in FPS1, `attack` in FCS). While FCS is not binding, outside FPS1 holders cannot be "shot".

### Lever table

| Lever | Who | Speed | Bounds | Evidence |
|-------|-----|-------|--------|----------|
| Collateral list | Anyone, by opening a position with any ERC-20; any qualified holder can `deny` it | At least 3 days (`_initPeriod >= 3 days`, opener may choose longer); clones of an approved position are immediate | Token must revert on a failed transfer, at most 24 decimals, minimum collateral worth at least 5,000 ZCHF at the liquidation price, 1,000 ZCHF non-refundable fee; denial only before `start` | source (`MintingHubV2.openPosition`, `PositionV2.deny`) |
| Synth list (minters) | Anyone suggests a minter; any qualified holder can `denyMinter`. Via MinterGovernance, anyone can also deny a minter not announced there and earn 10% of its reward pool (450 ZCHF in the pool on chain) | Direct path on ZCHF: at least 14 days and 1,000 ZCHF (chain). Sanctioned path via `MinterGovernance.suggestMinter`: at least 60 days and 1,200 ZCHF (chain constants on `0x4655...e73c`). Denial is immediate | Approval is permanent: `denyMinter` reverts `TooLate` after the period, and no removal function exists. Every approved minter can mint without limit at the token level, burn from any holder, and receives an infinite allowance over every holder's ZCHF (`_allowance` returns `INFINITY` for minters) | chain (`minters()`), source (`Frankencoin.sol`, `MinterGovernance.sol`) |
| Risk parameters | Set per position by its opener (owner); changed only by that owner. Protocol constants (opening fee, 2% challenger reward, 10x expired price factor) have no setter | Owner lowers the liquidation price immediately if collateralised; raising it triggers a 3 day minting cooldown | Raised price bounded only by `price * minimumCollateral <= limit` in the deployed V2 (the 3x cap exists only in the undeployed top-level `Position.sol`); any challenge can test the price | source (`PositionV2._adjustPrice`, `MintingHubV2` constants) |
| Oracle | No one: no oracle in V1 or V2 minting, challenge or expiry logic | n/a | Exception: the approved Uniswap Amplifier minter (`0xa130...8c58`) reads the spot price (`slot0()`) of the ZCHF/USDT Uniswap V3 pool `0x8E43...8cEF` and only lets positions open within a band around its deployment anchor; limit 2.5M ZCHF, 368,172 borrowed, expires 2027-03-31 | source (grep of V1, V2 and Sourcify hub source; Sourcify amplifier source); chain (`LIMIT()`, `totalBorrowed()`, `EXPIRATION()`, `UNISWAP_POOL()`) |
| Fees and rates | Lead rates: any qualified holder calls `proposeChange` (FPS1 directly, FCS via `InterestGovernance`); anyone calls `applyChange`. Risk premium: fixed per position at opening (immutable) | 7 days from the latest proposal; a new proposal (for example re-proposing the current rate) overwrites and cancels the pending one | Rate is a `uint24` in ppm (up to about 1,670%); no other bound. Current: borrowing lead rate used by MintingHub V2 (`0x3BF3...B2aE`) 10,000 ppm = 1.00%; main savings module (`0x27d9...7B38`) 35,000 ppm = 3.50%; no pending change on either | chain (`currentRatePPM()`, `nextRatePPM()`, `MintingHubV2.rate()`), source (`Leadrate.sol`, deployed via Sourcify) |
| Pause | No global pause exists. Partial levers: a qualified holder can deny positions and minters still in their waiting period, and can set CCIP rate limits per chain (including to zero) through `CCIPAdmin.applyRateLimit` | Immediate | Rate limits affect only the bridge; a challenge suspends minting on one position. Chainlink's Risk Management Network (`getRmnProxy()` = `0x411d...5e81`) can halt CCIP lanes outside Frankencoin governance (not read further) | source (`CCIPAdmin.sol`, verified); chain (pool `owner()` = CCIPAdmin, `getRateLimitAdmin()` = zero) |
| Code upgrades | No one can upgrade deployed code. Change happens by adding new minters (new hubs, savings modules, bridges) through the minter path | 14 days direct, 60 days via MinterGovernance | Old minters stay approved for ever (MintingHub V1 from 2023 is still a minter). Bridge config (add or remove chains, remote pools) via CCIPAdmin: 7 day delay; CCIP admin transfer: 21 days; any qualified holder may `deny` a pending proposal | chain (`minters()`), source (`CCIPAdmin.propose(hash, 7 / 21)`) |
| Shutdown | No one: there is no wind-down or global settlement | n/a | Positions expire individually; stablecoin bridges expire at `horizon` (XCHF 2024-10-26, VCHF 2026-04-15, CHFAU 2027-04-08). If equity falls below 1,000 ZCHF, a qualified FPS1 holder may call `restructureCapTable` to burn other holders' FPS, letting recapitalisers own the reserve; FCS has no equivalent. Losses beyond the reserve are minted (`coverLoss`), not passed to ZCHF holders as a haircut | chain (`horizon()`), source (`Equity.restructureCapTable`, `Frankencoin.coverLoss`, `FCS.sol` comment) |

### Minter set on Ethereum (chain, `minters(address)` = approval time)

The [indexer API](https://api.frankencoin.com/ecosystem/minter/list) lists 17 mainnet applications; each was re-checked on chain.

| Minter | Address | Approved | Note |
|--------|---------|----------|------|
| MintingHub V1 | `0x7546762fdb1a6d9146b33960545C3f6394265219` | 2023-10-28 (genesis via `initialize`) | still a minter |
| XCHF bridge | `0x7bbe8F18040aF0032f4C2435E7a76db6F1E346DF` | 2023-10-28 (genesis) | horizon passed 2024-10-26 |
| Savings V2 (also the V2 borrowing lead rate) | `0x3BF301B0e2003E75A3e86AB82bD1EFF6A9dFB2aE` | 2024-12-11 | rate 1.00% |
| PositionRoller V2 | `0xAD0107D3Da540Fd54b1931735b65110C909ea6B6` | 2024-12-11 | |
| MintingHub V2 | `0xDe12B620A8a714476A97EfD14E6F7180Ca653557` | 2024-12-11 | |
| VCHF bridge | `0x3b71ba73299f925a837836160c3e1fec74340403` | 2025-05-12 | horizon passed 2026-04-15 |
| Savings (referral) | `0x27d9AD987BdE08a0d083ef7e0e4043C857A17B38` | 2025-06-05 | rate 3.50% |
| TransferReference | `0xf98c221661F51578f5E5236B189a493E2a8a1916` | 2025-07-07 | transfer utility; minter status gives it infinite allowance |
| CCIP BurnMint token pool | `0x9359cd75549DaE00Cdd8D22297BC9B13FbBe4B79` | 2025-07-08 | inbound rate limits 110k to 550k ZCHF per chain; outbound limiter disabled |
| BridgeAccounting | `0x88fd2ECD0B9250F203e99E80eb78b0C32B8AdB16` | 2025-07-08 | books L2 profits and losses on mainnet |
| CHFAU bridge (AllUnity) | `0x3e445ff4ddDf0ff8aE7458c9746eD80bD664F6C1` | 2026-05-14 | limit 10M, 348,649 minted, horizon 2027-04-08 |
| Uniswap Amplifier | `0xa1304e5aaf83cdb7c2b367f50b99bb0647ed8c58` | 2026-09-03 | applied 2026-08-20 on the 14 day path |

Denied applications include five on the API list, among them one on 2026-10-31 (14 day direct path, message "FCS"), consistent with the MinterGovernance bounty working as designed. PositionFactory V1 and V2, CloneHelper, CCIPAdmin and the FCS contracts are not minters (chain: `minters()` = 0). The API list may be incomplete; event logs could not be scanned because the RPC endpoint refuses historical `eth_getLogs` ("archive requests require a personal token").

### Resolved conflicts

- **14 days and 1,000 ZCHF versus 60 days and 1,200 ZCHF.** Both are correct, for different contracts. `Frankencoin.suggestMinter` on the ZCHF token (deployed 2023, immutable) enforces `MIN_APPLICATION_PERIOD` = 14 days and `MIN_FEE` = 1,000 ZCHF. `MinterGovernance.suggestMinter` (`0x4655...e73c`, part of the 2026 FCS deployment) wraps it with 60 days and 1,200 ZCHF (up to 200 ZCHF refills a bounty pool), and adds `denyUnannouncedMinter`, which anyone can call to veto a minter that skipped the wrapper. The 60 day rule is therefore enforced socially and economically (by the FCS votes held in FPS1 and the bounty), not by the token. The docs describe the wrapper. The repository's own `docs/fcs.md` contradicts itself (its table says 90 days, its text and the code say 60).
- **1% versus 2% veto threshold.** FPS1 (deployed) uses 2%; FCS uses more than 1%. The docs say the shipped code uses a single 1% threshold and that FCS and direct FPS actions share it; the deployed FPS source says otherwise for direct FPS1 actors, who still need 2%. FCS holders effectively face 1% of FCS votes plus the FCS contract's 35% share of FPS1 votes (well above 2%).
- **`Ownable.sol` in the audit scope.** It is the per-object owner used by `Position` (the borrower who opened it; clones are owned first by the hub, then by the cloner), `PositionRoller`, `CloneHelper`, `LiquiditySource` and the amplifier positions. MintingHub imports it but does not inherit it. It confers no protocol-wide power (source).

### Not checked

- Distribution of FPS and FCS votes across holders and delegates (who can reach 2% or 1% alone), and whether any single address is qualified today.
- Bridged deployments on the seven other chains (their own `BridgedFrankencoin` minters, governance sync freshness, CCIPAdmin `0xdE4c...484E`); only the mainnet side was read.
- Chainlink CCIP's own controls (RMN curse, router and TokenAdminRegistry ownership) beyond confirming that the ZCHF token admin in the registry is the Frankencoin CCIPAdmin.
- Live positions: number, collateral types, liquidation prices and risk premiums ([NOT FOUND] here).
- Bytecode was not compared against the repository; deployed behaviour is taken from Sourcify-verified source for ZCHF and FPS, MintingHub V2, MinterGovernance, MainnetVotes, CCIPAdmin, both savings modules and the amplifier.

## Sources

- [DefiLlama Frankencoin API](https://api.llama.fi/protocol/frankencoin) : accessed 2026-11-16 (TVL figure as recorded in [[patterns/cdp-synthetics-landscape]])
- [Frankencoin docs: governance](https://docs.frankencoin.com/governance) : accessed 2026-11-16
- [Frankencoin docs: opening a position](https://docs.frankencoin.com/positions/open.md) : accessed 2026-11-16 (via [[patterns/cdp-synthetics-landscape]])
- [FrankenCoin repository at commit 8b4c4ab](https://github.com/Frankencoin-ZCHF/FrankenCoin/tree/8b4c4ab67bb361b91d58c474b87f4608fc4c0566) : accessed 2026-11-16; files read: `contracts/stablecoin/Frankencoin.sol`, `contracts/equity/Equity.sol`, `contracts/equity/Governance.sol`, `contracts/equity/shares/{FCS,MinterGovernance,InterestGovernance,CCIPGovernance,GovernanceFactory,MainnetVotes,AccumulatingVotesToken}.sol`, `contracts/minting/v2/{MintingHubV2,PositionV2}.sol`, `contracts/rate/Leadrate.sol`, `contracts/savings/AbstractSavings.sol`, `contracts/bridge/CCIPAdmin.sol`, `contracts/equity/BridgeAccounting.sol`, `contracts/swap/{StablecoinBridge,UniswapAmplifier}.sol`, `contracts/utils/Ownable.sol`, `docs/fcs.md`, `exports/address.config.ts`
- [Sourcify verified source, ZCHF](https://sourcify.dev/server/v2/contract/1/0xB58E61C3098d85632Df34EecfB899A1Ed80921cB?fields=sources) and [FPS](https://sourcify.dev/server/v2/contract/1/0x1bA26788dfDe592fec8bcB0Eaff472a42BE341B2?fields=sources) : accessed 2026-11-16
- [Sourcify, MinterGovernance](https://sourcify.dev/server/v2/contract/1/0x4655529a4bf8c26a97c2b7f6c044b69a2f98e73c?fields=sources), [MainnetVotes](https://sourcify.dev/server/v2/contract/1/0x3bc9575c1E41322De656ef93eF5cAd9BE9e90DcD?fields=sources), [CCIPAdmin](https://sourcify.dev/server/v2/contract/1/0x2527ec458c863073a303CF0a362Bf78aDD5dFEf8?fields=sources), [MintingHub V2](https://sourcify.dev/server/v2/contract/1/0xDe12B620A8a714476A97EfD14E6F7180Ca653557?fields=sources), [Savings V2](https://sourcify.dev/server/v2/contract/1/0x3BF301B0e2003E75A3e86AB82bD1EFF6A9dFB2aE?fields=sources), [Savings referral](https://sourcify.dev/server/v2/contract/1/0x27d9AD987BdE08a0d083ef7e0e4043C857A17B38?fields=sources), [Uniswap Amplifier](https://sourcify.dev/server/v2/contract/1/0xa1304e5aaf83cdb7c2b367f50b99bb0647ed8c58?fields=sources) : accessed 2026-11-16
- [Frankencoin API: minter applications](https://api.frankencoin.com/ecosystem/minter/list) : accessed 2026-11-16
- [Frankencoin API: lead rates](https://api.frankencoin.com/savings/leadrate/info) and [savings modules](https://api.frankencoin.com/savings/core/info) : accessed 2026-11-16
- Ethereum mainnet `eth_call` through [ethereum-rpc.publicnode.com](https://ethereum-rpc.publicnode.com) : read 2026-11-16 (block about 26,426,700); selectors computed locally
