# Model Conversion

Convert HuggingFace models and LoRA adapters to GGUF format for use with Tinfer.

> **Source code:** The conversion scripts are available in the [`conversion/`](https://github.com/Sunilk240/tinfer-documentation/tree/main/conversion) folder of this repository.

---

## Setup

```bash
cd conversion
pip install -r requirements.txt
```

**Dependencies:** PyTorch (CPU-only), transformers, numpy, gguf, sentencepiece, safetensors, protobuf.

---

## 1. Convert HuggingFace Model → GGUF

Use `convert_hf_to_gguf.py` to convert any HuggingFace model (safetensors/PyTorch) to GGUF format.

### Download a Model First

```bash
pip install huggingface-hub
python -c "
from huggingface_hub import snapshot_download
snapshot_download(repo_id='HuggingFaceTB/SmolLM-135M-Instruct', local_dir='./models/SmolLM-135M-Instruct')
"
```

### Convert

```bash
# Basic conversion (auto-detect output type)
python convert_hf_to_gguf.py ./models/SmolLM-135M-Instruct --outfile ./models/converted/SmolLM-135M-q8.gguf --outtype q8_0

# Convert to F16 (for further quantization)
python convert_hf_to_gguf.py ./models/SmolLM-135M-Instruct --outfile ./models/converted/SmolLM-135M-f16.gguf --outtype f16

# Convert to BFloat16
python convert_hf_to_gguf.py ./models/SmolLM-135M-Instruct --outfile ./models/converted/SmolLM-135M-bf16.gguf --outtype bf16
```

### Convert Directly from HuggingFace (No Download)

```bash
python convert_hf_to_gguf.py --remote HuggingFaceTB/SmolLM-135M-Instruct --outfile SmolLM-135M-q8.gguf --outtype q8_0
```

### Output Types

| Type | Description | When to Use |
|:---:|---|---|
| `f32` | Float 32-bit | Maximum quality, largest file |
| `f16` | Float 16-bit | Good for further quantization with `tinfer-quantize` |
| `bf16` | BFloat 16-bit | Training checkpoints |
| `q8_0` | 8-bit quantized | Ready to use, good quality |
| `auto` | Auto-detect | Uses model's native precision |

### Key Flags

| Flag | Description |
|------|-------------|
| `--outfile <path>` | Output GGUF file path |
| `--outtype <type>` | Output type: `f32`, `f16`, `bf16`, `q8_0`, `auto` |
| `--remote <model-id>` | Convert directly from HuggingFace without downloading |
| `--bigendian` | Use big-endian format |
| `--vocab-only` | Export only the vocabulary |
| `--model-name <name>` | Override model name in metadata |
| `--metadata <file>` | Override metadata from JSON file |
| `--split-max-tensors <n>` | Split output into shards by tensor count |
| `--split-max-size <n>` | Split output into shards by size |
| `--dry-run` | Show what would be done without writing |
| `--no-lazy` | Load all tensors into RAM (uses more memory) |

---

## 2. Convert LoRA Adapter → GGUF

Use `convert_lora_to_gguf.py` to convert HuggingFace PEFT LoRA adapters.

### Requirements

The LoRA adapter folder must contain:

- `adapter_config.json` — adapter configuration
- `adapter_model.safetensors` (or `adapter_model.bin`) — adapter weights

### Convert

```bash
# Auto-detect base model (reads from adapter_config.json)
python convert_lora_to_gguf.py ./my-lora-adapter --outfile lora-adapter.gguf

# Specify base model directory
python convert_lora_to_gguf.py ./my-lora-adapter --base ./base-model-folder --outfile lora.gguf

# Specify base model by HuggingFace ID
python convert_lora_to_gguf.py ./my-lora-adapter --base-model-id meta-llama/Llama-3.2-3B-Instruct --outfile lora.gguf
```

### Key Flags

| Flag | Description |
|------|-------------|
| `--outfile <path>` | Output GGUF file path |
| `--outtype <type>` | Output type: `f32`, `f16`, `bf16`, `q8_0`, `auto` (default: `f32`) |
| `--base <dir>` | Directory with base model config files |
| `--base-model-id <id>` | HuggingFace model ID for base model config |
| `--bigendian` | Use big-endian format |
| `--dry-run` | Show what would be done without writing |

### Use the Converted LoRA

```bash
# CLI with LoRA
tinfer -m base-model.gguf --lora lora-adapter.gguf -p "Hello!"

# Server with LoRA
tinfer-server -m base-model.gguf --lora lora-adapter.gguf --port 8080
```

---

## Full Pipeline Example

```bash
# 1. Download model from HuggingFace
python -c "
from huggingface_hub import snapshot_download
snapshot_download(repo_id='HuggingFaceTB/SmolLM-135M-Instruct', local_dir='./models/SmolLM-135M')
"

# 2. Convert to GGUF (F16 for max quality conversion)
python convert_hf_to_gguf.py ./models/SmolLM-135M --outfile ./models/SmolLM-135M-f16.gguf --outtype f16

# 3. Quantize to Q4_K_M (best size/quality ratio)
tinfer-quantize ./models/SmolLM-135M-f16.gguf ./models/SmolLM-135M-Q4_K_M.gguf Q4_K_M

# 4. Benchmark
tinfer-bench -m ./models/SmolLM-135M-Q4_K_M.gguf -ngl 99

# 5. Run inference
tinfer -m ./models/SmolLM-135M-Q4_K_M.gguf -p "Hello, how are you?" -n 100
```
