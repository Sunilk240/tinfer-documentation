# Tinfer — Tiny Inference Engine

**Run LLMs locally with GPU acceleration. No cloud, no API keys, no dependencies.**

Tinfer is a high-performance local inference engine for running large language models (LLMs) on your own hardware. It supports CUDA GPU acceleration, Mixture of Experts (MoE) models, and vision/multimodal models — all through a simple `pip install`.

## 📖 Documentation

**Live site → [sunilk240.github.io/tinfer-documentation](https://sunilk240.github.io/tinfer-documentation/)**

| Page | Description |
|------|-------------|
| [Installation](https://sunilk240.github.io/tinfer-documentation/installation/) | Install via `pip install tinfer-ai` |
| [Model Download](https://sunilk240.github.io/tinfer-documentation/models/) | Download GGUF models from HuggingFace |
| [CLI Reference](https://sunilk240.github.io/tinfer-documentation/cli/) | All CLI flags and options |
| [Server Reference](https://sunilk240.github.io/tinfer-documentation/server/) | Server flags, WebUI, config |
| [API Reference](https://sunilk240.github.io/tinfer-documentation/api/) | OpenAI-compatible HTTP endpoints |
| [Python SDK](https://sunilk240.github.io/tinfer-documentation/python-sdk/) | Python client and server management |

## ⚡ Quick Start

```bash
# Install
pip install tinfer-ai

# Download a model
pip install huggingface-hub
python -c "from huggingface_hub import hf_hub_download; import os; os.makedirs('models', exist_ok=True); hf_hub_download(repo_id='bartowski/Llama-3.2-3B-Instruct-GGUF', filename='Llama-3.2-3B-Instruct-Q4_K_M.gguf', local_dir='./models')"

# CLI chat
tinfer -m models/Llama-3.2-3B-Instruct-Q4_K_M.gguf -p "Hello, what is AI?"

# Start server with WebUI
tinfer-server -m models/Llama-3.2-3B-Instruct-Q4_K_M.gguf --port 8080
```

## 🔑 Features

- **CLI** — Chat and text completion from your terminal
- **Server** — HTTP server with built-in chat WebUI
- **API** — OpenAI-compatible `/v1/chat/completions` endpoint
- **GPU** — CUDA acceleration out of the box
- **MoE** — Run 30B-parameter models at 3B speed
- **Vision** — Multimodal image understanding and OCR

## 🛠️ Building the Docs Locally

```bash
pip install mkdocs-material
mkdocs serve
# Open http://localhost:8000
```

## 📦 PyPI

```bash
pip install tinfer-ai
```

[![PyPI](https://img.shields.io/pypi/v/tinfer-ai)](https://pypi.org/project/tinfer-ai/)
