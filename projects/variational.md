---
tags: [project, arbitrum, perpetuals, rfq, rwa, synthetics]
ecosystem: Ethereum L2 (Arbitrum)
category: RFQ Perpetuals and Swaps (RWA-heavy)
website: https://variational.io
docs: https://docs.variational.io
launched: 2025 (private mainnet Q1 2025 per roadmap; public mainnet status conflicts, see Limitations)
---

# Variational

Variational is an on-chain derivatives protocol whose first application, Omni, offers perpetual futures and total-return swaps on 500+ crypto, equity, ETF, commodity, FX and pre-IPO markets, with zero trading fees. There is no orderbook: every trade is a request for quote (RFQ) answered by one vertically integrated market maker, the Omni Liquidity Provider (OLP), which is the sole counterparty. Positions are collateralised in USDC on Arbitrum, so traders get synthetic exposure to assets such as US equities or OpenAI and Anthropic pre-IPO valuations without holding the underlying. It was not in the May 2026 vault snapshot; by November 2026 it is the third largest perp venue by open interest, second by October volume in one tracker and fourth by Q3 volume in DefiLlama-based data.

## Adoption metrics

| Metric | Value | Date | Source |
|--------|-------|------|--------|
| Open interest | $697.3M ($815M one week earlier; $994M 30 days earlier) | 2026-11-16 | [DefiLlama open-interest overview](https://api.llama.fi/overview/open-interest) (entry "Variational") |
| Open interest cross-check | about $0.70B | 2026-11-16 | [CoinGecko derivatives exchanges API](https://api.coingecko.com/api/v3/derivatives/exchanges) (entry "Variational Omni"; BTC-denominated, converted at BTC/USD 64,872, calculated here) |
| 24h volume | about $1.06B | 2026-11-16 | [CoinGecko derivatives exchanges API](https://api.coingecko.com/api/v3/derivatives/exchanges) (converted as above) |
| Share of on-chain perp open interest | 6.8% (6.2% in September) | 2026-11-15 | [Castle Labs perp report](https://castlelabs.io/research/reports/mapping-the-perpetual-trading-ecosystem) |
| Average daily volume | $1.29B (17 Aug to 15 Sep 2026) | 2026-09 | [Castle Labs perp report](https://castlelabs.io/research/reports/mapping-the-perpetual-trading-ecosystem) |
| Q3 2026 volume and share of perp DEX volume | $104.0B, 6.4% share (Q2: 3.5%); data to 25 Sep | 2026-09-25 | [Dwellir State of DeFi Q3 2026](https://www.dwellir.com/blog/state-of-defi-q3-2026) (cites DefiLlama perps; DefiLlama volume endpoint is paywalled to this fetch) |
| October 2026 volume | about $78B, second place behind Hyperliquid ($229B of a $659B total), roughly triple July's $25B (single search extract of a Cryptorank report; the page returned HTTP 403 to direct fetch) | 2026-10 | [Cryptorank October 2026 exchanges recap](https://cryptorank.io/insights/reports/october-2026-crypto-exchanges-recap) |
| RWA share of Variational volume | over 50% | 2026-09 | [Castle Labs perp report](https://castlelabs.io/research/reports/mapping-the-perpetual-trading-ecosystem) |
| TVL | [NOT FOUND] (DefiLlama lists Variational under Derivatives with no TVL series; an April 2026 CoinGecko case study cited near $100M by a search extract, not verified) | 2026-11-16 | [DefiLlama protocol API](https://api.llama.fi/protocol/variational) |
| Fees | [NOT FOUND] on DefiLlama (fees endpoint for the Variational slug returned an error); protocol charges no trading fee and earns the spread | 2026-11-16 | [Variational docs: Fees](https://docs.variational.io/omni/trading/fees.md) |
| Founding capital | $10M raised before building the protocol | undated (docs) | [Variational docs: Core Contributors](https://docs.variational.io/getting-started/core-contributors.md) |

## How it works

### User perspective

A trader connects an EVM wallet, deposits USDC on Arbitrum (gasless, with a flat 0.1 USDC fee per deposit or withdrawal) and opens a cross-margined or isolated position of up to 50x. Pressing buy or sell requests a firm quote from OLP; the displayed price is only an indicative quote, and the trader can set a slippage limit. Closing a position also means accepting an OLP quote. There are no maker or taker fees; the cost is the spread. Forced liquidations close at ask plus 0.5% (shorts) or bid minus 0.5% (longs).

Two instrument types: perpetuals (including TradFi perps on stocks, ETFs, commodities, FX and indices, and pre-IPO perps such as ANTHROPIC and OPENAI quoted on an assumed one billion shares) and swaps, a linear derivative tracking an asset's total return with liquidity sourced from traditional financial institutions and flatter funding.

### Protocol perspective

- **Settlement pools:** each user has an isolated bilateral pool with OLP, holding positions, USDC collateral and margin rules. Pools cannot be netted or transferred, and a liquidation in one pool affects no other user.
- **OLP:** a vault (smart contract holding USDC), a market-making engine (in-house algorithms pricing from CEX, DEX and TradFi data) and a risk system that hedges OLP's exposure. OLP posts margin too and can itself be liquidated, which Variational calls counterparty liquidation and equates to auto-deleveraging.
- **Variational Oracle:** a custom in-house oracle that streams and weights prices from many venues; it supplies the index price, which feeds the mark price together with funding and OLP's risk profile.
- **On-chain handling:** a transactor submits deposits, withdrawals and settlement transactions on behalf of users; a watcher confirms finality and guards against fake deposits.
- **Pre-IPO behaviour:** funding is fixed at 0.005% per 8 hours; Variational may delist and force-settle in adverse conditions, and the contract becomes a regular RWA perp after the IPO.

## Key behaviours

- [[patterns/oracle-pricing-model]] : a proprietary oracle sets index and mark prices, and OLP's quotes are the execution price; there is no independent price check on the fill.
- [[patterns/lp-counterparty-risk]] : the counterparty is a single professional market maker, not a public pool, so LP risk is concentrated in one balance sheet rather than spread across depositors.
- Funding on pre-IPO markets is fixed rather than market-derived, because no continuous underlying exists.
- Swaps are not yet 24/7: the roadmap marks 24/7 swap trading ("Phase 3") as open.

## Architecture decisions

- **RFQ with a vertically integrated market maker:** removes external market-maker rent and bootstraps long-tail and off-chain markets, because a new listing needs only a price feed, a quoting strategy and a hedge.
- **Zero fees, spread revenue:** the protocol takes a cut of OLP's spread revenue, and plans to burn 100% of treasury-directed revenue through $VAR.
- **Isolated bilateral settlement pools:** each trader is margined against OLP independently, limiting contagion between users.
- **Arbitrum and USDC only:** a single collateral asset on a mature Ethereum L2.
- **Founders from market making:** the team (ex-Genesis Trading) reuses a market-making engine it says it has refined for more than seven years.

## Differentiators

- An RFQ venue with no orderbook among the top perp venues (Castle Labs; one Substack headline calls it the only RFQ-based perp DEX), with over 50% of volume in RWA markets; with trade.xyz on [[projects/hyperliquid]] it accounts for about 90% of RWA perp volume (Castle Labs).
- Listing breadth: 500+ markets including pre-IPO equity exposure, which orderbook venues struggle to bootstrap. Compare [[projects/lighter]] (CLOB with internal pricing) and [[projects/ostium]] (RWA-first, hedged off-chain).
- Fast share gain: Q2 to Q3 2026 volume share rose from 3.5% to 6.4% (Dwellir, citing DefiLlama).

## Limitations and criticisms

- **Single counterparty:** OLP is the only liquidity source. Its solvency, hedging quality and quoting honesty are not observable on-chain; the OLP track record is not public and community deposits are not yet open (roadmap item unchecked).
- **No execution transparency:** firm quotes are generated off-chain at click time; the indicative-versus-firm gap is a slippage source the user cannot audit.
- **Proprietary oracle:** index, mark and liquidation prices come from an in-house oracle with no external cross-check described in the docs.
- **Status ambiguity [NOT FOUND]:** the docs' roadmap ticks "Omni public mainnet launch", yet the getting-started page (with a 2026-09-15 screenshot) still says Omni is in private beta and needs an access code. The actual access state in November 2026 was not confirmed. Also unchecked on the roadmap: API trading, partial fills of limit orders, the $VAR token and the Variational Pro app.
- **Token and points effects:** $VAR is scheduled for Q4 2026 (32% genesis airdrop to points holders, 18% ecosystem, 50% team and investors with 12-month lock then at least three years of unlocking). Pre-token points programmes can inflate volume; organic share [NOT FOUND].
- **Source conflicts:** lifetime volume is about $200B in an April 2026 CoinGecko case study and $260B+ in a September 2026 Datawallet guide (secondary search extracts, different dates and methods). Weak, affiliate-style sources should not be cited as primary.
- **Pre-IPO legal and venue risk:** the docs warn of higher instability and reserve the right to delist and force settle.
- **Incident record:** no Variational entry appears in the DefiLlama hacks list from 2026-03-01 to 2026-11-16 (negative finding). Audits listed: Zellic (December 2024) and Spearbit (March 2025); later feature audits not listed.

## Sources

- [Variational docs index](https://docs.variational.io/llms.txt) : accessed 2026-11-16
- [Variational docs: About Omni](https://docs.variational.io/omni/about-omni.md) : accessed 2026-11-16
- [Variational docs: The Omni Liquidity Provider](https://docs.variational.io/omni/the-omni-liquidity-provider-olp.md) : accessed 2026-11-16
- [Variational docs: Settlement Pools](https://docs.variational.io/variational-protocol/key-concepts/settlement-pools.md) : accessed 2026-11-16
- [Variational docs: Protocol Overview](https://docs.variational.io/variational-protocol/overview.md) : accessed 2026-11-16
- [Variational docs: Variational Oracle](https://docs.variational.io/variational-protocol/key-concepts/variational-oracle.md) : accessed 2026-11-16
- [Variational docs: Quoted, Index and Mark Prices](https://docs.variational.io/omni/trading/quoted-index-and-mark-prices.md) : accessed 2026-11-16
- [Variational docs: TradFi Perpetuals](https://docs.variational.io/omni/trading/tradfi-perpetuals.md) : accessed 2026-11-16
- [Variational docs: Pre-IPO Perpetuals](https://docs.variational.io/omni/trading/pre-ipo-perpetuals.md) : accessed 2026-11-16
- [Variational docs: Swaps](https://docs.variational.io/omni/trading/swaps.md) : accessed 2026-11-16
- [Variational docs: Fees](https://docs.variational.io/omni/trading/fees.md) : accessed 2026-11-16
- [Variational docs: Getting Started with Omni](https://docs.variational.io/omni/getting-started-with-omni.md) : accessed 2026-11-16
- [Variational docs: Roadmap](https://docs.variational.io/getting-started/roadmap.md) : accessed 2026-11-16
- [Variational docs: $VAR](https://docs.variational.io/token/usdvar.md) : accessed 2026-11-16
- [Variational docs: Audits](https://docs.variational.io/technical-documentation/audits.md) : accessed 2026-11-16
- [Variational docs: Core Contributors](https://docs.variational.io/getting-started/core-contributors.md) : accessed 2026-11-16
- [DefiLlama open-interest overview and hacks API](https://api.llama.fi/overview/open-interest) : accessed 2026-11-16
- [CoinGecko derivatives exchanges API](https://api.coingecko.com/api/v3/derivatives/exchanges) : accessed 2026-11-16
- [Castle Labs: Mapping the perpetual trading ecosystem](https://castlelabs.io/research/reports/mapping-the-perpetual-trading-ecosystem) : accessed 2026-11-16
- [Dwellir: State of DeFi Q3 2026](https://www.dwellir.com/blog/state-of-defi-q3-2026) : accessed 2026-11-16
- [Cryptorank: October 2026 crypto exchanges recap](https://cryptorank.io/insights/reports/october-2026-crypto-exchanges-recap) : accessed 2026-11-16 (search extract only)
