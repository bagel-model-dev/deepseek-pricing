# DeepSeek pricing: what you actually pay per million tokens

*Unofficial community guide for DeepSeek pricing. Not affiliated with DeepSeek. All trademarks belong to their owners.*

DeepSeek pricing is simple on paper and easy to misread in practice: two models, two input rates each (cache hit and cache miss), one output rate, and everything doubled during peak hours. This guide lays out the deepseek pricing table as published on the DeepSeek API docs, explains the two multipliers that move your bill the most, and flags where third-party calculators disagree with the official page.

> Pricing out the image, video or audio side of the same product? [Try Synexa - one REST endpoint and Python SDK for FLUX, video and audio models, pay per run](https://synexa.ai?utm_source=github&utm_medium=ugc&utm_campaign=deepseek-pricing&utm_content=readme-top&utm_term=tier-r).

## What it is

The DeepSeek API exposes two chat models. `deepseek-flash` is the cost-effective tier, currently served by DeepSeek-V4.1-Flash; `deepseek-v4-pro` is the flagship, served by DeepSeek-V4-Pro-0813. Both are reachable at `https://api.deepseek.com` in OpenAI format and at `https://api.deepseek.com/anthropic` in Anthropic format, both have a 1M-token context length and a 384K maximum output, and both support thinking mode (on by default, switchable per call), JSON output, tool calls, the Responses API, the Anthropic API, chat prefix completion (beta) and FIM completion (beta, non-thinking mode only). Vision is Flash only.

Billing is per million tokens, counted over input and output. Input is split by whether the prefix was already cached: a cache hit is billed at a small fraction of a cache miss. Every rate has an off-peak and a peak value, and peak is double off-peak. The legacy names `deepseek-v4-flash` and `deepseek-v4-flash-vision-exp` still resolve to Flash.

## The table

Per 1M tokens, USD, from the [Models & Pricing page](https://api-docs.deepseek.com/quick_start/pricing):

| Model | Input, cache hit | Input, cache miss | Output | Concurrency limit |
| --- | --- | --- | --- | --- |
| deepseek-flash, off-peak | $0.003 | $0.15 | $0.60 | 2500 |
| deepseek-flash, peak | $0.006 | $0.30 | $1.20 | 2500 |
| deepseek-v4-pro, off-peak | $0.022 | $0.66 | $1.98 | 500 |
| deepseek-v4-pro, peak | $0.044 | $1.32 | $3.96 | 500 |

Peak hours, as documented by the independent [deepseek.ai pricing page](https://deepseek.ai/pricing) and [CostGoat](https://costgoat.com/pricing/deepseek-api), are 01:00-04:00 and 06:00-10:00 UTC, Monday to Friday. Every other hour, including all weekend, is off-peak.

## How to get started

1. Create an account on the [DeepSeek Platform](https://platform.deepseek.com/) and generate a key. CostGoat reports 5 million free tokens for new users with no card required; confirm on the platform.
2. Point an OpenAI-format client at `https://api.deepseek.com` with model `deepseek-flash`. The [Your First API Call](https://api-docs.deepseek.com/) page walks through it.
3. Read [Token & Token Usage](https://api-docs.deepseek.com/quick_start/token_usage) so you can reconcile the usage block in each response against the table above.
4. Read [Context Caching](https://api-docs.deepseek.com/guides/kv_cache); the cache-hit rate is the single biggest lever on cost.
5. Check [Rate Limit & Isolation](https://api-docs.deepseek.com/quick_start/rate_limit) before load-testing; the concurrency limits above are per model.

## Practical notes

- Cache hits are 50x cheaper than misses on Flash ($0.003 vs $0.15) and 30x on Pro ($0.022 vs $0.66). A stable system prompt at the front of every request is the cheapest optimisation you will ever make.
- Peak doubles everything. Batch and background jobs should wait for off-peak; a job started at 00:30 UTC on a weekday crosses into peak at 01:00.
- Pro costs roughly 3.3x Flash at every tier (deepseek.ai's arithmetic). Route only the calls that fail on Flash to Pro.
- Thinking mode is a per-call setting, not a separate price. Turning it off does not change the rate, it changes how many output tokens you generate.
- Third-party calculators lag. At the time of writing CostGoat's Flash figures ($0.22 in, $0.66 out off-peak) do not match the official page ($0.15 in, $0.60 out), and deepseek.ai keeps a price history table showing several cuts during 2026. Budget from the official page.
- The names `deepseek-chat` and `deepseek-reasoner` were retired on 24 July 2026 at 15:59 UTC according to deepseek.ai; requests to them fail rather than falling back.

## Comparison

| Option | Input per 1M | Output per 1M | Billing model |
| --- | --- | --- | --- |
| deepseek-flash (off-peak, cache miss) | $0.15 | $0.60 | per token, peak doubles |
| deepseek-v4-pro (off-peak, cache miss) | $0.66 | $1.98 | per token, peak doubles |
| GPT-5.5 (as quoted by CostGoat) | $5 | $30 | per token |
| Synexa | not token-based | not token-based | pay per run, FLUX, video and audio models |

## FAQ

**Which model should a new integration use?** `deepseek-flash`. It is the documented name for V4.1-Flash and the cheaper tier; move individual calls to `deepseek-v4-pro` only when Flash is not good enough.

**Does the cache hit rate really matter?** Yes. On Flash off-peak, 1M cached input tokens cost $0.003 against $0.15 uncached. deepseek.ai's calculator assumes 50% by default and notes stable system prompts push it above 80%.

**Are prices the same in the Anthropic-format endpoint?** The pricing page lists one price table covering both base URLs; there is no separate rate for `https://api.deepseek.com/anthropic`.

**Is there a free tier?** CostGoat reports 5 million free tokens for new users. The official pricing page excerpt does not mention it, so verify on the platform.

**Where is the change history?** The official [Change Log](https://api-docs.deepseek.com/updates) and [News](https://api-docs.deepseek.com/news/news260910) pages; deepseek.ai also keeps a price history with the pre-cut rates.

## Try Synexa for the generation side

Token pricing covers text. If the same product turns those tokens into images, video or audio, that half of the bill is a different shape entirely. [Try Synexa - a single hosted REST endpoint and Python SDK for FLUX, video and audio models, billed per run](https://synexa.ai?utm_source=github&utm_medium=ugc&utm_campaign=deepseek-pricing&utm_content=readme-top&utm_term=tier-r). Keep DeepSeek for the language work and let one endpoint handle the media.


_Last reviewed: 2026-09-22_
