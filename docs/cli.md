# CLI Reference

The `tinfer` command provides direct text generation and interactive chat from your terminal.

## Basic Usage

```bash
# Simple prompt
tinfer -m model.gguf -p "What is AI?" -n 100

# Interactive conversation mode
tinfer -m model.gguf -cnv

# With GPU acceleration (offload all layers)
tinfer -m model.gguf -p "Hello" -ngl 99
```

---

## Model Options

| Flag | Description | Default |
|------|-------------|---------|
| `-m, --model FNAME` | Path to the GGUF model file | — |
| `-mu, --model-url URL` | Download model from URL | — |
| `-hf, --hf-repo <user>/<model>[:quant]` | HuggingFace model repository. Quant is optional, defaults to Q4_K_M | — |
| `-hff, --hf-file FILE` | Specific file from HuggingFace repo | — |
| `-hft, --hf-token TOKEN` | HuggingFace access token (env: `HF_TOKEN`) | — |

## Generation Options

| Flag | Description | Default |
|------|-------------|---------|
| `-p, --prompt TEXT` | Text prompt for generation | — |
| `-f, --file FNAME` | Read prompt from a file | — |
| `-n, --predict N` | Number of tokens to predict (-1 = infinity) | -1 |
| `-c, --ctx-size N` | Context window size (0 = loaded from model) | 0 |
| `-b, --batch-size N` | Logical maximum batch size | 2048 |
| `-ub, --ubatch-size N` | Physical maximum batch size | 512 |
| `-cnv, --conversation` | Enable conversation mode | off |
| `-e, --escape` | Process escape sequences (\\n, \\t, etc.) | on |
| `--keep N` | Tokens to keep from initial prompt (0 = none, -1 = all) | 0 |

## CPU / Thread Options

| Flag | Description | Default |
|------|-------------|---------|
| `-t, --threads N` | CPU threads for generation (env: `LLAMA_ARG_THREADS`) | auto |
| `-tb, --threads-batch N` | Threads for batch/prompt processing | same as `-t` |
| `-C, --cpu-mask M` | CPU affinity mask (hex) | — |
| `-Cr, --cpu-range lo-hi` | CPU range for affinity | — |
| `--cpu-strict <0\|1>` | Strict CPU placement | 0 |
| `--prio N` | Process priority: -1=low, 0=normal, 1=medium, 2=high, 3=realtime | 0 |

## GPU Options

| Flag | Description | Default |
|------|-------------|---------|
| `-ngl, --n-gpu-layers N` | Layers to offload to GPU (`auto`, number, or `all`) | auto |
| `-sm, --split-mode {none,layer,row}` | How to split model across GPUs | layer |
| `-ts, --tensor-split N0,N1,...` | Fraction of model per GPU | — |
| `-mg, --main-gpu INDEX` | Main GPU index | 0 |
| `-dev, --device <dev1,dev2,...>` | Devices for offloading | — |
| `--list-devices` | Print available devices and exit | — |
| `-fit, --fit [on\|off]` | Auto-adjust to fit in VRAM | on |
| `-fitt, --fit-target MiB` | Target margin per device for --fit | 1024 |

## Memory Options

| Flag | Description | Default |
|------|-------------|---------|
| `-ctk, --cache-type-k TYPE` | KV cache type for K (f32, f16, bf16, q8_0, q4_0, etc.) | f16 |
| `-ctv, --cache-type-v TYPE` | KV cache type for V | f16 |
| `--mlock` | Force model to stay in RAM | off |
| `--mmap, --no-mmap` | Memory-map the model file | on |
| `-kvo, --kv-offload` | Enable KV cache offloading | on |
| `--no-host` | Bypass host buffer | off |
| `-cmoe, --cpu-moe` | Keep all MoE weights in CPU | off |

## Layer Offloading

Run models **larger than your GPU VRAM** by dynamically swapping layers between Disk, CPU, and GPU using a sliding window with async prefetching.

