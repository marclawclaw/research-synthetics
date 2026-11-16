---
tags: [project, arbitrum, avalanche, perpetuals, synthetics]
ecosystem: Ethereum L2 (Arbitrum), Avalanche, MegaETH, Botanix (dormant), Ethereum mainnet access
category: Perpetuals
website: https://gmx.io
docs: https://docs.gmx.io
launched: 2021 (v1), 2023 (v2)
---

# GMX V2

GMX is a permissionless perpetual exchange that produces synthetic price exposure to BTC, ETH and other tracked assets through liquidity pools (GM and GLV) priced by Chainlink Data Streams oracles, not an orderbook. Traders take leveraged positions against the pool; LPs are the counterparty.

## Update 2026-11

**What changed since May 2026**

- **Headline metrics eased.** DefiLlama TVL for GMX V2 (now listed as "GMX EVM", previously "GMX V2 Perps") is $187.05M on 2026-11-16, down 8.1% from $203.47M in May. Open interest is $40.74M versus $83.6M on 2026-05-21 (DefiLlama series). 7d fees are $426,548 versus $456,470 (down 6.5%). Monthly fees (derived by summing DefiLlama daily data) fell from about $2.0M in May 2026 to about $1.3M in October 2026. TVL troughed near $159M in mid-June 2026 and recovered to about $210M by September to October before easing again.
- **Chain footprint shrank at the edges.** Arbitrum is $180.07M of the $187.05M total (96%). Avalanche fell from $10.99M to $6.74M. MegaETH fell from $1.03M to $0.23M (DefiLlama shows its TVL dropping from about $8.1M to $0.8M around 2026-05-30; the cause is [NOT FOUND]). Botanix shows $0 and has been about $0 since early July 2026 (no wind-down announcement found), so it should no longer be listed as an active chain.
- **GMX Solana is now GMTrade.xyz and is a separate venue.** Rebranded as GMTrade.xyz per the 2026-03-04 governance post. DefiLlama lists it separately (TVL $22.61M on 2026-11-16, down from $41.1M on 2026-05-22; 30d fees $2.78M, which is nearly double the EVM deployment's $1.43M). It should not be summed into the V2 figures.
- **Ethereum mainnet access (2025-12-22).** GMX announced it is available on Ethereum mainnet with 100+ perp markets and 23 spot markets, GM and GLV purchasable from mainnet, a unified "GMX Account", and cross-chain deposits via LayerZero, Gelato and GMX Express. DefiLlama does not break out Ethereum TVL for V2, so whether this is separate liquidity or a front end to Arbitrum liquidity is [NOT FOUND].
- **Tokenomics and governance.** 2026-03-04 Strategic Plan to Restore Price Discovery: about 600,000 GMX moved from Treasury-controlled Uniswap and Trader Joe pools into the GMX-USDC pool and GMTrade.xyz; staking rewards redirected to the Treasury and only released once GMX exceeds $90; a one-off 1,000,000 GMX buy-wall at $5; weekly deposit-buy-withdraw buybacks (2M+ GMX repurchased since late 2024); 27% of protocol fees go to the Treasury for buybacks. GMX Labs named its first CEO (an anonymous community member) on about 2026-05-08. A proposal (GMLabs-003, around 2026-10-21) would send remaining Solana-side revenue to GMX buybacks and add a 180-day unstaking countdown on Arbitrum and Avalanche; adoption status is [NOT FOUND] (reported by news outlets only).
- **Product expansion (RWA and dollar yield).** 24/7 gold and silver perps (2026-04-10), WTI, Brent and natural gas (2026-04-24), MEGA/USD (2026-04-30), pre-IPO SpaceX SPCX/USD (2026-06-10), QQQ/USD and SPY/USD (2026-08-25), lower funding costs (2026-05-15), referral and balancer programs (July 2026), and agent and developer integration stack (2026-06-26). On 2026-10-06 GMX launched the GMX Dollar Vault (GLV [USDG-USDG] on Arbitrum, USDG from Paxos): it earns 63% of fees from the BTC, ETH and SOL markets it backs, estimates 7% to 12% APR, and has an 8%+ launch boost for 8 weeks on up to $100M of program TVL funded with Arbitrum.
- **Oracle for new markets.** The gold, silver and energy markets are stated to be powered by Chainlink; the oracle source for the pre-IPO SPCX market is [NOT FOUND].
- **Audit.** DefiLlama lists a Guardian review report dated 2026-06-16 for gmx-synthetics (link only; contents not opened).
- **No V2 exploit found in 2026.** The 2025-07-09 GMX V1 reentrancy exploit (about $42M, per BlockSec) affected V1 only and is not new.

**Why it matters:** GMX V2 is stable but not growing. It remains a single-chain-dominated (Arbitrum) Chainlink-priced LP-pool venue; the secondary chains have mostly emptied; and the growth effort has shifted to RWA markets, a stablecoin-denominated LP vault, and the separately branded Solana deployment.

## Adoption metrics

| Metric | Value | Date | Source | Change since May 2026 (May value) |
|--------|-------|------|--------|-----------------------------------|
| TVL (V2 perps total) | $187.05M | 2026-11-16 | [DefiLlama API, gmx-v2-perps](https://api.llama.fi/protocol/gmx-v2-perps) | -8.1% (was $203.47M) |
| TVL on Arbitrum | $180.07M | 2026-11-16 | [DefiLlama API, gmx-v2-perps](https://api.llama.fi/protocol/gmx-v2-perps) | -5.9% (was $191.43M) |
| TVL on Avalanche | $6.74M | 2026-11-16 | [DefiLlama API, gmx-v2-perps](https://api.llama.fi/protocol/gmx-v2-perps) | -38.7% (was $10.99M) |
| TVL on MegaETH | $0.23M | 2026-11-16 | [DefiLlama API, gmx-v2-perps](https://api.llama.fi/protocol/gmx-v2-perps) | -77.6% (was $1.03M) |
| TVL on Botanix | $0 | 2026-11-16 | [DefiLlama API, gmx-v2-perps](https://api.llama.fi/protocol/gmx-v2-perps) | Not captured in May (about $0.02M on 2026-05-02) |
| GMX Solana (GMTrade.xyz) TVL, separate venue | $22.61M | 2026-11-16 | [DefiLlama API, gmx-solana](https://api.llama.fi/protocol/gmx-solana) | Was $41.1M on 2026-05-22 (DefiLlama series) |
| 24h volume | [NOT FOUND] on DefiLlama (volume endpoints paywalled; protocol page HTTP 403 to automated fetch) | 2026-11-16 | n/a | May value $106.46M. A secondary tracker reports a 24h perp volume benchmark of $67.82M, labelled as a benchmark so possibly not GMX-specific ([OpenChainBench](https://openchainbench.com/perp/gmx-v2)) |
| 30d volume | $2.17B (secondary tracker) | 2026-11-16 | [OpenChainBench GMX v2](https://openchainbench.com/perp/gmx-v2) | [NOT FOUND] for May; implied average about $72M per day |
| Open interest | $40.74M (latest daily value) | 2026-11-16 | [DefiLlama OI API, gmx-evm](https://api.llama.fi/summary/open-interest/gmx-evm) | -51% (was $83.6M on 2026-05-21 in the DefiLlama daily series). Series value was $60.5M on 2026-10-17 |
| 24h fees | $44,791 | 2026-11-15 | [DefiLlama fees API, gmx-evm](https://api.llama.fi/summary/fees/gmx-evm) | Not captured in May |
| 7d fees | $426,548 | 2026-11-16 | [DefiLlama fees API, gmx-evm](https://api.llama.fi/summary/fees/gmx-evm) | -6.5% (was $456,470) |
| 30d fees | $1,433,296 | 2026-11-16 | [DefiLlama fees API, gmx-evm](https://api.llama.fi/summary/fees/gmx-evm) | Not captured in May; May 2026 calendar-month fees about $2.0M (derived) |
| Tradable markets | 114 | 2026-11-16 | [OpenChainBench GMX v2](https://openchainbench.com/perp/gmx-v2) | [NOT FOUND] for May |
| GMX token price | $7.30 | 2026-11-16 | [DefiLlama coins API](https://coins.llama.fi/prices/current/coingecko:gmx) | [NOT FOUND] for May |
| Cumulative trading volume (all chains, since launch) | >$345B across >735K traders | 2026 | [Coinbureau GMX review](https://coinbureau.com/review/gmx-review/) | Stale, from May note; not re-verified |

Note: the DefiLlama protocol web pages returned HTTP 403 to automated fetches on 2026-11-16, so TVL, fees and open interest come from DefiLlama's public API endpoints that back those pages. The OI figure is a point-in-time snapshot (the 7d and 30d "totals" in that endpoint are sums of daily snapshots and are not used). OpenChainBench is a secondary tracker; its 30d volume is consistent with DefiLlama's 30d fees ($1.43M / $2.17B is about 6.6 bps, matching its stated 6 bps all-in fee) but is not a second primary source.

## How it works

### User perspective

A trader deposits supported collateral on Arbitrum or Avalanche (MegaETH is nearly empty and Botanix shows no TVL), or via the Ethereum mainnet and cross-chain GMX Account, then opens a long or short on a market (e.g. ETH/USD). Execution price is the Chainlink Data Streams index price at the moment a keeper picks up the request; the trader pays a borrow fee while the position is open and pays/receives funding based on long/short skew. No orderbook, no slippage based on order size: pricing is the oracle mid, and the cost of size is captured through funding and price impact.

LPs mint GM (single-market) or GLV (multi-market vault) tokens by depositing the pool's basket assets, or since 2026-10-06 USDG via the Dollar Vault. They earn 63% of fees (the DefiLlama methodology gives 37% to the protocol: 10% treasury and 27% GMX holders) but absorb the pool's PnL versus traders. The previous "60-70%" range in this note is superseded by the 63% figure.

### Protocol perspective

- **GM pools:** isolated single-market pools. Each backs one market (e.g. ETH/USD-WETH-USDC).
- **GLV vaults:** yield vaults that dynamically allocate liquidity across multiple GM pools based on utilisation. The Dollar Vault is a GLV holding GM pool tokens across BTC, ETH and SOL markets, funded in USDG.
- **Chainlink Data Streams:** pull-based, low-latency oracle. Keepers fetch a signed price report and post it with the order; the contract verifies the signature and uses the report price for execution.
- **Two-step keeper execution:** order requests are submitted, then a keeper executes against the next valid oracle update. This blocks oracle front-running because the executor cannot choose a stale price.

## Key behaviours

- [[patterns/lp-counterparty-risk]] : GM/GLV LPs are direct counterparty to traders.
- [[patterns/oracle-pricing-model]] : Chainlink Data Streams pull oracle as the canonical execution price.
- [[patterns/keeper-based-execution]] : keepers submit oracle updates and execute orders.

## Architecture decisions

- **Oracle-mid pricing:** zero slippage on execution because price comes from oracle, not from an orderbook depth curve. Size cost is captured via dynamic funding and price impact, not bid/ask depth.
- **Permissionless market listing in V2:** markets can be added with minimal governance, expanding beyond crypto majors (114 markets tradable per OpenChainBench on 2026-11-16).
- **Multi-chain expansion:** beyond Arbitrum and Avalanche, GMX deployed to MegaETH and Botanix (a BTC-anchored EVM), but by November 2026 their TVL is near zero. A separate Solana deployment, now GMTrade.xyz, runs as its own venue. Ethereum mainnet access was added on 2025-12-22.
- **GLV vault for capital efficiency:** allocates LP capital across GM pools by demand, lifting average capital utilisation.
- **Governance restructuring:** from March 2026 the DAO has centralised GMX token liquidity on its own venues, redirected staking rewards to the Treasury, and in May appointed a Labs CEO.

## Differentiators

- One of the largest oracle-priced (non-orderbook) perp DEXes by cumulative volume.
- RWA perp expansion: 24/7 gold, silver, oil, gas (April 2026), pre-IPO SpaceX (June), and Nasdaq-100 and S&P 500 ETF perps (August). The earlier "$1.18B volume in one week" claim was not re-verified.
- Stablecoin-denominated LP vault (Dollar Vault) removing crypto price exposure from the LP deposit asset (LPs still carry trader PnL risk).
- Multi-chain presence, though secondary chains are now small.

## Limitations and criticisms

- LPs (GM/GLV) carry directional risk: when traders are skewed long during a rally, the pool pays out. V1 LPs (GLP) saw drawdowns during sharp moves.
- Oracle dependence is total: a malicious or stale Chainlink Data Streams update would mis-price liquidations directly. Keeper enforcement of "next valid update" is the main mitigation. (The July 2026 Ostium incident, see [[projects/ostium]], shows that an operator-controlled price path is a larger risk than the oracle network itself; GMX V2 uses Chainlink reports verified on-chain.)
- Funding-rate model can be aggressive in skewed conditions, pushing traders out of positions; GMX cut funding costs on 2026-05-15.
- TVL, open interest and fees have trended down through 2026 as Hyperliquid and orderbook competitors took share; monthly fees fell from about $3.5M (March) to about $1.3M (October) on DefiLlama data (derived).
- Token governance has been contentious (staking rewards suspended until GMX exceeds $90; GMX at $7.30 on 2026-11-16).

## Sources

- [DefiLlama GMX V2 Perps page](https://defillama.com/protocol/gmx-v2-perps) : accessed 2026-05-19 (HTTP 403 to automated fetch on 2026-11-16)
- [DefiLlama API, gmx-v2-perps](https://api.llama.fi/protocol/gmx-v2-perps) : accessed 2026-11-16
- [DefiLlama API, gmx-solana](https://api.llama.fi/protocol/gmx-solana) : accessed 2026-11-16
- [DefiLlama fees API, gmx-evm](https://api.llama.fi/summary/fees/gmx-evm) : accessed 2026-11-16
- [DefiLlama OI API, gmx-evm](https://api.llama.fi/summary/open-interest/gmx-evm) : accessed 2026-11-16
- [DefiLlama fees API, gmx-solana](https://api.llama.fi/summary/fees/gmx-solana) : accessed 2026-11-16
- [DefiLlama coins API (GMX price)](https://coins.llama.fi/prices/current/coingecko:gmx) : accessed 2026-11-16
- [OpenChainBench GMX v2](https://openchainbench.com/perp/gmx-v2) : accessed 2026-11-16
- [GMX Substack archive (2026 posts)](https://gmxio.substack.com/archive) : accessed 2026-11-16
- [GMX Substack : Deposit dollars, earn dollars (GMX Dollar Vault, 2026-10-06)](https://gmxio.substack.com/p/deposit-dollars-earn-dollars-the) : accessed 2026-11-16
- [GMX Substack : Governance Update, Strategic Plan to Restore Price Discovery (2026-03-04)](https://gmxio.substack.com/p/governance-update-a-strategic-plan) : accessed 2026-11-16
- [GMX Substack : GMX is live on Ethereum (2025-12-22)](https://gmxio.substack.com/p/gmx-is-live-on-ethereum) : accessed 2026-11-16 (previously 2026-05-19)
- [GMX governance forum : CEO search](https://gov.gmx.io/t/ceo-search-applications-are-now-open-deadline-april-3-2026/5064) : listed in search results on 2026-11-16, not opened
- [KuCoin News : GMX Labs appoints first CEO, 168,500 GMX buybacks](https://www.kucoin.com/news/flash/gmx-labs-appoints-first-ceo-and-completes-168-500-gmx-token-buybacks) : listed in search results on 2026-11-16, not opened
- [KuCoin News : GMX proposes Solana revenue for GMX buybacks and 180-day unstaking lock (GMLabs-003)](https://www.kucoin.com/news/flash/gmx-proposes-solana-revenue-for-gmx-buybacks-and-180-day-unstaking-lock) : listed in search results on 2026-11-16, not opened; adoption status unverified
- [BlockSec : GMX V1 incident (2025-07-09)](https://blocksec.com/blog/gmx-incident-cross-contract-reentrancy-bypasses-a-four-year-old-guard) : accessed 2026-11-16
- [GMX docs](https://docs.gmx.io/docs/intro/) : accessed 2026-05-19
- [GMX-Solana adopts Chainlink Data Streams](https://gmxio.substack.com/p/gmx-solana-adopts-chainlink-data) : accessed 2026-05-19
- [GMX V2 a quick guide (blocmates)](https://www.blocmates.com/articles/gmx-v2-a-quick-guide-to-the-upgrade) : accessed 2026-05-19
- [Coinbureau GMX review 2026](https://coinbureau.com/review/gmx-review/) : accessed 2026-05-19
