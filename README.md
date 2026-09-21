# GPT-5.5 API (gpt-5.5 / gpt-5.5-pro)

<!-- conv-kit:v1 -->

<p align="center">
  <img src="assets/badges/price.svg" alt="observed unit price"> <img src="assets/badges/billing.svg" alt="billing model"> <img src="assets/badges/compat.svg" alt="OpenAI-compatible endpoint">
</p>

> **$4 / $24 per million tokens** (input / output, effective) — one OpenAI-compatible endpoint at `https://api.apimart.ai/v1`, no monthly plan required. *(observed 2026-09-17)*

**[Get an API key](https://go.apimart.ai/k-13f276)** · **[Live pricing](https://go.apimart.ai/k-afebde)** · **[Model page](https://go.apimart.ai/k-0a4ba4)** · [⚡ 60-second quickstart](#quickstart)

**Why teams call GPT-5.5 (`gpt-5.5`) through APIMart**

- **One key, entire catalog.** The same `https://api.apimart.ai/v1` base URL and `Authorization` header reach GPT-5.5 (`gpt-5.5`) and 300+ other image, video and language models — switch the `model` field, not your client.
- **$1 minimum, pay as you go.** No subscription and no prepaid plan to size up front: top up from $1 and spend it on calls. There is no free quota to burn through first, so the price in this table is the price you pay.
- **The charge comes back in the response.** Every call reports the amount billed (`cost` / `credits_cost`), so a spend number is read per call instead of guessed at month end.
- **Drop-in OpenAI shape.** `POST /v1/chat/completions` with the same request body your client already sends; only `base_url` and `model` change.

<!-- /conv-kit:v1 -->

GPT-5.5 on APIMart is an OpenAI-compatible chat route billed per million tokens, with a `-pro` variant for the hardest prompts. This repository documents the model ids, the measured token rates, the request shapes (non-streaming, streaming, tool calling, JSON mode) and the outputs of real calls.

## Model ids

| Model id | Tier | Typical use |
| --- | --- | --- |
| `gpt-5.5` | highest quality | hard prompts, long-form reasoning |
| `gpt-5.5-pro` | fast/cheap tier | high-volume extraction, classification, routing |

Endpoint: `POST https://api.apimart.ai/v1/chat/completions` (OpenAI-compatible). **Streaming is the default** — pass
`stream: false` when you want one JSON object back.

## Pricing (per million tokens)

<!-- pricing:token:start -->
| Token direction | List price / 1M | Effective price / 1M |
| --- | --- | --- |
| cached_input | $0.5 | $0.4 |
| input | $5.00 | $4.00 |
| output | $30.00 | $24.00 |
<!-- pricing:token:end -->

The effective column is what you pay after the default group discount; [`data/model.json`](data/model.json) is refreshed
daily by CI, and the `usage` block in every response tells you exactly which tokens were billed.

## Verified capabilities

| Capability | Verified behaviour |
| --- | --- |
| Non-streaming chat | `stream: false` returns a single `chat.completion` object with `usage` |
| Streaming | the endpoint streams by default; iterate the SSE `data:` lines and stop at `[DONE]` |
| Tool calling | `tools` with JSON-schema functions returns `finish_reason: tool_calls` and a populated `tool_calls` array |
| JSON mode | `response_format: {"type": "json_object"}` returns parseable JSON |
| System messages | standard `system`/`user` message roles are accepted |

## Quickstart

```bash
curl -sS https://api.apimart.ai/v1/chat/completions \
  -H "Authorization: Bearer $APIMART_API_KEY" -H 'Content-Type: application/json' \
  -d '{"model":"gpt-5.5","stream":false,"messages":[{"role":"user","content":"Name three retry rules."}]}'
```

```python
import os, requests

BASE = "https://api.apimart.ai/v1"
HEADERS = {"Authorization": f"Bearer {os.environ['APIMART_API_KEY']}", "Content-Type": "application/json"}

payload = requests.post(f"{BASE}/chat/completions", headers=HEADERS, timeout=120, json={
    "model": "gpt-5.5", "stream": False,
    "messages": [{"role": "user", "content": "Name three retry rules."}],
}).json()
print(payload["choices"][0]["message"]["content"])
print(payload["usage"])
```

Streaming, tool calling and JSON mode examples are in [`examples/`](examples) (`t_curl.sh`, `python_chat.py`).

## Real call outputs

These rows are actual completions recorded from this route, with the token usage the API returned and the cost computed
from the effective rates above.

| Prompt | Response excerpt | Tokens (in/out) | Reported cost |
| --- | --- | --- | --- |
| `Explain idempotency keys to a backend engineer in five sentences, then give one curl examp` | Idempotency keys are client-generated unique tokens attached to requests that may be retried, typically `POST` requests with side effects.   The server stores the first request’s outcome for that key, including success o… | 25 / 321 | $0.0078 |
| `Review this Python snippet and list concrete issues only: requests.post(url, json=payload)` | - No `timeout` specified: a retry attempt can hang indefinitely. - Retrying `POST` can create duplicate side effects if the server processed the request but the client timed out or lost the response. - Generating a new `… | 36 / 474 | $0.0115 |
| `Return strict JSON with keys model, unit, price for: 'gpt-image-2.5-ext bills 0.0085 USD p` | {"model":"gpt-image-2.5-ext","unit":"1080p image","price":0.0085} | 44 / 83 | $0.0022 |

Full transcripts (including longer answers) are in [`data/samples.json`](data/samples.json).

<!-- conv-kit:v1:fix -->
## First-call troubleshooting

| Symptom | Likely cause | Fix |
| --- | --- | --- |
| `401` / `invalid api key` | key missing, truncated, or a stray newline pasted into the header | Re-copy it from the console; the header is `Authorization: Bearer $APIMART_API_KEY` |
| balance / credit error | the account has no balance | Top up from $1 in the console — there is no free quota to fall back on |
| `429` | concurrent requests on one key | Back off, then retry the same request with the same `Idempotency-Key` |
| `400` / model not found | wrong route for the id: the per-unit alias needs its `version`, the official id must not send one | Copy the exact `model` value from the route table above |
| task ends `failed` | prompt rejected by the filter, or a reference image URL expired | Re-submit with a **new** `Idempotency-Key` and re-host the reference image |
| result URL stops working | result links expire | Download the file as soon as the task reports `completed` |
<!-- /conv-kit:v1:fix -->

## FAQ

**What are the GPT-5.5 API model ids?**

`gpt-5.5` and `gpt-5.5-pro`. Calls return a dated snapshot id (for example `gpt-5.5-2026-04-24`), which is useful to log if you need reproducible outputs.

**How is GPT-5.5 billed?**

Per million tokens, with separate input, cached-input and output rates — the table above is refreshed from the public pricing payload and the effective column is what you actually pay.

**Why do I get an empty body when I call the endpoint?**

Because streaming is the default. Pass `stream: false` for a single JSON response, or parse the SSE `data:` lines.

**Is the request shape the same as OpenAI's?**

For chat, streaming, tool calling and JSON mode, yes. Keep the base URL and key in environment variables so you can switch routes without touching business logic.

## Related searches

- `gpt 5.5 api`
- `gpt-5.5 api pricing`
- `gpt 5.5 api key`
- `openai compatible api`
- `llm api pricing comparison`
- `cheapest llm api`
- `tool calling api`

<!-- conv-kit:v1:cta -->
---

**Start with $1.** [Get an API key](https://go.apimart.ai/k-13f276) → [check live pricing](https://go.apimart.ai/k-afebde) → [open GPT-5.5 (`gpt-5.5`) in the model library](https://go.apimart.ai/k-0a4ba4). The first call is three steps: submit, poll `task_id`, read the charged amount off the response.
<!-- /conv-kit:v1:cta -->

## Attributed links (how this repository is measured)

| Purpose | Attributed link | Target |
| --- | --- | --- |
| Browse the model catalog | <https://go.apimart.ai/k-0a4ba4> | `apimart.ai/model` |
| Current pricing page | <https://go.apimart.ai/k-afebde> | `apimart.ai/pricing` |
| Get an API key | <https://go.apimart.ai/k-13f276> | `apimart.ai/keys` |

Outbound APIMart links are minted through the promo link API; hand-made tracking parameters are rejected by
`tools/check_links.py` in CI.

## Disclosure

GPT-5.5 is a third-party model served through APIMart; this repository publishes model ids, measured prices and
real call outputs, and does not claim official status. Model names, prices and documentation belong to their respective
owners. Endpoint reference: [https://docs.apimart.ai/en/api-reference/texts/general/chat-completions](https://docs.apimart.ai/en/api-reference/texts/general/chat-completions).

## Repository map

```text
README.md             model ids, token pricing, verified capabilities, real outputs
data/model.json       token rates for every tier (CI-refreshed)
data/samples.json     recorded completions with usage and computed cost
tools/snapshot.py     refresh pricing from the public payload
tools/check_links.py  attribution guard
examples/             curl and Python clients (streaming, tools, JSON mode)
.github/workflows/    daily price refresh + validation
```

## License

MIT — see [LICENSE](LICENSE).
