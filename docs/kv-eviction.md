# KV Cache Eviction

Smart KV eviction is an experimental feature that intelligently removes KV cache entries when the cache is full, enabling **infinite-length text generation** without the quality degradation of context shifting.

---

## The Problem

When the KV cache fills up during long generation, the default behavior is **context shifting** — discarding the oldest half of the cache and shifting positions. This causes quality degradation because the model loses important context from the middle of the conversation.

## The Solution

Smart KV eviction selectively removes **individual** cache entries based on their importance, preserving:

1. **Sink tokens** — first N positions (attention sinks essential for model stability)
2. **Protected tokens** — system prompt or other critical prefix tokens
3. **Recent window** — last 25% of each sequence (actively being generated)

Only the "middle" tokens — the least important for ongoing generation — are evicted.

---

## Two Eviction Modes

### Mode 1: Streaming (`--kv-eviction 1`)

Evicts the **oldest middle tokens** first (position-based). Simple, fast, and reliable.

Based on the [StreamingLLM paper](https://arxiv.org/abs/2309.17453) — which discovered that the first few tokens ("attention sinks") receive disproportionate attention from the model. Keeping sinks + a recent window preserves generation quality even over very long contexts.

```bash
tinfer -m model.gguf --kv-eviction 1 --ctx-size 2048 -p "Hello" -n 5000
```

### Mode 2: Scored (`--kv-eviction 2`)

Evicts the **least-recently-accessed** tokens first. Better quality for long conversations because it keeps tokens the model is actively referencing, regardless of their position.

Inspired by [H2O (Heavy-Hitter Oracle)](https://arxiv.org/abs/2306.14048) — which tracks cumulative attention scores to identify important tokens. This mode uses a lightweight access-recency approximation instead of full attention tracking.

```bash
tinfer -m model.gguf --kv-eviction 2 --ctx-size 2048 -p "Hello" -n 5000
```

### Comparison

| Aspect | Streaming (Mode 1) | Scored (Mode 2) |
|--------|:---:|:---:|
| **Strategy** | Evict oldest middle tokens | Evict least-recently-accessed |
| **Speed** | Faster (no scoring overhead) | Slightly slower |
| **Quality** | Good | Better for long conversations |
| **Best for** | Simple generation, speed-critical | Multi-turn chat, document processing |

---

## Quick Start

```bash
# Streaming mode
tinfer -m model.gguf --kv-eviction 1 --ctx-size 2048 -p "Hello" -n 5000

# Scored mode (better quality)
tinfer -m model.gguf --kv-eviction 2 --ctx-size 2048 -p "Hello" -n 5000

# Custom sink + protected tokens
tinfer -m model.gguf --kv-eviction 1 --kv-sink-tokens 8 --kv-protected-tokens 128

# Works with the server
tinfer-server -m model.gguf --kv-eviction 1 --port 8080

# Disabled (default, falls back to context shift)
tinfer -m model.gguf --kv-eviction 0 -p "Hello" -n 100
```

---

## Flags

| Flag | Default | Description |
|------|---------|-------------|
| `--kv-eviction MODE` | `0` (disabled) | `0` = none (context shift), `1` = streaming, `2` = scored |
| `--kv-sink-tokens N` | `4` | Number of initial positions to always keep (range: 0–256). These are "attention sinks" — tokens the model always references. |
| `--kv-protected-tokens N` | `0` | Number of positions to protect from eviction. Set this to your system prompt length to ensure it's never evicted. |

---

## How Eviction Triggers

Eviction happens automatically — no manual intervention needed:

```
prepare() fails (no free slots)
  → evict_cells(32+)    ← removes least important tokens
  → retry prepare()
  → success!             ← generation continues seamlessly
```

If eviction can't free enough cells (everything is protected), it falls back to the standard failure path.

---

## What Are Sink Tokens?

**Attention sinks** are the first few tokens in a sequence that consistently receive high attention scores from the model, regardless of their actual content. This was discovered by the StreamingLLM paper.

Why they matter:

- The model's attention mechanism relies on these positions as "anchors"
- Removing them destabilizes generation and causes quality collapse
- Even with a 2K context, keeping just 4 sink tokens prevents degradation

The default of `--kv-sink-tokens 4` works well for most models.

---

## Protecting System Prompts

If you use a system prompt (e.g., 128 tokens of instructions), protect it from eviction:

```bash
tinfer-server -m model.gguf --kv-eviction 2 --kv-protected-tokens 128 --port 8080
```

This ensures your system prompt stays in the cache even during very long conversations.

---

## Safety Features

- **Shared cells preserved** — cells used by multiple sequences are never evicted
- **Per-sequence recent window** — protects `max(32, seq_len/4)` most recent positions per sequence
- **Input validation** — range checks on all parameters
- **ISWA compatibility** — only the base cache is evicted; SWA cache uses native sliding window
- **Zero impact when disabled** — default `--kv-eviction 0` adds no runtime overhead

## Limitations

- No per-layer scoring (all layers share the same eviction decision)
- Scored mode uses access recency, not true attention weights (lightweight approximation)
- Not yet tested with speculative decoding
- Eviction granularity is per-cell (not per-block for PagedAttention mode)
