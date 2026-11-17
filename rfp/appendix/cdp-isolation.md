---
tags: [appendix, cdp, lez, risk-sharing, first-loss-pool, isolation, evidence]
related: [rfp/appendix/cdp-notion-map, rfp/appendix/cdp-admin-switch, patterns/cdp-vs-morpho-trust, projects/liquity, projects/indigo, projects/hylo, projects/frankencoin, projects/synthetix-v2, projects/monolith]
---

# CDP on LEZ: evidence on how a deployment shares risk

This note holds the evidence behind Part 4 ("How a deployment shares risk") of the Notion page "CDP-Backed Synthetics": how comparable protocols share losses among collaterals, synths, depositors and borrowers, and what precedent exists for each design. It holds no decisions; those are on the page. The filename stays `cdp-isolation` because other notes link to it. Terms (synth, branch, position, deployment, first-loss pool, cross-margin) are defined in [[rfp/appendix/cdp-notion-map]]. Background in [[patterns/cdp-vs-morpho-trust]]. Written 2026-11-16; fact-checked and reduced to evidence on 2026-11-17.

## How protocols differ: tokens per synth and width of the loss group

1. **Tokens per synth.** One token for the synth whatever the collateral: Liquity V2, Indigo, Hylo, Frankencoin and Synthetix V2. A separate token per collateral: Monolith only (one collateral per stablecoin), from its docs landing page ([[projects/monolith]]).
2. **Width of the group that carries a loss first.** One collateral: Liquity V2 (the pool and the redistribution stay on that collateral). One synth: Indigo. All collaterals of one synth: Hylo, Frankencoin. All synths: Synthetix V2, where every borrower carries the price moves of every synth.

Liquity V2 combines one token across collaterals with a narrow loss group (one collateral).

## How five protocols carry a loss first

Cell categories (as on the page) show behaviour that at least two protocols share; a cell with no category is behaviour only one protocol has, and the "What goes in, and what depositors earn" column has none.

- What one pool covers: **one pool per collateral**; **one pool per synth**; **one pool for all collaterals of a synth**.
- When the pool pays out: **swaps synth for collateral** of a liquidated position; **pays a shortfall** when backing falls short, with no such swap.
- Beyond the pool: **stays within the same collateral's positions**; **waits in the position** until the pool is refilled; **reaches every holder** of the synth; **not described** in the sources read.

