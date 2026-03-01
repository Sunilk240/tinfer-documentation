# Benchmarking

Measure inference speed (tokens/second) for prompt processing and text generation using `tinfer-bench`.

---

## Quick Start

```bash
# Basic benchmark (uses GPU by default)
tinfer-bench -m model.gguf

# Benchmark with all layers on GPU
tinfer-bench -m model.gguf -ngl 99

# Custom prompt/generation lengths
tinfer-bench -m model.gguf -p 512 -n 128 -ngl 99
```

---

## What It Measures

| Metric | Description |
|--------|-------------|
| **pp** (prompt processing) | Speed of processing the input prompt (tokens/sec) |
| **tg** (text generation) | Speed of generating output tokens (tokens/sec) |

Higher values = faster. The output shows model name, size, backend, and throughput with standard deviation.

### Example Output

```
| model                 |   size |  params | backend | ngl |  test  |         t/s |
| llama 256M Q8_0       | 136 MiB| 134.52 M| CUDA    |  99 | tg128  | 72.41 ± 11.67 |
```

---

## All Flags

### Model & Input

| Flag | Description | Default |
|------|-------------|---------|
| `-m, --model <file>` | Model file path (.gguf) | `models/7B/ggml-model-q4_0.gguf` |
| `-p, --n-prompt <n>` | Number of prompt tokens to process | `512` |
| `-n, --n-gen <n>` | Number of tokens to generate | `128` |
| `-pg <pp,tg>` | Combined prompt + generation test (e.g. `-pg 512,128`) | — |
| `-d, --n-depth <n>` | Context depth for the test | `0` |

### Batch Processing

| Flag | Description | Default |
|------|-------------|---------|
| `-b, --batch-size <n>` | Batch size for prompt processing | `2048` |
| `-ub, --ubatch-size <n>` | Micro-batch size | `512` |

### GPU & Device

| Flag | Description | Default |
|------|-------------|---------|
| `-ngl, --n-gpu-layers <n>` | Number of layers to offload to GPU | `99` |
| `-mg, --main-gpu <i>` | Main GPU index | `0` |
| `-sm, --split-mode <mode>` | Multi-GPU split: `none`, `layer`, `row` | `layer` |
| `-dev, --device <dev0/dev1>` | Specific devices to use | `auto` |
| `-ts, --tensor-split <t0/t1>` | Tensor split ratios across GPUs | `0` |
| `--list-devices` | List available devices and exit | — |
| `-nkvo, --no-kv-offload <0\|1>` | Disable KV cache offloading to GPU | `0` |
| `-fa, --flash-attn <0\|1>` | Enable flash attention | `0` |

### CPU & Threading

| Flag | Description | Default |
|------|-------------|---------|
| `-t, --threads <n>` | Number of CPU threads | auto-detect |
| `-C, --cpu-mask <hex>` | CPU affinity mask | `0x0` |
| `--cpu-strict <0\|1>` | Strict CPU affinity | `0` |
| `--poll <0...100>` | Poll percentage | `50` |
| `-ncmoe, --n-cpu-moe <n>` | CPU MoE expert count | `0` |
| `--numa <mode>` | NUMA mode: `distribute`, `isolate`, `numactl` | disabled |

### KV Cache

| Flag | Description | Default |
|------|-------------|---------|
| `-ctk, --cache-type-k <type>` | KV cache key type: `f16`, `bf16`, `q8_0`, `q4_0`, etc. | `f16` |
| `-ctv, --cache-type-v <type>` | KV cache value type | `f16` |

### Output & Control

| Flag | Description | Default |
|------|-------------|---------|
| `-r, --repetitions <n>` | Number of times to repeat each test | `5` |
| `-o, --output <format>` | Output format: `csv`, `json`, `jsonl`, `md`, `sql` | `md` |
| `-oe, --output-err <format>` | Stderr output format | `none` |
| `-v, --verbose` | Verbose output | off |
| `--progress` | Show progress indicators | off |
| `--no-warmup` | Skip warmup runs | off |
| `--prio <-1\|0\|1\|2\|3>` | Process priority | `0` |
| `--delay <seconds>` | Delay between tests | `0` |

### Memory

| Flag | Description | Default |
|------|-------------|---------|
| `-mmp, --mmap <0\|1>` | Use memory-mapped I/O | `0` |
| `-dio, --direct-io <0\|1>` | Use direct I/O | `0` |
| `-embd, --embeddings <0\|1>` | Benchmark embedding mode | `0` |

---

## Multi-Value Testing

You can test multiple configurations in a single run by using comma-separated values:

```bash
# Compare GPU vs CPU
tinfer-bench -m model.gguf -ngl 0,99

# Test multiple prompt lengths
tinfer-bench -m model.gguf -p 128,256,512,1024 -ngl 99

# Compare multiple models
tinfer-bench -m model-q4.gguf,model-q8.gguf -ngl 99

# Range syntax: first-last+step
tinfer-bench -m model.gguf -p 128-1024+128 -ngl 99
```

---

## Export Results

```bash
# Export as CSV
tinfer-bench -m model.gguf -ngl 99 -o csv > results.csv

# Export as JSON
tinfer-bench -m model.gguf -ngl 99 -o json > results.json

# Export as SQL
tinfer-bench -m model.gguf -ngl 99 -o sql > results.sql
```
