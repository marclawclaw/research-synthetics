---
tags: [appendix, cdp, lez, risk-sharing, first-loss-pool, isolation]
related: [rfp/appendix/cdp-notion-map, rfp/appendix/cdp-admin-switch, patterns/cdp-vs-morpho-trust, projects/liquity, projects/indigo, projects/hylo, projects/frankencoin, projects/synthetix-v2, projects/monolith]
---

# CDP on LEZ: how a deployment shares risk

Working framing for the synth CDP software on LEZ (sXMR and sBTC are the examples). It is mirrored in Part 4 of the Notion page "CDP-Backed Synthetics" and sits alongside [[rfp/appendix/cdp-admin-switch]] (Part 1). The Notion page renamed this part from "isolation" to "how a deployment shares risk"; the filename stays `cdp-isolation` because other notes link to it. The background is in [[patterns/cdp-vs-morpho-trust]]. Written 2026-11-16; restructured 2026-11-17 to match the page, with protocol evidence, a precedent check and conflicts added. A formal fact-check pass has not been run on this note.

## Part map

Which vault file backs which part of the Notion page (full map and terms in [[rfp/appendix/cdp-notion-map]]):

| Notion part | Vault file |
|---|---|
| 1 The rules, and the admin switch proposal | [[rfp/appendix/cdp-admin-switch]] |
| 2 Units and oracle requirements | [[patterns/collateral-debt-units]] |
| 3 Peg mechanisms | [[patterns/cdp-peg-mechanisms]] |
| 4 How a deployment shares risk | this note, with [[patterns/cdp-vs-morpho-trust]], [[projects/indigo]], [[projects/monolith]], [[projects/liquity]] |
| 5 Who carries the risk, 7 Privacy, 8 Market | [[rfp/appendix/cdp-risk-privacy-market]] |
| 6 Oracle | [[rfp/appendix/cdp-oracle]] |

## Terms

- **Deployer:** someone who takes the output of the RFP and decides to run it. They may or may not be competing for a Lambda Prize. They may build their own software or modify the delivered software.
- **Synth:** a token the CDP mints against collateral, such as sXMR or sBTC.
- **Synth stablecoin:** a synth that tracks a stable unit, such as a USD synth. The CDP mints it.
- **Collateral stablecoin:** a stablecoin that a borrower deposits as collateral. This CDP does not mint it. The reflexive stablecoin (RFP-013) is a candidate.
- **LGO:** Logos Native Gas token.
- **First-loss pool:** a pool of the synth, filled by opt-in depositors, that takes losses before other holders do. Liquity and Indigo call theirs a Stability Pool, Hylo an Earn Pool, and Frankencoin the FPS reserve.
- **Position:** what a borrower opens by locking collateral and minting synth against it in one transaction. It records that collateral and that debt; adding collateral or minting more later changes the same position, and a borrower can open several. Liquity calls it a Trove.
- **Deployment:** one running instance of the software, with its own tokens, state and roles.
- **Branch:** one collateral paired with one synth, with its own ratio, feed and debt ceiling, and optionally its own first-loss pool and shutdown.
- **Debt ceiling:** the most synth a branch can mint.
- **Role and mode:** a role is a group of the Part 1 levers that one controller can change; its mode (Immutable, Separate governance or Program gated) says how that controller works.
- **Reserve:** value the protocol holds itself, such as Sky's surplus buffer, rather than opt-in deposits.
- **Cross-margin:** one position that holds several collaterals or synths.