| Protocol, its synths and its pool | What one pool covers | What goes in, and what depositors earn | When the pool pays out | Beyond the pool |
|---|---|---|---|---|
| **Liquity V2.** Synths: BOLD (a USD stablecoin). Pool: Stability Pool | **One pool per collateral** (WETH, wstETH, rETH); the synth stays one fungible token across collaterals | The synth, deposited by anyone. Depositors earn the collateral of liquidated positions (at about a 5% discount) and 75% of the interest paid by positions on that collateral | **Swaps synth for collateral:** a position on that collateral is liquidated, the pool burns synth to repay the debt, and the depositors receive the position's collateral | **Stays within the same collateral's positions:** the part of the debt the pool cannot absorb, with the collateral against it, is spread across all other open positions on the same collateral, in proportion to their collateral (proportionality from the vault note, not re-read) |
| **Indigo.** Synths: iUSD, iBTC, iETH, with iJPY and iEUR being introduced in V3 (docs); iSOL appears in a 2025 tokenomics proposal (status unverified). Pool: Stability Pool | **One pool per synth** ("Each iAsset, such as iBTC, has its dedicated Stability Pool", docs) | The synth, deposited by any holder. Depositors earn the collateral of liquidated positions, 40% of interest (V3), rewards in the protocol token (INDY) and redistributed fees (docs) | **Swaps synth for collateral:** a position is liquidated, the pool burns synth to repay the debt, and the depositors receive the position's collateral | **Waits in the position:** the part of the debt the pool cannot absorb stays frozen in the position, with its collateral, until more synth is deposited and the liquidation is run again. Nothing is spread to other positions (whitepaper, V2 era; V3 not confirmed) |
| **Hylo.** Synths: hyUSD (a USD stablecoin). Pool: Earn Pool. Leveraged tokens: xSOL, xBTC, xHYPE, which own whatever collateral value is above what backs hyUSD, so they take all price falls first | **One pool for all collaterals of a synth** (SOL liquid staking tokens, BTC, HYPE, USDC) | The synth, staked by holders. Depositors earn a share of the protocol's yield (staking rewards and borrow rates) | **Pays a shortfall, no swap:** no liquidation. Holders of the leveraged tokens take price falls first. When the backing of a collateral falls below 100%, the pool's synth is burned to retire the unbacked supply, capped at the pool's balance; the pool also funds subsidised rebalances earlier | **Not described** in the sources read |
| **Frankencoin.** Synths: ZCHF (a Swiss franc stablecoin). Pool: FPS reserve. Pool shares: FPS | **One pool for all collaterals of a synth:** every position mints the same synth | The synth: anyone can deposit it to mint pool shares (FPS). The shareholders own the pool's equity and hold the votes that can veto new positions and minters | **Pays a shortfall, no swap:** no liquidation by the pool. A loss is paid from the pool: first the shareholders' equity, then the part contributed by borrowers | **Reaches every holder:** once the pool is empty, new synth is minted without backing, so the loss dilutes every holder (no haircut or redemption) |
| **Synthetix V2.** Synths: sUSD, sETH, sBTC and others. Pool: the global debt pool, which has no opt-in deposits. Collateral: SNX only; sUSD is the settlement unit | One debt pool for all synths and all borrowers (stakers) | No deposits: borrowers lock SNX and mint sUSD. They earn exchange fees in proportion to their share of the debt pool | No pool pays out. Each borrower's debt moves with the total value of all synths, so trader profits are paid by the borrowers | No separate pool: the borrowers carry everything |

Sources: the vault's notes on Liquity, Hylo, Frankencoin and Synthetix V2; Indigo's docs (`llms-full.txt`, accessed 2026-11-17) and whitepaper v2.0.0, section 2.6 (accessed 2026-11-17); Hylo's risk management docs (accessed 2026-11-17). The evidence behind each row is under "Protocol evidence" below.

In every design surveyed, a first-loss pool limits shared losses but does not remove them: a loss larger than the pool reaches every holder of the synth, or the peg.

## One shared debt pool: Synthetix V2

Synthetix V2 is the one multi-synth protocol in the survey that shares one pool across its synths. It is not a first-loss pool that depositors opt into, and it is now legacy (solo staking ended in April 2025). Borrowers (Synthetix calls them stakers) lock one collateral, SNX, and mint sUSD against it. Their debt is not a fixed amount of sUSD: each borrower owes a share of one global debt pool, which equals the total USD value of every synth in circulation. If sETH rises against the dollar, the global debt rises and every borrower's debt rises in proportion to their share; if it falls, their debt falls. So the borrowers are together the counterparty to everyone holding any synth, and they take the price moves of all synths at once. That is why swaps between synths cost no slippage; the cost is that trader profits are paid by the borrowers. Source: [[projects/synthetix-v2]].

## Protocol evidence

Tags: **chain** = read from Ethereum mainnet; **source** = verified contract source (Sourcify); **docs** = project documentation; **vault** = another note here; **agent** = reported by a research agent and not independently verified.

### Liquity V2: one token, one Stability Pool per collateral

Detail and numbers in [[projects/liquity]], section "Stability Pool and redistribution". Liquidation capacity is the BOLD deposited in the branch's pool; the debt the pool cannot absorb, with the collateral against it, goes to all other active positions of the same branch through per-unit-of-stake accumulators; BOLD stays one token across the three branches. On 2026-11-17 the WETH branch pool held 7,812,940.57 BOLD and 21.83 WETH of unclaimed collateral gains (chain, block 26,432,207; repeated by the fact-check at block 26,432,666). The wstETH and rETH branches have their own Stability Pools (`0x9502…e56b` and `0xd442…8695`, fact-check read 2026-11-17).

