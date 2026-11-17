---
tags: [pattern, cdp, morpho, trust, governance, collateral]
seen_in: [synthetix-v2, liquity, sky-maker, indigo, monolith, alchemix]
---

# CDPs versus Morpho: where does the collateral agreement live?

Question: can a CDP be as trust-minimised as Morpho Blue, or does someone have to agree which collateral is accepted? Short answer: a CDP that issues one shared, fungible token per synth needs a collective agreement on the collateral behind that synth, because every holder of the token is exposed to every collateral type that backs it. Morpho avoids this by moving the agreement from the protocol to each lender. Morpho is not agreement-free either: its owner is a multisig with a narrower power set.

Evidence labels: "chain" means read from Ethereum mainnet on 2026-11-16 (`eth_call`, `eth_getStorageAt` via `ethereum-rpc.publicnode.com`; method as in [[patterns/cdp-synthetics-landscape]] sections 4a and 4b); "design" means standard design of the protocol from memory, not re-read from source in this session; treat as unverified.

## Morpho Blue admin surface

| Item | Result | Label |
|------|--------|-------|
| Contract | `0xBBBBBbbBBb9cC5e90e3b3Af64bdAF62C37EEFFCb`; no EIP-1967 implementation slot | chain |
| `owner()` | `0xcBa28b38103307Ec8dA98377ffF9816C164f9AFa`, a Safe with 6 of 10 signers | chain |
| Fee recipient | zero address (no fee being routed) | chain |
| Owner-gated functions present in bytecode | `enableIrm`, `enableLltv`, `setFee`, `setFeeRecipient`, `setOwner`; no `upgradeTo`. `createMarket` is present | chain (selectors only; access control not read) |
| Safe guard, modules, timelock | not checked | not checked |
| Market creation | Permissionless but limited to owner-enabled interest rate models (IRMs) and loan-to-value limits (LLTVs); each market's oracle is chosen by its creator; the owner cannot change an existing market | design |
| Vault curators (MetaMorpho) | Allocate lender funds across markets | design |

So Morpho's governance is not zero: a 6-of-10 Safe controls the allow-list of IRMs and LLTVs, the fee and its own succession. What it cannot do (by the absence of an upgrade function and a proxy slot) is rewrite the lending logic or an existing market.

## Why a CDP is different

In Morpho the loan asset already exists (for example USDC). A market is a bilateral pairing of one loan asset, one collateral, one oracle, one IRM and one LLTV. A lender chooses the market; a bad collateral hurts only the lenders who chose it. The "agreement" is individual and opt-in, and it is priced.

In a CDP the protocol creates the liability. Whoever holds a synth is, in effect, a passive lender to every collateral type that backs that synth, and cannot opt out of a single one. A bad collateral type damages the peg for every holder of that synth, so the collateral set cannot be left to each user. Something has to bound it:

| Escape | How agreement is reached | Examples | Cost |
|--------|-------------------------|----------|------|
| Fix it at deployment | Hard-coded; owner renounced | Liquity V1 (ETH only; chain: owner of TroveManager and BorrowerOperations is the zero address) | Cannot add collateral; V2 adds wstETH and rETH as separate markets in a new deployment, with separate stability pools |
| Collective veto | Anyone proposes; token holders veto in a window | Frankencoin (see [[patterns/cdp-synthetics-landscape]] section 4b) | Plutocratic veto; trust moves to share-holders, and the minter set and position contracts were not read |
| Governance list | A DAO or council decides each collateral and parameter | Sky/Maker (48 hour delay), crvUSD (7 day Curve DAO vote), Synthetix V2 (4-of-7 Safe, no delay) | Admin power over the whole system; the fastest to react and the largest trust surface |
| Separate token per branch (Morpho-style) | Each branch (one collateral paired with one synth) issues its own token, and that token is the synth, so each holder opts in to one risk | Monolith: each stablecoin is made from one collateral token and one price feed, immutable at the latest four years after deployment ([docs](https://docs.monolith.market/), accessed 2026-11-16; more in [[projects/monolith]]) | Fragmented liquidity: several tokens for the same synth; trust returns at the market and oracle |
| One token per synth, with first-loss pools | One fungible token per synth, however many collaterals back it, with opt-in first-loss pools (per collateral type or per synth) | Liquity V2 per-collateral stability pools (one token across collaterals, with a narrow loss group: one collateral); Indigo, a multi-synth product, with a Stability Pool per iAsset ("Each iAsset, such as iBTC, has its dedicated Stability Pool", [Indigo docs](https://docs.indigoprotocol.io/readme/stability-pools-and-liquidations.md), accessed 2026-11-16; losses beyond the pool: the whitepaper says the shortfall waits frozen in the position, see [[projects/indigo]]); Hylo xASSET holders and one Earn Pool shared by all pools; Frankencoin FPS as reserve (chain: `ZCHF.reserve()` is the FPS contract; first-loss role is from design docs, not re-verified) | Does not fragment the synth into several tokens, but only bounds the shared exposure: losses beyond the first-loss pool reach every holder of that synth and its peg |

The synth's fungibility is what forces the agreement. A separate token per branch (so several tokens for the same synth) removes it but fragments liquidity. One token per synth with first-loss pools keeps the token fungible and bounds the damage with the pool, but a loss larger than the pool still reaches every holder of that synth. Morpho has no such tail because the loan asset is issued by someone else and market bad debt falls only on that market's lenders. Where the table rests on Liquity, Frankencoin or crvUSD, the precedent is single-synth; the multi-synth evidence is in [[rfp/appendix/cdp-isolation]].

## Evidence added 2026-11-17

- **Monolith (separate token per branch).** The only surveyed example. Anyone can deploy a stablecoin "by providing a collateral token address and a price feed"; each stablecoin "becomes irreversibly immutable when reaching a maximum immutability deadline of four years since deployment" (docs landing page, re-read 2026-11-17). A factory operator can set a global fee (cap 10%) paid to a fee recipient from each instance's global reserves; on chain both factories had fee 0 and no fee recipient on 2026-11-17. So even here a human holds a switch: the factory operator, though with a bounded power. Detail, addresses and conflicts in [[projects/monolith]] (docs and chain, from the Notion session; chain reads not repeated in this pass).
- **Alchemix (one synth per deployment).** The Alchemix docs say "Each alAsset on each chain is managed by a dedicated Alchemist" and "Alchemists can have multiple yield strategies but can only issue one type of alAsset" ([Alchemist page](https://alchemix-finance.gitbook.io/user-docs/alchemix-ecosystem/alchemist.md), both re-read verbatim 2026-11-17; V3 not checked). That is one synth per deployment; the comparison with multi-synth designs is in [[rfp/appendix/cdp-isolation]] (Alchemix section).

## What stays trusted even in the best case

- **Oracle.** Morpho puts the oracle choice on each market creator and its lenders; CDPs that price or liquidate through an oracle carry the same risk (Ostium, Metronome, Mirror in this vault). Frankencoin avoids an oracle in liquidation (per docs) at the price of proposer-set liquidation prices and share-holder veto.
- **Signers or voters.** Morpho Blue's owner can only widen or narrow what new markets may use; the CDP governance sets above can change what existing positions are exposed to.
- **Off-chain curation.** MetaMorpho-style curators and Frankencoin-style veto holders both move a human judgement off-protocol.

## For synthetic non-USD assets (inference, not a vault finding)

Morpho-style lending can only borrow assets that already exist; a market for an equity or commodity token still depends on whoever issued that token (custodian, SPV or a CDP). So Morpho does not remove the issuer trust for synthetics; it lets lenders price it. A synthetic that must be minted by the protocol faces the shared-liability problem above. The design space in between (a separate token per branch, oracle chosen per market, a fixed list of permitted pairs) is what I did not find in the survey and have not evaluated.

## Not checked

- Safe guard, modules or timelock on Morpho's owner; function-level access control on `enableIrm`, `enableLltv` and `createMarket` from source.
- Which LLTVs and IRMs are currently enabled, and how many markets exist.
- MetaMorpho vault curation.
- Monolith's liquidation design beyond what [[projects/monolith]] records; how large the first-loss pools are relative to collateral risk in the one-token-per-synth designs.