Highlight words (the Notion page's colours, written in words): "Recommendation: ..." (green), "Open decision: ..." (yellow, a decision with no recommendation yet) and "Risk to evaluate: ..." (red).

## Framing

We provide software through the RFP, not a product or a factory. There is no central party, and any deployer can modify the software before deploying. A deployer deploys the software and makes some decisions: some we fix in the software, and the deployer picks the rest. How deployers compete for users is out of scope. Deployments are assumed to share nothing: each has its own tokens and state, so deployment A's sXMR and deployment B's sXMR are different tokens. This part is about how one deployment shares risk among its collaterals, synths, depositors and borrowers: who carries a loss when a position fails, and what happens when others join and propose new collateral or synths. The table below lists what the software fixes and what the deployer picks.

## What the software fixes and what the deployer picks

| Angle | Fixed in the software | Picked by the deployer |
|-------|-----------------------|------------------------|
| Tokens per synth | Whether a synth can have a separate token per branch, or only one token across collaterals | Which of those the deployment uses |
| Loss group | How wide the group that carries a loss first can be (one collateral, one synth, all collaterals of a synth, all synths) | How wide it is in this deployment |
| Synths | Whether several synths per deployment are supported, and what they may share | How many synths, and whether they share a first-loss pool |
| Collaterals | Whether branches behind one token are supported | Which branches, and their ratios, feeds and ceilings (later changes: the Listing and Risk roles) |
| Adding collateral or synths after deployment | Whether a new branch can have its own first-loss pool, ceiling and shutdown | Whether new branches use that isolation; who may propose them (Listing role) |
| Positions | One branch per position, or cross-margin | Nothing, unless the software makes cross-margin optional |
| Loss path | Which steps exist and in what order | Pool sizes, and whether a reserve is used |
| Control | The level at which roles can be set | Controller and mode at each level |
| Failure | How narrow a shutdown can be | When to trigger it, through the Shutdown role |

## What changes once a deployment has users

At deployment the deployer picks a model, just as they pick a mode for each admin role ([[rfp/appendix/cdp-admin-switch]]). Then the deployment grows, and the people exposed to it are no longer only the deployer's choice:

- Holders and borrowers join because of the collateral and synths the deployment starts with.
- Others may propose new collateral or a new synth. Who may add them is the Listing role.
- **Risk to evaluate:** every addition exposes the holders already in the deployment to an asset they did not choose. The risk-sharing model decides how far a bad addition reaches.

The software's job is to make the more isolating models possible. The deployer picks the model, and later additions are the Listing role's job.

## Two questions that define the risk-sharing model

A deployment's risk-sharing model is its answer to two questions.

1. **How many tokens does a synth have?** One token for the synth whatever the collateral (Liquity, Indigo, Hylo, Frankencoin and Synthetix V2 all do this), or a separate token for each branch. With separate tokens, sXMR backed by LGO and sXMR backed by the reflexive stablecoin (RFP-013) would be two different tokens, so each holder takes on one collateral's risk only. The cost is fragmented liquidity: several tokens for the same synth. Separate tokens are seen only in Monolith (one collateral per stablecoin), from its docs landing page ([[projects/monolith]]).
2. **How wide is the group that carries a loss first?** One collateral (Liquity V2: the pool and the redistribution stay on that collateral), one synth (Indigo), all collaterals of one synth (Hylo, Frankencoin), or all synths (Synthetix V2, where every borrower carries the price moves of every synth). The table below compares them.

Liquity V2 is one token across collaterals with a narrow loss group (one collateral).

## How five protocols carry a loss first

Colour key (the page's cell colours, described in words). Cell categories show behaviour that at least two protocols share. A cell with no category is behaviour only one protocol has, and the "What goes in, and what depositors earn" column has no category.

- What one pool covers:
  - **One pool per collateral** (blue).
  - **One pool per synth** (purple).
  - **One pool for all collaterals of a synth** (orange).
- When the pool pays out:
  - **Swaps synth for collateral** of a liquidated position (pink).
  - **Pays a shortfall** when backing falls short, with no such swap (brown).
- Beyond the pool:
  - **Stays within the same collateral's positions:** the loss stays within the positions on the same collateral (green).
  - **Waits in the position:** the shortfall waits in the same position until the pool is refilled (yellow).
  - **Reaches every holder:** the loss reaches every holder of the synth (red).
  - **Not described** in the sources read (gray).

| Protocol, its synths and its pool | What one pool covers | What goes in, and what depositors earn | When the pool pays out | Beyond the pool |
|---|---|---|---|---|
| **Liquity V2.** Synths: BOLD (a USD stablecoin). Pool: Stability Pool | **One pool per collateral** (WETH, wstETH, rETH); the synth stays one fungible token across collaterals | The synth, deposited by anyone. Depositors earn the collateral of liquidated positions (at about a 5% discount) and 75% of the interest paid by positions on that collateral | **Swaps synth for collateral:** a position on that collateral is liquidated, the pool burns synth to repay the debt, and the depositors receive the position's collateral | **Stays within the same collateral's positions:** the part of the debt the pool cannot absorb, with the collateral against it, is spread across all other open positions on the same collateral, in proportion to their collateral |
| **Indigo.** Synths: iUSD, iBTC, iETH, iSOL. Pool: Stability Pool | **One pool per synth** | The synth, deposited by any holder. Depositors earn the collateral of liquidated positions, 40% of interest (V3), rewards in the protocol token (INDY) and redistributed fees (docs) | **Swaps synth for collateral:** a position is liquidated, the pool burns synth to repay the debt, and the depositors receive the position's collateral | **Waits in the position:** the part of the debt the pool cannot absorb stays frozen in the position, with its collateral, until more synth is deposited and the liquidation is run again. Nothing is spread to other positions (whitepaper, V2 era; V3 not confirmed) |
| **Hylo.** Synths: hyUSD (a USD stablecoin). Pool: Earn Pool. Leveraged tokens: xSOL, xBTC, xHYPE, which own whatever collateral value is above what backs hyUSD, so they take all price falls first | **One pool for all collaterals of a synth** (SOL liquid staking tokens, BTC, HYPE, USDC) | The synth, staked by holders. Depositors earn a share of the protocol's yield (staking rewards and borrow rates) | **Pays a shortfall, no swap:** no liquidation. Holders of the leveraged tokens take price falls first. When the backing of any one collateral falls below 100%, the pool's synth is burned to retire the unbacked supply | **Not described** in the sources read |
| **Frankencoin.** Synths: ZCHF (a Swiss franc stablecoin). Pool: FPS reserve. Pool shares: FPS | **One pool for all collaterals of a synth:** every position mints the same synth | The synth: anyone can deposit it to mint pool shares (FPS). The shareholders own the pool's equity and hold the votes that can veto new positions and minters | **Pays a shortfall, no swap:** no liquidation by the pool. A loss is paid from the pool: first the shareholders' equity, then the part contributed by borrowers | **Reaches every holder:** once the pool is empty, new synth is minted without backing, so the loss dilutes every holder (no haircut or redemption) |
| **Synthetix V2.** Synths: sUSD, sETH, sBTC and others. Pool: the global debt pool, which has no opt-in deposits. Collateral: SNX only; sUSD is the settlement unit | One debt pool for all synths and all borrowers (stakers) | No deposits: borrowers lock SNX and mint sUSD. They earn exchange fees in proportion to their share of the debt pool | No pool pays out. Each borrower's debt moves with the total value of all synths, so trader profits are paid by the borrowers | No separate pool: the borrowers carry everything |

Sources, as on the page: the vault's notes on Liquity, Hylo, Frankencoin and Synthetix V2; Indigo's docs (accessed 2026-11-16) and whitepaper v2.0.0, section 2.6 (accessed 2026-11-17). The evidence behind each row is under "Protocol evidence" below.

**Risk to evaluate:** a first-loss pool limits shared losses but does not remove them: a loss larger than the pool still reaches every holder and the peg.

## One shared debt pool: Synthetix V2

Synthetix V2 is the one multi-synth protocol in the survey that shares one pool across its synths. It is not a first-loss pool that depositors opt into, and it is now legacy (solo staking ended in April 2025). Borrowers (Synthetix calls them stakers) lock one collateral, SNX, and mint sUSD against it. Their debt is not a fixed amount of sUSD: each borrower owes a share of one global debt pool, which equals the total USD value of every synth in circulation (sUSD, sETH, sBTC and the others). If sETH rises against the dollar, the global debt rises and every borrower's debt rises in proportion to their share; if it falls, their debt falls. So the borrowers are together the counterparty to everyone holding any synth, and they take the price moves of all synths at once. That is why swaps between synths cost no slippage; the cost is that trader profits are paid by the borrowers. Source: [[projects/synthetix-v2]].

## Isolation

Isolation is how far a loss travels, and it follows from the two answers above. Separate tokens stop a loss at one branch. A first-loss pool stops it at the pool's size, then it reaches every holder of that synth. With one debt pool for all synths it is not contained: every borrower carries every synth. Isolation between deployments is assumed, as above.

## The angles

1. **Synths within a deployment.** A deployment can issue one synth or several. If several, what joins them: a shared first-loss pool, a fee sink, one Guardian, one Shutdown? Indigo keeps a pool per synth; Synthetix V2 shared one debt pool across all of them (every staker, not opt-in depositors, carries every synth's price moves). No surveyed protocol shares an opt-in first-loss pool across several synths. The alternative is that a deployment is one synth. Alchemix works this way: each alAsset (alUSD, alETH) has its own Alchemist, which issues only that alAsset, and collateral is not shared between them (docs). Liquity V1 and V2, Frankencoin and crvUSD are also single-synth, though none of them is a multi-synth product. The justification (inference): nothing is shared between synths, so a failure in sXMR cannot reach sBTC holders, and each deployment has one set of roles (Part 1). The cost: each synth needs its own collateral and first-loss pool, and positions cannot be cross-margined between synths. A multi-synth product would then be several deployments run by the same controllers.
2. **Collaterals per synth.** One collateral, or several branches behind one fungible token (Liquity V2). Each branch has its own ratio, feed, debt ceiling, first-loss pool and shutdown.
3. **Adding collateral or a synth to a live deployment.** Who may propose it is the Listing role (Part 1); the risk-sharing model decides the stakes. A new branch can carry its own first-loss pool, a debt ceiling that starts low and its own shutdown, or it can share the deployment's first-loss pool. Frankencoin shows the shared end: anyone can propose a collateral, every position mints the same ZCHF, and only the holders' veto and the FPS reserve stand between a bad collateral and every holder. Liquity V2 shows the opposite: no branch can be added after deployment, and each has its own Stability Pool. Both are seen only in parts: Sky's AutoLine ramps a debt ceiling up gradually, but no protocol adds a collateral to a live deployment with its own pool, and both examples cover adding collateral, not adding a synth.
4. **Positions.** A position holds one collateral and one synth (Liquity, crvUSD, Frankencoin), or works as cross-margin across several (Metronome, whose shared swap pool led to the $15.7M shortfall).
5. **Loss path.** Who pays, in what order, when a position goes underwater:
   1. the liquidator;
   2. the branch's first-loss pool;
   3. redistribution to the other borrowers in the branch (Liquity);
   4. a deployment-wide first-loss pool or reserve (Frankencoin's FPS reserve, Sky's surplus buffer);
   5. finally the holders, with the synth trading below its price or taking a haircut at shutdown.

   The software decides which of these steps exist; the deployer sizes them or switches them on. This order is a proposal assembled from Liquity, Frankencoin and Sky; no protocol has all five steps. Maker adds debt auctions that mint MKR before the holders.
6. **Control.** At what level each admin role from Part 1 is set: once per deployment, per synth or per branch. A Risk role per branch, held by a different controller, would make a deployment a Morpho-style curator layer. Per-market governors exist in lending (Morpho curators, Euler V2), not in minting protocols.
7. **Failure.** How narrow a shutdown can be: one branch (Liquity V2), one synth or the whole deployment. Shared oracle legs fail together whatever the shape: an LGO/USD outage hits every branch that reads it.

## Questions

1. **Open decision: how many tokens per synth does the software support?** One token across collaterals only, or also a separate token for each branch.
2. **Open decision: how wide can the group that carries a loss first be?** One collateral, one synth, all collaterals of a synth or all synths. Is it fixed by the software, or chosen by the deployer?
3. **Open decision: what is the smallest unit the software supports?** For example a (collateral, synth) branch: LGO to sXMR, LGO to sBTC, the reflexive stablecoin (RFP-013) to sXMR. Supporting the narrowest unit lets a deployer choose separate tokens; grouping branches is then their choice (inference).
4. **Open decision: one synth per deployment, or several in one?** If several, what can they share?
5. **Open decision: which loss steps does the code provide?** When a branch's losses exceed its first-loss pool, who pays: every holder of that synth, a shared reserve, or nobody, in which case the synth trades below its price?
6. **Open decision: at what level can each role be set:** deployment, synth or branch?
7. **Open decision: can one branch shut down without touching the others?** Liquity V2 does this per branch (Part 1, Shutdown row).
8. **Open decision: cross-margin:** supported, optional or never? It saves capital but joins risks that the branches were meant to keep apart (Metronome).
9. **Open decision: when a new collateral or synth is added to a live deployment, can it be isolated from the holders already in?** That means its own first-loss pool, ceiling and shutdown. Is that the default, or optional?
10. **Open decision: does a new branch start with a debt ceiling of zero, raised later by the Risk role, or with a value the deployer sets?**

## Protocol evidence

Tags: **chain** = read from Ethereum mainnet; **source** = verified contract source (Sourcify); **docs** = project documentation; **vault** = another note here; **agent** = reported by a research agent and not independently verified. Evidence gathered in the Notion session on 2026-11-16 and 2026-11-17.

### Liquity V2: one token, one Stability Pool per collateral

Detail and numbers (WETH branch only) are in [[projects/liquity]], section "Stability Pool and redistribution". In short: liquidation capacity is the BOLD deposited in the branch's pool; the debt the pool cannot absorb, with the collateral against it, goes to all other active positions of the same branch through per-unit-of-stake accumulators; BOLD stays one token across the three branches. On 2026-11-17 the WETH branch pool held 7,812,940.57 BOLD and 21.83 WETH of unclaimed collateral gains (chain, re-read in this pass).

### Indigo: a pool per synth, and what happens when it is too small

Detail in [[projects/indigo]]. The whitepaper (v2.0.0 draft, March 2024, section 2.6; re-read in this pass from the saved text) says: "If a CDP debt is higher than the entire amount of iAssets in the related SP, the protocol attempts to cancel as much debt as possible with the iAsset supply available. Any remaining non-liquidated collateral and debt of the CDP remains frozen until more iAsset is deposited into the SP and another liquidation is initiated." No redistribution to other positions, no auction and no reserve are described. The V3 docs pages read are silent on this case; V3 contracts were not read (no local clone). Bad debt is not mentioned in any docs page read.

### Hylo: a leveraged token takes the first loss, then the Earn Pool

Each Hylo collateral pool is split into a leveraged token (xSOL, xBTC, xHYPE) that owns whatever collateral value exceeds what backs hyUSD, plus a virtual dollar unit (vUSD) worth $1 ([[projects/hylo]]). Worked example (illustrative, not from the docs): a pool holding 150 USD of collateral that backs 100 hyUSD leaves 50 for the leveraged token, so a 10% fall in the collateral cuts it to 35 (30% down, 3x leverage), and at 100% backing it is worth nothing; only then is staked hyUSD in the Earn Pool burned. The Earn Pool is shared by all pools, which is why the page puts Hylo in "one pool for all collaterals of a synth".

### Frankencoin: the FPS reserve and dilution

Chain, 2026-11-16 ([[projects/frankencoin]]): the FPS contract held 11,070,613 ZCHF, of which equity 3,553,638 (owned by FPS holders) and the borrowers' minter reserve 7,516,975; ZCHF supply on Ethereum 27,771,913. A loss is paid first from the equity, then from the borrowers' reserve (which shrinks pro rata); once the reserve contract is empty, `Frankencoin.coverLoss` mints new unbacked ZCHF (source). There is no haircut or redemption; the loss becomes dilution of all ZCHF. FPS also carries the votes that can veto positions and minters.

### Synthetix V2: the shared debt pool

[[projects/synthetix-v2]]: debt is a share of the global pool; the structural peg support is that stakers must buy and burn sUSD (SCCP-232). Not a first-loss pool.

### Alchemix: one Alchemist per alAsset

Docs, read 2026-11-17 through a summariser that covered the first 100,000 of 271,009 characters of `https://alchemix-finance.gitbook.io/user-docs/llms-full.txt` (re-read in this pass with the same limit): "There is one Alchemist for each alAsset on each chain." "Alchemists can have multiple yield strategies but can only issue one type of alAsset." The alUSD Alchemist "issues alUSD but can accept yield-bearing tokens for any approved stablecoin"; "alETH can be borrowed against ETH-denominated yield token deposits." These are V2 docs; V3 was not checked. Not addressed in the part read: separate Transmuters or per-Alchemist debt accounting. See [[projects/alchemix]].

### Monolith: a token per market

Docs and chain, 2026-11-17. Detail in [[projects/monolith]]. It is the only example found of one collateral per stablecoin (a separate token per branch), from its docs landing page ("Monolith allows anyone to deploy their own decentralized crypto-backed stablecoin" by "providing a collateral token address and a price feed").

### Metronome: cross-margin and the swap pool

[[projects/metronome-synth]]: one position can hold several collaterals and mint several synths; its swap module mints and burns synths at oracle price against a shared pool. On 2026-07-30 the DAO disclosed about 6,367 msETH and 4.57M msUSD created without adequate collateral (about $15.7M to $16M); the page and [[patterns/cdp-synthetics-landscape]] attribute it to oracle lag at the swap module, not to repays or liquidations.

### Youves: an engine per synth and collateral pair

[[projects/youves]]: each synthetic and collateral pair has its own v3 engine contract with its own parameters, but all tracker tokens live in one FA2 contract (token ids 0 uUSD, 1 uDEFI, 2 uBTC and uXTZ), so this is isolation of accounting, not separate tokens.

## Precedent check

For each recommendation or proposal in Part 4: what is backed elsewhere, what is partly backed, what is novel. Source: a precedent audit run in the Notion session on 2026-11-17 by a research agent (vault notes, Aave V3, Morpho Blue and Vault V2, Alchemix, Lido, Balancer, OpenZeppelin, Indigo, Monolith and RedStone docs; not the LEZ repository or RFP texts). Agent-reported; where this pass re-checked an item it says so.

| Item on the page | Page label | What is backed | What is not seen |
|---|---|---|---|
| Five-step loss path (angle 5) | Proposal assembled from Liquity, Frankencoin and Sky | Each step exists: Liquity pool then redistribution; Frankencoin equity, then reserve, then dilution; Sky's surplus buffer; Maker's last step before holders is debt auctions minting MKR (Black Thursday: 20,980 MKR sold for 5.3M DAI, [[projects/sky-maker]]) | No protocol has the five-step sequence |
| New branch with its own pool, low starting ceiling and own shutdown (angle 3, questions 9 and 10) | Partly seen | Liquity V2 has per-branch pools and shutdown but cannot add branches; Sky's AutoLine ramps ceilings (150M gap, 6 hours); crvUSD has two markets at zero ceiling ([[projects/crvusd]]) | No protocol adds a collateral to a live deployment with its own pool; precedents cover adding collateral, not adding a synth |
| Separate token per branch (question 1) | Partly seen | Monolith issues a token per market (docs landing page only; [[projects/monolith]]); Youves has an engine contract per synth and collateral pair but its tokens sit in one FA2 contract ([[projects/youves]]); Liquity V2 has a branch per collateral but one token | Any protocol with separate tokens for one synth other than Monolith |
| Per-branch Risk controller (angle 6) | Proposal | Euler V2 per-vault governors and Morpho curators | Both are lending, not minting |
| One synth per deployment (angle 1) | Justification marked as inference | Alchemix: one Alchemist per alAsset (docs). Liquity V1 and V2, Frankencoin and crvUSD are single-synth, so they are single-synth precedent only | No surveyed protocol shares an opt-in first-loss pool across several synths |
| Cross-margin in a minting protocol (angle 4, question 8) | Seen once, with a cost | Metronome, whose shared swap pool led to the $15.7M shortfall | Cross-margin that keeps branches isolated |
| Shared oracle legs fail together (angle 7) | Inference | JPY DeFi's docs warn that JPY/USD is shared by all branches ([[patterns/collateral-debt-units]]) | Not checked per protocol |

## Conflicts and gaps

- **Words.** The earlier version of this note used "strict" and "loss layer" for the two kinds of isolation, "buffer" for the first-loss pool, and "unit of isolation" for the branch. [[patterns/cdp-vs-morpho-trust]] used "strict isolation" and "loss-layer isolation". The page uses the two questions (tokens per synth; width of the loss group), "first-loss pool" and "branch". This note and [[patterns/cdp-vs-morpho-trust]] follow the page.
- **Indigo V3.** The shortfall rule is from the V2-era whitepaper; the V3 docs are silent and V3 live status rests on a secondary source (see [[projects/indigo]]). The page marks it "V3 not confirmed".
- **Liquity V2 other branches.** The Stability Pool read covers the WETH branch only; wstETH and rETH are assumed to run the same code.
- **Alchemix.** V2 docs only, first 100,000 of 271,009 characters.
- **Monolith.** The "one collateral per stablecoin" claim is from the docs landing page and the factory's inputs; no source says what happens if two instances use the same collateral. The 2% GOV share and the local-fee caps differ between sources (see [[projects/monolith]]).
- **Hylo.** The "Not described in the sources read" cell is unchanged: the docs read do not say what happens if the Earn Pool is exhausted.
- **Not re-verified in this pass:** the Notion page's Synthetix V2, Frankencoin and Hylo cells (taken from the project notes).

## Sources

- Notion page "CDP-Backed Synthetics", Part 4, as saved 2026-11-17.
- [Indigo whitepaper v2.0.0 (March 2024 draft)](https://indigoprotocol.io/paper.pdf), section 2.6 : accessed 2026-11-17
- [Indigo docs: Stability Pools and liquidations](https://docs.indigoprotocol.io/readme/stability-pools-and-liquidations.md) : accessed 2026-11-17
- [Alchemix user docs, llms-full.txt](https://alchemix-finance.gitbook.io/user-docs/llms-full.txt) : accessed 2026-11-17 (first 100,000 of 271,009 characters)
- [Monolith docs](https://docs.monolith.market/) : accessed 2026-11-17
- Ethereum mainnet JSON-RPC (publicnode): `getTotalBoldDeposits()` and `getCollBalance()` on the WETH Stability Pool `0x5721cbbd64fc7ae3ef44a0a3f9a790a9264cf9bf`, read 2026-11-17 (block 26,432,207)
- Vault notes: [[projects/liquity]], [[projects/indigo]], [[projects/hylo]], [[projects/frankencoin]], [[projects/synthetix-v2]], [[projects/alchemix]], [[projects/monolith]], [[projects/metronome-synth]], [[projects/youves]], [[projects/crvusd]], [[projects/sky-maker]]
