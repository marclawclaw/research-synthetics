---
tags: [appendix, cdp, lez, notion, map, terms]
related: [rfp/appendix/cdp-admin-switch, patterns/collateral-debt-units, patterns/cdp-peg-mechanisms, rfp/appendix/cdp-isolation, rfp/appendix/cdp-risk-privacy-market, rfp/appendix/cdp-oracle]
---

# CDP on LEZ: map from the Notion page to this vault

The Notion page "CDP-Backed Synthetics" is the source of truth for structure, wording, terminology and decisions about the synth CDP program on LEZ (sXMR and sBTC are the examples). This vault holds the evidence behind each claim on that page. This note says which vault file backs which part, repeats the page's terms and highlight key, lists the decisions carried over, and records where the vault and the page disagreed. The page was saved on 2026-11-17 (about 66,000 characters); this note was written on the same day. The vault's research status for these notes: agent-assisted, with spot checks; a formal fact-check pass (`rfp/agents/fact-checker.md`) has not been run.

## Part map

| Notion part | Vault file | What the file adds to the page |
|---|---|---|
| Terms and highlight key | this note | The page's terms and key, and how colours become words |
| 1 The rules, and the admin switch proposal | [[rfp/appendix/cdp-admin-switch]] | The per-rule overview as a category matrix, the general governance paragraphs, the proposal, the RFP-001, RFP-002 and LEZ readings, a precedent check |
| 2 Units and oracle requirements | [[patterns/collateral-debt-units]] | The full comparison table with sources, the requirements, collateral with no outside market, a precedent check |
| 3 Peg mechanisms | [[patterns/cdp-peg-mechanisms]] | Each mechanism with the protocols that use it, numbers and sources, mechanisms seen but not on the page's table |
| 4 How a deployment shares risk | [[rfp/appendix/cdp-isolation]] (filename kept; retitled), [[patterns/cdp-vs-morpho-trust]] | The five-protocol table, angles, questions, protocol evidence, precedent check; per-protocol notes [[projects/liquity]], [[projects/indigo]], [[projects/hylo]], [[projects/frankencoin]], [[projects/synthetix-v2]], [[projects/alchemix]], [[projects/monolith]] |
| 5 Who carries the risk | [[rfp/appendix/cdp-risk-privacy-market]] | Evidence for who mints, the loss path and who liquidates |
| 6 Oracle | [[rfp/appendix/cdp-oracle]] | The interface and Proxy Oracle decisions, what is out of scope, existing multi-oracle designs, gaps |
| 7 Privacy | [[rfp/appendix/cdp-risk-privacy-market]] | The question with the evidence the vault holds |
| 8 Market | [[rfp/appendix/cdp-risk-privacy-market]] | The viability question with the evidence the vault holds |

Cross-protocol evidence used by several parts: [[patterns/cdp-synthetics-landscape]] (50-protocol survey; sections 4a and 4b for admin control), [[patterns/oracle-pricing-model]], and the project notes [[projects/sky-maker]], [[projects/crvusd]], [[projects/metronome-synth]], [[projects/youves]], [[projects/mirror-protocol]].

## Terms (as on the page)

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
- **Redemption:** handing the synth back for collateral at face value (Liquity). A redemption price, as in RAI, is something else: an internal target price.
- **Shield and unshield:** moving tokens from a public LEZ account to a private one, and back.
- **Oracle provider and feed:** a provider (such as RedStone) publishes signed prices; a feed is one price, such as XMR/USD.

Vocabulary notes from the page: "lever" is Part 1's word for the admin rules; "peg mechanism" and "rate mechanism" are Part 3's words and are not "lever"; "synth" is generic (sXMR and sBTC are examples only); the reflexive stablecoin is always "the reflexive stablecoin (RFP-013)", never "RAI-style", and it is a separate product that appears only as collateral; Maker's 48 hours is a "delay" (Maker calls the contract the pause), not a timelock or a pause; "first-loss pool" is the generic term and Stability Pool, Earn Pool and FPS reserve are each protocol's name for its own.

## Highlight key: colours become words

The page highlights text to show where a decision or risk stands. This vault has no colours, so the vault writes the meaning in words:

