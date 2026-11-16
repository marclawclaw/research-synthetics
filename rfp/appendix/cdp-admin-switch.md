# CDP on LEZ: an admin switch per role

Working proposal for the sXMR and sBTC CDP program on LEZ, mirrored in the Notion page "CDP-Backed Synthetics" (Part 1). Evidence for the protocol comparisons is in [[patterns/cdp-synthetics-landscape]] (sections 4a and 4b) and the admin-control sections of [[projects/liquity]], [[projects/sky-maker]], [[projects/synthetix-v2]], [[projects/frankencoin]] and [[projects/crvusd]]. Written 2026-11-16.

## The switch

The program does not pick a governance model. It groups its levers into roles, each with one controller account, and the deployer sets each role to one of three modes.

| Mode | Controller | What holders must trust | Closest surveyed example |
|------|------------|-------------------------|--------------------------|
| Immutable | None: renounced at deploy or later, one way | The code and the values fixed at deploy | Liquity V1 and V2 |
| Separate governance | A multisig program account. Governance can take any form, even with its own on-chain programs, but nothing in code ties it to the CDP: the signers carry out its decisions by choice | The signers; any process or delay binds them only socially, unless the core enforces a delay | Synthetix V2 (4 of 7 Safe, no delay; councils vote, nothing on chain binds the Safe); Morpho Blue owner (6 of 10 Safe, narrow powers) |
| Program gated | The governance program itself is the controller, so the CDP accepts a change only after that program's process has run | The gate program's rules and its voters | Sky/Maker (vote, then 48 hour pause, plus bounded fast paths); crvUSD (7 day DAO vote, plus a 5 of 9 Emergency DAO); Frankencoin (anyone proposes, qualified holders veto) |

The core only checks that the caller is authorised as the role's controller (a signer, or a program acting through a chained call), so modes can be mixed per role. The deployer is not a role: it sets every controller at deploy and keeps only what it assigns to itself.

## Roles

| Role | Levers | Notes |
|------|--------|-------|
| Program upgrade | Code upgrades | Native on LEZ (below). Must be immutable for any other renounce to be final |
| Listing | Collateral list, synth list | Removing stops new debt; never seizes positions |
| Risk | Collateral ratios, liquidation penalty, debt ceilings | Within bounds in code |
| Oracle | Feeds, signer set, staleness rules | See [[patterns/collateral-debt-units]] for the requirements |
| Fees | Mint fee, borrower rate, fee recipient | Within bounds in code |
| Guardian | Pause new minting | Cannot block repay, top up, withdrawal of excess collateral or liquidation; expires on its own |
| Shutdown | Global settlement | Starts the exit path |

## Rules in every mode (proposed)

1. Renounce is per role and one way; transfer takes two steps (nominate, accept).
2. A renounced role is only as final as Program upgrade. Front ends show each lever's effective mode as the weaker of its role and Program upgrade.
3. Bounds and rate limits live in code. Evidence: Synthetix V2's caps did not bind its owner, who could swap the settings contract; Sky's SP-BEAM bounds rates to 2% to 30%, at most 4 points per change, once per 16 hours; Morpho's owner can only enable options.
4. No role can move a position's collateral or stop a borrower from repaying and withdrawing.
5. Every change and controller change emits an event, and each role's controller is readable on chain.

Whatever oracle is chosen brings its own admin, whatever mode the roles are in: Liquity has no admin, but Chainlink's ETH/USD proxy is owned by a 4 of 9 multisig ([[projects/liquity]]).

## Open choices

- Minimum delay enforced by the core per role, fixed at deploy (recommended; Drift lost about $285M to $295M through a 2 of 5 council with no timelock, see [[projects/drift]]).
- Guardian as its own fast, pause-only, time-limited role (recommended). crvUSD's Emergency DAO is the nearest example but broader: it can also swap rate contracts.
- Whether Risk changes apply to open positions or only new ones.
- Whether to ship a reference Program gated controller or only define the interface.

## Logos RFPs this builds on

Read from `logos-co/rfp` at `920fad9` (2026-11-06).

- [RFP-001 Admin Authority Library](https://github.com/logos-co/rfp/blob/master/RFPs/RFP-001-admin-authority-lib.md): admin authority set at initialisation, can transfer or revoke (renounce), gates privileged instructions; "There can only be one admin authority (signer) at a time"; a deployed PDA as authority is a soft requirement. Gaps: one authority per program rather than per role; transfer is one step.
- [RFP-002 Freeze Authority Library](https://github.com/logos-co/rfp/blob/master/RFPs/RFP-002-freeze-authority-lib.md): freeze authority set at initialisation and changed or revoked by the admin authority; can freeze the program ("rejecting any attempt to interact with it") or a single `AccountId`. Both conflict with the Guardian limits and rule 4 (a full freeze blocks repay and liquidation; an account freeze can block one borrower from repaying).
- [RFP-020 RedStone Off-Chain Oracle Adaptor](https://github.com/logos-co/rfp/blob/master/RFPs/RFP-020-redstone-oracle-adaptor.md) (assumed delivered by 0.4): push aggregator writing an RFP-019 price account (the only read path for private execution) and public-mode pull; M of N signers (default 3 of N); per-feed `maxAge`; feeds and signer sets managed by an RFP-001 admin authority, which is an upstream admin the CDP inherits.

## What LEZ provides

Read from `logos-blockchain/logos-execution-zone`, branch `dev`, commit `cb075d5` (2026-11-13).

- User programs are deployed through the program loader (`lee/program_loader/src/lib.rs`) as a header account pointing to a chain of bytecode segments. `ProgramHeader` carries `immutable: bool` ("Once `true`, this header can never be updated again"). `UpdateHeader` asserts the old header is mutable and that the header account is authorised; it can also flip `immutable` to true (test `update_header_flip_to_immutable_lands_immutable_mirror_commitment` in `lee/state_machine/src/state/tests/deploy.rs`).
- `ReceiveInput.is_authorized`: "Input is authorized when signed by the receiver or authorized by owning program" (`lee/state_machine/core/src/program/mod.rs`); programs derive PDAs from their own account ID. Inference: a header held at a multisig's or gate program's PDA would be upgradable only through that program. No test in the repo covers this; to confirm with the LEZ team.
- System programs (clock, fee, bridge and others) are installed and upgraded by a threshold of the sequencer committee with a scheduled activation height; `system_upgrader` is seeded immutable and only a hard fork changes it (`lez/programs/system_upgrader/README.md`).
- No mechanism to freeze a user program was found (keyword search for freeze, frozen, blacklist and denylist in `lee` and `lez`; not an audit).
- An LEZ multisig exists: `logos-co/lez-multisig` (Squads v4 style: create, propose, approve M of N, execute through `ChainedCall` to any program). Last commit `a057fc4` on 2026-03-31, which predates the current LEZ code; compatibility not verified.
