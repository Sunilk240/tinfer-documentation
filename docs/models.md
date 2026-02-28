# Model Download

Tinfer runs **GGUF** format models. You can download them from [HuggingFace](https://huggingface.co) using any of the methods below.

---

## Method 1: Terminal One-Liner

The quickest way to download a model directly from your terminal:

```bash
pip install huggingface-hub
python -c "from huggingface_hub import hf_hub_download; import os; os.makedirs('models', exist_ok=True); hf_hub_download(repo_id='bartowski/Llama-3.2-3B-Instruct-GGUF', filename='Llama-3.2-3B-Instruct-Q4_K_M.gguf', local_dir='./models')"
```

Replace the `repo_id` and `filename` with any model from the tables below.

## Method 2: Python Script

For more control, use a Python script:

```python
import os
from huggingface_hub import hf_hub_download

# Create models directory
os.makedirs('models', exist_ok=True)

# Download model
hf_hub_download(
    repo_id='bartowski/Llama-3.2-3B-Instruct-GGUF',   # HuggingFace repo
    filename='Llama-3.2-3B-Instruct-Q4_K_M.gguf',      # Specific file
    local_dir='./models'                                 # Where to save
)
```

!!! tip "Use absolute paths"
    When passing the model path to `tinfer` or `tinfer-server`, use the **full absolute path** to avoid errors.
    For example: `tinfer -m C:\Users\you\models\model.gguf` instead of `tinfer -m models/model.gguf`.

---

## Recommended Models

### Text Models (MoE — Best Performance/Size Ratio)

| Model | Params (Total/Active) | Q4 Size | HuggingFace Repo | Filename |
|---|:---:|:---:|---|---|
| **Granite 3.1-1B MoE** | 1B / 400M | ~0.6 GB | `itlwas/granite-3.1-1b-a400m-instruct-Q4_K_M-GGUF` | `granite-3.1-1b-a400m-instruct-Q4_K_M.gguf` |
| **OLMoE-1B-7B** | 7B / 1.3B | 4.2 GB | `allenai/OLMoE-1B-7B-0924-GGUF` | `olmoe-1b-7b-0924-q4_k_m.gguf` |
| **GLM-4.7-Flash** ⭐ | 30B / 3B | 18.3 GB | `unsloth/GLM-4.7-Flash-GGUF` | `GLM-4.7-Flash-Q4_K_M.gguf` |
| **Qwen3-30B-A3B** | 30B / 3B | 18.6 GB | `bartowski/Qwen_Qwen3-30B-A3B-GGUF` | `Qwen3-30B-A3B-Q4_K_M.gguf` |

### Vision / Multimodal Models

| Model | Params | Q4 Size | Capability | HuggingFace Repo |
|---|:---:|:---:|---|---|
| **MiniCPM-V 4.0** ⭐ | 4B | 2.2 GB | Image understanding | `openbmb/MiniCPM-V-4.0-GGUF` |
| **Gemma 3-4B** | 4B | 2.5 GB | Vision + text | `bartowski/gemma-3-4b-it-GGUF` |
| **Qwen2.5-VL-7B** | 7B | 4.7 GB | Vision + video + docs | `Qwen/Qwen2.5-VL-7B-Instruct-GGUF` |

### General Text Models

| Model | Params | Q4 Size | Best For | HuggingFace Repo |
|---|:---:|:---:|---|---|
| **Llama 3.2-3B** | 3B | ~2 GB | General chat | `bartowski/Llama-3.2-3B-Instruct-GGUF` |
| **Qwen2.5-3B** | 3B | ~2 GB | Multilingual, code | `Qwen/Qwen2.5-3B-Instruct-GGUF` |
| **Mistral-7B** | 7B | ~4.1 GB | Fast general purpose | `mistralai/Mistral-7B-Instruct-v0.3-GGUF` |

---

## Understanding Quantization

GGUF models come in different quantization levels that trade quality for size:

| Quantization | Quality | Size | Use Case |
|:---:|:---:|:---:|---|
| **Q8_0** | Highest | Largest | Best quality, needs more RAM |
| **Q5_K_M** | High | Medium | Good balance |
| **Q4_K_M** | Good | Small | **Recommended** — best size/quality ratio |
| **Q3_K_M** | Acceptable | Smaller | Low-RAM systems |
| **Q2_K** | Lower | Smallest | Extreme memory constraints |

!!! info "What is MoE?"
    **Mixture of Experts** models have many total parameters but only activate a small subset per token. For example, GLM-4.7-Flash has 30B total parameters but only uses 3B at a time, giving you 30B-quality responses at 3B speed.

---

## After Downloading

Once you have a model, you can use it with any Tinfer command:

```bash
# CLI chat
tinfer -m C:\path\to\model.gguf -p "Hello!" -n 100

# Start server
tinfer-server -m C:\path\to\model.gguf --port 8080
```
