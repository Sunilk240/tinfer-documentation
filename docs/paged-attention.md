# PagedAttention

PagedAttention is an experimental memory management feature for the KV cache that reduces memory fragmentation and enables efficient context shifting for multi-sequence workloads.

---

## The Problem

The default KV cache uses a **contiguous ring buffer**. When multiple sequences are active (e.g., parallel requests on a server), memory gets fragmented — gaps appear between sequences, wasting VRAM.

## The Solution

PagedAttention divides the KV cache into fixed-size **blocks** (default: 32 tokens). Sequences map logical token positions to physical blocks via a **block table** — similar to how an operating system manages virtual memory with paging.

---

## Key Benefits

| Benefit | Description |
|---------|-------------|
| **Zero fragmentation** | Blocks are allocated on demand — no wasted gaps between sequences |
| **O(1) context shift** | Remove or remap blocks in the table instead of moving actual data |
| **Copy-on-Write (CoW)** | Shared sequences (e.g., beam search) share blocks until one writes, then a copy is made |

---

## Quick Start

```bash
# Enable PagedAttention
tinfer -m model.gguf --kv-cache-paged -p "Hello" -n 100

# With the server (great for multi-user)
tinfer-server -m model.gguf --kv-cache-paged --port 8080

# Disable (default behavior)
tinfer -m model.gguf --no-kv-cache-paged -p "Hello" -n 100
```

---

## Flags

| Flag | Default | Description |
|------|---------|-------------|
| `--kv-cache-paged` | disabled | Enable paged KV cache |
| `--no-kv-cache-paged` | — | Explicitly disable paged KV cache |

---

## Block Size

The default block size is **32 tokens**, which aligns with:

- F16 KV cache
- Q8_0 quantized KV cache (quant block = 32)
- Q4_0 quantized KV cache (quant block = 32)

!!! note
    Q4_K quantization uses a quant block of 256, which may need a larger block size for optimal alignment. This is a future optimization.

---

## How It Works Internally

```
CLI (--kv-cache-paged)
  → llama_context_params.kv_cache_paged
  → llama_kv_cache constructor (paged=true)
      → BlockAllocator (manages free block pool)
      → BlockTable (maps seq_id → block list)
```

**BlockAllocator** — manages a pool of free blocks. When a sequence needs more room, it allocates a block. When a sequence is removed, its blocks return to the free pool.

**BlockTable** — maps each sequence ID to a list of blocks. Lookup is O(1). Context shifting just remaps the table entries instead of copying data.

---

## Performance

Tested with SmolLM2-135M-Instruct-Q8_0 on CPU:

| Mode | Prompt (tokens/sec) | Generation (tokens/sec) |
|------|:---:|:---:|
| Baseline (ring buffer) | 374.8 | 24.3 |
| **PagedAttention** | **512.5** | **27.6** |

**No throughput regression** — PagedAttention matches or exceeds baseline performance.

---

## When to Use PagedAttention

| Scenario | Recommendation |
|----------|----------------|
| Single-user CLI chat | Optional — minimal fragmentation anyway |
| **Multi-user server** | **Recommended** — significant memory savings |
| Beam search / parallel decoding | **Recommended** — CoW reduces memory by sharing |
| Long context generation | **Recommended** — O(1) context shifting |

## Limitations

- Block size is fixed at 32 (not yet configurable)
- Q4_K quantization may not align optimally with block_size=32
- No custom paged flash attention kernel yet (future optimization)
