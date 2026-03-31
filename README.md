# PromptThin

> **Reduce LLM API costs through caching, compression, and smart routing. Zero code changes.**

PromptThin is a transparent proxy that sits between your AI agents and LLM providers. Two environment variables and you're done — every API call gets four compounding savings routes applied automatically.

```
Your app ──→ PromptThin ──→ OpenAI / Anthropic / Gemini / Groq
```

[![Website](https://img.shields.io/badge/website-promptthin.tech-brightgreen)](https://promptthin.tech)
[![Free tier](https://img.shields.io/badge/free_tier-500_req%2Fmonth-blue)](https://promptthin.tech)
[![7-day trial](https://img.shields.io/badge/trial-7_days_unlimited-orange)](https://promptthin.tech)

---

## How much can I save?

Savings depend on your workload. The four routes compound — each one reduces what the next has to work with.

- **High cache hit rate** (repeated or similar queries): up to 90%+ reduction
- **Long context agents** (multi-turn, large prompts): 40–60% reduction from pruning + compression
- **Mixed workloads** (some unique, some repeated): typically 20–40% reduction

Check your actual savings anytime from the dashboard or API:

```bash
curl https://promptthin.tech/usage/summary -H "X-API-Key: ts_your_key"
```

---

## Four savings routes

| Route | What it does | Saving |
|---|---|---|
| **Semantic Cache** | Returns cached answers for similar questions — even if worded differently | Up to 100% on repeated queries |
| **Prompt Compression** | Compresses verbose prompts with LLMLingua 2 before sending | Up to 50% on input tokens |
| **Model Router** | Automatically routes simple tasks to cheaper models in <1ms | Up to 90% per request |
| **Context Pruning** | Summarises long conversation history when it exceeds 8K tokens | Up to 60% on long threads |

All four routes run on every request. You control which to enable or skip per-request.

---

## Get started in 2 minutes

### Step 1 — Create a free account

```bash
curl -X POST https://promptthin.tech/auth/register \
  -H "Content-Type: application/json" \
  -d '{"email": "you@example.com", "password": "yourpassword"}'
```

Returns your API key (`ts_xxx`). Save it.

Or sign up at **[promptthin.tech](https://promptthin.tech)** and get your key from the dashboard.

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

# Identical to regular OpenAI usage
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

const response = await client.chat.completions.create({
  model: "gpt-4o",
  messages: [{ role: "user", content: "Hello!" }],
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

### LangChain — Python

```python
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(
    base_url="https://promptthin.tech/v1",
    api_key="ts_your_key",
    model="gpt-4o",
)
```

### LangChain — JavaScript

```typescript
import { ChatOpenAI } from "@langchain/openai";

const llm = new ChatOpenAI({
  configuration: { baseURL: "https://promptthin.tech/v1" },
  apiKey: "ts_your_key",
});
```

### AutoGen — Python

```python
config_list = [{
    "model": "gpt-4o",
    "base_url": "https://promptthin.tech/v1",
    "api_key": "ts_your_key",
}]

assistant = AssistantAgent(
    name="assistant",
    llm_config={"config_list": config_list},
)
```

### CrewAI

```bash
# .env — CrewAI reads from environment automatically
OPENAI_BASE_URL=https://promptthin.tech/v1
OPENAI_API_KEY=ts_your_key
```

### Vercel AI SDK

```typescript
import { createOpenAI } from "@ai-sdk/openai";
import { generateText } from "ai";

const openai = createOpenAI({
  baseURL: "https://promptthin.tech/v1",
  apiKey: "ts_your_key",
});

const { text } = await generateText({
  model: openai("gpt-4o"),
  prompt: "Hello!",
});
```

### LiteLLM

```python
import litellm

litellm.api_base = "https://promptthin.tech/v1"
litellm.api_key = "ts_your_key"

response = litellm.completion(
    model="gpt-4o",
    messages=[{"role": "user", "content": "Hello!"}],
)
```

### Cursor / Continue.dev / Open WebUI

In settings, set:
- **OpenAI API Base URL**: `https://promptthin.tech/v1`
- **API Key**: `ts_your_key`

---

## Supported models

PromptThin infers the provider from the model name — no extra config needed:

| Model prefix | Routes to |
|---|---|
| `gpt-*`, `o1-*`, `o3-*` | OpenAI |
| `claude-*` | Anthropic |
| `gemini-*` | Google Gemini |
| `llama-*`, `mixtral-*`, `gemma-*` | Groq |

---

## MCP server

PromptThin exposes an MCP server for agents that support the Model Context Protocol (Claude Desktop, Claude Code, Cursor, Cline, Windsurf, Continue.dev).

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

Available tools your agent can call:

| Tool | What it does |
|---|---|
| `get_usage_summary` | Tokens saved, cache hit rate, cost saved |
| `get_billing_status` | Plan, requests remaining this month |
| `flush_cache` | Mark all cached responses as stale |
| `get_recent_requests` | Last N proxied requests with details |

---

## Per-request controls

Skip specific savings routes on individual requests by adding headers:

```python
# Skip cache for this request only
client.chat.completions.create(
    model="gpt-4o",
    messages=[{"role": "user", "content": "What's the latest news?"}],
    extra_headers={"X-Cache-Control": "no-cache"},
)
```

| Header | Value | Effect |
|---|---|---|
| `X-Cache-Control` | `no-cache` | Skip cache lookup and storage |
| `X-Prune-Control` | `no-prune` | Skip context pruning |
| `X-Compress-Control` | `no-compress` | Skip prompt compression |
| `X-Router-Control` | `no-route` | Skip model routing |

---

## Check your savings

```bash
curl https://promptthin.tech/usage/summary \
  -H "X-API-Key: ts_your_key"
```

Or open the **[live dashboard](https://promptthin.tech/dashboard)** — shows total tokens saved, tokens you would have used, tokens actually used, and cache hit rate.

---

## Pricing

| Plan | Price | Requests/month |
|---|---|---|
| **Free** | $0 | 500 req + 7-day unlimited trial |
| **Pro** | $4.99 first month, then $11.99/mo | 10,000 req |
| **Enterprise** | Custom | 100,000+ req, SLA, dedicated support |

[Sign up free →](https://promptthin.tech)

---

## Security

- Provider keys are encrypted with **AES-256** before storage — never in logs or responses
- All traffic is **HTTPS only**
- Keys are stored in **GCP Secret Manager**, not in the application database
- PromptThin never modifies response content — only compresses prompts and manages context

---

## FAQ

**Do I need to change my code?**
No. Set two environment variables and everything works automatically.

**Does PromptThin slow down my requests?**
The semantic cache adds <2ms. Prompt compression and model routing add <1ms. Cache hits completely skip the LLM call, dramatically reducing latency.

**What if I want to use my own provider key instead of registering it?**
Pass it directly in the Authorization header alongside your PromptThin key:
```python
client = OpenAI(
    base_url="https://promptthin.tech/v1",
    api_key="ts_your_key",
    default_headers={"Authorization": "Bearer sk-your-openai-key"},
)
```
PromptThin detects the provider key prefix and uses it directly.

**Can I use multiple providers?**
Yes. Register keys for each provider. PromptThin routes to the right one based on the model name.

**Is this open source?**
The integration documentation and examples are public. The proxy server is proprietary.

---

## Contact

- **Website**: [promptthin.tech](https://promptthin.tech)
- **Enterprise**: [kaztesla2025@yahoo.com](mailto:kaztesla2025@yahoo.com)
- **Issues**: Open an issue on this repository
