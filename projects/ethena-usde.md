---
tags: [project, ethereum, synthetic-dollar, delta-neutral, stablecoin, synthetics]
ecosystem: Ethereum (L1 issuance; USDe also on Base, Robinhood Chain, Plasma, Solana and others)
category: Synthetic Dollar (delta-neutral backed)
website: https://ethena.fi
docs: https://docs.ethena.fi
launched: 2024 (February per Stablecoin Insider)
---

# Ethena USDe

USDe is a synthetic dollar: a dollar-denominated token backed by a portfolio of assets held by the protocol, where volatile assets (spot crypto, and now tokenised equities and commodities) are paired with short futures positions in roughly equal notional so that the combined position stays near one dollar. Holders get dollar exposure without any bank-held fiat reserve, and the stability comes from derivatives hedging on centralised venues. sUSDe is the staked form that passes through incentive rewards. USDtb is a separate Ethena-linked dollar issued by Anchorage Digital Bank. USDe is the largest synthetic-dollar instrument in the Ethereum ecosystem but has shrunk by about two thirds from its October 2025 peak, and its backing is being re-weighted away from the pure crypto basis trade.

## Adoption metrics

| Metric | Value | Date | Source |
|--------|-------|------|--------|
| USDe circulating supply | $4.887B | 2026-11-16 | [DefiLlama stablecoins: USDe](https://defillama.com/stablecoin/ethena-usde) (data via [stablecoins.llama.fi](https://stablecoins.llama.fi/stablecoin/146)) |
| USDe protocol TVL | $4.880B ($4.351B on 2026-05-19; $3.960B on 2026-08-15) | 2026-11-16 | [DefiLlama Ethena USDe](https://defillama.com/protocol/ethena-usde) (data via [api.llama.fi](https://api.llama.fi/protocol/ethena-usde)) |
| USDe price | $0.9995 | 2026-11-16 | [DefiLlama coins API](https://coins.llama.fi/prices/current/coingecko:ethena-usde) |
| Peak supply | $14.82B (2025-10-04) | 2025-10-04 | [DefiLlama stablecoins: USDe](https://stablecoins.llama.fi/stablecoin/146) |
| Supply path in 2026 | $5.93B (2026-03-20), $4.35B (2026-05-19), $4.45B (2026-07-01), $4.12B (2026-09-01), $4.76B (2026-10-15) | 2026 | [DefiLlama stablecoins: USDe](https://stablecoins.llama.fi/stablecoin/146) |
| Supply by chain | Ethereum $2.845B; Plasma $510M; Base $382M; Robinhood Chain $365M; Solana $277M; Avalanche $104M | 2026-11-16 | [DefiLlama stablecoins: USDe](https://stablecoins.llama.fi/stablecoin/146) (chainBalances; rounded) |
| USDtb TVL | $317.9M ($1,076.5M on 2026-05-19; $357.8M on 2026-08-15) | 2026-11-16 | [DefiLlama Ethena USDtb](https://defillama.com/protocol/ethena-usdtb) (data via [api.llama.fi](https://api.llama.fi/protocol/ethena-usdtb)) |
| Protocol fees, trailing 30 days | $18.34M | 2026-11-16 | [DefiLlama fees overview](https://api.llama.fi/overview/fees) |
| Q3 2026 fees | $53.5M | 2026-09-26 | [Dwellir State of DeFi Q3 2026](https://www.dwellir.com/blog/state-of-defi-q3-2026) (cites DefiLlama fees) |
| sUSDe staking yield | 4.75% (30d average 4.74%; 90d average 4.53%) | 2026-10-14 | [Stablecoin Insider USDe report](https://stablecoininsider.org/ethena-usde-q1-2026-report/) (October 2026 postscript on a March article) |
| Backing ratio and Reserve Fund | 101.59% backing; Reserve Fund about $62M (June 2026 close) | 2026-08-19 | [Dealroom: Ethena moves $1B into FalconX facility](https://dealroom.co/news/145914-ethena-moves-1b-of-usde-backing-into-falconx-credit-facility/) (secondary) |
| Backing mix (June 2026 close) | about $2B DeFi lending, $2B liquid stables, $501M tokenised RWAs, $310M direct institutional lending | 2026-08-19 | [Dealroom](https://dealroom.co/news/145914-ethena-moves-1b-of-usde-backing-into-falconx-credit-facility/) (secondary; Ethena's transparency dashboard not fetched) |
| Yield-bearing stablecoin supply, Q2 2026 | fell by more than $3.5B (15%); sUSDe lost 52% of supply (nearly $2B) | 2026-07-02 | [Cointelegraph](https://cointelegraph.com/news/yield-bearing-stablecoins-lose-3-5-billion-q2) (citing CEX.io Q2 report) |
| Peer synthetic dollars (supply) | Falcon USDf $1.48B (was $1.72B 30 days earlier); Solstice USX $201M; Usual USD0 $538M | 2026-11-16 | [DefiLlama stablecoins list](https://stablecoins.llama.fi/stablecoins) |

## How it works

### User perspective

A whitelisted user mints USDe by sending USDT or USDC (or ETH, BTC or liquid staking tokens) to the protocol and receives about one USDe per dollar, less execution and hedging costs. Everyone else buys USDe on a DEX or lending market. Staking USDe gives sUSDe, whose USDe value rises as rewards accrue; negative-revenue periods are absorbed by the Reserve Fund and generally pause rewards instead of cutting principal. Redemption by a whitelisted user closes the matching short.

### Protocol perspective

- **Delta-neutral hedge:** a mint opens a short perpetual position of roughly equal notional on a derivatives exchange; a redemption closes it. Volatile backing assets (spot crypto, tokenised equities, tokenised commodities) carry matching shorts; stable assets (liquid stablecoins, short-duration RWAs) need no hedge.
- **Off-exchange settlement:** backing assets stay on-chain with custody providers, and Ethena delegates (never transfers) them to exchanges to margin the shorts. Ethena trades derivatives on the major centralised exchanges supported by these providers.
- **Backing diversification:** Ethena's docs now list non-crypto basis trades (equity and commodity perps), DeFi lending, institutional lending, tokenised RWAs and liquid stablecoins alongside the crypto basis trade, with allocation set dynamically under Risk Committee limits and published on a transparency dashboard (not fetched here).
- **Reserve Fund:** pays when combined funding, staking and carry revenue is negative, to protect the spot backing.
- **USDtb:** an Anchorage-issued dollar (since October 2025) designed to give USDe a stable leg when funding turns negative; Stablecoin Insider says over 90% of its reserves were in BlackRock's BUIDL (as at launch period; current mix [NOT FOUND]).

## Key behaviours

- [[patterns/synthetic-collateral-models]] : a candidate sixth model for that note, the hedged-collateral dollar: the token is backed by volatile assets whose price risk is cancelled by short derivatives, rather than by overcollateralisation or a 1:1 reserve.
- Funding-rate dependence: when perpetual funding compresses, yield falls and supply leaves. April 2026 sUSDe APY had fallen from about 8% to about 3.5%, below Treasury bills (Stock3, 2026-04-10).
- The hedge instrument is the same class of product as the perps covered elsewhere in this vault, so perp funding conditions feed directly into USDe yield and supply. Ethena also launched HyENA on [[projects/hyperliquid]] in December 2025 (Stablecoin Insider).
- Multi-chain footprint: USDe circulates on Base, Robinhood Chain, Plasma and Solana as well as Ethereum; the cross-chain mechanism (bridge or messaging) was not reviewed here, so its trust assumptions are [NOT FOUND].

## Architecture decisions

- **Hedge with perps, not overcollateralise:** capital-efficient (nearly every backing dollar earns yield) but moves risk to funding, exchange solvency and custody.
- **Whitelisted mint and redeem, open secondary market:** keeps the hedge book controlled, relies on arbitrageurs for the peg.
- **Dynamic backing mix under a Risk Committee:** reduces dependence on one yield source; the committee shrank from five to three voting members in February 2026 (Stablecoin Insider).
- **Separate USDtb leg issued by a bank:** isolates a Treasury-like product with a regulated issuer from the derivatives-backed USDe.
- **Credit pivot in 2026:** in August 2026 Ethena moved about $1B of backing (about a quarter of supply) into an over-collateralised institutional credit facility with FalconX through an SPV (Dealroom, 2026-08-19).

## Differentiators

- The largest dollar token in the vault's scope whose peg is maintained by short derivatives rather than reserves or CDP collateral; compare [[projects/synthetix-v3]] (debt-pool synths) and the CDP and bridged models in [[patterns/synthetic-collateral-models]].
- Supply of $4.9B is the largest figure among delta-neutral or basis-trade dollars in the DefiLlama stablecoin list (Falcon USDf is second at $1.5B).
- Distribution inside perp and RWA ecosystems: USDe is present on Robinhood Chain ($365M) where [[projects/lighter]] and tokenised stocks operate.

## Limitations and criticisms

- **Supply collapse:** from $14.82B (2025-10-04) to $4.89B (2026-11-16), about 67%; Q2 2026 was a roughly 15% fall for the whole yield-bearing stablecoin category and a 52% fall for sUSDe supply.
- **Yield compression and leverage unwinds:** sUSDe's 4.75% yield is close to Treasury yields, removing the premium that attracted leveraged loops.
- **October 2025 stress:** Stablecoin Insider reports USDe briefly fell to $0.97 and recovered within hours during a liquidation cascade; its own dates for the event are inconsistent (10 versus 11 October), so treat the detail as secondary.
- **Counterparty and custody exposure:** the hedge sits on centralised exchanges and custodians; an exchange failure or a large exchange hack is a direct risk (a Binance hack of about $6.3B on 2026-10-29 is reported by Al Jazeera and TRM; its effect on USDe is [NOT FOUND]).
- **Credit and liquidity risk from the pivot:** credit positions take time to unwind and haircuts can widen under stress, which could complicate redemptions.
- **Fee switch dormant:** passed 2026-09-02 but inactive until USDe supply reaches $7.5B (Dwellir).
- **Peer failure shows the category risk:** Resolv's USR lost its peg on 2026-03-22 after an attacker minted about 80M unbacked tokens and extracted about $25M (The Block); USR supply is now about $0.2M at a price near $0.035 on DefiLlama. Resolv said its collateral pool was intact; one analysis disputes that.
- **Unverified claim:** one source (Stablecoin Insider) says BaFin barred USDe under MiCA and Ethena exited the EU and EEA; no second source found.
- **Source conflicts on current supply:** Messari lists about 4.5B (page updated 2026-10-29), Stablecoin Insider about $4.66B (2026-10-21), DefiLlama $4.76B (2026-10-15) and $4.89B (2026-11-16); some widely shared figures of $6.4B to $9B refer to late 2025.

## Sources

- [Ethena docs: Overview](https://docs.ethena.fi/overview/ethena-overview.md) : accessed 2026-11-16
- [Ethena docs: How USDe Works](https://docs.ethena.fi/overview/how-usde-works.md) : accessed 2026-11-16
- [Ethena docs: Backing Assets Overview](https://docs.ethena.fi/backing-assets/overview.md) : accessed 2026-11-16
- [Ethena docs: Underlying Derivatives](https://docs.ethena.fi/protocol-overview/underlying-derivatives.md) : accessed 2026-11-16
- [Ethena docs: Funding Risk](https://docs.ethena.fi/protocol-overview/risks/funding-risk.md) : accessed 2026-11-16
- [Ethena docs: USDtb](https://docs.ethena.fi/overview/usdtb.md) : accessed 2026-11-16
- [DefiLlama stablecoins API](https://stablecoins.llama.fi/stablecoin/146) : accessed 2026-11-16
- [DefiLlama protocol, fees and hacks APIs](https://api.llama.fi/protocol/ethena-usde) : accessed 2026-11-16
- [DefiLlama coins API](https://coins.llama.fi/prices/current/coingecko:ethena-usde) : accessed 2026-11-16
- [Dwellir: State of DeFi Q3 2026](https://www.dwellir.com/blog/state-of-defi-q3-2026) : accessed 2026-11-16
- [Cointelegraph: Yield-bearing stablecoins lose $3.5B in Q2](https://cointelegraph.com/news/yield-bearing-stablecoins-lose-3-5-billion-q2) : accessed 2026-11-16
- [Stock3: Ethena moves beyond crypto in search of higher yields](https://stock3.com/news/ethena-moves-beyond-crypto-in-search-of-higher-yields-17018707) : accessed 2026-11-16
- [Dealroom: Ethena moves $1B of USDe backing into FalconX credit facility](https://dealroom.co/news/145914-ethena-moves-1b-of-usde-backing-into-falconx-credit-facility/) : accessed 2026-11-16
- [Stablecoin Insider: Ethena USDe Q1 2026 report](https://stablecoininsider.org/ethena-usde-q1-2026-report/) : accessed 2026-11-16
- [The Block: Resolv USR depeg](https://www.theblock.co/news/defi/2026-03-22-resolvs-usr-stablecoin-depegs-after-attacker-mints-80-million-unbacked-tokens-extracts-roughly-25-million-394582) : accessed 2026-11-16
- [Al Jazeera: Binance says $6.3bn stolen](https://www.aljazeera.com/news/2026/10/30/binance-says-6-3bn-stolen-in-biggest-crypto-heist-in-history-what-we-know) : accessed 2026-11-16
- [Messari: Ethena USDe](https://messari.io/asset/ethena-usde) : accessed 2026-11-16 (search extract only)
