---
tags: [appendix, cdp, lez, notion, map, terms]
related: [rfp/appendix/cdp-admin-switch, patterns/collateral-debt-units, patterns/cdp-peg-mechanisms, rfp/appendix/cdp-isolation, rfp/appendix/cdp-risk-privacy-market, rfp/appendix/cdp-oracle]
---

# CDP on LEZ: map from the Notion page to this vault

The Notion page "CDP-Backed Synthetics" holds the structure, wording and decisions for the synth CDP program on LEZ (sXMR and sBTC are the examples). This vault holds only the evidence behind the page's claims. This note says which vault notes hold the evidence for each part of the page, and keeps the page's terms so the evidence notes use the same words. Written 2026-11-17; reduced to a map on the same day after the fact-check.

## Part map

| Notion part | Vault notes holding its evidence | What they hold |
|---|---|---|
| 1 The rules | [[rfp/appendix/cdp-admin-switch]] | Who can change each rule in six protocols (category matrix), general governance per protocol, admin structures elsewhere, the RFP-001, RFP-002, RFP-005 and RFP-020 readings, what LEZ provides for program upgrades |
| 2 Units and oracle requirements | [[patterns/collateral-debt-units]] | The comparison table with sources, collateral with no outside market |
| 3 Peg mechanisms | [[patterns/cdp-peg-mechanisms]] | Each mechanism with the protocols that use it, numbers and sources, mechanisms seen but not on the page's table |
| 4 How a deployment shares risk | [[rfp/appendix/cdp-isolation]] (filename kept), [[patterns/cdp-vs-morpho-trust]] | The five-protocol loss table, protocol evidence, precedent findings; per-protocol notes [[projects/liquity]], [[projects/indigo]], [[projects/hylo]], [[projects/frankencoin]], [[projects/synthetix-v2]], [[projects/alchemix]], [[projects/monolith]] |
| 5 Who carries the risk | [[rfp/appendix/cdp-risk-privacy-market]] | Who mints, the loss path and who liquidates |
| 6 Oracle | [[rfp/appendix/cdp-oracle]] | Existing oracle designs, the RFP-019 and RFP-020 readings, feed coverage |
| 7 Privacy | [[rfp/appendix/cdp-risk-privacy-market]] | What the vault holds on private interaction |
| 8 Market | [[rfp/appendix/cdp-risk-privacy-market]] | Market evidence |

Cross-protocol evidence used by several parts: [[patterns/cdp-synthetics-landscape]] (50-protocol survey; sections 4a and 4b for admin control), [[patterns/oracle-pricing-model]], and the project notes [[projects/sky-maker]], [[projects/crvusd]], [[projects/metronome-synth]], [[projects/youves]], [[projects/mirror-protocol]].

## Terms (as on the page)

- **Synth:** a token the CDP mints against collateral, such as sXMR or sBTC.
- **Synth stablecoin:** a synth that tracks a stable unit, such as a USD synth.
- **Collateral stablecoin:** a stablecoin that a borrower deposits as collateral, such as the reflexive stablecoin (RFP-013).
- **LGO:** the page's name for the Logos native gas token (not found in `logos-docs`, `rfp` or `ecosystem`; unverified).
- **First-loss pool:** a pool of the synth, filled by opt-in depositors, that takes losses before other holders do. Liquity and Indigo call theirs a Stability Pool, Hylo an Earn Pool, and Frankencoin the FPS reserve.
- **Position:** what a borrower opens by locking collateral and minting synth against it. It records that collateral and that debt; a borrower can open several. Liquity calls it a Trove.
- **Deployment:** one running instance of the software, with its own tokens, state and roles.
- **Branch:** one collateral paired with one synth, with its own ratio, feed and debt ceiling, and optionally its own first-loss pool and shutdown.
- **Debt ceiling:** the most synth a branch can mint.
- **Reserve:** value the protocol holds itself, such as Sky's surplus buffer, rather than opt-in deposits.
- **Cross-margin:** one position that holds several collaterals or synths.
- **Redemption:** handing the synth back for collateral at face value (Liquity). A redemption price, as in RAI, is something else: an internal target price.
- **Shield and unshield:** moving tokens from a public LEZ account to a private one, and back.
- **Oracle provider and feed:** a provider (such as RedStone) publishes signed prices; a feed is one price, such as XMR/USD.

Wording used in the evidence notes, to match the page: "lever" for an admin rule (Part 1); "peg mechanism" and "rate mechanism" for Part 3; "synth" is generic; "the reflexive stablecoin (RFP-013)", never "RAI-style"; Maker's 48 hours is a "delay" (Maker calls the contract the pause); "first-loss pool" is the generic term. The Part 3 table does not list redemption at face value as a peg mechanism; the vault's evidence for it ([[projects/liquity]] calls redemption the hard floor) is kept under "seen in the survey" in [[patterns/cdp-peg-mechanisms]].

## Known gaps in the vault's evidence

- Indigo contract source and V3 behaviour were not read; the shortfall rule is from the V2-era whitepaper ([[projects/indigo]]).
- The Liquity V2 Stability Pool balance read covers the WETH branch ([[projects/liquity]]).
- Most chain and source facts in the Part 1 matrix are from the 2026-11-16 reads in the project notes; the 2026-11-17 fact-check repeated some of them.
- Oracle feeds: Pyth publishes an XMR/USD feed (`Crypto.XMR/USD`, "MONERO / US DOLLAR", id `46b8cc93…c321d`, Hermes API, read 2026-11-17); its publisher count is unverified. RedStone XMR/USD rests on RFP-020 and, per the 2026-11-17 fact-check (not repeated), a RedStone token registry last changed in 2021; no live RedStone XMR feed was found ([[rfp/appendix/cdp-oracle]]).
- The RFP texts (`logos-co/rfp` at `920fad9`) and the LEZ repository (`cb075d5`, `dev`) were re-read from local clones on 2026-11-17 ([[rfp/appendix/cdp-admin-switch]]).
