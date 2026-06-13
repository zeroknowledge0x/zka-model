# ZKA Model — AI Inference API

> **Documentation:** [zeroknowledge0x.github.io/zka-model/](https://zeroknowledge0x.github.io/zka-model/)

ZKA Model is the **Model Layer** of the [ZKA Labs](https://zeroknowledge0x.github.io/ZKA-Labs/) ecosystem — a high-performance AI inference platform that provides fast, reliable access to large language models through a RESTful API. It is designed for production workloads, offering low-latency completions, streaming responses, and flexible authentication.

---

## Available Models

| Model | ID | Description | Best For |
|---|---|---|---|
| **ZK-V3Z** | `zk-v3z` | Flagship reasoning model — highest accuracy and largest context window | Complex reasoning, code generation, research tasks |
| **ZK-V2B** | `zk-v2b` | Balanced model — strong quality with faster response times | General-purpose chat, content creation, summarization |
| **ZK-V1B** | `zk-v1b` | Lite model — ultra-fast inference with minimal latency | High-throughput pipelines, classification, simple Q&A |

> **Tip:** Use `zk-v1b` for latency-sensitive workloads and `zk-v3z` when output quality is the priority.

---

## API Base URL

```
https://api.zk0x.app/v1
```

All endpoints are versioned under `/v1`. Responses are JSON-encoded.

---

## Authentication

All requests require an API key passed via the `Authorization` header:

```
Authorization: Bearer ***
```

API keys are provisioned through the ZKA Model dashboard. Each key is scoped to a project and can be rotated or revoked independently.

| Key Type | Description |
|---|---|
| **Production** | Full model access, production rate limits |
| **Development** | Full model access, reduced rate limits |

> **Security:** Never expose API keys in client-side code, public repositories, or logs.

---

## Endpoints

### Chat Completions

```
POST /v1/chat/completions
```

Generate a model response from a conversation.

**Request Body:**

```json
{
  "model": "zk-v3z",
  "messages": [
    { "role": "system", "content": "You are a helpful assistant." },
    { "role": "user", "content": "Explain zero-knowledge proofs in one paragraph." }
  ],
  "temperature": 0.7,
  "max_tokens": 1024,
  "stream": false
}
```

| Parameter | Type | Required | Default | Description |
|---|---|---|---|---|
| `model` | `string` | Yes | — | Model ID (`zk-v3z`, `zk-v2b`, or `zk-v1b`) |
| `messages` | `array` | Yes | — | List of message objects (`role` + `content`) |
| `temperature` | `float` | No | `1.0` | Sampling temperature (0.0–2.0) |
| `max_tokens` | `integer` | No | Model default | Maximum tokens in the response |
| `stream` | `boolean` | No | `false` | Enable streaming (see below) |
| `top_p` | `float` | No | `1.0` | Nucleus sampling threshold |
| `stop` | `string \| array` | No | `null` | Stop sequences |

**Response (non-streaming):**

```json
{
  "id": "chatcmpl-abc123",
  "object": "chat.completion",
  "model": "zk-v3z",
  "choices": [
    {
      "index": 0,
      "message": { "role": "assistant", "content": "..." },
      "finish_reason": "stop"
    }
  ],
  "usage": {
    "prompt_tokens": 42,
    "completion_tokens": 128,
    "total_tokens": 170
  }
}
```

---

### List Models

```
GET /v1/models
```

Returns the list of models available to the authenticated account.

**Response:**

```json
{
  "data": [
    { "id": "zk-v3z", "object": "model", "created": 1700000000 },
    { "id": "zk-v2b", "object": "model", "created": 1700000000 },
    { "id": "zk-v1b", "object": "model", "created": 1700000000 }
  ]
}
```

---

### Health Check

```
GET /v1/health
```

Returns service status. No authentication required.

---

## Streaming

Set `"stream": true` in your request body to receive tokens as they are generated via **Server-Sent Events (SSE)**.

**Response format (streaming):**

```
data: {"id":"chatcmpl-abc123","object":"chat.completion.chunk","choices":[{"delta":{"content":"Hello"},"index":0}]}

data: {"id":"chatcmpl-abc123","object":"chat.completion.chunk","choices":[{"delta":{"content":" world"},"index":0}]}

data: {"id":"chatcmpl-abc123","object":"chat.completion.chunk","choices":[{"delta":{},"finish_reason":"stop","index":0}]}

data: [DONE]
```

Each `data:` line contains a JSON object with a `delta` field containing incremental content. The stream terminates with `data: [DONE]`.

---

## Rate Limits

Rate limits are enforced per API key and vary by key type and model.

| Key Type | Requests / min | Tokens / min |
|---|---|---|
| **Production** | 60 | 150,000 |
| **Development** | 10 | 10,000 |

When a rate limit is exceeded, the API returns `429 Too Many Requests` with a `Retry-After` header indicating when to retry.

**Rate limit headers included in every response:**

```
X-RateLimit-Remaining: 42
X-RateLimit-Reset: 1700000060
```

---

## Error Codes

| Code | Status | Description |
|---|---|---|
| `invalid_api_key` | `401` | Missing or invalid API key |
| `model_not_found` | `404` | Requested model does not exist or is not available |
| `invalid_request` | `400` | Malformed request body or missing required parameters |
| `rate_limit_exceeded` | `429` | Rate limit hit — check `Retry-After` header |
| `internal_error` | `500` | Server-side error — retry with exponential backoff |

---

## Quickstart

```bash
curl https://api.zk0x.app/v1/chat/completions \
  -H "Authorization: Bearer *** \
  -H "Content-Type: application/json" \
  -d '{
    "model": "zk-v2b",
    "messages": [
      { "role": "user", "content": "What is ZK-SNARK?" }
    ]
  }'
```

---

## SDKs & Integration

ZKA Model uses an OpenAI-compatible API format. You can integrate with existing OpenAI SDK clients by changing the base URL:

```python
from openai import OpenAI

client = OpenAI(
    base_url="https://api.zk0x.app/v1",
    api_key="YOUR_API_KEY"
)

response = client.chat.completions.create(
    model="zk-v3z",
    messages=[{"role": "user", "content": "Hello!"}]
)
print(response.choices[0].message.content)
```

---

## Support

- **Dashboard:** [app.zka-model.dev](https://app.zka-model.dev)
- **Email:** support@zka-model.dev

---

*ZKA Model — Fast, reliable AI inference.*