### Indigo: a pool per synth, and what happens when it is too small

Detail in [[projects/indigo]]. The whitepaper (v2.0.0 draft, March 2024, section 2.6) says: "If a CDP debt is higher than the entire amount of iAssets in the related SP, the protocol attempts to cancel as much debt as possible with the iAsset supply available. Any remaining non-liquidated collateral and debt of the CDP remains frozen until more iAsset is deposited into the SP and another liquidation is initiated." No redistribution to other positions, no auction and no reserve are described. The V3 docs read are silent on this case; V3 contracts were not read. Bad debt is not mentioned in any docs page read.

Synths over time (docs, `llms-full.txt`, 2026-11-17): "Upon launch, iBTC and iUSD were available for minting … iETH followed after DAO approval"; "With Indigo V3 and the Indigo Limitless rollout, additional iAssets such as iJPY and iEUR are being introduced through DAO governance, alongside expanded collateral markets and PSM pools." The Limitless tranches also add collateral to existing synths (for example "NIGHT as collateral for iUSD"). The forum's "Proposal: Update of Indigo Tokenomics" (2025-01-16, [thread 14298](https://forum.indigoprotocol.io/t/proposal-update-of-indigo-tokenomics/14298)) lists iSOL allocations; iSOL is not in the current docs and its status is unverified.

### Hylo: a leveraged token takes the first loss, then the Earn Pool

