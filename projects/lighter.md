---
tags: [project, ethereum, zk-rollup, perpetuals, orderbook, rwa, synthetics]
ecosystem: Ethereum (zk-rollup settled on Ethereum L1; separate instance on Robinhood Chain, an Arbitrum-stack L2)
category: Orderbook Perpetuals (RWA-extended)
website: https://lighter.xyz
docs: https://docs.lighter.xyz
launched: "[NOT FOUND] for mainnet date; LIT token launched December 2025 (Castle Labs); Robinhood Chain instance live 2026-07-01"
---

# Lighter

Lighter is an on-chain central limit orderbook (CLOB) perpetual exchange whose matching and liquidation engine is proven with zero-knowledge proofs and anchored on Ethereum. Traders get synthetic price exposure to crypto, equities, ETFs, commodities, FX and pre-IPO names through perpetual contracts, settled in USDC (mainnet) or USDG (Robinhood Chain instance), without holding the underlying. Users keep custody in Ethereum contracts and can force an exit through an on-chain priority queue. It was not in the May 2026 vault snapshot; by November 2026 it is the largest in-scope (Ethereum-anchored) perp venue by Q3 volume.

## Adoption metrics

| Metric | Value | Date | Source |
|--------|-------|------|--------|
| Lighter Bridge TVL (user collateral held in the Ethereum contract; DefiLlama classes it as a bridge) | $549.62M ($487.1M on 2026-05-19; $632.9M on 2026-10-15) | 2026-11-16 | [DefiLlama Lighter Bridge](https://defillama.com/protocol/lighter-bridge) (data via [api.llama.fi](https://api.llama.fi/protocol/lighter-bridge)) |
| LLP TVL (liquidity pool on mainnet) | $97.07M (series starts 2026-10-21 on DefiLlama, so no May comparison) | 2026-11-16 | [DefiLlama Lighter LLP](https://defillama.com/protocol/lighter-llp) (data via [api.llama.fi](https://api.llama.fi/protocol/lighter-llp)) |
| Lighter Robinhood Perps TVL (separate exchange on Robinhood Chain) | $111.65M (series starts 2026-07-20; $17.7M on 2026-08-15) | 2026-11-16 | [DefiLlama Lighter Robinhood Perps](https://defillama.com/protocol/lighter-robinhood-perps) (data via [api.llama.fi](https://api.llama.fi/protocol/lighter-robinhood-perps)) |
| Open interest, mainnet | $418.8M | 2026-11-16 | [DefiLlama open-interest overview](https://api.llama.fi/overview/open-interest) (entry "Lighter Perps") |
| Open interest, Robinhood Chain instance | $186.0M | 2026-11-16 | [DefiLlama open-interest overview](https://api.llama.fi/overview/open-interest) (entry "Lighter Robinhood Perps") |
| Open interest cross-check (mainnet / Robinhood Chain) | about $0.43B / about $0.19B | 2026-11-16 | [CoinGecko derivatives exchanges API](https://api.coingecko.com/api/v3/derivatives/exchanges) (BTC-denominated, converted at BTC/USD 64,872; conversion calculated here) |
| 24h volume (mainnet / Robinhood Chain) | about $2.46B / about $0.66B | 2026-11-16 | [CoinGecko derivatives exchanges API](https://api.coingecko.com/api/v3/derivatives/exchanges) (converted as above) |
| Average daily volume | $1.80B (17 Aug to 15 Sep 2026) | 2026-09 | [Castle Labs perp report](https://castlelabs.io/research/reports/mapping-the-perpetual-trading-ecosystem) |
| Q3 2026 volume and share of perp DEX volume | $126.9B, 7.8% share (Q2: 7.1%); data to 25 Sep | 2026-09-25 | [Dwellir State of DeFi Q3 2026](https://www.dwellir.com/blog/state-of-defi-q3-2026) (cites DefiLlama perps; DefiLlama volume endpoint is paywalled to this fetch) |
| October 2026 volume | about $73B (single social-post figure relayed by KuCoin; unverified) | 2026-10 | [KuCoin insight post](https://www.kucoin.com/news/insight/PERP/6aedd695abae5e0007c536f0) |
| Share of on-chain perp open interest | 5.7% | 2026-11-15 | [Castle Labs perp report](https://castlelabs.io/research/reports/mapping-the-perpetual-trading-ecosystem) |
| 30d fees (mainnet / Robinhood Chain) | $4.54M / $2.21M | 2026-11-16 | [DefiLlama fees overview](https://api.llama.fi/overview/fees) |
| Last funding round | $68M raised at a $1.5B valuation | reported 2026-07-01 | [The Block, Robinhood Chain launch](https://www.theblock.co/post/406918/robinhood-chain-goes-live-mainnet-alongside-24-7-tokenized-stocks-lighter-perps-planned-crypto-agentic-trading) |

## How it works

### User perspective

A trader deposits USDC into the Lighter contract on Ethereum (or bridges in), and trades perpetual futures against a central orderbook. Standard accounts pay no maker or taker fee on regular orders (1 basis point on TWAP, advanced orders and RFQ) but face 300 ms taker latency; Premium accounts opt in to maker/taker fees with LIT-staking discounts and lower latency. Markets include crypto majors and long-tail tokens, equities, ETFs, commodities, FX and pre-IPO names such as OPENAI and ANTHROPIC, quoted 24/7. Leverage on an RWA market does not change outside the underlying's trading hours. Accounts are cross-margined by default with an isolated option, and a Public Pool lets depositors follow a whitelisted operator's strategy without lockups.

Since 2026-07-01 Lighter also runs a separate exchange on Robinhood Chain ("Lighter RH"), used by Robinhood Wallet in eligible jurisdictions. It uses USDG as main collateral, accepts tokenised stocks and ETFs as deposit assets, and shares nothing with mainnet (accounts, balances and API keys do not carry over).

### Protocol perspective

- **Sequencer and provers:** a sequencer orders user-signed transactions first-in first-out and gives soft finality. Witness generators and a purpose-built prover produce proofs that are aggregated into one batch proof verified on Ethereum before state updates.
- **State on Ethereum:** Ethereum contracts hold deposits and the canonical state root. Each state update ships compressed blob data, so any user can reconstruct account state from immutable on-chain data.
- **Escape hatch:** users can submit withdrawals, pool exits or reduce-only IOC orders to a priority request queue on Ethereum. If the sequencer ignores these within the set time, the contract freezes and users withdraw by proving ownership of balances, positions and pool shares from blob data.
- **Pricing:** mark price is the median of the orderbook impact price, an index plus an 8-minute EMA premium (clamped to 0.5% of index) and a median of centralised-exchange marks. Index prices come from Chainlink, Stork and Pyth. When external feeds go stale (for example RWA markets outside trading hours, stale threshold typically 1 minute) pricing switches to an orderbook-derived internal price with a 30-minute EMA; pre-IPO and prelaunch markets use only the internal price.
- **LLP:** the Lighter Liquidity Provider is a community-owned pool that market-makes, takes over liquidated positions (if it stays above its initial margin) and serves as the insurance fund. Where it cannot absorb a bankrupt account, auto-deleveraging closes opposing positions at the bankrupt account's zero price. LLP deposits are capped at 10 USDC per staked LIT.

## Key behaviours

- [[patterns/oracle-pricing-model]] : oracles feed an index, but the mark price blends orderbook impact and CEX prices; off-hours RWA pricing is endogenous to the book.
- [[patterns/lp-counterparty-risk]] : the LLP is market maker, liquidator and insurance fund in one, so it carries directional and tail risk; ADL is the last resort.
- [[patterns/bridge-trust-assumptions]] : custody sits in Ethereum contracts with a forced-exit path, in contrast to [[projects/hyperliquid]]'s validator-multisig USDC bridge.
- Equity and ETF perp positions are not adjusted for splits or corporate actions: affected markets are delisted and settled before the event.
- LIT is bought back by the protocol with trading-fee revenue through daily 24-hour TWAPs (docs). Whether bought tokens are burned is stated by a secondary source (Tokenomist, search extract only, not fetched) as true since 2026-06-30; [NOT FOUND] in the primary docs.

## Architecture decisions

- **Verifiable matching via zk proofs:** the exchange claims to be the first with verifiable order matching and liquidations, trading a centralised sequencer for provable rule-following and an Ethereum-enforced exit.
- **Ethereum as settlement and data layer:** deposits, state root and blobs live on Ethereum rather than an app-specific chain.
- **Zero fees for standard accounts:** revenue comes from Premium/Plus tiers, RFQ and TWAP fees, and LLP economics.
- **Separate Robinhood Chain instance:** a distinct exchange with USDG margin and tokenised-stock collateral, rather than one shared book, isolating regulatory and distribution risk.
- **Internal pricing for markets without a 24/7 feed:** lets RWA, pre-IPO and prelaunch markets trade continuously.

## Differentiators

- Largest Ethereum-anchored perp venue by Q3 2026 volume ($126.9B, behind only [[projects/hyperliquid]] and Aster, which is BNB Chain centred); on open interest it is level with [[projects/variational]] ($605M across both Lighter exchanges versus $697M).
- Non-custodial exit enforced by Ethereum contracts, unusual among top perp venues.
- Distribution through Robinhood Wallet, with $11M of LIT committed to the Robinhood community (The Block, 2026-07-01).
- Broad RWA coverage including pre-IPO equity exposure through perps; compare [[projects/ostium]] (RWA-first, off-chain hedged) and [[projects/gmx-v2]] (oracle-priced pools).

## Limitations and criticisms

- **Sequencer centralisation:** a single sequencer orders transactions; the escape hatch protects funds but not liveness or fair ordering. Standard accounts carry a 300 ms taker speed bump.
- **Closed implementation:** prover, sequencer and matching engine are in-house; audits are published (nine files on the docs page) but their scope and findings were not reviewed here.
- **LLP concentration and access:** LLP is both insurance fund and market maker, and only LIT stakers may deposit, which couples pool capital to token demand.
- **RWA off-hours risk:** Castle Labs found venue indices diverging by up to 30 bps on weekends and Lighter's WTI funding reaching +95% annualised for longs.
- **Jurisdiction limits:** the Robinhood Wallet perps integration is unavailable to residents of the UK, US, Canada, Switzerland, UAE, Singapore and others.
- **Volume quality [NOT FOUND]:** a share of volume may be points and airdrop driven; no primary measurement located.
- **Incident record:** no Lighter entry appears in the DefiLlama hacks list from 2026-03-01 to 2026-11-16 (negative finding). A different venue on the same chain, Arcus Perps, lost about $2.23M on 2026-11-10 (not a Lighter incident).
- **Source conflict:** one secondary article (egamers.io, not fetched) gives 30d volume near $39B and open interest near $766M for October, which disagrees with DefiLlama and CoinGecko open interest (about $605M combined) and with the Q3 volume above; treated as unreliable.

## Sources

- [Lighter docs: Technical Architecture, Lighter Core](https://docs.lighter.xyz/about-lighter/technical-architecture-lighter-core.md) : accessed 2026-11-16
- [Lighter docs: Liquidations and LLP](https://docs.lighter.xyz/trading/liquidations-and-llp-insurance-fund.md) : accessed 2026-11-16
- [Lighter docs: Fair Price Marking](https://docs.lighter.xyz/trading/fair-price-marking.md) : accessed 2026-11-16
- [Lighter docs: Internal Pricing Mechanism](https://docs.lighter.xyz/trading/internal-pricing-mechanism.md) : accessed 2026-11-16
- [Lighter docs: Real World Assets](https://docs.lighter.xyz/trading/real-world-assets-rwas.md) : accessed 2026-11-16
- [Lighter docs: Pre-IPO Markets](https://docs.lighter.xyz/trading/real-world-assets-rwas/pre-ipo-markets.md) : accessed 2026-11-16
- [Lighter docs: Trading Fees](https://docs.lighter.xyz/trading/trading-fees.md) : accessed 2026-11-16
- [Lighter docs: Public Pools](https://docs.lighter.xyz/trading/public-pools.md) : accessed 2026-11-16
- [Lighter docs: LIT Utility](https://docs.lighter.xyz/about-lighter/lit-utility.md) : accessed 2026-11-16
- [Lighter docs: Lighter on Robinhood Chain](https://docs.lighter.xyz/developers/robinhood-chain/overview.md) : accessed 2026-11-16
- [Lighter docs: Security Audits](https://docs.lighter.xyz/security/security-audits.md) : accessed 2026-11-16
- [DefiLlama protocols, fees and open-interest APIs](https://api.llama.fi/protocols) : accessed 2026-11-16
- [DefiLlama hacks API](https://api.llama.fi/hacks) : accessed 2026-11-16
- [CoinGecko derivatives exchanges API](https://api.coingecko.com/api/v3/derivatives/exchanges) : accessed 2026-11-16
- [Castle Labs: Mapping the perpetual trading ecosystem](https://castlelabs.io/research/reports/mapping-the-perpetual-trading-ecosystem) : accessed 2026-11-16
- [Dwellir: State of DeFi Q3 2026](https://www.dwellir.com/blog/state-of-defi-q3-2026) : accessed 2026-11-16
- [The Block: Robinhood Chain goes live on mainnet](https://www.theblock.co/post/406918/robinhood-chain-goes-live-mainnet-alongside-24-7-tokenized-stocks-lighter-perps-planned-crypto-agentic-trading) : accessed 2026-11-16
- [KuCoin insight post relaying October 2026 volumes](https://www.kucoin.com/news/insight/PERP/6aedd695abae5e0007c536f0) : accessed 2026-11-16 (search extract only)
