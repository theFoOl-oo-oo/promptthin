# PromptThin

> **Reduce LLM API costs through caching, compression, and smart routing. Zero code changes.**

PromptThin is a transparent proxy that sits between your AI agents and LLM providers. Two environment variables and you're done — every API call gets five compounding savings routes applied automatically.

```
Your app ──→ PromptThin ──→ OpenAI / Anthropic / Gemini / Groq
```

[![Website](https://img.shields.io/badge/website-promptthin.tech-brightgreen)](https://promptthin.tech)
[![Free trial](https://img.shields.io/badge/trial-7_days_free-blue)](https://promptthin.tech)

---

## Where PromptThin works

PromptThin saves tokens when **you control the API call** — your own code, AI agents, or a self-hosted chat UI. It does **not** intercept calls made by managed chat interfaces like claude.ai, ChatGPT, or similar products; those platforms call LLM APIs internally and cannot be proxied.

| Scenario | PromptThin works? |
|---|---|
| Your app code calling OpenAI / Anthropic / Gemini / Groq | ✅ Yes |
| AI agents (LangChain, AutoGen, CrewAI, etc.) | ✅ Yes |
| Self-hosted chat UIs (Open WebUI, LibreChat, Cursor, Continue.dev) | ✅ Yes |
| `proxy_chat` MCP tool called by Claude Desktop / Claude Code | ✅ Yes — for that specific outbound LLM call |
| claude.ai chat interface | ❌ No — Anthropic controls that pipe |
| ChatGPT / Gemini web apps | ❌ No — provider controls that pipe |

> **Tip for heavy claude.ai users:** If you're hitting usage quota limits in the claude.ai chat, the fix is to use a self-hosted UI like [Open WebUI](https://openwebui.com) or [LibreChat](https://librechat.ai) pointed at the Anthropic API through PromptThin. You get the same chat experience with compression and caching reducing every turn's token cost.

---

## Five savings routes

| Route | What it does | Saving |
|---|---|---|
| **Semantic Cache** | Returns cached answers for similar questions — even if worded differently | Up to 100% on repeated queries |
| **Prompt Compression** | Compresses verbose prompts with LLMLingua 2 before sending | Up to 50% on input tokens |
| **Model Router** | Automatically routes simple tasks to cheaper models in <1ms | Up to 90% per request |
| **Context Pruning** | Summarises long conversation history when it exceeds 8K tokens | Up to 60% on long threads |
| **Thinking Budget** | Caps reasoning tokens on thinking models (Claude, o-series, Gemini 2.5/3) based on task complexity | Up to 80% on thinking tokens |

All five routes run on every request. You control which to skip per-request via headers.

The semantic cache only ever stores successful, well-formed answers — see [Cache correctness](#cache-correctness) below — and skips multimodal (image) requests by default — see [Vision and image requests](#vision-and-image-requests). Need part of a prompt to survive compression untouched? See [Protecting parts of a prompt from compression](#protecting-parts-of-a-prompt-from-compression).

---

## Get started in 2 minutes

### Step 1 — Create an account

Sign up at **[promptthin.tech](https://promptthin.tech)** — verify your email, then start your 7-day free trial (no charge for 7 days).

Or via API:

```bash
curl -X POST https://promptthin.tech/auth/register \
  -H "Content-Type: application/json" \
  -d '{"email": "you@example.com", "password": "yourpassword"}'
```

**Password requirements:** 8+ characters, uppercase, lowercase, number, special character.
Check your inbox for a verification email before making API calls.

### Step 2 — Register your LLM provider key

```bash
# OpenAI
curl -X POST https://promptthin.tech/keys/openai \
  -H "X-API-Key: ts_your_key" \
  -H "Content-Type: application/json" \
  -d '{"api_key": "sk-your-openai-key"}'

# Anthropic
curl -X POST https://promptthin.tech/keys/anthropic \
  -H "X-API-Key: ts_your_key" \
  -H "Content-Type: application/json" \
  -d '{"api_key": "sk-ant-your-anthropic-key"}'

# Gemini
curl -X POST https://promptthin.tech/keys/gemini \
  -H "X-API-Key: ts_your_key" \
  -H "Content-Type: application/json" \
  -d '{"api_key": "AIza-your-gemini-key"}'

# Groq
curl -X POST https://promptthin.tech/keys/groq \
  -H "X-API-Key: ts_your_key" \
  -H "Content-Type: application/json" \
  -d '{"api_key": "gsk_your-groq-key"}'
```

Your provider keys are encrypted with AES-256 and never appear in logs or responses.

### Step 3 — Point your app at PromptThin

```bash
# .env — two lines, no other changes needed
OPENAI_BASE_URL=https://promptthin.tech/v1
OPENAI_API_KEY=ts_your_key
```

Done. Every LLM call now routes through PromptThin and savings start immediately.

---

## Authentication

PromptThin accepts your PromptThin account key (`ts_...`) two ways:

- **`X-API-Key` header** — the dedicated header, works with any HTTP client
- **`Authorization: Bearer ts_...`** — for SDKs (like the OpenAI client) that only expose a single `api_key` field and always send it via `Authorization`

Both resolve to the same account; use whichever is more convenient for your client.

### Using the X-API-Key header

```bash
curl -X POST https://promptthin.tech/v1/chat/completions \
  -H "X-API-Key: ts_YOUR_API_KEY_HERE" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gemini-2.5-flash",
    "messages": [{"role": "user", "content": "Hello"}],
    "max_tokens": 300
  }'
```

`model` can be any supported model name (`gpt-*`, `claude-*`, `gemini-*`, `llama-*`/`mixtral-*`/`gemma-*`) — PromptThin infers the provider and translates the request/response shape automatically, so the same OpenAI-style call works across all four providers.

### With the Python OpenAI client

```python
from openai import OpenAI

client = OpenAI(
    base_url="https://promptthin.tech/v1",
    api_key="ts_your_key",   # sent as Authorization: Bearer ts_your_key
)

response = client.chat.completions.create(
    model="gemini-2.5-flash",
    messages=[{"role": "user", "content": "Hello"}],
    max_tokens=300,
)
```

This is the same pattern used throughout this README's integration examples — no `default_headers` needed. If you'd rather use the dedicated header instead, that also works:

```python
client = OpenAI(
    api_key="dummy",  # required by the SDK but unused when X-API-Key is set
    base_url="https://promptthin.tech/v1",
    default_headers={"X-API-Key": "ts_YOUR_API_KEY_HERE"},
)
```

**Note on `Authorization`:** this header serves a second purpose beyond carrying your `ts_` key — it's also how you pass a *provider* key directly in pass-through mode (see `What if I want to pass my provider key directly?` in the FAQ below). PromptThin distinguishes the two by prefix: `ts_` is treated as your account key, while `sk-`, `sk-ant-`, `AIza`, and `gsk_` are treated as provider keys and used directly for that request.

---

## Integration examples

### OpenAI SDK — Python

```python
from openai import OpenAI

client = OpenAI(
    base_url="https://promptthin.tech/v1",
    api_key="ts_your_key",
)

response = client.chat.completions.create(
    model="gpt-4o",
    messages=[{"role": "user", "content": "Hello!"}]
)
```

### OpenAI SDK — JavaScript / TypeScript

```typescript
import OpenAI from "openai";

const client = new OpenAI({
  baseURL: "https://promptthin.tech/v1",
  apiKey: "ts_your_key",
});
```

### Anthropic SDK — Python

```python
import anthropic

client = anthropic.Anthropic(
    base_url="https://promptthin.tech",
    api_key="ts_your_key",
)
```

### Anthropic SDK — JavaScript

```typescript
import Anthropic from "@anthropic-ai/sdk";

const client = new Anthropic({
  baseURL: "https://promptthin.tech",
  apiKey: "ts_your_key",
});
```

### LangChain

```python
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(
    base_url="https://promptthin.tech/v1",
    api_key="ts_your_key",
    model="gpt-4o",
)
```

### AutoGen

```python
config_list = [{
    "model": "gpt-4o",
    "base_url": "https://promptthin.tech/v1",
    "api_key": "ts_your_key",
}]
```

### CrewAI / any OpenAI-compatible framework

```bash
OPENAI_BASE_URL=https://promptthin.tech/v1
OPENAI_API_KEY=ts_your_key
```

### Vercel AI SDK

```typescript
import { createOpenAI } from "@ai-sdk/openai";

const openai = createOpenAI({
  baseURL: "https://promptthin.tech/v1",
  apiKey: "ts_your_key",
});
```

### LiteLLM

```python
import litellm

litellm.api_base = "https://promptthin.tech/v1"
litellm.api_key = "ts_your_key"
```

### Cursor / Continue.dev / Open WebUI

In settings, set:
- **OpenAI API Base URL**: `https://promptthin.tech/v1`
- **API Key**: `ts_your_key`

---

## Supported models

PromptThin infers the provider from the model name automatically:

| Model prefix | Routes to |
|---|---|
| `gpt-*`, `o1-*`, `o3-*` | OpenAI |
| `claude-*` | Anthropic |
| `gemini-*` | Google Gemini |
| `llama-*`, `mixtral-*`, `gemma-*` | Groq |

---

## Preview savings before committing

Use the `POST /predict-savings` endpoint to get a cost estimate before making a real LLM call — no tokens billed, no LLM call made:

```bash
curl -X POST https://promptthin.tech/predict-savings \
  -H "X-API-Key: ts_your_key" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-4o",
    "provider": "openai",
    "messages": [
      {"role": "user", "content": "your long prompt here..."}
    ]
  }'
```

Response:

```json
{
  "original_tokens": 4200,
  "estimated_tokens_after_savings": 2100,
  "estimated_cost_original": 0.0105,
  "estimated_cost_after_savings": 0.0013,
  "estimated_saving": 0.0092,
  "saving_percent": 87.5,
  "recommendation": "proceed"
}
```

---

## MCP server

PromptThin supports both Streamable HTTP (recommended) and SSE transports.

Add to `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "promptthin": {
      "command": "cmd",
      "args": [
        "/c", "npx", "mcp-remote@latest",
        "https://promptthin.tech/mcp",
        "--header", "X-API-Key: ts_your_key"
      ]
    }
  }
}
```

On Mac/Linux, replace `"command": "cmd"` and remove `"/c"` — use `"command": "npx"` directly.

Available tools:

| Tool | What it does |
|---|---|
| `billing_start_trial` | Start a 7-day free Pro trial — returns Stripe checkout URL |
| `proxy_chat` | Send a chat request through PromptThin — all five savings routes applied |
| `proxy_predict` | Estimate savings before the real call — free, no tokens billed |
| `usage_summary` | Total tokens saved, cache hit rate, cost saved |
| `billing_status` | Plan status and requests remaining |
| `cache_flush` | Clear the semantic cache |
| `usage_recent` | Recent proxied requests with details |

> **What `proxy_chat` is (and isn't):** `proxy_chat` routes a single outbound LLM call through PromptThin from within an AI assistant's response — for example, when you ask Claude to "use GPT-4 to summarise this file." It does **not** proxy the main conversation between you and a managed chat interface like claude.ai or ChatGPT — those platforms control their own API calls internally and cannot be intercepted. PromptThin saves tokens where **you** control the API call: your own code, agents, or self-hosted chat UIs.

**Recommended agent pattern:**
```python
# 1. Check savings estimate first (free)
estimate = call_tool("proxy_predict", model="gpt-4o", messages=messages)
# → "87% saving — compression + routing to gpt-4o-mini"

# 2. Send through PromptThin (savings applied automatically)
response = call_tool("proxy_chat", model="gpt-4o", messages=messages)
# → Returns answer + "[PromptThin] Tokens: 420 in / 85 out"
```

---

## Per-request controls

| Header | Value | Effect |
|---|---|---|
| `X-Cache-Control` | `no-cache` | Skip both cache lookup **and** cache storage for this request — the response also won't be written to the cache for future requests |
| `X-Cache-Control` | `force-image-cache` | Allow caching for this one request even though it contains image content (see [Vision and image requests](#vision-and-image-requests)) |
| `X-Prune-Control` | `no-prune` | Skip context pruning |
| `X-Compress-Control` | `no-compress` | Skip prompt compression |
| `X-Router-Control` | `no-route` | Skip model routing |
| `X-Thinking-Control` | `no-cap` | Skip thinking budget caps (use full reasoning) |

---

## Vision and image requests

The semantic cache fingerprints a request from the **text** portion of its messages only — image content blocks are never embedded. This means two requests with identical text but different images would otherwise hash to the same cache key and risk returning a cached answer about the wrong image.

To prevent this, PromptThin **skips the semantic cache by default for any request containing image content** — covering OpenAI/Anthropic-style image blocks (`image_url`, `image`, `input_image`) and Gemini-style inline/file image parts, in any message of the conversation, not just the latest one.

All other savings routes (compression, pruning, routing, thinking budget) are unaffected and still apply normally to vision requests.

If you have a workload where this is safe — for example, the image is decorative and the answer is fully determined by the text — you can opt back in for a single request:

```bash
curl -X POST https://promptthin.tech/v1/chat/completions \
  -H "X-API-Key: ts_your_key" \
  -H "X-Cache-Control: force-image-cache" \
  -H "Content-Type: application/json" \
  -d '{ ... }'
```

This is a per-request override, not a setting — each request containing images still needs `force-image-cache` explicitly to be cached.

---

## Protecting parts of a prompt from compression

Prompt compression (Route B) compresses the entire text of your last user message. If a part of that message must survive byte-for-byte — JSON you're going to parse, code, an exact template, anything format-sensitive — wrap it in markers instead of disabling compression for the whole message:

```
Please summarize this:
<<<no-compress>>>
{"id": 123, "exact": "json"}
<<<end-no-compress>>>
```

How it works under the hood:

1. Before compression runs, each `<<<no-compress>>>...<<<end-no-compress>>>` block is extracted and replaced with a unique placeholder token.
2. LLMLingua-2 compresses the remaining text, with the placeholder tokens hinted as force-preserved.
3. After compression, PromptThin verifies every placeholder token survived intact. If even one was split, stripped, or altered by the tokenizer, the **entire compression result for that message is discarded** and the original uncompressed message is sent instead — this guarantees the protected content is never silently corrupted, at the cost of losing compression savings on that one message.
4. If the verification passes, the placeholders are replaced back with the original protected text and the markers are removed from the final message.

Malformed markers (e.g. unmatched start/end tags) are treated as plain text — the message compresses normally without raising an error.

This is a finer-grained alternative to `X-Compress-Control: no-compress`, which disables compression for the whole request rather than just a portion of one message.

---

## Cache correctness

The semantic cache is only ever populated with responses that PromptThin can verify are well-formed. Before any response is written to the cache, it must pass all of the following checks:

- **No transport or provider error** — the upstream call must return HTTP 200. Timeouts, gateway errors, and provider-side error payloads are never cached.
- **Non-empty, substantive content** — responses with empty or near-empty text (e.g. a thinking model that returned nothing because its reasoning budget consumed the entire output) are rejected.
- **No bad finish reason** — provider-specific signals that the response was cut short or blocked are checked: OpenAI/Groq content-filter stops, Gemini `SAFETY` / `RECITATION` / `OTHER` / `BLOCKLIST` finish reasons, and Anthropic `stop_reason == "error"` all skip the cache.

These checks catch **errors and malformed responses**, not factual correctness — PromptThin has no way to verify whether a fluent, well-formed answer is actually *right*. If you're working with prompts where you don't want a possibly-imperfect answer cached for future similar requests by anyone, send `X-Cache-Control: no-cache` on that request. It skips both the cache read and the cache write, so that response is never reused.

For multimodal requests specifically, see [Vision and image requests](#vision-and-image-requests) above — those are skipped by default regardless of response quality, because the risk there is a cache-key collision, not a bad response.

---

## Pricing

| Plan | Price | Requests |
|---|---|---|
| **No card** | Free | 20 requests to explore |
| **Pro** | 7-day free trial · then $4.99 first month · then $11.99/mo | 10,000 req/month |
| **Enterprise** | Custom | Unlimited + SLA + dedicated support |

[Start free trial →](https://promptthin.tech)

---

## Security

- Provider keys encrypted with **AES-256** — never in logs or responses
- **Email verification** required before making API calls
- **Strong passwords** enforced (8+ chars, upper, lower, number, special character)
- All traffic **HTTPS only**
- Keys stored in **GCP Secret Manager**

---

## FAQ

**Do I need to change my code?**
No. Set two environment variables.

**Does PromptThin slow down my requests?**
Cache hits completely skip the LLM call — dramatically lower latency. Cache misses add <2ms overhead.

**Does PromptThin reduce my claude.ai / ChatGPT chat quota usage?**
No. Managed chat interfaces like claude.ai and ChatGPT control their own API calls internally — PromptThin cannot intercept those. PromptThin works when *you* control the API call: your own code, AI agents, or self-hosted chat UIs (e.g. Open WebUI, LibreChat). If you're hitting quota limits in a managed chat app, the fix is to use a self-hosted UI that calls the API directly through PromptThin instead.

**What if I want to pass my provider key directly?**
```python
client = OpenAI(
    base_url="https://promptthin.tech/v1",
    api_key="ts_your_key",
    default_headers={"Authorization": "Bearer sk-your-openai-key"},
)
```
PromptThin detects the key prefix and uses it directly.

**Can I use multiple providers?**
Yes. Register keys for each provider. PromptThin routes to the right one based on the model name.

**What happens after the 7-day trial?**
Your card is charged $4.99 for the first month, then $11.99/month. Cancel anytime from the dashboard — no charge if cancelled within 7 days.

---

## Contact

- **Website**: [promptthin.tech](https://promptthin.tech)
- **Enterprise**: [kaztesla2025@yahoo.com](mailto:kaztesla2025@yahoo.com)
- **Issues**: Open an issue on this repository
