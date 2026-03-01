# Supported Inference Types

Tinfer supports **four types of model inference**, each serving a different use case. Built on top of [llama.cpp](https://github.com/ggml-org/llama.cpp), Tinfer brings these capabilities through a single `pip install`.

---

## 1. Text Generation

Standard text completion and chat — the most common use case.

### Start Server

```bash
tinfer-server -m models/Llama-3.2-3B-Instruct-Q4_K_M.gguf --port 8080 -ngl 99
```

### CLI

```bash
tinfer -m models/Llama-3.2-3B-Instruct-Q4_K_M.gguf -p "What is artificial intelligence?" -n 200
```

### curl

```bash
curl http://localhost:8080/v1/chat/completions -H "Content-Type: application/json" -d "{\"messages\":[{\"role\":\"system\",\"content\":\"You are a helpful assistant.\"},{\"role\":\"user\",\"content\":\"What is artificial intelligence?\"}],\"max_tokens\":200,\"temperature\":0.7}"
```

### PowerShell

```powershell
Invoke-RestMethod -Uri "http://localhost:8080/v1/chat/completions" -Method Post -Body '{"messages":[{"role":"system","content":"You are a helpful assistant."},{"role":"user","content":"What is artificial intelligence?"}],"max_tokens":200,"temperature":0.7}' -ContentType "application/json" | Select-Object -ExpandProperty choices | Select-Object -ExpandProperty message | Select-Object -ExpandProperty content
```

### Python

```python
import requests

response = requests.post(
    "http://localhost:8080/v1/chat/completions",
    json={
        "messages": [
            {"role": "system", "content": "You are a helpful assistant."},
            {"role": "user", "content": "What is artificial intelligence?"}
        ],
        "max_tokens": 200,
        "temperature": 0.7
    }
)
print(response.json()["choices"][0]["message"]["content"])
```

---

## 2. Vision / Multimodal

Image understanding, OCR, and visual question answering. Requires a vision model and its multimodal projector (`mmproj`).

### Download Vision Model (Example: MiniCPM-V 4.0)

```bash
# Download model
python -c "from huggingface_hub import hf_hub_download; import os; os.makedirs('models', exist_ok=True); hf_hub_download(repo_id='openbmb/MiniCPM-V-4-gguf', filename='ggml-model-Q8_0.gguf', local_dir='./models')"

# Download multimodal projector
python -c "from huggingface_hub import hf_hub_download; hf_hub_download(repo_id='openbmb/MiniCPM-V-4-gguf', filename='mmproj-model-f16.gguf', local_dir='./models')"
```

### Download OCR Model (Example: GLM-OCR)

```bash
# Download model
python -c "from huggingface_hub import hf_hub_download; import os; os.makedirs('models', exist_ok=True); hf_hub_download(repo_id='ggml-org/GLM-OCR-GGUF', filename='GLM-OCR-Q8_0.gguf', local_dir='./models')"

# Download multimodal projector
python -c "from huggingface_hub import hf_hub_download; hf_hub_download(repo_id='ggml-org/GLM-OCR-GGUF', filename='mmproj-GLM-OCR-Q8_0.gguf', local_dir='./models')"
```

### Start Server

```bash
# MiniCPM-V 4.0
tinfer-server -m models/ggml-model-Q8_0.gguf -mm models/mmproj-model-f16.gguf --port 8080 --layer-window auto

# GLM-OCR
tinfer-server -m models/GLM-OCR-Q8_0.gguf -mm models/mmproj-GLM-OCR-Q8_0.gguf --port 8080 -ngl 99
```

### CLI

```bash
# Text prompt with vision model
tinfer -m models/ggml-model-Q8_0.gguf -mm models/mmproj-model-f16.gguf -p "What is AI?" -n 100

# Describe an image
tinfer -m models/ggml-model-Q8_0.gguf -mm models/mmproj-model-f16.gguf --image path/to/image.png -p "Describe this image" -n 200

# With layer offloading
tinfer -m models/ggml-model-Q8_0.gguf -mm models/mmproj-model-f16.gguf --image image.png -p "Describe this image" -n 200 --layer-window auto

# Interactive mode
tinfer -m models/ggml-model-Q8_0.gguf -mm models/mmproj-model-f16.gguf --image image.jpg -p "What do you see?" -n 150 --interactive
```

### Python (API)

```python
import requests
import base64

# Read and encode the image
with open("image.png", "rb") as f:
    image_base64 = base64.b64encode(f.read()).decode()

response = requests.post(
    "http://localhost:8080/v1/chat/completions",
    json={
        "messages": [{
            "role": "user",
            "content": [
                {"type": "image_url", "image_url": {"url": f"data:image/png;base64,{image_base64}"}},
                {"type": "text", "text": "Describe this image in detail"}
            ]
        }],
        "max_tokens": 200,
        "temperature": 0.7
    }
)
print(response.json()["choices"][0]["message"]["content"])
```

### PowerShell (API)

```powershell
$imageBytes = [System.IO.File]::ReadAllBytes("image.png")
$base64Image = [Convert]::ToBase64String($imageBytes)
$body = @{
    messages = @(@{
        role = "user"
        content = @(
            @{ type = "image_url"; image_url = @{ url = "data:image/png;base64,$base64Image" } },
            @{ type = "text"; text = "Describe this image in detail" }
        )
    })
    max_tokens = 200
    temperature = 0.7
} | ConvertTo-Json -Depth 5
Invoke-RestMethod -Uri "http://localhost:8080/v1/chat/completions" -Method Post -Body $body -ContentType "application/json" | Select-Object -ExpandProperty choices | Select-Object -ExpandProperty message | Select-Object -ExpandProperty content
```

!!! note "Vision Models need a multimodal projector"
    Vision models require both the model file (`-m`) and the multimodal projector file (`-mm`). The projector maps image features into the language model's embedding space. Both files are available from the same HuggingFace repo.

---

## 3. Embedding & Reranking

Generate text embeddings for semantic search, or rerank documents by relevance.

### Download Embedding Model

```bash
python -c "from huggingface_hub import hf_hub_download; import os; os.makedirs('models', exist_ok=True); hf_hub_download(repo_id='second-state/All-MiniLM-L6-v2-Embedding-GGUF', filename='all-MiniLM-L6-v2-Q4_K_M.gguf', local_dir='./models')"
```

### Embedding — Start Server

```bash
tinfer-server -m models/all-MiniLM-L6-v2-Q4_K_M.gguf --embedding --port 8080 -ngl 99
```

!!! warning "Use `--embedding` flag"
    The `--embedding` flag is required to enable the embedding endpoint. Without it, `/v1/embeddings` will return an error.

### Embedding — Test

=== "PowerShell"

    ```powershell
    Invoke-RestMethod -Uri "http://localhost:8080/v1/embeddings" -Method Post -Body '{"input":"Hello world","model":"all-MiniLM-L6-v2"}' -ContentType "application/json"
    ```

=== "Python"

    ```python
    import requests

    response = requests.post(
        "http://localhost:8080/v1/embeddings",
        json={"input": "Hello world", "model": "all-MiniLM-L6-v2"}
    )
    data = response.json()["data"][0]["embedding"]
    print(f"Embedding vector (first 10 dims): {data[:10]}")
    print(f"Total dimensions: {len(data)}")
    ```

### Reranking — Start Server

Uses a text model with the `--rerank` flag:

```bash
tinfer-server -m models/Llama-3.2-3B-Instruct-Q4_K_M.gguf --rerank --port 8080 -ngl 99
```

### Reranking — Test

=== "PowerShell"

    ```powershell
    Invoke-RestMethod -Uri "http://localhost:8080/v1/rerank" -Method Post -Body '{"query":"What is AI?","documents":["AI is artificial intelligence","Dogs are animals","Python is a language"]}' -ContentType "application/json"
    ```

=== "Python"

    ```python
    import requests

    response = requests.post(
        "http://localhost:8080/v1/rerank",
        json={
            "query": "What is AI?",
            "documents": [
                "AI is artificial intelligence",
                "Dogs are animals",
                "Python is a language"
            ]
        }
    )
    for result in response.json()["results"]:
        print(f"Index: {result['index']}, Score: {result['relevance_score']:.4f}")
    ```

---

## 4. Fine-Tuned Model Inference (LoRA)

Run fine-tuned models using **LoRA adapters** — small weight files applied on top of a base model for specialized tasks (code, medical, translation, etc.).

```
Base Model (e.g., Llama-3.2-3B) + LoRA Adapter = Specialized Model
```

### Basic LoRA Usage

```bash
# Load base model with LoRA adapter
tinfer-server -m models/Llama-3.2-3B-Instruct-Q4_K_M.gguf --lora path/to/lora-adapter.gguf --port 8080 -ngl 99
```

### Multiple LoRA Adapters

```bash
tinfer-server -m models/Llama-3.2-3B-Instruct-Q4_K_M.gguf --lora lora1.gguf,lora2.gguf --port 8080 -ngl 99
```

### LoRA with Custom Scaling

Control how strongly the adapter influences the output (0.0 = no effect, 1.0 = full effect):

```bash
# Single adapter at 80% strength
tinfer-server -m models/Llama-3.2-3B-Instruct-Q4_K_M.gguf --lora-scaled lora-adapter.gguf:0.8 --port 8080 -ngl 99

# Multiple adapters with different strengths
tinfer-server -m models/Llama-3.2-3B-Instruct-Q4_K_M.gguf --lora-scaled lora1.gguf:0.5,lora2.gguf:1.0 --port 8080 -ngl 99
```

### Dynamic LoRA Switching (via API)

Load adapters without applying, then switch at runtime:

```bash
tinfer-server -m models/Llama-3.2-3B-Instruct-Q4_K_M.gguf --lora lora-adapter.gguf --lora-init-without-apply --port 8080 -ngl 99
```

Then apply via API:

```bash
curl http://localhost:8080/lora-adapters -X POST -H "Content-Type: application/json" -d '{"lora_adapters":[{"path":"lora-adapter.gguf","scale":1.0}]}'
```

### LoRA Flags Reference

| Flag | Description |
|------|-------------|
| `--lora FNAME` | Path to LoRA adapter (comma-separated for multiple) |
| `--lora-scaled FNAME:SCALE,...` | LoRA with custom scaling factor |
| `--lora-init-without-apply` | Load LoRA without applying (switch via API later) |

### Where to Get LoRA Adapters

- **HuggingFace** — search for "LoRA GGUF" adapters
- **Train your own** — use [Unsloth](https://github.com/unslothai/unsloth), [PEFT](https://github.com/huggingface/peft), or [LLaMA-Factory](https://github.com/hiyouga/LLaMA-Factory)
- **Convert existing** — convert PyTorch LoRA weights to GGUF format

!!! tip "LoRA adapters are small"
    A typical LoRA adapter is only 10–100 MB — much smaller than the base model. This makes it easy to have multiple specialized adapters for different tasks while sharing one base model.
