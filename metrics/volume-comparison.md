---
tags: [metrics, volume, perpetuals]
updated: 2026-11-16
---

# Volume Comparison

Trading volume across the protocols, where "volume" generally means notional perpetuals volume. Refreshed 2026-11-16. DefiLlama's volume endpoints returned HTTP 402 and its web pages HTTP 403 during this refresh, so most volume figures come from secondary sources that cite DefiLlama or from venue APIs. That is weaker than the May snapshot's direct DefiLlama reads; treat every volume row as approximate and see the conflicts listed below.

## Ethereum, L2s and Solana

| Protocol | 24h volume | 30d / quarterly volume | Open interest | Date | Source |
|----------|-----------|------------------------|---------------|------|--------|
| [[projects/lighter]] | ~$3.1B | Q3 $126.9B | ~$605M | 2026-11 | [CoinGecko](https://api.coingecko.com/api/v3/derivatives/exchanges), [Dwellir](https://www.dwellir.com/blog/state-of-defi-q3-2026) |
| [[projects/variational]] | ~$1.06B | Q3 $104.0B | $697M | 2026-11 | [Dwellir](https://www.dwellir.com/blog/state-of-defi-q3-2026), [DefiLlama OI](https://api.llama.fi/overview/open-interest) |
| Aster (BNB Chain centred; not a vault note) | [NOT FOUND] | Q3 $154.8B | $1.16B | 2026-11 | [Dwellir](https://www.dwellir.com/blog/state-of-defi-q3-2026) |
| [[projects/jupiter-perps]] | $318.8M (SOL, ETH, BTC markets summed); $321.6M on a third-party tracker | Q3 $16.7B, Q2 $19.1B, Sep $6.3B; 7d/30d [NOT FOUND] | $28.0M (DefiLlama; other sources give $31M to $106M, definitions differ) | 2026-11-16 | [Jupiter API](https://perps-api.jup.ag/v1/market-stats?mint=So11111111111111111111111111111111111111112), [Pine Analytics](https://pineanalytics.substack.com/p/jupiter-quarterly-report-q3-2026) |
| [[projects/gmx-v2]] | [NOT FOUND] | 30d $2.17B (secondary); Q3 $64.5B for "GMX combined" (secondary; unreconciled, see below) | $40.74M (May: $83.6M) | 2026-11-16 | [OpenChainBench](https://openchainbench.com/perp/gmx-v2), [DefiLlama](https://api.llama.fi/summary/open-interest/gmx-evm) |
| [[projects/ostium]] | [NOT FOUND] | 30d $47.08M (May: ~$6B a month) | $9.58M (DefiLlama; May $183.5M) | 2026-11-15 | [OpenChainBench](https://openchainbench.com/perp/ostium), [DefiLlama](https://api.llama.fi/summary/open-interest/ostium) |
| [[projects/drift]] / Velocity | $232K | [NOT FOUND] | $193K (pre-hack: $62M on 2026-04-01) | 2026-11-16 | [bykaranteli tracker](https://bykaranteli.com/solana-perps), [DefiLlama](https://api.llama.fi/summary/open-interest/velocity) |
| [[projects/synthetix-v3]] / mainnet exchange | $2.11M (ETH-USDT market only) | V3: no activity; cumulative V3 volume [NOT FOUND] (May: $6.962B) | $0.297M (DefiLlama) to $2.68M (homepage, ETH-USDT only); unresolved | 2026-11-16 | [Synthetix](https://synthetix.io/), [DefiLlama](https://api.llama.fi/summary/open-interest/synthetix-v4) |

## Fees (DefiLlama API, 2026-11-15 and 2026-11-16)

| Protocol | 24h | 7d | 30d |
|----------|-----|----|-----|
| Jupiter Perps | $599K | $3.85M | $10.0M |
| GMX V2 | [NOT FOUND] | $426,548 (May: $456,470) | $1.43M |
| Ostium | $1,040 | $21,573 | $36,710 (May: about $4.9M, derived) |
| Synthetix mainnet exchange | $136 | [NOT FOUND] | $3,718 (all-time $24,716) |
| Ethena USDe | [NOT FOUND] | [NOT FOUND] | $18.3M |

Sources: [Jupiter](https://api.llama.fi/summary/fees/jupiter-perpetual-exchange), [GMX](https://api.llama.fi/summary/fees/gmx-evm), [Ostium](https://api.llama.fi/summary/fees/ostium), [Synthetix](https://api.llama.fi/summary/fees/synthetix-v4), [category](https://api.llama.fi/overview/fees).

## Category totals (secondary sources citing DefiLlama)

- Perp DEX volume: Q2 2026 $1,813B; Q3 to 25 September $1,619.7B; rolling 30 days about $633B to $647B in early to mid October; October about $659B (one search extract). Peak was $1.36T in October 2025 ([Cointelegraph](https://cointelegraph.com/news/onchain-perp-dex-volumes-decline-hyperliquid-share), [Dwellir](https://www.dwellir.com/blog/state-of-defi-q3-2026)).
- Share of global perp volume: 12.33% in September 2026, against a 14.25% peak in December 2025 ([Castle Labs via Forklog](https://forklog.com/en/perp-dex-trading-volume-share-rises-to-12/)).
- Q3 share by venue (to 25 September): Hyperliquid 37.3%, Aster 9.6%, Lighter 7.8%, Variational 6.4%, ApeX 6.0%, edgeX 5.4%, GMX 4.0%, Grvt 3.7%, dYdX 0.2% (Dwellir). Other sources put Hyperliquid between 31% and 63% depending on window and denominator.
- Perps-only open interest: $10.38B (sum of DefiLlama's Derivatives category, matching Castle Labs' $10.34B across 43 venues on 15 November); Hyperliquid 54%, Aster 11%, Variational 6.8%, Lighter 5.7%.
- RWA perp volume: $147.5B in July and $123B in August; trade.xyz and Variational are about 90% of it (Castle Labs).

## Observations

- **The orderbook venues overtook the oracle-priced pool venues.** Lighter and Variational each did more Q3 volume than GMX, Jupiter and Ostium combined; Jupiter's Q3 volume ($16.7B) is a small fraction of its claimed 2025 run rate.
- **Jupiter's "$263B in 2025" could not be reproduced from any primary source,** and the Q2 and Q3 2026 figures imply roughly $6B a month. The May note repeated the $263B figure without a primary citation.
- **Synthetix and Drift are now near zero**, and Ostium's 30d volume is about 1% of its April monthly level.

## Unreconciled conflicts

- GMX: Dwellir's $64.5B Q3 "GMX combined" against OpenChainBench's $2.17B for 30 days cannot both describe GMX V2 on EVM (about $21B a month versus $2.17B). Possibly the combined figure includes GMTrade (Solana), which a DefiLlama snippet showed at $2.1B 24h on 2026-11-04, while another tracker shows GMTrade at only $91M. Unresolved; check DefiLlama's paid volume series or Dune before citing either.
- Jupiter open interest ranges from $28M (DefiLlama) to $106M (Pine, 2026-09-30); definitions differ.

## Methodology notes

- Daily volume is highly variable; favour 7d or 30d averages where possible.
- "Notional volume" varies by venue: Hyperliquid reports gross taker volume; GMX reports executed position size in USD; dYdX reports matched trade notional. Direct cross-comparison is approximate.
- For LP-pool venues (GMX, Jupiter), the headline volume figure includes both opening and closing of positions, so $X in "volume" implies roughly $X/2 in unique trader exposure.
- Hyperliquid and dYdX were not refreshed (outside this update's Ethereum and Solana scope); see their project notes for May 2026 values.
