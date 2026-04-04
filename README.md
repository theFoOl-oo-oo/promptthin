# PromptThin

> **Reduce LLM API costs through caching, compression, and smart routing. Zero code changes.**

PromptThin is a transparent proxy that sits between your AI agents and LLM providers. Two environment variables and you're done — every API call gets four compounding savings routes applied automatically.

```
Your app ──→ PromptThin ──→ OpenAI / Anthropic / Gemini / Groq
```

[![Website](https://img.shields.io/badge/website-promptthin.tech-brightgreen)](https://promptthin.tech)
[![Free trial](https://img.shields.io/badge/trial-7_days_free-blue)](https://promptthin.tech)

---

## Four savings routes

| Route | What it does | Saving |
|---|---|---|
| **Semantic Cache** | Returns cached answers for similar questions — even if worded differently | Up to 100% on repeated queries |
| **Prompt Compression** | Compresses verbose prompts with LLMLingua 2 before sending | Up to 50% on input tokens |
| **Model Router** | Automatically routes simple tasks to cheaper models in <1ms | Up to 90% per request |
| **Context Pruning** | Summarises long conversation history when it exceeds 8K tokens | Up to 60% on long threads |

All four routes run on every request. You control which to skip per-request via headers.

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
  -d '{"key": "sk-your-openai-key"}'

# Anthropic
curl -X POST https://promptthin.tech/keys/anthropic \
  -H "X-API-Key: ts_your_key" \
  -H "Content-Type: application/json" \
  -d '{"key": "sk-ant-your-anthropic-key"}'

# Gemini
curl -X POST https://promptthin.tech/keys/gemini \
  -H "X-API-Key: ts_your_key" \
  -H "Content-Type: application/json" \
  -d '{"key": "AIza-your-gemini-key"}'

# Groq
curl -X POST https://promptthin.tech/keys/groq \
  -H "X-API-Key: ts_your_key" \
  -H "Content-Type: application/json" \
  -d '{"key": "gsk_your-groq-key"}'
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

Add to `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "promptthin": {
      "url": "https://promptthin.tech/mcp",
      "headers": {
        "X-API-Key": "ts_your_key"
      }
    }
  }
}
```

Available tools:

| Tool | What it does |
|---|---|
| `start_trial` | Start a 7-day free Pro trial — returns Stripe checkout URL |
| `chat_completion` | Send a chat request through PromptThin — all savings applied automatically |
| `predict_savings` | Estimate savings before the real call — free, no tokens billed |
| `get_usage_summary` | Total tokens saved, cache hit rate, cost saved |
| `get_billing_status` | Plan status and requests remaining |
| `flush_cache` | Clear the semantic cache |
| `get_recent_requests` | Recent proxied requests with details |

**Recommended agent pattern:**
```python
# 1. Check savings estimate first (free)
estimate = call_tool("predict_savings", model="gpt-4o", messages=messages)
# → "87% saving — compression + routing to gpt-4o-mini"

# 2. Send through PromptThin (savings applied automatically)
response = call_tool("chat_completion", model="gpt-4o", messages=messages)
# → Returns answer + "[PromptThin] Tokens: 420 in / 85 out"
```

---

## Per-request controls

| Header | Value | Effect |
|---|---|---|
| `X-Cache-Control` | `no-cache` | Skip cache lookup and storage |
| `X-Prune-Control` | `no-prune` | Skip context pruning |
| `X-Compress-Control` | `no-compress` | Skip prompt compression |
| `X-Router-Control` | `no-route` | Skip model routing |

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
