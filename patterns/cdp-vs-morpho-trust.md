---
tags: [pattern, cdp, morpho, trust, governance, collateral]
seen_in: [synthetix-v2, liquity, sky-maker]
---

# CDPs versus Morpho: where does the collateral agreement live?

Question: can a CDP be as trust-minimised as Morpho Blue, or does someone have to agree which collateral is accepted? Short answer: a CDP that issues one shared, fungible liability needs a collective agreement on collateral, because every holder of the token is exposed to every collateral type. Morpho avoids this by moving the agreement from the protocol to each lender. Morpho is not agreement-free either: its owner is a multisig with a narrower power set.

Evidence labels: "chain" means read from Ethereum mainnet on 2026-11-16 (`eth_call`, `eth_getStorageAt` via `ethereum-rpc.publicnode.com`; method as in [[patterns/cdp-synthetics-landscape]] sections 4a and 4b); "design" means standard design of the protocol from memory, not re-read from source in this session; treat as unverified.

## Morpho Blue admin surface

| Item | Result | Label |
|------|--------|-------|
| Contract | `0xBBBBBbbBBb9cC5e90e3b3Af64bdAF62C37EEFFCb`; no EIP-1967 implementation slot | chain |
| `owner()` | `0xcBa28b38103307Ec8dA98377ffF9816C164f9AFa`, a Safe with 6 of 10 signers | chain |
| Fee recipient | zero address (no fee being routed) | chain |
| Owner-gated functions present in bytecode | `enableIrm`, `enableLltv`, `setFee`, `setFeeRecipient`, `setOwner`; no `upgradeTo`. `createMarket` is present | chain (selectors only; access control not read) |
| Safe guard, modules, timelock | not checked | not checked |
| Market creation is permissionless but limited to owner-enabled interest rate models (IRMs) and loan-to-value limits (LLTVs); each market's oracle is chosen by its creator; owner cannot change an existing market | design |
| Vault curators (MetaMorpho) allocate lender funds across markets | design |

So Morpho's governance is not zero: a 6-of-10 Safe controls the allow-list of IRMs and LLTVs, the fee and its own succession. What it cannot do (by the absence of an upgrade function and a proxy slot) is rewrite the lending logic or an existing market.

## Why a CDP is different

In Morpho the loan asset already exists (for example USDC). A market is a bilateral pairing of one loan asset, one collateral, one oracle, one IRM and one LLTV. A lender chooses the market; a bad collateral hurts only the lenders who chose it. The "agreement" is individual and opt-in, and it is priced.

In a CDP the protocol creates the liability. Whoever holds the stablecoin is, in effect, a passive lender to every collateral type at once, and cannot opt out of a single one. A bad collateral type damages the peg for everyone, so the collateral set cannot be left to each user. Something has to bound it:

| Escape | How agreement is reached | Examples | Cost |
|--------|-------------------------|----------|------|
| Fix it at deployment | Hard-coded; owner renounced | Liquity V1 (ETH only; chain: owner of TroveManager and BorrowerOperations is the zero address) | Cannot add collateral; V2 adds wstETH and rETH as separate markets in a new deployment, with separate stability pools |
| Collective veto | Anyone proposes; token holders veto in a window | Frankencoin (see [[patterns/cdp-synthetics-landscape]] section 4b) | Plutocratic veto; trust moves to share-holders, and the minter set and position contracts were not read |
| Governance list | A DAO or council decides each collateral and parameter | Sky/Maker (48 hour pause), crvUSD (7 day Curve DAO vote), Synthetix V2 (4-of-7 Safe, no delay) | Admin power over the whole system; the fastest to react and the largest trust surface |
| Strict isolation (Morpho-style) | Each collateral market issues its own token, so each holder opts in to one risk | Not surveyed | Fragmented, non-fungible stablecoin liquidity; trust returns at the market and oracle |
| Loss-layer isolation | One fungible token, but each collateral type has an opt-in first-loss tranche | Liquity V2 per-collateral stability pools; Hylo xASSET holders; Frankencoin FPS as reserve (chain: `ZCHF.reserve()` is the FPS contract; first-loss role is from design docs, not re-verified) | Does not fragment the token, but only bounds the shared exposure: losses beyond the junior tranche reach every holder and the peg |

The stablecoin's fungibility is what forces the agreement. Strict isolation (a token per collateral market) removes it but fragments liquidity. Loss-layer isolation keeps one token and bounds the damage with a junior tranche, but a loss larger than that tranche still reaches every holder. Morpho has no such tail because the loan asset is issued by someone else and market bad debt falls only on that market's lenders.

## What stays trusted even in the best case

- **Oracle.** Morpho puts the oracle choice on each market creator and its lenders; CDPs that price or liquidate through an oracle carry the same risk (Ostium, Metronome, Mirror in this vault). Frankencoin avoids an oracle in liquidation (per docs) at the price of proposer-set liquidation prices and share-holder veto.
- **Signers or voters.** Morpho Blue's owner can only widen or narrow what new markets may use; the CDP governance sets above can change what existing positions are exposed to.
- **Off-chain curation.** MetaMorpho-style curators and Frankencoin-style veto holders both move a human judgement off-protocol.

## For synthetic non-USD assets (inference, not a vault finding)

Morpho-style lending can only borrow assets that already exist; a market for an equity or commodity token still depends on whoever issued that token (custodian, SPV or a CDP). So Morpho does not remove the issuer trust for synthetics; it lets lenders price it. A synthetic that must be minted by the protocol faces the shared-liability problem above. The design space in between (strict isolation with a token per collateral, oracle chosen per market, a fixed list of permitted pairs) is what I did not find in the survey and have not evaluated.

## Not checked

- Safe guard, modules or timelock on Morpho's owner; function-level access control on `enableIrm`, `enableLltv` and `createMarket` from source.
- Which LLTVs and IRMs are currently enabled, and how many markets exist.
- MetaMorpho vault curation.
- Any CDP design with strict per-collateral issuance, and how large the first-loss tranches are relative to collateral risk in the loss-layer designs.
