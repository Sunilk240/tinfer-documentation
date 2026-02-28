# API Reference

When `tinfer-server` is running, it exposes OpenAI-compatible HTTP endpoints. All requests go to `http://localhost:8080` by default.

---

## Health Check

### `GET /health`

Check if the server is ready.

```bash
curl http://localhost:8080/health
```

| Status | Response | Meaning |
|:---:|---|---|
| 200 | `{"status": "ok"}` | Server is ready |
| 503 | `{"error": {"code": 503, "message": "Loading model"}}` | Model still loading |

!!! note
    This endpoint is public — no API key required. Also available at `/v1/health`.

---

## Chat Completions (OpenAI-compatible)

### `POST /v1/chat/completions`

The primary endpoint for conversational AI. Fully compatible with OpenAI's API format.

```bash
curl http://localhost:8080/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "messages": [
      {"role": "system", "content": "You are a helpful assistant."},
      {"role": "user", "content": "What is artificial intelligence?"}
    ],
    "max_tokens": 200,
    "temperature": 0.7
  }'
```

**Request body:**

| Parameter | Type | Description | Default |
|---|---|---|---|
| `messages` | array | Array of `{role, content}` message objects | required |
| `model` | string | Model name (optional, uses loaded model) | — |
| `max_tokens` | int | Maximum tokens to generate | -1 (infinite) |
| `temperature` | float | Randomness (0.0 = deterministic) | 0.8 |
| `top_p` | float | Nucleus sampling | 0.95 |
| `top_k` | int | Top-K sampling | 40 |
| `min_p` | float | Min-P sampling | 0.05 |
| `stream` | bool | Stream tokens as they generate | false |
| `stop` | array | Stop strings | [] |
| `seed` | int | RNG seed (-1 = random) | -1 |
| `frequency_penalty` | float | Frequency penalty | 0.0 |
| `presence_penalty` | float | Presence penalty | 0.0 |
| `repeat_penalty` | float | Repeat penalty | 1.0 |
| `logit_bias` | object | Token probability adjustments | — |
| `n_probs` | int | Return top-N token probabilities | 0 |
| `grammar` | string | BNF grammar constraint | — |
| `json_schema` | object | JSON schema constraint | — |

**Response:**

```json
{
  "id": "chatcmpl-xxxx",
  "object": "chat.completion",
  "created": 1234567890,
  "model": "tinfer-server",
  "choices": [
    {
      "index": 0,
      "message": {
        "role": "assistant",
        "content": "Artificial intelligence is..."
      },
      "finish_reason": "stop"
    }
  ],
  "usage": {
    "prompt_tokens": 25,
    "completion_tokens": 100,
    "total_tokens": 125
  }
}
```

---

## Text Completions (OpenAI-compatible)

### `POST /v1/completions`

Raw text completion (non-chat format).

```bash
curl http://localhost:8080/v1/completions \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "The capital of France is",
    "max_tokens": 50,
    "temperature": 0.5
  }'
```

Same parameters as chat completions, but uses `prompt` instead of `messages`.

---

## List Models

### `GET /v1/models`

List all loaded models.

```bash
curl http://localhost:8080/v1/models
```

**Response:**

```json
{
  "object": "list",
  "data": [
    {
      "id": "model-name",
      "object": "model",
      "owned_by": "tinfer"
    }
  ]
}
```

---

## Completions (Non-OAI)

### `POST /completion`

Native completion endpoint with more options than the OAI-compatible version.

```bash
curl http://localhost:8080/completion \
  -H "Content-Type: application/json" \
  -d '{"prompt": "Building a website in 10 steps:", "n_predict": 128}'
```

**Additional parameters (beyond OAI-compatible ones):**

| Parameter | Type | Description | Default |
|---|---|---|---|
| `prompt` | string/array | Text prompt or token array | required |
| `n_predict` | int | Max tokens to predict | -1 |
| `cache_prompt` | bool | Reuse KV cache from previous request | true |
| `stream` | bool | Stream tokens in real-time | false |
| `id_slot` | int | Assign to specific slot (-1 = auto) | -1 |
| `return_tokens` | bool | Return raw token IDs | false |
| `samplers` | array | Order of samplers to apply | ["dry","top_k",...] |
| `t_max_predict_ms` | int | Time limit for generation (0 = disabled) | 0 |
| `timings_per_token` | bool | Include speed info per response | false |
| `n_probs` | int | Top-N probabilities per token | 0 |

**Response fields:**

| Field | Description |
|---|---|
| `content` | Generated text |
| `stop` | Whether generation stopped |
| `stop_type` | Why it stopped: `none`, `eos`, `limit`, or `word` |
| `stopping_word` | The stop word that triggered stop |
| `model` | Model alias |
| `tokens_evaluated` | Prompt tokens processed |
| `tokens_cached` | Tokens reused from cache |
| `truncated` | Whether context was exceeded |
| `timings` | Speed statistics |

---

## Tokenize

### `POST /tokenize`

Convert text to tokens.

```bash
curl http://localhost:8080/tokenize \
  -H "Content-Type: application/json" \
  -d '{"content": "Hello, world!"}'
```

| Parameter | Type | Description | Default |
|---|---|---|---|
| `content` | string | Text to tokenize | required |
| `add_special` | bool | Insert BOS token | false |
| `with_pieces` | bool | Return token text pieces | false |

---

## Detokenize

### `POST /detokenize`

Convert tokens back to text.

```bash
curl http://localhost:8080/detokenize \
  -H "Content-Type: application/json" \
  -d '{"tokens": [123, 456, 789]}'
```

---

## Embeddings

### `POST /v1/embeddings`

Generate text embeddings. Requires `--embedding` flag.

```bash
curl http://localhost:8080/v1/embeddings \
  -H "Content-Type: application/json" \
  -d '{"input": "Hello world", "model": "model-name"}'
```

Non-OAI endpoint `/embedding` is also available with `content` parameter and optional `embd_normalize`.

---

## Reranking

### `POST /v1/rerank`

Rerank documents by query relevance. Requires `--rerank` flag and a reranker model.

```bash
curl http://localhost:8080/v1/rerank \
  -H "Content-Type: application/json" \
  -d '{
    "query": "What is a panda?",
    "documents": ["hi", "it is a bear", "The giant panda is a bear species."],
    "top_n": 3
  }'
```

---

## Server Properties

### `GET /props`

Get server configuration and default generation settings.

```bash
curl http://localhost:8080/props
```

Returns model path, chat template, default sampling parameters, modalities, and slot count.

---

## Apply Template

### `POST /apply-template`

Convert chat messages to a prompt string using the model's template, without running inference.

```bash
curl http://localhost:8080/apply-template \
  -H "Content-Type: application/json" \
  -d '{"messages": [{"role": "user", "content": "Hello"}]}'
```

---

## Code Infill

### `POST /infill`

Fill-in-the-middle code completion (FIM).

```bash
curl http://localhost:8080/infill \
  -H "Content-Type: application/json" \
  -d '{
    "input_prefix": "def hello():\n    ",
    "input_suffix": "\n    return result"
  }'
```

| Parameter | Description |
|---|---|
| `input_prefix` | Code before the cursor |
| `input_suffix` | Code after the cursor |
| `input_extra` | Additional context: `[{"filename": "...", "text": "..."}]` |

---

## Authentication

If `--api-key` is set, include the key in your requests:

```bash
curl http://localhost:8080/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer your-api-key" \
  -d '{"messages": [{"role": "user", "content": "Hello"}]}'
```
