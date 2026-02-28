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

## RoPE / Context Extension

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