```bash
# Auto-detect window size based on free VRAM
tinfer -m model.gguf -ngl 5 --layer-window auto -p "Hello" -n 100

# Manual window size (4 CPU layers windowed at a time)
tinfer -m model.gguf -ngl 5 --layer-window 4 -p "Hello" -n 100
```

**How it works:** When `-ngl` is less than total layers, remaining layers normally compute on slow CPU. Layer offloading instead keeps a sliding **window** of N layers in GPU staging buffers, temporarily swapping them in for fast GPU compute, then swapping back.

| Tier | Location | Behavior |
|------|----------|----------|
| **GPU** | VRAM (permanent) | Controlled by `-ngl`, always on GPU |
| **CPU** | System RAM | Windowed into GPU staging as needed |
| **Disk** | GGUF file | Loaded into CPU cache on demand (LRU eviction) |

| Flag | Description | Default |
|------|-------------|---------|
| `--layer-window N` | `auto` = detect from free VRAM, or exact number of layers to window (env: `LLAMA_ARG_LAYER_WINDOW`) | 0 (disabled) |
| `--no-layer-prefetch` | Disable async prefetching of next window | enabled |

!!! tip
    Use `--layer-window auto` with a small `-ngl` to run models that don't fit in VRAM. The system will auto-detect how many layers it can window through GPU staging.

## PagedAttention

Reduces KV cache memory fragmentation and enables efficient context shifting for multi-sequence workloads.

```bash
# Enable PagedAttention
tinfer -m model.gguf --kv-cache-paged -p "Hello" -n 100
```

**How it works:** Instead of a contiguous ring buffer, the KV cache is divided into fixed-size **blocks** (32 tokens). Sequences map positions to physical blocks via a block table — like OS virtual memory paging.

**Benefits:**

- **Zero fragmentation** — blocks allocated on demand, no wasted gaps
- **O(1) context shift** — remap blocks instead of moving data
- **Copy-on-Write** — shared sequences (beam search) share blocks until written

| Flag | Description | Default |
|------|-------------|---------|
| `--kv-cache-paged` | Enable paged KV cache | disabled |
| `--no-kv-cache-paged` | Disable paged KV cache | — |

## KV Cache Eviction

Intelligently removes KV cache entries when full, enabling **infinite-length text generation** without quality loss from context shifting.

```bash
# StreamingLLM mode: keep sinks + recent, evict oldest middle
tinfer -m model.gguf --kv-eviction 1 --ctx-size 2048 -p "Hello" -n 5000

# Scored mode: evict least-recently-accessed (better quality)
tinfer -m model.gguf --kv-eviction 2 --ctx-size 2048 -p "Hello" -n 5000

# Protect a 128-token system prompt from eviction
tinfer -m model.gguf --kv-eviction 1 --kv-sink-tokens 8 --kv-protected-tokens 128
```

**How it works:** When the cache fills, instead of discarding the oldest half (context shift), smart eviction selectively removes individual entries:

1. **Sink tokens** — first N positions always kept (attention sinks)
2. **Protected tokens** — system prompt or critical prefix preserved
3. **Recent window** — last 25% of each sequence preserved

| Mode | Flag | Strategy | Best For |
|------|------|----------|----------|
| **Disabled** | `--kv-eviction 0` | Falls back to context shift | Default |
| **Streaming** | `--kv-eviction 1` | Evict oldest middle tokens | Simple, fast |
| **Scored** | `--kv-eviction 2` | Evict least-recently-accessed | Better quality |

| Flag | Description | Default |
|------|-------------|---------|
| `--kv-eviction MODE` | Eviction mode: 0=none, 1=streaming, 2=scored | 0 |
| `--kv-sink-tokens N` | Initial positions to always keep (0-256) | 4 |
| `--kv-protected-tokens N` | Positions to protect (e.g. system prompt length) | 0 |

| Flag | Description | Default |
|------|-------------|---------|
| `--rope-scaling {none,linear,yarn}` | RoPE frequency scaling method | model default |
| `--rope-scale N` | RoPE context scaling factor | — |
| `--rope-freq-base N` | RoPE base frequency (NTK-aware) | model default |
| `--rope-freq-scale N` | RoPE frequency scaling factor | — |
| `--yarn-orig-ctx N` | YaRN original context size | 0 |
| `--yarn-ext-factor N` | YaRN extrapolation mix factor | -1.0 |

