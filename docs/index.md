---
hide:
  - toc
---

# Tinfer — Tiny Inference Engine

## Run LLMs locally with GPU acceleration

Tinfer is a high-performance local inference engine for running large language models on your own hardware. No cloud, no API keys, no dependencies.

<div class="grid-container" markdown>
<div class="grid-item" markdown>
### ⚡ CLI Chat
Run any GGUF model from your terminal with a single command.
</div>
<div class="grid-item" markdown>
### 🖥️ Server + WebUI
HTTP server with a built-in chat interface at `localhost:8080`.
</div>
<div class="grid-item" markdown>
### 🔌 OpenAI-Compatible API
Drop-in replacement for OpenAI's `/v1/chat/completions` endpoint.
</div>
<div class="grid-item" markdown>
### 📦 Layer Offloading
Run models larger than VRAM — dynamic Disk → CPU → GPU layer swapping.
</div>
<div class="grid-item" markdown>
### 🧩 PagedAttention
Zero-fragmentation KV cache with O(1) context shifting and Copy-on-Write.
</div>
<div class="grid-item" markdown>
### ♻️ KV Cache Eviction
Infinite-length generation — smart eviction keeps critical tokens.
</div>
<div class="grid-item" markdown>
### 🚀 CUDA GPU Acceleration
Offload layers to your NVIDIA GPU for faster inference.
</div>
<div class="grid-item" markdown>
### 🧠 MoE Support
Run Mixture-of-Experts models — 30B params at 3B speed.
</div>
<div class="grid-item" markdown>
### 👁️ Vision Models
Multimodal support for image understanding and OCR.
</div>
</div>

---

## Quick Start

```bash
# 1. Install
pip install tinfer-ai

# 2. Download a model
pip install huggingface-hub
python -c "from huggingface_hub import hf_hub_download; import os; os.makedirs('models', exist_ok=True); hf_hub_download(repo_id='bartowski/Llama-3.2-3B-Instruct-GGUF', filename='Llama-3.2-3B-Instruct-Q4_K_M.gguf', local_dir='./models')"

# 3. Run
tinfer -m models/Llama-3.2-3B-Instruct-Q4_K_M.gguf -p "Hello, what is AI?"
```

---

## Three Ways to Use Tinfer

### 1. CLI — Direct Chat

```bash
tinfer -m model.gguf -p "Explain quantum computing" -n 200
```

### 2. Server — WebUI + API

```bash
tinfer-server -m model.gguf --port 8080
# Open http://localhost:8080 for the chat UI
```

### 3. Python — Programmatic Access

```python
from tinfer import Server, chat

with Server("model.gguf", port=8080) as s:
    response = chat("What is artificial intelligence?")
    print(response)
```

---

## Documentation

| Page | Description |
|------|-------------|
| [Installation](installation.md) | Install Tinfer via pip |
| [Model Download](models.md) | Download GGUF models from HuggingFace |
| [CLI Reference](cli.md) | All CLI flags and options |
| [Server Reference](server.md) | Server flags, WebUI, and configuration |
| [API Reference](api.md) | OpenAI-compatible HTTP endpoints |
| [Python SDK](python-sdk.md) | Python client and server management |
