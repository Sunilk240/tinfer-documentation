# Layer Offloading

Layer offloading is an experimental feature that enables running models **larger than your GPU VRAM** by dynamically swapping layer weights between Disk, CPU, and GPU using a sliding window with async prefetching.

---

## The Problem

When `-ngl` is less than the total layer count, the remaining layers compute on CPU, which is significantly slower. Users with limited VRAM are stuck with slow CPU inference for those layers.

## The Solution

Layer offloading keeps only a **window** of N layers in GPU staging buffers at a time. Before each layer computation:

1. **Swap in** — CPU layer weights are temporarily redirected to GPU staging via pointer swaps
2. **Transfer** — Weight data is copied from CPU → GPU staging buffer
3. **Compute** — GPU computes using the staged weights (fast!)
4. **Swap back** — Pointers are restored to original CPU locations

This means even if you can only fit 5 layers permanently in VRAM, all layers still get **GPU-speed compute** through the sliding window.

---

## Three Tiers

| Tier | Location | Behavior |
|------|----------|----------|
| **GPU** | VRAM (permanent) | Controlled by `-ngl`, always on GPU |
| **CPU** | System RAM | Windowed into GPU staging as needed |
| **Disk** | GGUF file | Loaded into CPU cache on demand (LRU eviction) |

```
Model: 48 layers total, -ngl 5

Layer 0-42:  CPU tier  ─── windowed through GPU staging ───→ GPU compute
Layer 43-47: GPU tier  ─── permanently on GPU ──────────────→ GPU compute
```

- `-ngl` controls how many layers are **permanently** on GPU
- `--layer-window` controls how CPU-tier layers are **temporarily** windowed into GPU
- If `-ngl` covers all layers, `--layer-window` has no effect

---

## Quick Start

```bash
# Auto-detect window size based on free VRAM
tinfer -m model.gguf -ngl 5 --layer-window auto -p "Hello" -n 100

# Manual window size (4 CPU layers windowed at a time)
tinfer -m model.gguf -ngl 5 --layer-window 4 -p "Hello" -n 100

# Disable async prefetching (sync transfer only)
tinfer -m model.gguf -ngl 5 --layer-window auto --no-layer-prefetch -p "Hello" -n 100

# Works with the server too
tinfer-server -m model.gguf -ngl 5 --layer-window auto --port 8080
```

---

## Flags

| Flag | Default | Description |
|------|---------|-------------|
| `--layer-window N` | `0` (disabled) | `auto` = detect from free VRAM, or exact number of layers to window. (env: `LLAMA_ARG_LAYER_WINDOW`) |
| `--no-layer-prefetch` | prefetch enabled | Disable async prefetching of the next window |

---

## How Auto-Detection Works

When you use `--layer-window auto`, the system:

1. Measures free VRAM after loading the model
2. Calculates the largest layer size
3. Determines how many layers can fit in a double-buffered staging area
4. Sets the window size accordingly

---

## Architecture

```
CLI (--layer-window N)
  → llama_model_params.layer_window / .layer_prefetch
  → load_tensors(): init window, assign tiers, allocate staging
  → process_ubatch(): swap_layer_to_gpu → graph_compute → swap_layer_to_cpu

Phase A (scheduler-level):
  Prefetch next split's inputs during current split compute

Phase B (layer-level):
  Sliding window of N layers on double-buffered GPU staging
  Pointer swap preserves graph topology

Phase C (disk tier):
  Read layers from GGUF file via direct I/O
  LRU CPU cache with eviction
```

### Double-Buffered Staging

Two staging buffer slots allow computing from one while loading data into the other, enabling overlap of data transfer and computation.

---

## Key Design Decisions

- **Graph reuse preserved** — only `tensor->data` and `tensor->buffer` pointers are swapped, graph nodes never change
- **Double-buffered staging** — overlap data transfer and computation
- **Automatic tensor handling** — adapts to upstream model struct changes automatically

## Limitations

- Pinned memory (`cudaMallocHost`) not yet used (fallback to `malloc`)
- Disk tier I/O is synchronous reads (not yet producer-consumer queued)
- Not yet tested with speculative decoding or multi-GPU split modes