## Sampling Options

| Flag | Description | Default |
|------|-------------|---------|
| `--temp N` | Temperature | 0.8 |
| `--top-k N` | Top-K sampling (0 = disabled) | 40 |
| `--top-p N` | Top-P / nucleus sampling (1.0 = disabled) | 0.95 |
| `--min-p N` | Min-P sampling (0.0 = disabled) | 0.05 |
| `-s, --seed N` | RNG seed (-1 = random) | -1 |
| `--repeat-penalty N` | Repetition penalty (1.0 = disabled) | 1.0 |
| `--repeat-last-n N` | Tokens to consider for penalty (0 = disabled) | 64 |
| `--presence-penalty N` | Presence penalty (0.0 = disabled) | 0.0 |
| `--frequency-penalty N` | Frequency penalty (0.0 = disabled) | 0.0 |
| `--mirostat N` | Mirostat sampling (0=off, 1=v1, 2=v2) | 0 |
| `--mirostat-lr N` | Mirostat learning rate (eta) | 0.1 |
| `--mirostat-ent N` | Mirostat target entropy (tau) | 5.0 |
| `--typical N` | Locally typical sampling (1.0 = disabled) | 1.0 |
| `--dynatemp-range N` | Dynamic temperature range (0.0 = disabled) | 0.0 |
| `--samplers SAMPLERS` | Sampler order, separated by `;` | penalties;dry;top_k;... |

## DRY Sampling

| Flag | Description | Default |
|------|-------------|---------|
| `--dry-multiplier N` | DRY penalty multiplier (0.0 = disabled) | 0.0 |
| `--dry-base N` | DRY base value | 1.75 |
| `--dry-allowed-length N` | Allowed repeat length before penalty | 2 |
| `--dry-penalty-last-n N` | Tokens to scan for repeats (-1 = ctx size) | -1 |
| `--dry-sequence-breaker STR` | Sequence breaker strings | `\n`, `:`, `"`, `*` |

## Grammar / Structured Output

| Flag | Description | Default |
|------|-------------|---------|
| `--grammar GRAMMAR` | BNF-like grammar to constrain output | — |
| `--grammar-file FNAME` | Read grammar from file | — |
| `-j, --json-schema SCHEMA` | JSON schema constraint | — |
| `-jf, --json-schema-file FILE` | Read JSON schema from file | — |

## Advanced Options

| Flag | Description | Default |
|------|-------------|---------|
| `--lora FNAME` | Path to LoRA adapter (comma-separated for multiple) | — |
| `--lora-scaled FNAME:SCALE,...` | LoRA with custom scaling | — |
| `--control-vector FNAME` | Control vector file | — |
| `--override-kv KEY=TYPE:VALUE` | Override model metadata | — |
| `--check-tensors` | Validate model tensor data | off |
| `--numa {distribute,isolate,numactl}` | NUMA optimizations | — |
| `-fa, --flash-attn [on\|off\|auto]` | Flash Attention | auto |
| `--verbose-prompt` | Print prompt before generation | off |

## Logging Options

| Flag | Description | Default |
|------|-------------|---------|
| `-v, --verbose` | Maximum verbosity (debug all) | off |
| `-lv, --verbosity N` | Verbosity level: 0=output, 1=error, 2=warn, 3=info, 4=debug | 3 |
| `--log-file FNAME` | Log to file (env: `LLAMA_LOG_FILE`) | — |
| `--log-disable` | Disable logging | off |
| `--log-colors [on\|off\|auto]` | Colored log output | auto |

## Misc

| Flag | Description |
|------|-------------|
| `-h, --help` | Print usage and exit |
| `--version` | Show version and build info |
| `--license` | Show license info |
| `--completion-bash` | Print bash completion script |
| `--offline` | Offline mode (no network access) |
