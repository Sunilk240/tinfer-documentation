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
tinfer -m C:\path\to\model.gguf -p "Hello!" -n 100

# Start server
tinfer-server -m C:\path\to\model.gguf --port 8080
```
