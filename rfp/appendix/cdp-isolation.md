# CDP on LEZ: isolation when anyone can deploy

This is a working framing for the sXMR and sBTC CDP software on LEZ. It is mirrored in the Notion page "CDP-Backed Synthetics" (Part 3b) and sits alongside [[rfp/appendix/cdp-admin-switch]]. The background is in [[patterns/cdp-vs-morpho-trust]]. Written 2026-11-16.

## Framing

We are building software, not one product. Anyone can deploy it and launch their own synths, and holders and borrowers pick the deployment they trust: the best deployer wins. Isolation works at two levels:

- **Between deployments**, it follows from how the software is shipped.
- **Within a deployment**, the deployer chooses a shape from the options the software supports, as they choose a mode for each admin role. Changes after launch, such as adding a branch, are the Listing role's job.

The software's job is to make the narrow shapes possible and every shape readable.

## Precedents for "anyone deploys"

| Example | Model | Isolation between deployers | Source |
|---------|-------|-----------------------------|--------|
| Monolith (Ethereum, 2026) | Factory: "allows anyone to deploy their own decentralized crypto-backed stablecoin" from one collateral token and one price feed. Each stablecoin "becomes irreversibly immutable when reaching a maximum immutability deadline of four years since deployment". invUSD and swanUSD are two deployments, at $7.8M and $6.8M TVL ([[patterns/cdp-synthetics-landscape]]) | Strict: one token per deployment, one collateral each | [docs](https://docs.monolith.market/), accessed 2026-11-16. Other parameters and roles are not described on that page |
| Liquity V2 friendly forks | Licensed copies of the code. More than 15 teams signed, for example Felix (Hyperliquid), Quill (Scroll) and Orki (Swellchain) | Strict: separate programs and tokens | [liquity.org/forks](https://www.liquity.org/forks), [docs](https://docs.liquity.org/v2-documentation/friendly-fork-program), accessed 2026-11-16 through search results; lists differ between pages |
| Morpho Blue | One immutable program; anyone creates a market; curators compete for lenders | Per market; the loaned asset already exists | [[patterns/cdp-vs-morpho-trust]] |
| Frankencoin (counter-example) | Anyone opens positions, but all mint ZCHF | None between proposers; the veto is the only protection | [[projects/frankencoin]] |

## Angles

1. **Between deployments: copies or instances.**
   - Copies: each deployer runs their own program, as the Liquity forks do. The Program upgrade role belongs to each deployer, but holders must check each copy's code.
   - Instances: one shared program, with each deployment held as accounts inside it, as with Monolith and Morpho. There is one audited code, but the Program upgrade role cannot be set per deployer. Whoever can upgrade the shared program sits above every deployment, so it would have to be immutable.

   Either way, deployment A's sXMR is a different token from deployment B's. Strict isolation between deployers comes free; liquidity splits.
2. **Synths per deployment.** One or several. If several, decide what they share: a buffer, a fee sink, one Guardian, one Shutdown. Indigo has a pool per synth; Synthetix V2 had one debt pool for all synths. The alternative is one synth per deployment, with a multi-synth brand made of several deployments under the same controllers.
3. **Collaterals per synth.** One (Monolith), or several branches behind one fungible token (Liquity V2), each with its own ratio, feed, debt ceiling, first-loss pool and shutdown.
4. **Positions.** One collateral and one synth (Liquity, crvUSD, Frankencoin), or cross-margin (Metronome, whose shared swap pool led to the $15.7M shortfall).
5. **Loss path.** The order of who pays:
   1. the liquidator;
   2. the branch first-loss pool;
   3. redistribution to the other borrowers in the branch (Liquity);
   4. a deployment-wide reserve (Frankencoin FPS, Sky surplus buffer);
   5. the holders, through a discount or a shutdown haircut.

   The software decides which steps exist; the deployer sizes them or switches them on.
6. **Control.** The level at which each admin role is set: deployment, synth or branch. A Risk role per branch, held by a different controller, gives a Morpho-style curator layer.
7. **Failure.** Shutdown granularity: branch, synth or deployment. Shared oracle legs fail together: an ETH/USD outage hits every branch and every deployment that reads the same RFP-020 feed.
8. **Comparability.** "Best deployer wins" needs comparable deployments, so each one should expose on chain:
   - its code and upgrade mode;
   - each role's mode, controller, delay and bounds;
   - its synths, branches and feeds;
   - its buffer sizes, debt and collateral.

   Anyone can name a token sXMR; which deployments a front end or registry shows is a separate choice.

## Fixed in the software or picked by the deployer

| Angle | Fixed in the software | Picked by the deployer |
|-------|-----------------------|------------------------|
| Between deployments | Copies or instances | Which deployment to launch; only the oracle and LEZ are shared |
| Synths | Whether several are supported, and what they may share | How many, and whether they share a buffer |
| Collaterals | Whether branches behind one token are supported | Branches and their parameters (later changes: the Listing and Risk roles) |
| Positions | One pair or cross-margin | Nothing, unless cross-margin is optional |
| Loss path | Which steps exist, and their order | Buffer sizes, and whether a reserve is used |
| Control | The level at which roles can be set | Controller and mode at each level |
| Failure | Shutdown granularity | When to trigger it, through the Shutdown role |
| Comparability | What every deployment must publish | Nothing |

## Open questions

1. Copies or instances? If instances, the shared program must be immutable.
2. What is the smallest unit, for example a (collateral, synth) branch? Supporting the narrowest unit lets a deployer choose strict isolation, and grouping is then their choice (inference).
3. Can one deployment hold several synths, and what can they share?
4. Which loss steps does the code provide, and who pays beyond the buffer?
5. At what level can each role be set?
6. Can a branch shut down on its own?
7. Is cross-margin supported, optional or never?
8. What profile must every deployment publish, and is there a registry?
