# Quantization

Reduce model size and speed up inference by quantizing GGUF models using `tinfer-quantize`.

---

## Quick Start

```bash
# Quantize a model (input → output → type)
tinfer-quantize model-f16.gguf model-Q4_K_M.gguf Q4_K_M
```

---

## Syntax

```
tinfer-quantize [options] input.gguf [output.gguf] type [nthreads]
```

If `output.gguf` is omitted, the output file is auto-named based on the quantization type.

---

## Quantization Types

### Recommended (Best Quality/Size Ratio)

| Type | Size (7B) | Quality Loss | Description |
|:---:|:---:|:---:|---|
| **Q4_K_M** | 4.58 GB | +0.1754 ppl | ⭐ **Best overall balance** — recommended default |
| **Q5_K_M** | 5.33 GB | +0.0569 ppl | Higher quality, slightly larger |
| **Q6_K** | 6.14 GB | +0.0217 ppl | Near-lossless quality |
| **Q8_0** | 7.96 GB | +0.0026 ppl | Virtually lossless |

### All Available Types

| Type | Size (7B) | Description |
|:---:|:---:|---|
| **F32** | 26.00 GB | Full 32-bit float — no quantization |
| **BF16** | 14.00 GB | BFloat16 — best for training checkpoints |
| **F16** | 14.00 GB | Float16 — standard unquantized |
| **Q8_0** | 7.96 GB | 8-bit — virtually no quality loss |
| **Q6_K** | 6.14 GB | 6-bit K-quant |
| **Q5_K_M** | 5.33 GB | 5-bit K-quant mixed |
| **Q5_K_S** | 5.21 GB | 5-bit K-quant small |
| **Q5_1** | 5.65 GB | 5-bit legacy |
| **Q5_0** | 5.21 GB | 5-bit legacy |
| **Q4_K_M** | 4.58 GB | 4-bit K-quant mixed |
| **Q4_K_S** | 4.37 GB | 4-bit K-quant small |
| **Q4_1** | 4.78 GB | 4-bit legacy |
| **Q4_0** | 4.34 GB | 4-bit legacy |
| **Q3_K_L** | 4.03 GB | 3-bit K-quant large |
| **Q3_K_M** | 3.74 GB | 3-bit K-quant medium |
| **Q3_K_S** | 3.41 GB | 3-bit K-quant small |
| **Q2_K** | 2.96 GB | 2-bit K-quant (significant quality loss) |
| **Q2_K_S** | 2.96 GB | 2-bit K-quant small |
| **IQ4_NL** | — | 4.50 bpw non-linear quantization |
| **IQ4_XS** | — | 4.25 bpw non-linear quantization |
| **IQ3_M** | — | 3.66 bpw quantization mix |
| **IQ3_S** | — | 3.44 bpw quantization |
| **IQ3_XS** | — | 3.3 bpw quantization |
| **IQ3_XXS** | — | 3.06 bpw quantization |
| **IQ2_M** | — | 2.7 bpw quantization |
| **IQ2_S** | — | 2.5 bpw quantization |
| **IQ2_XS** | — | 2.31 bpw quantization |
| **IQ2_XXS** | — | 2.06 bpw quantization |
| **IQ1_M** | — | 1.75 bpw quantization |
| **IQ1_S** | — | 1.56 bpw quantization |
| **TQ2_0** | — | 2.06 bpw ternarization |
| **TQ1_0** | — | 1.69 bpw ternarization |
| **COPY** | — | Copy tensors without quantizing |

!!! warning "IQ1/IQ2/Q2_K_S require importance matrix"
    The extreme quantizations (IQ1_S, IQ1_M, IQ2_S, IQ2_XXS, IQ2_XS, Q2_K_S) **require** an importance matrix (`--imatrix`) to maintain acceptable quality.

---

## Options

| Flag | Description |
|------|-------------|
| `--allow-requantize` | Allow re-quantizing already quantized tensors (⚠️ may reduce quality) |
| `--leave-output-tensor` | Leave `output.weight` un-quantized (increases size, may improve quality) |
| `--pure` | Disable K-quant mixtures — quantize all tensors to the same type |
| `--imatrix <file>` | Use importance matrix for optimized quantization |
| `--include-weights <name>` | Only apply importance matrix to matching tensors |
| `--exclude-weights <name>` | Exclude matching tensors from importance matrix |
| `--output-tensor-type <type>` | Set output tensor GGML type |
| `--token-embedding-type <type>` | Set token embedding GGML type |
| `--tensor-type <name=type>` | Quantize specific tensor(s) to a specific type |
| `--tensor-type-file <file>` | Load tensor type overrides from a file |
| `--prune-layers <L0,L1,...>` | Prune specific layers from the model (⚠️ advanced) |
| `--keep-split` | Keep the same shard structure as input |
| `--override-kv <KEY=TYPE:VALUE>` | Override model metadata (⚠️ advanced) |
| `--dry-run` | Calculate quantized size without performing quantization |

---

## Examples

### Basic Quantization

```bash
# F16 → Q4_K_M (recommended)
tinfer-quantize model-f16.gguf model-Q4_K_M.gguf Q4_K_M

# Q8_0 → Q4_K_M
tinfer-quantize model-q8.gguf model-Q4_K_M.gguf Q4_K_M --allow-requantize

# Use 8 threads for faster quantization
tinfer-quantize model-f16.gguf model-Q4_K_M.gguf Q4_K_M 8
```

### With Importance Matrix

```bash
# Better quality quantization with imatrix
tinfer-quantize model-f16.gguf model-IQ4_XS.gguf IQ4_XS --imatrix imatrix.dat
```

### Dry Run (Check Size Only)

```bash
# See final size without actually quantizing
tinfer-quantize --dry-run model-f16.gguf Q4_K_M
```

### Preserve Output Quality

```bash
# Keep output layer unquantized for better quality
tinfer-quantize model-f16.gguf model-Q4_K_M.gguf Q4_K_M --leave-output-tensor
```

---

## Quantization Pipeline

The typical workflow for converting and quantizing a model:

```bash
# Step 1: Convert HuggingFace model to GGUF (F16 or Q8_0)
python conversion/convert_hf_to_gguf.py model-folder --outfile model-f16.gguf --outtype f16

# Step 2: Quantize to desired size
tinfer-quantize model-f16.gguf model-Q4_K_M.gguf Q4_K_M

# Step 3: Benchmark to verify speed
tinfer-bench -m model-Q4_K_M.gguf -ngl 99

# Step 4: Run inference
tinfer -m model-Q4_K_M.gguf -p "Hello!" -n 100
```
