# Second-Hand Market Pricing Methodology

Source: archived skill `second-hand-market-pricing`

## Goal

Produce a **defensible pricing recommendation** based on **comparable second-hand samples**, not generic new-product prices.

## Core Rules

1. **Do not mix markets** — second-hand primary; new retail only as ceiling reference
2. **Prefer directly comparable samples** — same model, same generation, same config
3. **Separate buckets** — domestic vs overseas; ask price vs 成交价
4. **When platform blocks scraping, say so** — fall back to V2EX 二手区, forum posts
5. **Do not over-infer from weak samples** — say "sample quality limited" rather than inventing certainty

## Workflow

1. Define sale unit precisely (bare host, RAM, SSD separately)
2. Search: `<model> 二手 价格`, `site:v2ex.com <model> 二手交易`
3. Gather component samples separately (RAM, SSD capacity/health)
4. Normalize samples: source, date, exact config, price, sold/asking, confidence level
5. Produce three price bands:
   - **保守快出价**: moves quickly
   - **正常挂价**: most reasonable
   - **偏高试探价**: optimistic

## Pricing Logic

Host value + RAM standalone × 0.7 + SSD standalone × 0.7 = rough bundle floor
(installed parts typically add < their standalone resale value)

## Output Template

1. 结论先行 (three price bands)
2. 样本来源与口径
3. 整机样本 (config, price, source, confidence)
4. 配件样本
5. 定价推导
6. 最终建议挂价

## Pitfalls

- **Don't** blend Amazon/JD new prices as "market price"
- **Don't** call blocked Goofish data "verified" — say "非法访问" blocked
- **Don't** treat installed parts as fully additive
- eBay/Facebook prices are reference only, not domestic proof
