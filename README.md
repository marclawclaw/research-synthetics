# Research: Synthetic Assets

Obsidian vault containing ecosystem research on synthetic asset protocols across Ethereum (L1 and L2s), Solana, and Bitcoin L2s.

## Scope

Synthetic assets are on-chain instruments that derive their value from an external reference (price, index, or asset) without requiring custody of the underlying. This vault covers:

- Collateralised debt-position synthetics (Synthetix family)
- Perpetuals platforms producing synthetic exposure (GMX, dYdX, Drift, Jupiter Perps, Hyperliquid)
- BTC-backed synthetics on Bitcoin L2s (sBTC on Stacks, Babylon-related)
- Cross-chain synthetic representations of ETH, BTC and SOL
- CDP, debt-pool, collateral-split and delta-neutral synthetic dollars and assets (Sky, Liquity, Alchemix, Hylo, Ethena, Indigo, Monolith and others; see `patterns/cdp-synthetics-landscape.md`)
- Working notes for a CDP on LEZ (rules and admin switch, units and oracle requirements, peg mechanisms, how a deployment shares risk, oracle interface), which back the Notion page "CDP-Backed Synthetics"; start at `rfp/appendix/cdp-notion-map.md`

Last refreshed: Ethereum, Solana and CDP coverage 2026-11-16; the CDP-on-LEZ notes were aligned with the Notion page on 2026-11-17 (agent-assisted, not yet through a formal fact-check pass); Hyperliquid, dYdX and sBTC May 2026.

## Structure

- `projects/` : one atomic note per protocol
- `metrics/` : cross-project metric comparisons (TVL, volume, fees)
- `patterns/` : recurring design and trust patterns
- `rfp/appendix/` : appendix notes for RFP work; the CDP-on-LEZ set (`cdp-notion-map.md` maps each part of the Notion page "CDP-Backed Synthetics" to a vault file) sits here with `patterns/collateral-debt-units.md` and `patterns/cdp-peg-mechanisms.md`; `rfp/` also holds agent prompts and templates
- `_index.md` : ranking and selection table
- `summary.md` : final synthesis

## Navigation

Open this folder in Obsidian for full graph and backlink support. Wikilinks: `[[projects/synthetix]]`.

## Conventions

- Australian English spelling
- No em dashes or hyphenated punctuation dashes in prose
- All metrics carry a source URL and access date
- `[NOT FOUND]` where a metric cannot be sourced; never estimated
