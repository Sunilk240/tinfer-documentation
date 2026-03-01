---
hide:
  - toc
---

# Tinfer — Tiny Inference Engine

## Run LLMs locally with GPU acceleration

Tinfer is a high-performance local inference engine built on top of [llama.cpp](https://github.com/ggml-org/llama.cpp). Run large language models on your own hardware — no cloud, no API keys, no C++ build tools.

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
### 👁️ Vision & OCR
Image understanding, visual QA, and OCR with multimodal models.
</div>
<div class="grid-item" markdown>
### 🔍 Embedding & Reranking
Generate text embeddings and rerank documents for semantic search.
</div>
<div class="grid-item" markdown>
### 🎯 LoRA Fine-Tuning
Run fine-tuned models with LoRA adapters — hot-swap at runtime.
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
### 📥 pip Install
`pip install tinfer-ai` — no C++ compiler or CMake needed.
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
| [Inference Types](inference-types.md) | Text, Vision, Embedding, Reranking, LoRA |
| [CLI Reference](cli.md) | All CLI flags and options |
| [Server Reference](server.md) | Server flags, WebUI, and configuration |
| [API Reference](api.md) | OpenAI-compatible HTTP endpoints |
| [Python SDK](python-sdk.md) | Python client and server management |
| [Layer Offloading](layer-offloading.md) | Run models larger than VRAM |
| [PagedAttention](paged-attention.md) | Zero-fragmentation KV cache |
| [KV Cache Eviction](kv-eviction.md) | Infinite-length generation |
