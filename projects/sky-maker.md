---
tags: [project, ethereum, cdp, stablecoin, synthetics, governance]
ecosystem: Ethereum (USDS also on 7 other chains)
category: Synthetics/CDP stablecoin
status: live (rebranded from MakerDAO/DAI to Sky/USDS; DAI still in circulation)
status_date: 2026-11-16
website: https://sky.money
docs: https://developers.skyeco.com
launched: [NOT FOUND in primary sources retrieved]; DefiLlama TVL series starts 2019-01-04
---

# Sky Protocol (formerly MakerDAO): DAI and USDS

Status: live and by far the largest CDP stablecoin. Users lock collateral in a Vault and draw DAI or USDS, a USD-pegged synthetic dollar. It is included as the reference per-position CDP design and as the best-documented failure case for auction liquidations (Black Thursday, March 2020). Sky now also lends to "allocators" and holds real-world assets and stablecoin reserves, so a large share of supply is no longer backed by user CDPs. That share is not quantified in the sources retrieved: [NOT FOUND].

## Adoption metrics

| Metric | Value | Date | Source |
|--------|-------|------|--------|
| TVL, Sky Lending | $3.756B (plus $555.8M in a separate "staking" series) | 2026-11-16 | [DefiLlama Sky Lending](https://api.llama.fi/protocol/sky-lending) |
| TVL peak | $19.82B | 2021-12-01 | [DefiLlama Sky Lending](https://api.llama.fi/protocol/sky-lending) |
| DAI circulating | $4.623B | 2026-11-16 | [DefiLlama stablecoins](https://stablecoins.llama.fi/stablecoins) |
| USDS circulating | $5.351B | 2026-11-16 | [DefiLlama stablecoins](https://stablecoins.llama.fi/stablecoins) |
| Fees: 24h / 30d / 1y / all time | $934,505 / $27.40M / $381.18M / $1.30B | 2026-11-16 | [DefiLlama fees overview](https://api.llama.fi/overview/fees) |
| Revenue (fees minus savings rate paid): 30d / 1y / all time | $13.52M / $189.20M / $784.98M | 2026-11-16 | [DefiLlama revenue overview](https://api.llama.fi/overview/fees?dataType=dailyRevenue) |

Check on units: the $3.756B TVL is far below the $9.97B combined DAI and USDS supply. The gap is consistent with supply backed by non-CDP reserves (PSM, RWA, allocator assets), but the split is [NOT FOUND].

## How it works

### User perspective

A user opens a Vault for a collateral type ("ilk"), deposits collateral, draws DAI or USDS up to the ilk's liquidation ratio, and pays a per-ilk stability fee accruing in the debt. Repaying returns the collateral. Holders of DAI or USDS may deposit into the DSR or SSR savings contracts.

### Protocol perspective

- **Stability fee accounting:** the Vat keeps a cumulative rate per ilk, debt is `art * rate`, and the Jug sets a per-second fee equal to the ilk `duty` plus a global `base`. Fee revenue increases the Vow's surplus. Source: [Sky rates overview](https://developers.skyeco.com/protocol/rates/ovreview/).
- **Liquidation 2.0 (Dutch auction):** `Dog.bark` seizes the whole Vault and calls `Clipper.kick`. Start price is the oracle (OSM) price plus a buffer `buf`; a pluggable calculator lowers the price over time; auctions reset after a `tail` timeout or when price falls below `cusp` times the start price. The penalty is `chop` (example in the docs: 1.13 for a 13% penalty). Keepers receive a flat `tip` and a percentage `chip`. Global and per-ilk `hole` caps limit DAI needed by active auctions. A four level circuit breaker can stop kicks, resets and takes. Bidders can use flash-lent funds. Source: [Sky collateral liquidation](https://developers.skyeco.com/protocol/vaults/collateral-liquidation/).
- **Peg:** DAI debt is repaid at $1 per unit; arbitrage through Vault repayment, the Peg Stability Module (USDC swaps, per DefiLlama's fee methodology) and the savings rate. PSM fee and DSR/SSR current values: [NOT FOUND].
- **Oracle delay:** the OSM price is one to two hours behind the market by design (same Sky docs page), a safety delay against oracle attacks that also creates liquidation lag.

## Key behaviours

- [[patterns/synthetic-collateral-models]] : multi-collateral CDP with governance-set per-collateral risk parameters.
- [[patterns/oracle-pricing-model]] : delayed oracle (OSM) plus Dutch auction; the 2020 stale-price failure is the origin of both.
- [[patterns/cdp-synthetics-landscape]] : per-position CDP reference design.

## Architecture decisions

- **Governance-managed risk parameters per collateral type:** flexible, but requires continuous governance and an off-chain risk function (the Atlas and Stability Scope, not retrieved).
- **Move from English-style auctions (Liquidation 1.0) to Dutch auctions (2.0):** after the March 2020 zero-bid events, with instant settlement and flash-loan-compatible bidding. Quantstamp audited Liquidations 2.0 (link in [DefiLlama Sky Lending](https://api.llama.fi/protocol/sky-lending)).
- **Rebrand to Sky and USDS:** date and mechanics of the DAI to USDS upgrade path are [NOT FOUND] in the primary documents retrieved; DefiLlama now lists both DAI and USDS.

## Differentiators

- Scale: $3.76B TVL and $9.97B combined DAI and USDS supply on 2026-11-16.
- Longest operating history of any decentralised stablecoin; its main failure mode (Black Thursday) is the best-studied of any CDP.
- Revenue: DefiLlama shows $189.2M revenue over the last year.

## Limitations and criticisms

- **Black Thursday, 12 March 2020:** ETH fell roughly half over 12 to 13 March; the Medianizer oracle did not update under congestion and gas costs, and bidders won collateral in auctions for 0 DAI. Quantification conflicts across sources (see below). Debt auctions then minted MKR; 20,980 MKR were sold for 5.3M DAI and Paradigm won 68% of the 106 auctions (The Block). MKR holders voted not to compensate affected Vault owners (Decrypt, The Defiant).
- Centralisation of collateral: USDC and RWAs entered as collateral after 2020 (Cointelegraph notes USDC was added after Black Thursday).
- Third-party keeper hygiene: on 2026-10-06 a dormant third-party ETH-A liquidation keeper bot was drained of about $538k (about 200 WETH) via an unprotected function; this was not a protocol contract failure (DefiLlama hacks list description).
- Governance complexity and rebrand churn; token (MKR to SKY) migration risk: [NOT FOUND] for details.

### Fees and borrower cost

| Item | Value | Date | Source |
|------|-------|------|--------|
| Stability fee per Vault type | Set per ilk as `duty + base`; the current per-ilk percentages are [NOT FOUND] (Sky docs and daistats pages retrieved contained no rates; search snippets gave 5 to 8% on ETH vaults but from low-quality aggregators and are not used) | 2026-11-16 | [Sky rates overview](https://developers.skyeco.com/protocol/rates/ovreview/) |
| Liquidation penalty | `chop` per ilk, docs example 13%; current values [NOT FOUND] | n/a | [Sky collateral liquidation](https://developers.skyeco.com/protocol/vaults/collateral-liquidation/) |
| Keeper incentive | flat `tip` (docs example 100 DAI) plus `chip` (docs example 2% of the target debt) | n/a | same |
| Revenue booked | stability fees, liquidation income, PSM fees minus savings rate paid | 2026-11-16 | [DefiLlama fees methodology](https://api.llama.fi/overview/fees) |
| Fee run rate | $381.18M in fees and $189.20M in revenue over 1 year; $27.40M and $13.52M over 30 days | 2026-11-16 | [DefiLlama fees overview](https://api.llama.fi/overview/fees) |
| Fee destination | savings rate (DSR/SSR) to DAI/USDS depositors; remaining revenue to SKY buybacks and staking rewards, per DefiLlama methodology | 2026-11-16 | [DefiLlama fees overview](https://api.llama.fi/overview/fees) |
| DSR/SSR current rate | [NOT FOUND] | n/a | n/a |

Conflict on Black Thursday bad debt: Cointelegraph (author estimate, 2020-04-29) about 6.65M DAI undercollateralised with 1,461 near-zero auctions and 62,843 ETH; DefiLlama hacks list $8.32M; Decrypt cites 2.5M; Linum Labs 5.67M DAI; a Whiterabbit analysis cited by search results gives $8.325M for 1,462 zero-bid auctions. Not reconciled; the post-mortem on the official Maker blog is no longer served (redirects to sky.money). See [Cointelegraph](https://cointelegraph.com/news/maker-debt-crisis-post-mortem-recommends-new-safeguards).

## Admin control: who can change the rules (read 2026-11-16)

Tags: **chain** = read from Ethereum mainnet at block 26,426,703 via `eth_call` and `eth_getStorageAt` against `ethereum-rpc.publicnode.com`; **source** = verified source in the `sky-ecosystem` GitHub repos (commit noted in Sources); **docs** = Sky developer docs, executive vote pages or secondary write-ups. Addresses come from the on-chain ChainLog `0xdA0Ab1e0017DEbCd72Be8599041a2aa3bA7e740F` (version 1.20.22, 508 keys, all dumped).

### Voting layer

- **Root of trust (chain):** the DSPause `MCD_PAUSE` `0xbE28...38f3` has `owner()` = zero address and `authority()` = `0x929d9A1435662357F54AdcF64DcEE4d6b867a6f9`, which is ChainLog `MCD_ADM`. That contract's `gov()` is SKY (`0x5607...9279`, also ChainLog `MCD_GOV`), `live()` = 1, `maxYays()` = 5, `launchThreshold()` = 2,400,000,000 SKY, `liftCooldown()` = 10 blocks. Docs call it "Chief V3 (SKY)"; the previous MKR chief `0x0a3f...ddc0` still holds 431.7 MKR but has no Vat ward and is not the pause authority (chain). The swap to a SKY chief was approved in the "MKR-to-SKY Upgrade Phase One" executive of 15 May 2025 (docs).
- **How a change is made (source):** SKY holders `lock` SKY in the Chief and `vote` for a slate of up to 5 candidate addresses (continuous approval voting). Anyone can `lift` a candidate to `hat` if it has strictly more approval than the current hat, at most once per 10 blocks. The Chief's `canCall` returns true only for the `hat`, so only the hat can `plot` (schedule) or `drop` a plan in the pause. A spell that becomes hat calls `schedule()`, which plots it with `eta` at least `delay()` = 172,800 s (48 hours) ahead; after `eta` anyone can `cast()`, and the pause `exec`s it by `delegatecall` from the PauseProxy `0xBE8E...98FB`, which holds `wards` on Vat, Jug, Pot, Spot, Dog, End, ESM, sUSDS, USDS, the IlkRegistry and the modules below (chain, each `wards(PauseProxy)` = 1). `MCD_PROTEGO` (chain: `pause()` = the DSPause) lets anyone deploy a single-purpose spell that drops one planned action; it still needs to become hat to act (source).
- **Current hat (chain):** `0xabc702420dcc1868f7f8df06a89da766bcb01136`, description "2026-11-05 MakerDAO Executive Spell", `done()` = true (already cast), `eta` 2026-11-08 18:52 UTC. Approval on it is 7,028,338,153 SKY. Total SKY held by the Chief is 7,137,371,832 of a 23,452,431,916 SKY supply (30.4%). So a new hat, including any emergency spell, needs more than about 7.03B SKY behind it. The SKY held by the Chief includes SKY routed through Vote Delegate V3 contracts and the Staking Engine (docs); how concentrated that is among delegates was not read.
- **Who votes:** SKY holders, directly or via delegate contracts (docs, Vote Delegate V3 `0x4Cf3...cCD5`). MKR no longer votes in the live chief (chain).

### The eight levers

| Lever | Who | Speed | Bounds |
|-------|-----|-------|--------|
| Collateral list (new or removed ilks) | Executive spell via the PauseProxy, which is a Vat ward and IlkRegistry ward (chain). 33 ilks in `ILK_REGISTRY` (chain). Removal in practice: `LINE_MOM.wipe` by the hat sets an ilk's debt ceiling to zero (source, chain: ETH-A is registered in LineMom) | Add: at least 48 h after the spell is scheduled (chain). Freeze new debt on an ilk: instant once a wipe spell is hat (source) | None on chain: a spell has root access to the Vat (source, [emergency spells README](https://github.com/sky-ecosystem/dss-emergency-spells)) |
| Synth list (DAI, USDS) | Spell. USDS (`0xdC035D45...7384F`) and sUSDS (`0xa3931d71...7fbd`) are ERC-1967 proxies whose implementation slot points to `USDS_IMP` and `SUSDS_IMP`; PauseProxy is a ward on both (chain). DAI has no PauseProxy ward (chain; it is minted through `MCD_JOIN_DAI`, not checked) | 48 h (chain) | None (chain: unbounded `wards`) |
| Risk parameters (debt ceiling `line`, `mat`, `chop`, `hole`, `dust`, auction params) | Spell for all of them (chain: PauseProxy ward on Vat, Spot, Dog, Clipper). Debt ceilings are also moved by the permissionless `DssAutoLine.exec` (source; chain: AutoLine is a Vat ward) | Spell 48 h. AutoLine: anyone, any block; raises at most once per `ttl` (ETH-A 21,600 s = 6 h; LitePSM 43,200 s) and lowers instantly (source, chain) | AutoLine ETH-A: maximum line 15B, `gap` 150M above current debt (chain). Spell: none. Current ETH-A (chain): `mat` 1.45, `chop` 1.13, `hole` 40M, global `Hole` 150M, `dust` 7,500, debt about 181.3M against a 329.2M line |
| Oracle (ETH-A worked example) | ETH-A `PIP_ETH` is an OSM (`0x81FE...5763`, `hop()` = 3,600 s, `stopped()` = 0) reading `src()` = `0x46ef...7F4E`, a Chronicle Scribe-style feed ("ETH/USD", `bar()` = 13 of 25 `feeds()`, `opChallengePeriod()` = 600 s) (chain). PauseProxy is a ward of the OSM but not of the Chronicle feed; the feed's `authed()` lists 5 contracts, one of which exposes `getMinDelay()` = 604,800 s (7 days), consistent with a timelock (chain; owners not identified). Feed signers are therefore lifted and dropped by Chronicle, not by Sky governance (chain inference from `authed()`). Sky can `stop` the OSM via `OSM_MOM.stop` (hat, instant) or replace `src` by spell (chain, source) | OSM delay 1 h on every price (chain). OSM stop: instant once hat. Source swap: 48 h. Chronicle side: [NOT FOUND] beyond the 7 day timelock reading | `ClipperMom.tolerance(ETH-A clipper)` = 0.5: anyone can trip the liquidation breaker if the next OSM price is below 50% of the current (chain, source). No bound on what a spell can set |
| Fees and rates (stability fee, DSR, SSR) | Spell (PauseProxy ward of Jug, Pot, sUSDS; chain). Also SP-BEAM `0x36B0...2b22`, a Jug, Pot and sUSDS ward (chain), callable by its bud `0xe1c6f81D0c3CD570A77813b81AA064c5fff80309`, a Safe 1.4.1 with threshold 2 of 3 owners (chain). Docs: a Stability Advisor proposes, the Stability Facilitator approves, an approved operator executes ([Block Analitica](https://blockanalitica.substack.com/p/sky-sp-beam-overview)) | SP-BEAM: one batch per `tau` = 57,600 s (16 h), last used 2026-10-07 (chain). Spell: 48 h. `SPBEAM_MOM.halt` by the hat: instant (source) | SP-BEAM (chain, `cfgs`): ETH-A, ETH-B, ETH-C, WSTETH-A, WBTC-A and SSR 2% to 30% with at most 4 percentage points per change; DSR 0% to 30%, same step. Current (chain): `base` 0, ETH-A 10.50% a year, DSR 1.25%, SSR 3.80%; LitePSM `tin` and `tout` 0. Spell: no bound |
| Pause (emergency stops) | The hat, through MOM contracts whose `owner()` is the PauseProxy and `authority()` is the Chief (chain, all ten read: OSM, Clipper, Line, SP-BEAM, LitePSM, Splitter, stUSDS, Direct, Starknet escrow, PAS). Pre-deployed emergency spells are listed in the ChainLog (`EMSP_*`: global OSM stop, global clip breaker, global line wipe, SP-BEAM halt, splitter stop, LitePSM halt factory and others) (chain, source). Permissionless: `ClipperMom.tripBreaker` within the 50% tolerance (source). No guardian multisig with pause power found | Instant once the emergency spell is hat, which today means outvoting about 7.03B SKY (chain). ETH-A clipper `stopped()` = 0 (chain) | Each MOM can only stop, halt or wipe; none can unpause or reconfigure (source). Unpausing is by spell, 48 h |
| Code upgrades | Spell. Core modules (Vat, Jug, Dog, Clipper, OSM) are not proxies; upgrades are new contracts wired in by `rely`/`deny` from the PauseProxy. USDS and sUSDS are proxies upgradeable by the PauseProxy (chain, ERC-1967 slot read) | 48 h (chain) | None (source: spells have root access) |
| Shutdown | `End.cage()` is `auth`; wards include the PauseProxy and the ESM (chain). ESM `0x09e0...4c58`: `gem()` = MKR (not SKY), `min()` = 2^256 minus 1, `Sum()` = 0.0013 MKR, `live()` = 1 (chain). Docs: End and ESM are "deprecated"; the threshold was set to infinity in the May 2025 upgrade (docs) | Spell: 48 h, then `End.wait()` = 262,800 s (73 h) cooldown before `thaw` (chain) | ESM unreachable unless a spell lowers `min` (PauseProxy is an ESM ward, chain). Exit for holders: `pack` DAI, then `cash` per ilk for a fixed collateral basket (docs, archived). USDS exit path not documented: [NOT FOUND]; likely via the DAI/USDS converter first, unverified |

### Delegated and bounded paths that bypass the 48 hour delay

1. **SP-BEAM (rates):** a 2 of 3 Safe can move stability fees, DSR and SSR within the caps above every 16 h. Worst case per update: plus or minus 4 percentage points on each configured rate. Initialised by the 17 April 2025 executive with the same `tau`, caps and bud (docs, matches chain).
2. **stUSDS rate setter and SBE-BEAM:** similar bounded setters exist (`STUSDS_RATE_SETTER`, `tau` 57,600 s, str 2% to 50% and duty 2.1% to 50% with 5 point steps; `MCD_SBEBEAM`, `tau` 1,800 s) (chain). Their buds were not identified; the SP-BEAM bud is not a bud of either (chain).
3. **DssAutoLine (debt ceilings):** permissionless, bounded by governance-set maximum line, gap and ttl (source, chain).
4. **MOMs and emergency spells:** stop-only powers, instant, but gated by the hat, so they bypass the delay only, not the vote (chain, source).
5. **ClipperMom.tripBreaker:** permissionless, only if the oracle's next price is below 50% of the current (chain, source).
6. **Chronicle oracle:** the ETH/USD source contract is administered by Chronicle, not by Sky governance (chain); Sky's protection is the 1 h OSM delay plus the hat's ability to stop the OSM.

### Method and what was not checked

- `eth_getLogs` was not used, so `wards` and `buds` maps could not be enumerated. I checked specific addresses only (PauseProxy, MOMs, ESM, End, SP-BEAM, AutoLine). Other wards on Vat, the OSM or the Clipper (for example a Chronicle or keeper address on the OSM) may exist and were not ruled out.
- The 5 `authed()` addresses on the Chronicle feed were not identified beyond one exposing `getMinDelay()` = 7 days; the 25 feed signers were not attributed.
- Star `SubProxy` and `StarGuard` contracts (Spark, Grove and others) and allocator vaults were not analysed; StarGuard's README expects only the PauseProxy as ward (source), not checked on chain.
- Delegate concentration, the Atlas text defining the Stability Facilitator role, and spell office hours were not read. A 2026 governance poll proposed renaming the Stability Facilitator role (search result only).
- In March 2025 an executive stated the GSM delay was 18 hours ([oracle migration executive](https://vote.makerdao.com/executive/template-executive-vote-eth-and-wsteth-oracle-migration-rate-changes-smart-burn-engine-parameter-update-bug-bounty-payout-aligned-delegate-compensation-atlas-core-development-payments-integration-boost-top-up-spark-proxy-spell-march-20-2025)); it reads 48 hours now. When it changed was not checked.
- **Conflicts:** Sky's [security mechanisms page](https://developers.skyeco.com/security/security-measures/security-mechanisms/) says the OSM Mom is controlled by Chronicle; chain shows `OSM_MOM.owner()` = PauseProxy and `authority()` = Chief, so Chronicle can use it only if it holds the hat (a direct Chronicle ward on the OSM was not ruled out). The same page says the ESM "can be triggered by a subset of SKY holders"; chain shows the ESM `gem()` is still MKR with an infinite threshold.

## Sources

- [DefiLlama Sky Lending API](https://api.llama.fi/protocol/sky-lending) : accessed 2026-11-16
- [DefiLlama stablecoins list](https://stablecoins.llama.fi/stablecoins) : accessed 2026-11-16
- [DefiLlama fees overview](https://api.llama.fi/overview/fees) : accessed 2026-11-16
- [DefiLlama hacks list](https://api.llama.fi/hacks) : accessed 2026-11-16
- [Sky developer docs: collateral liquidation](https://developers.skyeco.com/protocol/vaults/collateral-liquidation/) : accessed 2026-11-16
- [Sky developer docs: rates overview](https://developers.skyeco.com/protocol/rates/ovreview/) : accessed 2026-11-16
- [Cointelegraph: Maker debt crisis post-mortem](https://cointelegraph.com/news/maker-debt-crisis-post-mortem-recommends-new-safeguards) : accessed 2026-11-16
- [The Block: MKR auctions conclude](https://www.theblock.co/post/60447/mkr-auctions-conclude-with-crypto-venture-firm-paradigm-emerging-as-the-biggest-winner) : accessed 2026-11-16 (search summary)
- [Decrypt: Maker votes not to compensate Black Thursday victims](https://decrypt.co/42992/maker-votes-not-compensate-black-thursday-victims) : accessed 2026-11-16 (search summary)
- [Sky ChainLog contract 0xdA0Ab1e0017DEbCd72Be8599041a2aa3bA7e740F](https://etherscan.io/address/0xdA0Ab1e0017DEbCd72Be8599041a2aa3bA7e740F) : read on chain via `https://ethereum-rpc.publicnode.com` (`count()`, `get(uint256)`), accessed 2026-11-16; all other chain values in the admin control section were read through the same endpoint at block 26,426,703 on 2026-11-16
- [sky-ecosystem/chief, Chief.sol (commit ed7f912)](https://github.com/sky-ecosystem/chief/blob/master/src/Chief.sol) : accessed 2026-11-16
- [sky-ecosystem/ds-pause, pause.sol (commit 5e798dd)](https://github.com/sky-ecosystem/ds-pause) : accessed 2026-11-16
- [sky-ecosystem/sp-beam, SPBEAM.sol and SPBEAMMom.sol (commit bc88579)](https://github.com/sky-ecosystem/sp-beam) : accessed 2026-11-16
- [sky-ecosystem/dss-auto-line (commit bff7e6c)](https://github.com/sky-ecosystem/dss-auto-line) : accessed 2026-11-16
- [sky-ecosystem/dss-lite-psm, DssLitePsmMom.sol (commit dbf0022)](https://github.com/sky-ecosystem/dss-lite-psm) : accessed 2026-11-16
- [sky-ecosystem/clipper-mom (commit 1473b44)](https://github.com/sky-ecosystem/clipper-mom), [osm-mom (820324a)](https://github.com/sky-ecosystem/osm-mom), [line-mom (5935e56)](https://github.com/sky-ecosystem/line-mom) : accessed 2026-11-16
- [sky-ecosystem/esm (commit 225ddf2)](https://github.com/sky-ecosystem/esm) and [sky-ecosystem/dss end.sol (commit fa4f663)](https://github.com/sky-ecosystem/dss/blob/master/src/end.sol) : accessed 2026-11-16
- [sky-ecosystem/osm (commit e36c874)](https://github.com/sky-ecosystem/osm) and [median (commit 0316acd)](https://github.com/sky-ecosystem/median) : accessed 2026-11-16
- [sky-ecosystem/protego (commit dd96f9d)](https://github.com/sky-ecosystem/protego) : accessed 2026-11-16
- [sky-ecosystem/dss-emergency-spells README (commit ef95bc1)](https://github.com/sky-ecosystem/dss-emergency-spells) : accessed 2026-11-16
- [sky-ecosystem/stusds, StUsdsRateSetter.sol (commit 486bfe3)](https://github.com/sky-ecosystem/stusds) and [star-guard README (commit 707c84d)](https://github.com/sky-ecosystem/star-guard) : accessed 2026-11-16
- [Sky developer docs: Chief](https://developers.skyeco.com/protocol/governance/chief/), [Pause](https://developers.skyeco.com/protocol/governance/pause/), [Spell](https://developers.skyeco.com/protocol/governance/spell/), [OSM](https://developers.skyeco.com/protocol/core/osm/) : accessed 2026-11-16
- [Sky developer docs: security mechanisms](https://developers.skyeco.com/security/security-measures/security-mechanisms/) : accessed 2026-11-16
- [Sky developer docs: Emergency Shutdown (archived)](https://developers.skyeco.com/archive/shutdown/emergency-shutdown/) and [Shutdown, End (archived)](https://developers.skyeco.com/archive/shutdown/shutdown/) : accessed 2026-11-16
- [Sky developer docs: SKY token and governance upgrade, key information](https://developers.skyeco.com/guides/sky/token-governance-upgrade/key-info/) : accessed 2026-11-16
- [Executive vote: MKR-to-SKY Upgrade Phase One, Protego (15 May 2025)](https://vote.makerdao.com/executive/template-executive-vote-mkr-to-sky-upgrade-phase-one-adding-protego-to-the-chainlog-spark-proxy-spell-may-15-2025) : accessed 2026-11-16
- [Executive vote: SP-BEAM initialisation (17 April 2025)](https://vote.makerdao.com/executive/template-executive-vote-sp-beam-initialization-sky-token-rewards-rebalance-set-aave-lido-market-prime-market-ddm-dc-to-0-sbe-changes-launch-project-funding-spark-aave-revenue-share-payment-ad-compensation-atlas-core-development-payments-spark-proxy-spell-april-17-2025) : accessed 2026-11-16
- [Executive vote: ETH and wstETH oracle migration (20 March 2025)](https://vote.makerdao.com/executive/template-executive-vote-eth-and-wsteth-oracle-migration-rate-changes-smart-burn-engine-parameter-update-bug-bounty-payout-aligned-delegate-compensation-atlas-core-development-payments-integration-boost-top-up-spark-proxy-spell-march-20-2025) : accessed 2026-11-16
- [Block Analitica: Sky SP-BEAM overview (7 May 2025)](https://blockanalitica.substack.com/p/sky-sp-beam-overview) : accessed 2026-11-16
