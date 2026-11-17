---
tags: [project, ethereum, cdp, stablecoin, synthetics, multi-synth, stability-pool, immutable, bonding-curve, pre-launch]
ecosystem: Ethereum
category: Synthetics/Multi-synth CDP on one shared collateral (pETH)
status: pre-launch (public testnet on Sepolia; no mainnet contracts found)
status_date: 2026-11-17
website: https://polaris.finance (links to x.com/polarisfnd; the older domain polarisfinance.io now says "Polaris has moved to polaris.finance")
docs: https://docs.polaris.finance
launched: not launched (internal testnet 2026-03-04, open testnet 2026-05-07, per the Telegram announcement channel); mainnet date [DATA NEEDED]
---

# Polaris (Ethereum: USDp, GOLDp and other pAssets against pETH)

Status: pre-launch. Polaris is a CDP system on Ethereum, built by a team that grew out of Liquity research, in which every synth ("pAsset") is minted against a single collateral, pETH. pETH is not wrapped ETH: it is a token issued by a Polaris bonding curve against deposited ETH, with an ETH-denominated floor that "can only rise" and a market premium above it ([pETH](https://docs.polaris.finance/core-assets/peth)). The first two pAssets are USDp (US dollar) and GOLDp (gold; "one troy ounce" per Pharos, which a co-founder runs, see Team and funding), both at a 115% minimum collateral ratio ([USDp](https://docs.polaris.finance/core-assets/usdp), [GOLDp](https://docs.polaris.finance/core-assets/goldp)). The docs say "Any asset with an oracle can become a pAsset" and that launching a new pAsset market is permissionless ([pAsset Markets](https://docs.polaris.finance/architecture/passet-markets)); an earlier blog post said a stewardship-granted licence was needed (see Conflicts and gaps). The docs' own agent guidance says: "Polaris is currently on the Sepolia public testnet; testnet assets have no monetary value" ([llms.txt](https://docs.polaris.finance/llms.txt), read 2026-11-17). Written 2026-11-17 from the docs bundle, the project's blog (via the Wayback Machine), its Telegram announcement channel and news coverage. No contract source, audit report or mainnet address was found.

For this vault the design points are: one collateral type shared by many synths, each synth market with its own first-loss pool, oracle setup and stress thresholds; a two-sided peg mechanism (redeem at peg and "buy at peg") whose effects are spread across all positions in the market; a rate mechanism that reads mint and redemption flow instead of a market-price oracle; and an immutable core with a bounded "stewardship" layer.

## Identity: which Polaris is @polarisfnd

Confidence: high.

- polaris.finance links to `https://x.com/polarisfnd`, `https://t.me/polaris_ann`, `https://docs.polaris.finance/` and `https://testnet.polaris.finance` (page HTML fetched 2026-11-17).
- The Telegram channel `polaris_ann`, post 30 of 2026-08-19: "we now are @polarisfnd!" ([t.me/s/polaris_ann](https://t.me/s/polaris_ann)). Earlier posts link to `x.com/polarisfinance_`, so the handle was @polarisfinance_ until 2026-08-19.
- ChainCatcher's report of the angel round links a post from @polarisfnd ([ChainCatcher, 2026-09-04](https://www.chaincatcher.com/en/article/2287671)).

Other projects named Polaris that are not this one:

- **Polaris Finance (Aurora and Telos, 2022).** An algorithmic stablecoin pegged to NEAR by seigniorage, with POLAR, SPOLAR and PBOND tokens ([Polaris Finance on Medium](https://medium.com/@PolarisFinance/viva-la-innovation-in-defi-eca7b169e284); [DefiLlama polaris-finance](https://api.llama.fi/protocol/polaris-finance), TVL $6,441 on 2026-11-17, listed 2022-02-20). DefiLlama records its X handle as `PolarisFinance_` and its URL as `https://polarisfinance.io`, the same handle and domain the new Polaris used until mid-2026. Whether the 2026 team acquired the handle and domain or is the same team is not established; the 2026 origin story (below) does not mention the 2022 project.
- **Name collisions on the token:** Parallel's USDp (a live multichain stablecoin on DefiLlama) and Polymarket's pUSD are unrelated. [[projects/monolith]] lists two Monolith instances named pUSD; no link to Polaris was found.
- A ChainCatcher piece says a "Polaris" launched by Sky is building on Berachain ([ChainCatcher](https://www.chaincatcher.com/en/article/2172525), per a search summary, not opened); unrelated.

## Team and funding

- **Origin.** Per co-founder TokenBrice: "About two years ago, Robert Lauko (founder of Liquity) began researching a novel approach ... using a bonding-curve backed ETH derivative as collateral." The early model was called "NG" (Next Gen). Laurens (CTO) joined the research; Robert Mullins (0xLuude, from The DeFi Collective) became CEO; TokenBrice was CEO at first and is now a strategic adviser. "After having hardened the base model until late Summer 2025, we then got support from some of Liquity's devs." The Polaris name was chosen in October 2025 ([The Path to the North Star, 2026-04-27, archived](http://web.archive.org/web/20260512023301/https://polarisfinance.io/blog/path-to-the-north-star/); [tokenbrice.xyz: Why Polaris, 2026-01-27](https://tokenbrice.xyz/why-polaris/)).
- **Founders round.** "$500k from the team itself, with a Founders Round: every single team member and advisor of Polaris invested" ([Stewardship, not Governance, 2026-03-03, archived](http://web.archive.org/web/20260304063627/https://polarisfinance.io/blog/stewardship-not-governance/)).
- **Angel round.** $1M, announced 2026-09-03 (Telegram post 33), with the Ethereum Foundation, Fusion (IPOR), Altitude, Liquity and LI.FI, plus "more than 30 founders and contributors", including Anthony Sassano ([ChainCatcher, 2026-09-04](https://www.chaincatcher.com/en/article/2287671); [Dealroom](https://dealroom.co/news/148966-polaris-raises-1m-angel-round-backed-by-ethereum-foundation/)).
- **Legal entity or foundation:** [DATA NEEDED]. The handle "fnd" was not explained in any source read; no foundation is named.
- **Conflict of interest in a data source.** TokenBrice runs Pharos ([pharos.watch](https://pharos.watch/stablecoin/pusd-polaris/)), a stablecoin tracker whose Polaris pages are among the few third-party profiles. Pharos is not an independent source for Polaris and is cited below only where marked.

## Timeline

| Date | Event | Source |
|------|-------|--------|
| 2026-01-17 | Telegram announcement channel created | [t.me/s/polaris_ann](https://t.me/s/polaris_ann) |
| 2026-01-21 to 2026-01-27 | Public announcement; "Why Polaris?" blog post (pUSD) | same; [Why Polaris?, archived](http://web.archive.org/web/20260720042630/https://polarisfinance.io/blog/why-polaris/) |
| 2026-03-04 | "Testnet is live internally" (Pathfinders only) | Telegram post 15 |
| 2026-05-07 | "The Polaris second testnet is live, open access to all" | Telegram post 20 |
| 2026-06-11 to 2026-07-01 | Rebrand: pUSD becomes USDp (2026-06-17), POLAR announced (2026-06-24), pGOLD becomes GOLDp (2026-07-01) | Telegram posts 21 to 25 |
| 2026-08-05 | Documentation live | Telegram post 26 |
| 2026-08-14 to 2026-08-17 | Test of the "bootstrap phase that will happen just before mainnet launch" (seeding the bonding curve); Testnet V2 announced | Telegram posts 28, 29 |
| 2026-08-19 | X handle changed to @polarisfnd | Telegram post 30 |
| 2026-09-03 | $1M angel round announced | Telegram post 33 |
| 2026-10-26 | Four audits "underway", auditors unnamed, no mainnet date | Pharos (co-founder's site, not independent): [pharos.watch](https://pharos.watch/stablecoin/pusd-polaris/); the original Polaris post was not found |

## Adoption metrics

| Metric | Value | Date | Source |
|------|-------|------|--------|
| TVL | None: not on mainnet. No DefiLlama record for this project (the "Polaris Finance" record is the 2022 Aurora project) | 2026-11-17 | [DefiLlama protocols list](https://api.llama.fi/protocols) |
| USDp / GOLDp supply | None on mainnet; testnet values have no meaning (docs agent guidance) | 2026-11-17 | [llms.txt](https://docs.polaris.finance/llms.txt) |
| Number of pAssets | 2 on testnet (USDp, GOLDp); CHFp named as a "potential future" asset | 2026-11-17 | [pAsset Minting](https://docs.polaris.finance/architecture/passet-minting) |
| Collateral types | 1 (pETH), by design; stewardship "cannot ... Introduce new collateral" | 2026-11-17 | [Stewardship](https://docs.polaris.finance/architecture/stewardship) |
| POLAR initial supply | 100 million; allocation and vesting "will be published closer to launch" | 2026-11-17 | [Tokenomics](https://docs.polaris.finance/architecture/tokenomics) |
| Fees | None (pre-launch) | 2026-11-17 | n/a |

## How it works

### User perspective

A user swaps ETH for pETH through the bonding curve, deposits pETH into a pAsset market (USDp or GOLDp) and mints the pAsset at or above a 115% collateral ratio. "Every \$1 of USDp debt requires at least \$1.15 worth of pETH collateral at minting", and positions below 115% are eligible for liquidation ([USDp](https://docs.polaris.finance/core-assets/usdp)). Holders can deposit the pAsset in that market's Earn Pool (the first-loss pool) for a share of minter interest (paid in the pAsset) and liquidation gains (paid in pETH); each deposit is a transferable NFT ([Earn Pools](https://docs.polaris.finance/architecture/earn-pools)). A user can also open a "collateral-only position" (pETH in a market with no debt), which earns peg fees and Protocol Safety Rate payments ([pAsset Minting](https://docs.polaris.finance/architecture/passet-minting)).

Outside the CDP, pETH can be split into fpETH (the floor) and vpETH (the premium), and fpETH can back a non-liquidatable "Reserve Loan" of ETH from the bonding curve reserve, up to 100% of the floor value ([fpETH](https://docs.polaris.finance/core-assets/fpeth); [Reserve Loans](https://docs.polaris.finance/architecture/reserve-loans)).

### Protocol perspective

- **Collateral: pETH from a bonding curve.** ETH deposited into a power-law bonding curve mints pETH; returning pETH burns it and releases ETH. The curve's exponent β sets its shape; its value is not given ([Bonding Curve](https://docs.polaris.finance/architecture/bonding-curve)). pETH "has both a market price and a mathematically guaranteed floor price", and "the pETH floor can only rise; it cannot fall" ([pETH](https://docs.polaris.finance/core-assets/peth)). Fees from bonding curve swaps, Reserve Loans and pETH-to-POLAR conversions go to a Fee Router that burns pETH at the floor (raising the floor) and buys new pETH with the released ETH (raising the spot price) before distributing it ([Fee Router](https://docs.polaris.finance/design/fee-router)). The blog says Polaris "will not accept staked ETH tokens like wstETH as collateral" ([CDPs Mint Dollars. Polaris Mints Anything, 2026-02-02, archived](http://web.archive.org/web/20260203024944/https://polarisfinance.io/blog/polaris-mints-anything)).
- **One collateral, many markets.** "Every pAsset maintains its own accounting, oracle configuration, risk parameters and Earn Pool while relying on the same pETH collateral foundation" ([pAsset Markets](https://docs.polaris.finance/architecture/passet-markets)). The blog calls it "a CDP protocols factory ... all instances sharing the same pETH collateral" and argues that a gold shock would cause liquidations "on the pGold instance" while pUSD and pCHF "barely moved" (archived Mints Anything post). Stress states are per market: "Stress affecting one market does not automatically place every other market into the same state" ([Defensive Mode](https://docs.polaris.finance/design/defensive-mode)).
- **Liquidation.** Below 115%, a position's debt is cancelled with pAssets from that market's Earn Pool and its pETH goes to the pool. "If the Earn Pool does not hold enough deposits to absorb the entire liquidation, the remaining debt and collateral are redistributed across the other open positions in the same pAsset market" ([Liquidations](https://docs.polaris.finance/design/liquidations)). No external auction and no liquidator reward are described. Who triggers a liquidation, and any liquidation penalty or gas compensation: [DATA NEEDED].
- **Who absorbs bad debt.** First the market's Earn Pool depositors ("a first-loss backstop", [Risks](https://docs.polaris.finance/risks)), then the other positions in the same market by redistribution. No treasury or insurance backstop is described. Losses stay within one pAsset market; the docs do not say that one market's bad debt can reach another.
- **Defensive Mode (per market).** Entered when the market's "reserve-to-debt ratio falls below 110%" while backing is above 100%. Adding collateral and repaying stay open; withdrawing collateral or increasing debt is allowed only if the position stays at 150% or more and its ratio does not fall; closing a position and buying at peg are restricted ([Defensive Mode](https://docs.polaris.finance/design/defensive-mode)).
- **Recovery Mode (per market).** Entered below 100% backing. "Redemptions are paid in proportion to the collateral that remains" (a market at 80% pays 80% of face value to every redeemer); "Liquidation by redistribution is disabled"; a liquidation that would lower backing "is not carried out" ([Recovery Mode](https://docs.polaris.finance/design/recovery-mode)). This is a defined path for an undercollateralised synth market to keep running rather than halt.
- **Peg: two-sided, spread across positions ("Adaptive Peg Defense").** Below peg, anyone can redeem the pAsset for pETH at its target value; above peg, anyone can "buy newly minted supply at peg" with pETH. Neither side opens or targets a single position: "the resulting collateral, debt, and fees are distributed across all open positions in that pAsset market" (above peg, positions are upsized; below peg, downsized). Every buy and redemption pays a fee distributed to the market's positions ([Adaptive Peg Defense](https://docs.polaris.finance/design/adaptive-peg-defence); [pAsset Minting](https://docs.polaris.finance/architecture/passet-minting)). Peg fees use "spike-and-decay" pricing: demand raises the fee and it decays with inactivity ([Spike-and-Decay Pricing](https://docs.polaris.finance/design/spike-and-decay-fees)); numeric parameters [DATA NEEDED]. This differs from Liquity, where redemption hits the lowest-rate troves, and from a PSM, since the buy side is paid in pETH, not a stablecoin. See [[patterns/cdp-peg-mechanisms]].
- **Rate mechanism without a market-price oracle.** The Peg Stability Rate "is not altered by governance or manual intervention"; it falls toward "its minimum base fee of 0.05%" as net minting rises and rises with net redemptions ([Interest Rates](https://docs.polaris.finance/design/interest-rates)). The blog explains why: "a spike in redemption volume is a proxy signal for a downward depeg", so the system reads the peg "without needing access to its price feed" (archived Mints Anything post). The docs: "USDp also does not require an external market-price oracle to maintain its peg" ([Oracles](https://docs.polaris.finance/design/oracles)). Interest is paid in the pAsset, accrues to debt and is minted to the Earn Pool and sPOLAR.
- **Protocol Safety Rate.** An always-on transfer inside a market from higher-leverage to lower-leverage and collateral-only positions; "not protocol revenue" ([Interest Rates](https://docs.polaris.finance/design/interest-rates)). Formula and size [DATA NEEDED].
- **Oracle.** pETH's price in ETH comes from the bonding curve only. External prices go through a "Medianizer", one per price (USDp uses ETH/USD; GOLDp uses ETH/USD and XAU/USD), each with up to three feeds: the median if three are valid, "the first valid feed in the configured order" if one or two are, and "the last valid price datapoint" if none are. Feeds are valid if the call succeeds, the value is positive and it is not stale ([Oracles](https://docs.polaris.finance/design/oracles)). "The set of oracle feeds used at launch will be finalized closer to the mainnet launch." GOLDp is a non-USD synth priced from two USD legs; see [[patterns/collateral-debt-units]]. Whether the CR check values pETH at its spot price or its floor: [DATA NEEDED].

### Fees and borrower cost

| Item | Value | Source |
|------|-------|--------|
| Minimum collateral ratio (USDp, GOLDp) | 115% (also the liquidation threshold) | [USDp](https://docs.polaris.finance/core-assets/usdp); [Liquidations](https://docs.polaris.finance/design/liquidations) |
| Peg Stability Rate (borrow rate) | Floor 0.05%; rises with net redemptions; no cap stated | [Interest Rates](https://docs.polaris.finance/design/interest-rates) |
| Interest split | "a protocol-defined minimum share" to the market's Earn Pool; for USDp and GOLDp a share to sPOLAR; rest to approved recipients. Split values [DATA NEEDED]; stewardship sets "Interest rate splits" | [Flows](https://docs.polaris.finance/architecture/flows); [Stewardship](https://docs.polaris.finance/architecture/stewardship) |
| Peg buy and redemption fee | Spike-and-decay; paid to the market's positions; values [DATA NEEDED] | [Adaptive Peg Defense](https://docs.polaris.finance/design/adaptive-peg-defence) |
| Bonding curve swap fee | Spike-and-decay; to the Fee Router; values [DATA NEEDED] | [Spike-and-Decay Pricing](https://docs.polaris.finance/design/spike-and-decay-fees) |
| Reserve Loan | 0.05% opening fee plus a utilisation-based share of fpETH floor growth | [Reserve Loans](https://docs.polaris.finance/architecture/reserve-loans) |
| Effective rate | Can be "low, or even negative" because minters receive pETH flows in proportion to debt and peg fees | [pAsset Minting](https://docs.polaris.finance/architecture/passet-minting) |

The treasury gets no direct fee stream: "If the treasury wants exposure to protocol revenue, it must stake POLAR as sPOLAR" ([Tokenomics](https://docs.polaris.finance/architecture/tokenomics)).

## Admin and governance

- **Immutable core, no admin keys (claimed).** "The core protocol has no admin privileges", "The core protocol is immutable", "Neither pETH nor any pAsset includes blacklist or freeze functionality", and no "privileged actors, multisigs or administrators" can mint unbacked assets ([Security Guarantees](https://docs.polaris.finance/risks/security-properties)). "Core contracts are not designed to be upgraded or paused after deployment" ([Risks](https://docs.polaris.finance/risks)). Not verifiable: no mainnet contracts or source found.
- **Bounded stewardship by sPOLAR.** POLAR staked for 2 weeks to 4 years gives sPOLAR (an NFT position; called vePOLAR in the March 2026 blog) ([POLAR](https://docs.polaris.finance/core-assets/polar)). Stewards control: "Interest rate splits", "POLAR conversion parameters", "Replacing faulty oracle feeds", "Whitelisting or removing flow recipients", "Adjusting flow weightings". They cannot "Introduce new collateral", "Modify the bonding curve" or "Change the liquidation mechanisms" ([Stewardship](https://docs.polaris.finance/architecture/stewardship)). Parameters are "stewarded within immutable bounds" ([Spike-and-Decay Pricing](https://docs.polaris.finance/design/spike-and-decay-fees)); the bound values are [DATA NEEDED].
- **Timelock.** Only for oracles: "Failed feeds can be replaced immediately, whereas replacing an active feed or populating an empty slot requires a timelock" ([Oracles](https://docs.polaris.finance/design/oracles)). Timelock length [DATA NEEDED]. No timelock is stated for the other stewardship levers.
- **Multisig.** None named. Who holds stewardship before POLAR launches, and the voting mechanics, are [DATA NEEDED]. The March 2026 blog said founder influence "will be outsized at first: this is by design" and that treasury positions would be "Initially managed by team" (archived Stewardship post).
- **Can anyone deploy?** Two readings, see Conflicts and gaps. Current docs: "Launching a new pAsset market is permissionless, meaning builders can introduce new assets without approval from the protocol itself"; only access to pETH flows is stewarded. Each market must "route a minimum share of its primary interest to its own Earn Pool"; the builder decides the rest ([pAsset Markets](https://docs.polaris.finance/architecture/passet-markets)). All markets must use pETH as collateral. Whether the whole system (bonding curve plus markets) can be redeployed as a separate instance is not addressed; the March blog said the CDP code is "not just open code anyone can fork: teams are granted a license". Source code licence [DATA NEEDED]; no public repository found.

## Privacy and the reflexive stablecoin (RFP-013)

- **Privacy:** nothing found. The docs stress censorship resistance (no freeze or blacklist, "without KYC"), not transaction privacy.
- **Reflexive stablecoin:** Polaris is not one. USDp and GOLDp are hard-pegged to an oracle target. Relevant pieces: (1) the Peg Stability Rate is a rate mechanism driven by on-chain mint and redemption flow instead of a market-price feed for the synth; (2) "Any asset with an oracle can become a pAsset", and the blog lists "pBigMac" (Big Mac index) and commodities, so a floating or index target needs only an oracle under this design (no such market is documented); (3) the collateral is itself reflexive: pETH's premium (vpETH) reflects "expectations about the future growth of Polaris" ([vpETH](https://docs.polaris.finance/core-assets/vpeth)), so collateral value depends partly on demand for the protocol, while the ETH-denominated floor only rises.

## Key behaviours

- [[rfp/appendix/cdp-isolation]] : a new case: one shared collateral type, with a first-loss pool, redistribution, oracle and stress modes per synth market. Debt-side losses stay in one market; collateral risk (ETH price and pETH premium) is common to all markets.
- [[patterns/cdp-peg-mechanisms]] : two-sided peg at face value with effects spread across all positions; a flow-signal rate mechanism with no synth price oracle.
- [[rfp/appendix/cdp-oracle]] and [[patterns/collateral-debt-units]] : a three-feed median with ordered fallback and last-good-price, and a non-USD synth (gold) priced from two USD legs.
- [[rfp/appendix/cdp-admin-switch]] : immutable core with a bounded steward layer; oracle replacement is the one lever with a stated timelock.
- [[projects/liquity]] : same lineage (Liquity founder's research, Liquity devs, Liquity as an investor): Earn Pool plus redistribution mirrors Liquity's Stability Pool plus redistribution, and Recovery Mode reuses the name with different rules.

## Architecture decisions

- **One collateral by design**, issued by the protocol's own bonding curve, so pETH's ETH price needs no oracle and all fee flows can strengthen it.
- **A first-loss pool, oracle setup and stress modes per synth market**, while every market shares the same collateral token.
- **Peg actions spread across all positions** rather than aimed at one position (Liquity) or backed by a reserve of another stablecoin (PSM).
- **Algorithmic borrow rate from flow signals**; stewards set splits, not the rate.
- **Defined behaviour below 100% backing**: proportional redemptions and no redistribution.

## Differentiators

- The only surveyed design that issues its own collateral token through a bonding curve and requires every synth to use it.
- The only surveyed design with a buy-at-peg path paid in collateral and spread across existing positions.
- A documented Recovery Mode for an under-backed synth market that keeps it running rather than halting it.

## Limitations and criticisms

- Pre-launch: no mainnet contracts, no published audits, no public source code found. Every mechanism claim is from docs and blog posts.
- Concentrated collateral risk: every pAsset depends on pETH, so a fall in ETH or a collapse of the pETH premium hits every market at once. Per-market isolation covers debt-side shocks (for example gold), not collateral shocks. The docs' risk page says users "remain exposed to ETH price movements against USD, gold or any other reference asset".
- The Medianizer returns "the first valid feed in the configured order" when one or two feeds are valid, rather than a median, and falls back to "the last valid price datapoint" when none are; liquidations could then run on an old price.
- Buy-at-peg and redemption change every position's size, so a minter's debt and collateral move without their action (the docs say fees compensate this "under certain market conditions").
- Several parameters that set risk (β, fee curves, interest splits, steward bounds, timelock length, launch oracle feeds) are unpublished.

## Conflicts and gaps

- **Permissionless markets or licensed markets.** Docs (read 2026-11-17): launching a pAsset market is permissionless; stewardship only directs flows. Blog of 2026-03-03: stewards grant a "CDP Licensing" right; an applicant must "burn a set amount of pETH" and win a vote, and "This design is still to be implemented and will be refined as mainnet approaches." The docs are newer; treated as current, not confirmed in code.
- **POLAR emissions.** The March 2026 blog had "~25% of supply earmarked over 4 years" for gauge emissions and "16.4% of total POLAR supply for Treasury". The current docs say "POLAR has no perpetual emissions schedule" and that the final allocation is unpublished. These may be compatible (a fixed initial allocation with vesting) but are not reconciled.
- **Naming.** pUSD and pGOLD (blog, before June 2026) are now USDp and GOLDp. The X handle was @polarisfinance_ until 2026-08-19.
- **Liquidation pricing of pETH (spot or floor), liquidation penalty, liquidator incentive, steward bounds, interest split values, timelock length:** [DATA NEEDED].
- **Audits:** "four audits underway" (2026-10-26) is from Pharos only, which a co-founder runs; no audit firm or report found.

## Not checked

- No GitHub organisation or contract repository found (GitHub search was not run; it needed an approval this session did not have). testnet.polaris.finance HTML showed no repository link or contract address.
- Sepolia testnet contracts were not read.
- The X account itself was not read (X does not render to fetchers); Telegram posts were used instead.
- Blog posts on the bonding curve, pUSD yield, pGOLD, POLAR conversion, flows and the Ethereum bull case (archived on the Wayback Machine) were not read.
- [RootData](https://www.rootdata.com/projects/detail/Polaris?k=MjM1NzM%3D) and [CryptoRank](https://cryptorank.io/ico/polaris-protocol) did not render; their content was seen only in search summaries.

## Sources

- [Polaris website](https://polaris.finance/) : HTML fetched 2026-11-17 (links to @polarisfnd, Telegram, docs, testnet)
- [Polaris docs, full bundle](https://docs.polaris.finance/llms-full.txt) and [index](https://docs.polaris.finance/llms.txt) : read 2026-11-17; pages cited above: [pETH](https://docs.polaris.finance/core-assets/peth), [USDp](https://docs.polaris.finance/core-assets/usdp), [GOLDp](https://docs.polaris.finance/core-assets/goldp), [fpETH](https://docs.polaris.finance/core-assets/fpeth), [vpETH](https://docs.polaris.finance/core-assets/vpeth), [POLAR](https://docs.polaris.finance/core-assets/polar), [Bonding Curve](https://docs.polaris.finance/architecture/bonding-curve), [Earn Pools](https://docs.polaris.finance/architecture/earn-pools), [Flows](https://docs.polaris.finance/architecture/flows), [pAsset Markets](https://docs.polaris.finance/architecture/passet-markets), [pAsset Minting](https://docs.polaris.finance/architecture/passet-minting), [Reserve Loans](https://docs.polaris.finance/architecture/reserve-loans), [Stewardship](https://docs.polaris.finance/architecture/stewardship), [Tokenomics](https://docs.polaris.finance/architecture/tokenomics), [Adaptive Peg Defense](https://docs.polaris.finance/design/adaptive-peg-defence), [Defensive Mode](https://docs.polaris.finance/design/defensive-mode), [Fee Router](https://docs.polaris.finance/design/fee-router), [Interest Rates](https://docs.polaris.finance/design/interest-rates), [Liquidations](https://docs.polaris.finance/design/liquidations), [Oracles](https://docs.polaris.finance/design/oracles), [Recovery Mode](https://docs.polaris.finance/design/recovery-mode), [Spike-and-Decay Pricing](https://docs.polaris.finance/design/spike-and-decay-fees), [Risks](https://docs.polaris.finance/risks), [Security Guarantees](https://docs.polaris.finance/risks/security-properties)
- [Polaris Telegram announcements](https://t.me/s/polaris_ann) : posts 1 to 46 (2026-01-17 to 2026-11-13), read 2026-11-17
- Polaris blog (archived): [Why Polaris?, 2026-01-21](http://web.archive.org/web/20260720042630/https://polarisfinance.io/blog/why-polaris/); [CDPs Mint Dollars. Polaris Mints Anything, 2026-02-02](http://web.archive.org/web/20260203024944/https://polarisfinance.io/blog/polaris-mints-anything); [Stewardship, not Governance, 2026-03-03](http://web.archive.org/web/20260304063627/https://polarisfinance.io/blog/stewardship-not-governance/); [The Path to the North Star, 2026-04-27](http://web.archive.org/web/20260512023301/https://polarisfinance.io/blog/path-to-the-north-star/) : read 2026-11-17
- [TokenBrice: Why Polaris, 2026-01-27](https://tokenbrice.xyz/why-polaris/) : read 2026-11-17
- [ChainCatcher: angel round, 2026-09-04](https://www.chaincatcher.com/en/article/2287671); [Dealroom: angel round](https://dealroom.co/news/148966-polaris-raises-1m-angel-round-backed-by-ethereum-foundation/) : read 2026-11-17
- [Pharos: USDp](https://pharos.watch/stablecoin/pusd-polaris/) and [GOLDp](https://pharos.watch/stablecoin/pgold-polaris/) : read 2026-11-17; run by a Polaris co-founder, not independent
- [DefiLlama protocols list](https://api.llama.fi/protocols), [DefiLlama polaris-finance (2022 Aurora project)](https://api.llama.fi/protocol/polaris-finance), [DefiLlama stablecoins](https://stablecoins.llama.fi/stablecoins) : read 2026-11-17
