# Model Download

Tinfer runs **GGUF** format models. You can download them from [HuggingFace](https://huggingface.co) using any of the methods below.

---

## Method 1: Terminal One-Liner

The quickest way to download a model directly from your terminal:

```bash
pip install huggingface-hub
python -c "from huggingface_hub import hf_hub_download; import os; os.makedirs('models', exist_ok=True); hf_hub_download(repo_id='bartowski/Llama-3.2-3B-Instruct-GGUF', filename='Llama-3.2-3B-Instruct-Q4_K_M.gguf', local_dir='./models')"
```

Replace the `repo_id` and `filename` with any GGUF model from [HuggingFace](https://huggingface.co).

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
tinfer -m C:\path\to\model.gguf -p "Hello!" -n 100 -c 1024

# Start server
tinfer-server -m C:\path\to\model.gguf --port 8080 -c 1024
```

!!! info "What is `-c 1024`?"
    The `-c` flag sets the **context size** — how much memory the model allocates for its conversation window. Without it, some models try to allocate 14 GB+ of RAM by default, which will crash on most systems. Start with `-c 1024` and increase if your hardware allows. See [Troubleshooting](troubleshooting.md) for details.

---

## Quick Start on Google Colab

Want to try Tinfer instantly without installing anything locally? Open our ready-to-run notebook:

[![Open in Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/Sunilk240/tinfer-ai/blob/main/Tinfer_Setup.ipynb)

The notebook installs Tinfer, sets up the engine, downloads a model, and runs inference — all in under 5 minutes on a free T4 GPU.
