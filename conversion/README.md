# Tinfer Model Conversion Tools

Convert HuggingFace models and LoRA adapters to GGUF format for use with Tinfer.

## Setup

```bash
cd conversion
pip install -r requirements.txt
```

## Convert HuggingFace Model → GGUF

```bash
# Basic conversion (auto-detect output type)
python convert_hf_to_gguf.py path/to/model-folder --outfile model.gguf --outtype auto

# Convert to specific quantization
python convert_hf_to_gguf.py path/to/model-folder --outfile model-f16.gguf --outtype f16
python convert_hf_to_gguf.py path/to/model-folder --outfile model-q8.gguf --outtype q8_0
python convert_hf_to_gguf.py path/to/model-folder --outfile model-bf16.gguf --outtype bf16

# Convert directly from HuggingFace (no download needed)
python convert_hf_to_gguf.py --remote meta-llama/Llama-3.2-3B-Instruct --outfile llama3.2-3b.gguf --outtype q8_0
```

### Output Types

| Type | Description | Size |
|------|-------------|------|
| `f32` | Float 32-bit (highest quality) | Largest |
| `f16` | Float 16-bit | Large |
| `bf16` | BFloat 16-bit | Large |
| `q8_0` | 8-bit quantized | Medium |
| `auto` | Auto-detect from model | Varies |

> **Note:** For smaller quantizations (Q4_K_M, Q5_K_M, etc.), first convert to f16/q8_0, then use `tinfer-quantize` to further quantize.

## Convert LoRA Adapter → GGUF

```bash
# Basic LoRA conversion (auto-loads base model config from HuggingFace)
python convert_lora_to_gguf.py path/to/lora-adapter-folder --outfile lora-adapter.gguf

# Specify base model explicitly
python convert_lora_to_gguf.py path/to/lora-adapter-folder --base path/to/base-model --outfile lora.gguf

# Specify base model by HuggingFace ID
python convert_lora_to_gguf.py path/to/lora-adapter-folder --base-model-id meta-llama/Llama-3.2-3B-Instruct --outfile lora.gguf
```

## After Conversion

```bash
# Run converted text model
tinfer -m model.gguf -p "Hello!" -n 100

# Run with LoRA adapter
tinfer-server -m model.gguf --lora lora-adapter.gguf --port 8080

# Further quantize (e.g., f16 → Q4_K_M)
tinfer-quantize model-f16.gguf model-q4km.gguf Q4_K_M
```
