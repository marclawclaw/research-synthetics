---
tags: [appendix, cdp, lez, admin, governance, proposal]
related: [rfp/appendix/cdp-notion-map, rfp/appendix/cdp-isolation, rfp/appendix/cdp-oracle, patterns/cdp-synthetics-landscape, patterns/cdp-vs-morpho-trust]
---

# CDP on LEZ: the rules and an admin switch per role

Working proposal for the synth CDP program on LEZ (sXMR and sBTC are the examples), mirrored in the Notion page "CDP-Backed Synthetics" (Part 1). This note holds the evidence behind Part 1. The part-by-part map of the vault is in [[rfp/appendix/cdp-notion-map]]. Evidence for the protocol comparisons is in [[patterns/cdp-synthetics-landscape]] (sections 4a and 4b) and the admin-control sections of [[projects/liquity]], [[projects/sky-maker]], [[projects/synthetix-v2]], [[projects/frankencoin]] and [[projects/crvusd]]. Written 2026-11-16; reworked 2026-11-17 to match the page, with the per-rule overview and a precedent check added. A formal fact-check pass has not been run on this note.

## Terms

The full list is in [[rfp/appendix/cdp-notion-map]]. The ones this note uses:

- **Deployer:** someone who takes the output of the RFP and decides to run it. They may or may not be competing for a Lambda Prize. They may build their own software or modify the delivered software.
- **Synth:** a token the CDP mints against collateral, such as sXMR or sBTC.
- **Position:** what a borrower opens by locking collateral and minting synth against it in one transaction. It records that collateral and that debt; adding collateral or minting more later changes the same position, and a borrower can open several. Liquity calls it a Trove.
- **Deployment:** one running instance of the software, with its own tokens, state and roles.
- **Branch:** one collateral paired with one synth, with its own ratio, feed and debt ceiling, and optionally its own first-loss pool and shutdown.
- **Debt ceiling:** the most synth a branch can mint.
- **Role and mode:** a role is a group of the Part 1 levers that one controller can change; its mode (Immutable, Separate governance or Program gated) says how that controller works.
- **Reserve:** value the protocol holds itself, such as Sky's surplus buffer, rather than opt-in deposits.
- **Oracle provider and feed:** a provider (such as RedStone) publishes signed prices; a feed is one price, such as XMR/USD.

Highlight words. The Notion page colours decisions and risks; this vault has no colours, so the same meaning is written in words: "Recommendation: ..." (green: a recommendation made on a decision), "Open decision: ..." (yellow: a decision with no recommendation yet) and "Risk to evaluate: ..." (red: a risk that may not need a decision now, but needs to be evaluated).

## Part 1: the rules