| Page colour | Meaning on the page | Vault wording |
|---|---|---|
| Green | A recommendation made on a decision | "Recommendation: ..." (or "Recommendation (partly seen elsewhere): ..." when a precedent is partial) |
| Yellow | A decision with no recommendation yet | "Open decision: ..." |
| Red | A risk that may not need a decision now, but needs to be evaluated | "Risk to evaluate: ..." |

Two tables on the page use their own cell-colour keys, given above each table. Both are written in text in the vault: the per-rule table in Part 1 (Nobody, Borrowers or anyone, Equity-token holders, Token vote, Token vote plus a faster path, Multisig alone; [[rfp/appendix/cdp-admin-switch]]) and the five-protocol table in Part 4 (what one pool covers; when the pool pays out; beyond the pool; [[rfp/appendix/cdp-isolation]]).

## Decisions carried over from the page

- The RFP delivers software with no factory and no central party; deployers may modify the software; how deployers compete is out of scope; deployments share nothing (each has its own tokens and state).
- No reference Program gated controller is shipped. The RFP defines only the interface; a demo or test shows it works with the RFP-005 multisig as a role's controller.
- Rule 5 is only "every change and every controller change emits an event". Controller readability on chain is not a protocol rule. Rule 2's front-end sentence is conditional and a proposal.
- The Oracle role pins which price account each pair reads; the signer set belongs to the oracle provider, not to this role.
- The oracle is out of scope except an interface (an RFP-019 price account per pair, with the CDP applying its own maximum age) and a median Proxy Oracle program (write versus compute, missing or stale inputs and fallbacks left to implementors).
- Privacy: positions are public; no shielded positions or pool; one open question on direct interaction from private accounts.
- Part 4: two questions define a deployment's model (how many tokens does a synth have; how wide is the group that carries a loss first). Liquity V2 is one token across collaterals with a narrow loss group (one collateral).
- Part 2's worked example still uses ETH as the example collateral; whether to switch it to LGO is pending. [[patterns/collateral-debt-units]] therefore keeps ETH.

## Where the vault and the page disagreed, and which was followed

| Item | Vault (before this rework) | Page | Followed |
|---|---|---|---|
| Rule 5 | "... and each role's controller is readable on chain" | Only events | The page |
| Oracle role levers | "Feeds, signer set, staleness rules" | Which price account each pair reads; signer set belongs to the provider | The page |
| Maker's 48 hours | "48 hour pause" | "delay" (Maker calls it the pause) | The page |
| Reflexive stablecoin | Context line: a planned "RAI-style floating stablecoin", possibly a synth stablecoin and a collateral stablecoin | A separate product that appears only as collateral | The page |
| Part 2 worked example | ETH swapped to LGO in the working tree (worked example, requirements 1, 2, 9, 11, and an LGO/USD pair) | ETH | The page |
| Part 2 fallback item | "A new chain with one oracle provider gets only the last two" | Everything except a second provider remains available | The page (the findings file's correction) |
| Part 4 vocabulary | "strict" and "loss layer" isolation; "buffer"; "unit of isolation" | Two questions; "first-loss pool"; "branch" | The page |
| Redemption as a peg mechanism | [[projects/liquity]] calls redemption the hard floor | The Part 3 table does not list redemption at face value | The page for the table; the vault's evidence is kept under "seen in the survey" in [[patterns/cdp-peg-mechanisms]] |
| crvUSD Pause cell colour | n/a | Coloured Multisig alone although the first line of its Who entry says "nobody" | Reproduced as on the page; flagged in [[rfp/appendix/cdp-admin-switch]] |

## Known gaps in the vault's evidence

- The RFP texts (RFP-001, 002, 005, 019, 020) and the LEZ repository were read in an earlier session (`logos-co/rfp` at `920fad9`, LEZ `cb075d5`) and were not re-read: no local clone of either is available. The files to read when a clone is available are named in [[rfp/appendix/cdp-oracle]] and [[rfp/appendix/cdp-admin-switch]].
- Indigo contract source and V3 behaviour were not read; the shortfall rule is from the V2-era whitepaper ([[projects/indigo]]).
- The Liquity V2 Stability Pool read covers the WETH branch only ([[projects/liquity]]).
- Chain and source facts in the Part 1 matrix are those of the 2026-11-16 reads in the project notes; they were not repeated.
- RedStone's XMR/USD feed and Pyth's XMR/USD publisher count are asserted only by RFP-020 ([[rfp/appendix/cdp-oracle]]).