Each Hylo collateral pool is split into a leveraged token (xSOL, xBTC, xHYPE) that owns whatever collateral value exceeds what backs hyUSD, plus a virtual dollar unit (vUSD) worth $1 ([[projects/hylo]]). Worked example (illustrative, not from the docs): a pool holding 150 USD of collateral that backs 100 hyUSD leaves 50 for the leveraged token, so a 10% fall in the collateral cuts it to 35 (30% down, 3x leverage), and at 100% backing it is worth nothing; then Earn Pool hyUSD is burned to absorb the loss. The Earn Pool can also be drawn earlier: during rebalancing, "subsidized ones burn hyUSD from it"; "The burn is capped at the Earn Pool's balance, and the amount retired is recorded as debt on a pool drawdown ledger", repaid first from future yield ([Hylo docs](https://docs.hylo.so/technical-addendum/additional-risk-management.md), accessed 2026-11-17). The Earn Pool is shared by all pools (vault note), which is why the page puts Hylo in "one pool for all collaterals of a synth".

### Frankencoin: the FPS reserve and dilution

Chain, 2026-11-16 ([[projects/frankencoin]]): the FPS contract held 11,070,613 ZCHF, of which equity 3,553,638 (owned by FPS holders) and the borrowers' minter reserve 7,516,975; ZCHF supply on Ethereum 27,771,913. A fact-check read on 2026-11-17 reported 11,211,215 held, equity 3,694,239, supply 28,261,511 and the reserve unchanged (not repeated). A loss is paid first from the equity, then from the borrowers' reserve (which shrinks pro rata); once the reserve contract is empty, `Frankencoin.coverLoss` mints new unbacked ZCHF (source). There is no haircut or redemption; the loss becomes dilution of all ZCHF. FPS also carries the votes that can veto positions and minters.

### Synthetix V2: the shared debt pool

[[projects/synthetix-v2]]: debt is a share of the global pool. Stakers buying sUSD to repay debt is a source of pressure on the peg, not support: SCCP-232 lowered the target ratio partly to "prevent some pressure on the sUSD peg which would be caused by stakers buying additional sUSD on the market to pay their debt" ([SCCP-232](https://sips.synthetix.io/sccp/sccp-232/), accessed 2026-11-17). Not a first-loss pool.

### Alchemix: one Alchemist per alAsset

Docs, accessed 2026-11-17: "Each alAsset on each chain is managed by a dedicated Alchemist" ([Alchemist page](https://alchemix-finance.gitbook.io/user-docs/alchemix-ecosystem/alchemist.md)). From `llms-full.txt` (271,365 characters; quotes found verbatim on 2026-11-17, see [[projects/alchemix]]): "Alchemists can have multiple yield strategies but can only issue one type of alAsset." The alUSD Alchemist "issues alUSD but can accept yield-bearing tokens for any approved stablecoin"; "alETH can be borrowed against ETH-denominated yield token deposits." These are V2 docs; V3 was not checked. Separate Transmuters and per-Alchemist debt accounting were not addressed in the part read. See [[projects/alchemix]].

### Monolith: a token per market

Docs and chain, 2026-11-17. Detail in [[projects/monolith]]. It is the only example found of one collateral per stablecoin (a separate token per branch), from its docs landing page ("Monolith allows anyone to deploy their own decentralized crypto-backed stablecoin" by "providing a collateral token address and a price feed"). Instances on one factory share at least the factory's `InterestModel` (fact-check of [[projects/monolith]], 2026-11-17).

### Metronome: cross-margin and the swap pool

[[projects/metronome-synth]]: one position can hold several collaterals and mint several synths; its swap module mints and burns synths at oracle price against a shared pool. On 2026-07-30 the DAO disclosed about 6,367 msETH and 4.57M msUSD created without adequate collateral (about $15.7M to $16M); the cause was stale Chainlink ETH/USD prices on Base at the swap module, not repays, liquidations or cross-margin as such ([[patterns/cdp-synthetics-landscape]]; Crypto Briefing, 2026-07-31).

### Youves: an engine per synth and collateral pair

[[projects/youves]]: each synthetic and collateral pair has its own v3 engine contract with its own parameters, but all tracker tokens live in one FA2 contract (`KT1XRPEPXbZK25r3Htzp2o1x7xdMMmfocKNW`; token ids 0 uUSD, 1 uDEFI, 2 uBTC, 3 uXTZ, 4 uXAU; TzKT, read 2026-11-17), so this is isolation of accounting, not separate tokens.

### Maker: surplus buffer and debt auctions

Maker's last step before holders was debt auctions that mint MKR (Black Thursday: 20,980 MKR sold for 5.3M DAI, [[projects/sky-maker]]). They are disabled today: `MCD_VOW.sump()` (`0xA950524441892A31ebddF91d3cEEFa04Bf454466`) is 2^256 − 1, so `flop()` cannot run (chain, read 2026-11-17).

## Precedent findings

From a precedent audit run in the Notion session on 2026-11-17 by a research agent (vault notes; Aave V3, Morpho Blue and Vault V2, Alchemix, Lido, Balancer, OpenZeppelin, Indigo, Monolith and RedStone docs), corrected by the 2026-11-17 fact-check and the re-reads above.

- **One synth per deployment:** found. Alchemix runs one Alchemist per alAsset, with no collateral shared between them (docs). Liquity V1 and V2, Frankencoin and crvUSD are single-synth.
- **An opt-in first-loss pool shared across several synths:** not found. Indigo keeps a pool per synth; Synthetix V2 shares one debt pool, carried by every staker rather than opt-in depositors.
- **Adding a synth with its own pool to a live deployment:** found in Indigo (iETH after launch by DAO approval; iJPY and iEUR in V3; each iAsset has its own Stability Pool).
- **Adding a collateral to a live deployment with its own pool:** not found. Indigo's new collaterals join an existing synth and its pool. Liquity V2 has a pool and shutdown per branch but cannot add branches. Frankencoin lets anyone propose a collateral, and only the holders' veto and the FPS reserve stand between it and every holder.
- **A low starting ceiling for new collateral:** Sky's AutoLine ramps ceilings (gap 150M above current debt, at most once per 6 hours for ETH-A); crvUSD has two markets at a zero ceiling ([[projects/crvusd]]).
- **Loss path steps:** each step exists somewhere (liquidator; Liquity's pool then redistribution; Frankencoin's equity, then reserve, then dilution; Sky's surplus buffer; Maker's now disabled debt auctions). No protocol has all of them in sequence.
- **Separate token per branch:** Monolith only (docs landing page). Youves separates engines, not tokens. Liquity V2 has a branch per collateral but one token.
- **Per-branch risk controller:** found in lending only (Euler V2 per-vault governors, Morpho curators), not in minting protocols.
- **Per-branch shutdown:** found in Liquity V2 ([[rfp/appendix/cdp-admin-switch]], Shutdown row).
- **Cross-margin in a minting protocol:** found once, Metronome; its 2026 loss came from oracle lag at the swap module.
- **Shared oracle legs failing together:** JPY DeFi's docs warn that JPY/USD is shared by all branches ([[patterns/collateral-debt-units]]); not checked per protocol.

## Gaps

- **Indigo V3.** The shortfall rule is from the V2-era whitepaper; the V3 docs are silent, and V3 contracts were not read ([[projects/indigo]]). iSOL's status is unverified.
- **Liquity V2 other branches.** The Stability Pool balance read covers the WETH branch; wstETH and rETH are assumed to run the same code.
- **Alchemix.** V2 docs only; V3 not checked.
- **Monolith.** The "one collateral per stablecoin" claim is from the docs landing page and the factory's inputs; no source says what happens if two instances use the same collateral.
- **Hylo.** The docs read do not say what happens once the Earn Pool is exhausted; "one shared Earn Pool" rests on the vault note.
- The page's Synthetix V2, Frankencoin and Hylo cells are taken from the project notes.

## Sources

- [Indigo whitepaper v2.0.0 (March 2024 draft)](https://indigoprotocol.io/paper.pdf), section 2.6 : accessed 2026-11-17
- [Indigo docs, llms-full.txt](https://docs.indigoprotocol.io/llms-full.txt) and [Stability Pools and liquidations](https://docs.indigoprotocol.io/readme/stability-pools-and-liquidations.md) : accessed 2026-11-17
- [Indigo forum: Proposal: Update of Indigo Tokenomics](https://forum.indigoprotocol.io/t/proposal-update-of-indigo-tokenomics/14298) (2025-01-16) : accessed 2026-11-17
- [Hylo docs: additional risk management](https://docs.hylo.so/technical-addendum/additional-risk-management.md) : accessed 2026-11-17
- [Alchemix docs: Alchemist](https://alchemix-finance.gitbook.io/user-docs/alchemix-ecosystem/alchemist.md) and [llms-full.txt](https://alchemix-finance.gitbook.io/user-docs/llms-full.txt) : accessed 2026-11-17
- [Monolith docs](https://docs.monolith.market/) : accessed 2026-11-17
- [SCCP-232](https://sips.synthetix.io/sccp/sccp-232/) : accessed 2026-11-17
- [TzKT tokens API, Youves FA2 contract](https://api.tzkt.io/v1/tokens?contract=KT1XRPEPXbZK25r3Htzp2o1x7xdMMmfocKNW) : accessed 2026-11-17
- Ethereum mainnet JSON-RPC (publicnode): Liquity V2 WETH Stability Pool `0x5721cbbd64fc7ae3ef44a0a3f9a790a9264cf9bf` `getTotalBoldDeposits()` and `getCollBalance()` (block 26,432,207); `MCD_VOW.sump()`; read 2026-11-17
- Vault notes: [[projects/liquity]], [[projects/indigo]], [[projects/hylo]], [[projects/frankencoin]], [[projects/synthetix-v2]], [[projects/alchemix]], [[projects/monolith]], [[projects/metronome-synth]], [[projects/youves]], [[projects/crvusd]], [[projects/sky-maker]]