Every item below is a lever someone can pull. For each one we need who can change it, how fast, within what bounds, and whether that power can be given up for good. ("Lever" is Part 1's word for these rules; Part 3 uses "peg mechanism" and "rate mechanism" for something else.)

| Lever | What it covers |
|-------|----------------|
| Collateral list | Which collateral can back the synths, including collateral stablecoins, such as the reflexive stablecoin (RFP-013) |
| Synth list | Adding sBTC and later synths |
| Risk parameters | Collateral ratios, liquidation penalties, debt ceilings |
| Oracle | Which price account each pair reads, and swapping it |
| Fees and rates | Mint fee, borrower rate, where fees go |
| Pause | Emergency stop, what it can and can't freeze |
| Code | Upgrades, and whether any program can be immutable on LEZ |
| Shutdown | Who can wind it down, and how holders exit |

- **Open decision: at deployment, who holds each lever?**
- **Open decision: path to renounce.** Which levers get a path to renunciation, and which stay governed permanently?
- **Vault reference points:** Liquity V1 has no admin, Maker has a 48-hour delay, and Synthetix V2 had a 4-of-7 key with no delay. Frankencoin offers a holder veto.
- **LEZ:** who controls upgrades on the canonical LEZ, and can programs there be frozen? (See "What LEZ provides" below.)

### Who can change each rule: ecosystem overview

Each cell on the Notion page answers: who can change this rule, how fast, and within what limits. Evidence tags: **chain** = read from Ethereum mainnet on 2026-11-16; **source** = the contract source code; **docs** = the project's documentation; **vault** = these research notes. The full lever table of each protocol is in its note ([[projects/liquity]], [[projects/sky-maker]], [[projects/synthetix-v2]], [[projects/frankencoin]], [[projects/crvusd]]); the matrix below keeps the page's category and the one fact that decides it. "Not checked" means we did not look, not that the rule cannot be changed.

Category key (the page's cell colours, described in words). A cell's category is taken from the first line of its Who entry on the page; where the cell text names a further power, the category does not show it.

- **Nobody:** fixed in code, or no such power (green).
- **Borrowers or anyone, within rules fixed in code** (blue).
- **Equity-token holders:** holders of the protocol's equity token (Frankencoin: a qualified holder can propose or veto) (purple).
- **Token vote** (yellow).
- **Token vote plus a faster path:** a multisig or a bounded setter that can also change it (orange).
- **Multisig alone:** no vote on chain (red).

| Rule | Liquity V1 | Liquity V2 | Sky/Maker | Synthetix V2 | Frankencoin | crvUSD |
|------|------------|------------|-----------|--------------|-------------|--------|
| Collateral list | Nobody: ETH only, fixed in immutable code (chain: owners renounced) | Nobody: WETH, wstETH, rETH fixed in the collateral registry (chain: three collaterals, no owner function, no proxy slot); more branches only by a new deployment | Token vote: an executive spell voted in Chief V3 with SKY, executed by the PauseProxy, at least 48 hours after the vote (chain) | Multisig alone: a 4 of 7 Safe owning the core contracts including the CollateralManager, immediate, no delay, guard or modules (chain); governance proposals decide (docs, unverified) | Equity-token holders: anyone opens a position with any token; a qualified holder (2% of FPS votes, or over 1% of FCS votes) can deny it; at least 3 days (source) | Token vote plus faster path: only the Curve DAO can add a market (7 day vote, 51% support, 30% quorum, no timelock); no market can be removed, but the Emergency DAO (5 of 9) can cut a ceiling at once (source, chain) |
| Synth list | Nobody: LUSD only | Nobody: BOLD only; its owner is the zero address (chain) | Token vote: a spell; USDS and sUSDS are upgradeable proxies administered by the PauseProxy; 48 hours (chain) | Multisig alone: the same Safe; its AddressResolver decides which contract serves each synth; 10 synths listed (chain) | Equity-token holders: anyone suggests a minter, a qualified holder can deny it; 14 days and 1,000 ZCHF on the immutable token, 60 days and 1,200 ZCHF through the 2026 MinterGovernance wrapper; approval is permanent (chain, source) | Nobody: crvUSD only, minter fixed to the factory; but the DAO can mint any amount of crvUSD to any address through `set_debt_ceiling` (source) |
| Risk parameters | Nobody: hard-coded 110% minimum ratio, 150% Recovery Mode threshold, 1,800 LUSD minimum net debt plus a 200 LUSD reserve (chain, source) | Nobody: fixed per branch, WETH minimum ratio 110%, wstETH and rETH 120%, minimum debt 2,000 BOLD (chain, source) | Token vote plus faster path: a spell (48 hours), or AutoLine for debt ceilings, which anyone can trigger (ETH-A raised at most once per 6 hours in steps of 150M, up to 15B) (chain, source) | Multisig alone: the Safe owns SystemSettings, immediate; some caps in code, but the Safe can swap in a different settings contract (source) | Borrowers or anyone: each position's owner sets its own liquidation price, risk premium, term and limits; raising the price starts a 3 day minting cooldown (source) | Token vote plus faster path: the DAO sets loan and liquidation discounts and the AMM fee per market (7 day vote); the Emergency DAO can only raise the loan discount (source, chain) |
| Oracle | Nobody inside Liquity: Chainlink first, Tellor fallback; but Chainlink's feed proxy is owned by a 4 of 9 multisig that can repoint it (chain; its delay not checked) | Nobody inside Liquity: Chainlink feeds, same 4 of 9 multisig upstream; wstETH and rETH exchange rates come from the Lido and Rocket Pool token contracts (chain, source) | Token vote: Sky controls the Oracle Security Module (stopped by the leading spell, source changed by spell); Chronicle runs the ETH/USD feed behind it (13 of 25 signers) and the PauseProxy has no admin rights there (chain) | Multisig alone: the Safe owns ExchangeRates and picks the Chainlink feed per currency; sUSD fixed at $1 in code; no bounds on stale period or deviation threshold (source, chain) | Nobody: minting, challenges and expiry use no oracle; one exception, the Uniswap Amplifier minter reads the ZCHF/USDT pool's spot price (2.5M ZCHF limit, expires 2027-03-31) (source, chain) | Nobody can replace a market's oracle (source); the DAO or Emergency DAO can switch the Chainlink clamp (plus or minus 1.5%) on or off, off on the WBTC and wstETH oracles today (chain) |
| Fees and rates | Nobody: one-time borrowing fee that rises with redemptions, no interest; the algorithm is in immutable code (vault) | Borrowers: each sets their own interest rate (0.5% to 250% a year); the 75% to Stability Pool, 25% to governance split is hard-coded (source) | Token vote plus faster path: a spell (48 hours), or SP-BEAM, a bounded setter run by a 2 of 3 Safe (rates 2% to 30%, DSR 0% to 30%, at most 4 points per change, one batch per 16 hours); today ETH-A 10.50%, DSR 1.25%, SSR 3.80% (chain) | Multisig alone: the Safe owns FeePool and SystemSettings, immediate; exchange fee at most 10% per synth, today 35 bps for sETH and sBTC on L1 (source, chain) | Equity-token holders: any qualified holder proposes a lead rate and anyone applies it; 7 days; today 1.00% borrowing lead rate, 3.50% savings (source, chain) | Token vote plus faster path: the DAO changes rate settings (7 day vote); the DAO or Emergency DAO can swap any market's rate contract, at once for the Emergency DAO; the rate itself is algorithmic with a hard cap at which debt at most quadruples in a year (source, chain) |
| Pause | Nobody: no pause function exists; Recovery Mode is automatic (source) | Nobody: no pause function; the only freeze is branch shutdown (source) | Token vote: the leading spell (the "hat") through ten emergency modules (MOMs) and pre-deployed emergency spells; instant, but only after outvoting the current hat (about 7.03B SKY); stop-only, unpausing needs a 48 hour spell; no guardian multisig found (chain, source) | Multisig alone: any account the Safe grants suspend rights to; of 17 accounts checked, only the Safe can suspend the whole system (chain) | Nobody: no global pause; a qualified holder can set bridge rate limits per chain, including to zero, through CCIPAdmin; a challenge suspends minting on one position only (source, chain) | Multisig alone (colour as on the page): no designed pause; brakes are ceiling cuts and a PegKeeper kill switch, both available to the Emergency DAO, immediate; repaying and liquidating cannot be switched off by design (source, chain) |
| Code | Nobody: immutable; owner of TroveManager and BorrowerOperations is the zero address (chain, vault) | Nobody: no owner function and no proxy slot on the collateral registry or the three trove managers (chain); not upgradeable (docs, via vault) | Token vote: a spell; core contracts are not proxies, changes come by rewiring which contracts are authorised on the Vat; 48 hour delay (chain, source) | Multisig alone: the Safe, via `setTarget` on the SNX proxy and `importAddresses` on the resolver, both owner-only; immediate (source, chain) | Nobody can upgrade deployed code (no proxies); change comes only by adding new minters, 14 or 60 days; old minters stay approved for ever (source) | Nobody can upgrade deployed code (source, chain); the DAO can set new templates for future markets and swap a market's rate contract and rewards callback (source) |
| Shutdown | Nobody: no shutdown path exists (source) | Borrowers or anyone: once a branch's total collateral ratio falls below its shutdown ratio (110% WETH, 120% LSTs), or automatically on oracle failure; one transaction, irreversible, per branch; no branch is shut today, WETH ratio about 155.8% (source, docs, chain) | Token vote: a spell or the Emergency Shutdown Module (still on MKR, threshold set to the maximum, so in practice only a spell); 48 hours, then a 73 hour wait; the USDS exit path is [NOT FOUND] (chain, archived docs) | Multisig alone: the Safe, by combining the levers above; no dedicated shutdown function; what was done: minting blocked, exchange suspended, transfers of 10 synths frozen, sUSD replaced by a retired stub (SIP-423), V2 debt moved to the V3 LegacyMarket; for remaining sETH and sBTC holders the exit route is [NOT FOUND] (chain, source) | Nobody: no wind-down or global settlement; positions and bridges expire one by one; if equity falls below 1,000 ZCHF a qualified holder can burn the other holders' FPS to recapitalise (source) | Token vote plus faster path: the DAO, market by market, no global settlement or redemption; a ceiling cut is immediate through the Emergency DAO; all five PegKeepers held no deposits on 2026-11-16 (source, chain) |

Details beyond these cells (for example Liquity V2's per-branch parameters, Sky's eight levers, crvUSD's Lending Owner Proxy functions, Frankencoin's governance path) are in each project note. Where the matrix and a project note differ, the project note holds the evidence.

### General governance, per protocol

Who governs the protocol overall, kept separate from the per-rule table.

- **Liquity V1:** no governance. Owners of the core contracts are the zero address (chain).
- **Liquity V2:** no governance over protocol rules; contracts are immutable. Governance only directs where the 25% interest stream goes. LQTY stakers vote, weighted by stake age, in 7 day epochs; the governance contract's owner is zero and it has no role in any core contract (chain, source).
- **Sky/Maker:** SKY holders vote in Chief V3. A spell needs 2.4B SKY to launch; only the leading spell (the hat) can schedule into the 48 hour delay. The Chief holds 7.14B of 23.45B SKY (30.4%) (chain). Bounded fast paths sit beside it: SP-BEAM (2 of 3 Safe, rates within 2% to 30%), AutoLine (debt ceilings, anyone can trigger) and the stop-only emergency modules (chain). Who the large SKY voters are was not checked.
- **Synthetix V2:** a Safe multisig, 4 of 7 signers, with no delay, guard or modules (chain). Synthetix's developer docs name that address as the Protocol DAO multisig (search-result summary; the page itself returned 404). Stakers elect councils that vote on proposals (docs, unverified), but nothing on chain binds the Safe to a vote. SIP-124 expected at least 5 signatures and 9 or more members; the chain shows 4 of 7. SIP-401, which would have replaced the DAO with a Risk Council, was rejected.
- **Frankencoin:** no owner, admin or proxy (source). Rule changes need a qualified holder: 2% of time-weighted FPS votes acting directly, or more than 1% of FCS votes acting through the 2026 FCS modules; the FCS contract holds 35% of FPS votes, so its decisions clear the 2% bar (chain). Delegation is transitive, and a holder can sacrifice votes to destroy another's. The audit's trust model assumes no malicious majority of votes.
- **crvUSD:** the Curve DAO, by on-chain vote: 7 days, 51% support, 30% quorum, then no delay, executed through the Aragon Agent and the factory's admin forwarder (chain). Next to it, the Emergency DAO (a 5 of 9 Safe, no guard or modules) can act at once: cut debt ceilings, raise loan discounts, switch the Chainlink clamp, kill PegKeepers, and swap a market's rate contract (source, chain). The voter set was not checked.

What the table shows: Liquity V1 and V2 have no rule anyone can change. Sky/Maker and crvUSD put most rules behind a public vote with a delay, but both add faster paths: Sky through bounded, rate-limited setters and stop-only modules, crvUSD through a 5 of 9 Emergency DAO with immediate powers, most of them only able to reduce risk. Synthetix V2 put every rule behind one multisig with no delay. Frankencoin replaces most rules with a veto, so the proposer's choices are cheap and holders' attention is the scarce resource.

An immutable core does not remove every admin. Liquity's own contracts have none, but it inherits Chainlink's (a 4 of 9 multisig can repoint the ETH/USD feed) and, in V2, Lido's and Rocket Pool's control over the exchange rates. **Risk to evaluate: whatever oracle we choose brings its own admin, whatever mode our roles are in.**

## Proposal: an admin switch per role, chosen by the deployer

The program does not pick a governance model. It groups the levers above into a few roles, and each role has one controller account. The deployer sets each role to one of three modes.

| Mode | Controller | What holders must trust | Closest surveyed example |
|------|------------|-------------------------|--------------------------|
| Immutable | None. The role is renounced at deployment (or later); renouncing is one way | The code and the values fixed at deployment | Liquity V1 and V2: no rule anyone can change |
| Separate governance | A multisig program account (for example the LEZ multisig). Governance can take any form, even with its own on-chain programs (votes, councils), but nothing in code ties it to the CDP: the signers carry out its decisions by choice | The signers. Any process or delay binds them only socially, unless the core enforces a delay | Synthetix V2: a 4 of 7 Safe, no delay, can change everything. Morpho Blue: a 6 of 10 Safe that can only enable new rate models and LTVs, set the fee and pass ownership |
| Program gated | The governance program itself is the controller, so the CDP accepts a change only once that program's process has run (public proposal window, delay, vote or veto) | The gate program's rules and whoever votes in it | Sky/Maker: vote, then a 48 hour delay (Maker calls it the pause), plus bounded fast paths (a 2 of 3 Safe can move rates within 2% to 30%, at most 4 points per 16 hours). crvUSD: a 7 day Curve DAO vote, plus a 5 of 9 Emergency DAO that acts at once (close to our Guardian, but broader: it can also swap rate contracts). Frankencoin: anyone proposes, holders with 2% of FPS votes (or over 1% of FCS votes) can veto; approved minters can never be removed |

The core program only checks that the caller is authorised as the role's controller (a signer, or a program acting through a chained call). It does not need to know which mode it is in, and a deployer can mix modes: for example the Program upgrade role Immutable, the Oracle role under Separate governance, the Listing role Program gated.

Ideally, those 3 modes fold into one: the ability to set an account address as admin, whether this address is a public account, a program account or the default account (no owner), can then be left to the deployer.

### Proposed roles

The deployer is not a role: it sets every role's controller at deployment and then holds only what it kept for itself. This grouping is a proposal: per-role controllers exist in Aave V3 and Morpho Vault V2, but this split is not seen elsewhere.

| Role | Levers it controls | Notes |
|------|--------------------|-------|
| Program upgrade | Code upgrades | Native on LEZ: the right to update the program's header (see below). Must be immutable for any other renounce to mean anything |
| Listing | Collateral list, synth list | Removing a collateral or synth stops new debt; it never seizes positions |
| Risk | Collateral ratios, liquidation penalty, debt ceilings | Only within bounds written into the code |
| Oracle | Which price account each pair reads | The signer set belongs to the oracle provider, not to this role. See [[rfp/appendix/cdp-oracle]] |
| Fees | Mint fee, borrower rate, fee recipient | Only within bounds written into the code |
| Guardian | Pause new minting | Cannot block repaying, adding collateral, withdrawing excess collateral or liquidating; a pause expires on its own |
| Shutdown | Global settlement | Starts the exit path for holders and borrowers |

Whether a role has one controller per deployment, per synth or per branch is an open question in [[rfp/appendix/cdp-isolation]] (Part 4, question 6).

### Rules that hold in every mode (proposed)

1. Renounce is per role and one way. Transfer takes two steps (nominate, then the new controller accepts), as Synthetix's owned contracts do, so a typo cannot lock a role forever.
2. A renounced role is only as final as the Program upgrade role. If the program stays upgradeable, an upgrade can bring the power back. A front end that can read the roles can show each lever's effective mode as the weaker of its own role and Program upgrade. This is a proposal; no surveyed protocol does it. The conditional front-end sentence is part of the proposal, not a requirement.
3. Bounds live in the code, not with the controller: a floor on the collateral ratio, caps on fees, a maximum pause length. Synthetix V2's caps did not bind its owner, who could swap the settings contract. Morpho's owner can only enable options, never change an existing market. Sky's SP-BEAM shows bounds and rate limits together: rates only between 2% and 30%, at most 4 points per change, at most once per 16 hours.
4. No role can move a position's collateral, or stop a borrower from repaying and withdrawing.
5. Every change and every controller change emits an event. (Whether a controller can be read on chain is not a protocol rule.)

### Open choices within the proposal

- **A minimum delay in the core.** The core could enforce a delay floor per role, fixed at deployment (zero allowed), so a Separate governance multisig cannot act instantly. **Recommendation (partly seen elsewhere): yes,** because holders can read the delay without auditing the controller. Partly seen: Morpho Vault V2 enforces per-function timelocks in the contract, zero allowed; no surveyed CDP does this, and none fixes the delay at deployment. Drift lost about $285M to $295M through a 2 of 5 council with no delay ([[projects/drift]]), and Synthetix V2's Safe also has no delay.
- **Guardian as its own fast role.** **Recommendation (partly seen elsewhere): yes,** limited to pausing and time-limited, as above. Unpausing belongs to the Risk or Shutdown controller. Each limit exists elsewhere (Aave's freeze, crvUSD's ceiling cut, Sky's stop-only modules, Lido's self-expiring pause); no protocol combines them in one role.
- **Open decision: do Risk changes apply to open positions or only to new ones?** Maker, crvUSD and Synthetix apply them to open positions. Morpho cannot change an existing market at all.
- **Do we ship a reference Program gated controller (proposal plus veto), or only define the interface?** Decided: no reference controller; the RFP defines only the interface. A demo or test shows the interface works with the [RFP-005](https://github.com/logos-co/rfp/blob/master/RFPs/RFP-005-multisig-and-coordination.md) multisig as a role's controller.

### Logos RFPs this builds on

Read from `logos-co/rfp` at `920fad9` (2026-11-06; earlier session, not re-read in this pass because no local clone of `logos-co/rfp` is available).

- [RFP-001 Admin Authority Library](https://github.com/logos-co/rfp/blob/master/RFPs/RFP-001-admin-authority-lib.md): one admin authority per program, set at initialisation, which can transfer or renounce (revoke); gates privileged instructions; "There can only be one admin authority (signer) at a time"; a PDA is allowed as the authority (soft requirement). **Risk to evaluate: gaps against this proposal.** It allows only one authority per program, not one per role, and its transfer is one step.
- [RFP-002 Freeze Authority Library](https://github.com/logos-co/rfp/blob/master/RFPs/RFP-002-freeze-authority-lib.md): a freeze authority, set at initialisation and changed or revoked by the admin authority, that can freeze the whole program ("rejecting any attempt to interact with it") or a single `AccountId`. **Risk to evaluate: both conflict with the Guardian limits and rule 4.** A full freeze also blocks repaying, adding collateral and liquidating, and a per-account freeze can stop one borrower from repaying. The CDP would use it only in a narrower form, or not at all.
- [RFP-020 RedStone Off-Chain Oracle Adaptor](https://github.com/logos-co/rfp/blob/master/RFPs/RFP-020-redstone-oracle-adaptor.md) (assumed delivered by 0.4): push aggregator writing an RFP-019 price account (the only read path for private execution) and public-mode pull; M of N signers (default 3 of N); per-feed `maxAge`; feeds and signer sets managed by an RFP-001 admin authority, which is an upstream admin the CDP inherits. Detail in [[rfp/appendix/cdp-oracle]].

### What LEZ provides

Read from `logos-blockchain/logos-execution-zone`, branch `dev`, commit `cb075d5` (2026-11-13); earlier session, not re-read in this pass.

- A user program is deployed through the program loader (`lee/program_loader/src/lib.rs`) as a header account pointing to a chain of bytecode segments. `ProgramHeader` carries `immutable: bool` ("Once `true`, this header can never be updated again"). `UpdateHeader` asserts the old header is mutable and that the header account is authorised; it can also flip `immutable` to true (test `update_header_flip_to_immutable_lands_immutable_mirror_commitment` in `lee/state_machine/src/state/tests/deploy.rs`). Deploying immutable and renouncing later are therefore both native.
- `ReceiveInput.is_authorized`: "Input is authorized when signed by the receiver or authorized by owning program" (`lee/state_machine/core/src/program/mod.rs`); programs derive PDAs from their own account ID. Inference: a header held at a multisig's or gate program's PDA would be upgradable only through that program. **Risk to evaluate: this is not tested in the LEZ repo; confirm it with the LEZ team.**
- System programs (clock, fee, bridge and others) are installed and upgraded by a threshold of the sequencer committee with a scheduled activation height; `system_upgrader` is seeded immutable and only a hard fork changes it (`lez/programs/system_upgrader/README.md`). **Risk to evaluate: no mechanism to freeze a user program was found** (keyword search for freeze, frozen, blacklist and denylist in `lee` and `lez`; not an audit).
- An LEZ multisig exists: `logos-co/lez-multisig` (modelled on Squads v4: create, propose, approve M of N, execute through `ChainedCall` to any program). **Risk to evaluate: its last commit `a057fc4` is from 2026-03-31, which predates the current LEZ code, so whether it works with today's LEZ is not verified.**

## Precedent check

For each recommendation or proposal on the page: what is backed elsewhere, what is partly backed, what is novel. Source: a precedent audit run in the Notion session on 2026-11-17 by a research agent (it read the vault notes and the docs of Aave V3, Morpho Blue and Vault V2, Alchemix, Lido, Balancer, OpenZeppelin, Indigo, Monolith and RedStone; it did not check the LEZ repository or the RFP texts). The audit is agent-reported; the two checks marked "re-read" were repeated in this pass against the docs on 2026-11-17.

| Item on the page | Page label | What is backed | What is not seen | Sources |
|---|---|---|---|---|
| Minimum delay enforced by the core, fixed at deployment | Recommendation: yes. Partly seen | Morpho Vault V2 enforces a per-function timelock inside the contract, zero allowed, plus abdication (setting a function's timelock to infinity); Aave runs 1 day and 7 day executors by action class | A CDP core that enforces a delay, or one fixed at deployment (Vault V2's curator can change it through a timelocked step). Maker's 48 hours is an external pause contract. Drift (2 of 5 council, no delay, about $285M to $295M lost) is a failure case, not a precedent | [Morpho timelock docs](https://docs.morpho.org/curate/concepts/timelock) (re-read 2026-11-17: per-function, zero allowed, increase is itself timelocked, decrease uses the delay of the function being decreased); [[projects/drift]] |
| Guardian: pause new minting only; repay, add collateral, withdraw excess and liquidate stay open; pause expires on its own; unpause by another role | Recommendation: yes. Partly seen | Aave V3 `setReserveFreeze`; crvUSD Emergency DAO ceiling cut ([[projects/crvusd]]); Sky's stop-only emergency modules, unpaused by a 48 hour spell ([[projects/sky-maker]], Pause row); a self-expiring pause in Lido's CircuitBreaker and Balancer V2's pause window | All of these together in one role. Sky has no guardian multisig | [Lido CircuitBreaker](https://docs.lido.fi/contracts/circuit-breaker) (re-read 2026-11-17: DAO-set duration, 21 days on mainnet within a 5 to 60 day range; the pauser cannot choose the duration; the contract cannot resume the pausable) |
| Seven-role split (Program upgrade, Listing, Risk, Oracle, Fees, Guardian, Shutdown) | Proposal, not seen elsewhere | Per-role controllers exist in Aave V3 ACLManager, Morpho Vault V2 roles, Alchemix Admin, Sentinel and Keeper | This particular split. Maker's `End.cage()` is auth-gated but the ESM threshold is set to the maximum, so only a spell can shut down ([[projects/sky-maker]], Shutdown row) | Agent audit; vault notes |
| Rule 2: front end shows each lever's effective mode (weaker of its role and Program upgrade) | Proposal; no surveyed protocol does it | Nothing | The whole idea | Agent audit |
| Oracle role pins which price account each pair reads; the signer set belongs to the oracle provider | Decision | The split matches practice: Sky does not control Chronicle's signers ([[projects/sky-maker]], Oracle row), and Synthetix V2's Safe picks the Chainlink feed and stale period only | An oracle role that holds the signer set | Agent audit; vault notes |
| Rule 3: bounds in code, not with the controller | Proposal | Sky SP-BEAM bounds (chain), Synthetix V2 caps that did not bind its owner (source), Morpho's owner limited to enabling options ([[patterns/cdp-vs-morpho-trust]]) | Not audited separately for novelty | Vault notes |
| Rule 1: two-step transfer (nominate, accept) | Proposal | Synthetix's owned contracts (page; [[projects/synthetix-v2]]) | Not audited for other protocols | Vault notes |
| Rule 5: every change and controller change emits an event | Proposal | Not checked per protocol | Not checked | Agent audit: unclear |
| Header held at a PDA and upgradable only through the program | Risk to evaluate | Nothing in the LEZ repo tests it | Untested platform inference | LEZ reading above |
| Per-branch Risk controller (Part 4, angle 6) | Proposal | Euler V2 per-vault governors and Morpho curators | Both are lending, not minting protocols | [[rfp/appendix/cdp-isolation]] |

## Conflicts and gaps

- **Rule 5.** An earlier version of this note added "and each role's controller is readable on chain". The Notion page has only "every change and every controller change emits an event" (controller readability is not a protocol rule). This note follows the page.
- **Oracle role levers.** An earlier version listed "Feeds, signer set, staleness rules". The page says the role pins which price account each pair reads, and the signer set belongs to the oracle provider. This note follows the page; the freshness limit is applied by the CDP itself ([[rfp/appendix/cdp-oracle]]).
- **Words.** The page calls Maker's 48 hours a "delay" ("Maker calls it the pause"), not a timelock or a pause; this note follows it. Drift's council is described as having "no delay" (older wording: "no timelock"; the TRM Labs source says zero timelock).
- **crvUSD Pause cell.** The page colours it as Multisig alone although the first line of its Who entry says "nobody: no designed pause". The cell then names the Emergency DAO as holding the brakes. Reproduced as on the page; the colour does not follow the page's own first-line rule.
- **Not checked** (carried from the page): who the large SKY voters are; the Curve DAO voter set; Chainlink's delay on its 4 of 9 multisig; whether other accounts hold suspend rights on Synthetix V2 (event logs not read).
- **Not re-verified in this pass:** the RFP texts and the LEZ commit reading (no local clone; see above), and the chain and source facts in the matrix (taken from the project notes and the page).

## Sources

- Notion page "CDP-Backed Synthetics", Part 1, as saved 2026-11-17.
- Project notes cited above, each with its own sources and access dates.
- [Morpho timelock docs](https://docs.morpho.org/curate/concepts/timelock) : accessed 2026-11-17
- [Lido CircuitBreaker docs](https://docs.lido.fi/contracts/circuit-breaker) : accessed 2026-11-17
- [RFP-001](https://github.com/logos-co/rfp/blob/master/RFPs/RFP-001-admin-authority-lib.md), [RFP-002](https://github.com/logos-co/rfp/blob/master/RFPs/RFP-002-freeze-authority-lib.md), [RFP-005](https://github.com/logos-co/rfp/blob/master/RFPs/RFP-005-multisig-and-coordination.md), [RFP-020](https://github.com/logos-co/rfp/blob/master/RFPs/RFP-020-redstone-oracle-adaptor.md) : read 2026-11-06 at `920fad9` (earlier session)
