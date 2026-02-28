# Server Reference

The `tinfer-server` command starts an HTTP server with a built-in WebUI and OpenAI-compatible API endpoints.

## Basic Usage

```bash
# Start server with default settings
tinfer-server -m model.gguf --port 8080

# With GPU acceleration
tinfer-server -m model.gguf --port 8080 -ngl 99

# Custom host and context size
tinfer-server -m model.gguf --host 0.0.0.0 --port 9090 -c 4096
```

After starting, open **http://localhost:8080** in your browser for the chat WebUI.

---

## Server Configuration

| Flag | Description | Default |
|------|-------------|---------|
| `--host HOST` | IP address to listen on (env: `LLAMA_ARG_HOST`) | 127.0.0.1 |
| `--port PORT` | Port to listen on (env: `LLAMA_ARG_PORT`) | 8080 |
| `-np, --parallel N` | Number of parallel request slots (-1 = auto) (env: `LLAMA_ARG_N_PARALLEL`) | -1 |
| `-to, --timeout N` | Read/write timeout in seconds (env: `LLAMA_ARG_TIMEOUT`) | 600 |
| `--threads-http N` | Threads for HTTP processing (env: `LLAMA_ARG_THREADS_HTTP`) | -1 |
| `-a, --alias STRING` | Model alias for REST API (env: `LLAMA_ARG_ALIAS`) | — |
| `--path PATH` | Serve static files from this path (env: `LLAMA_ARG_STATIC_PATH`) | — |
| `--api-prefix PREFIX` | URL prefix for all endpoints (env: `LLAMA_ARG_API_PREFIX`) | — |

## Security

| Flag | Description | Default |
|------|-------------|---------|
| `--api-key KEY` | API key for authentication (comma-separated for multiple) (env: `LLAMA_API_KEY`) | none |
| `--api-key-file FNAME` | File containing API keys | — |
| `--ssl-key-file FNAME` | PEM-encoded SSL private key (env: `LLAMA_ARG_SSL_KEY_FILE`) | — |
| `--ssl-cert-file FNAME` | PEM-encoded SSL certificate (env: `LLAMA_ARG_SSL_CERT_FILE`) | — |

## WebUI

| Flag | Description | Default |
|------|-------------|---------|
| `--webui, --no-webui` | Enable/disable Web UI (env: `LLAMA_ARG_WEBUI`) | enabled |
| `--webui-config JSON` | Default WebUI settings as JSON (env: `LLAMA_ARG_WEBUI_CONFIG`) | — |
| `--webui-config-file PATH` | WebUI settings from JSON file (env: `LLAMA_ARG_WEBUI_CONFIG_FILE`) | — |

## Caching and Performance

| Flag | Description | Default |
|------|-------------|---------|
| `--cache-prompt, --no-cache-prompt` | Prompt caching (env: `LLAMA_ARG_CACHE_PROMPT`) | enabled |
| `--cache-reuse N` | Min chunk size for KV cache reuse (env: `LLAMA_ARG_CACHE_REUSE`) | 0 |
| `-cram, --cache-ram N` | Max cache size in MiB (-1=unlimited, 0=disable) (env: `LLAMA_ARG_CACHE_RAM`) | 8192 |
| `-kvu, --kv-unified` | Single unified KV buffer for all sequences (env: `LLAMA_ARG_KV_UNIFIED`) | auto |
| `--ctx-checkpoints N` | Max context checkpoints per slot (env: `LLAMA_ARG_CTX_CHECKPOINTS`) | 8 |
| `--context-shift, --no-context-shift` | Context shift for infinite generation (env: `LLAMA_ARG_CONTEXT_SHIFT`) | disabled |
| `--warmup, --no-warmup` | Warmup run on startup (env: `LLAMA_ARG_WARMUP`) | enabled |
| `-cb, --cont-batching` | Continuous/dynamic batching (env: `LLAMA_ARG_CONT_BATCHING`) | enabled |
| `-sps, --slot-prompt-similarity N` | Min prompt match to reuse slot | 0.10 |

## Multimodal / Vision

| Flag | Description | Default |
|------|-------------|---------|
| `-mm, --mmproj FILE` | Path to multimodal projector (env: `LLAMA_ARG_MMPROJ`) | — |
| `-mmu, --mmproj-url URL` | URL to multimodal projector (env: `LLAMA_ARG_MMPROJ_URL`) | — |
| `--mmproj-auto, --no-mmproj` | Auto-detect multimodal projector (env: `LLAMA_ARG_MMPROJ_AUTO`) | enabled |
| `--mmproj-offload, --no-mmproj-offload` | GPU offload for multimodal (env: `LLAMA_ARG_MMPROJ_OFFLOAD`) | enabled |
| `--image-min-tokens N` | Min tokens per image (env: `LLAMA_ARG_IMAGE_MIN_TOKENS`) | model default |
| `--image-max-tokens N` | Max tokens per image (env: `LLAMA_ARG_IMAGE_MAX_TOKENS`) | model default |

## Chat Template

| Flag | Description | Default |
|------|-------------|---------|
| `--chat-template TEMPLATE` | Custom Jinja2 chat template (env: `LLAMA_ARG_CHAT_TEMPLATE`) | model default |
| `--chat-template-file FILE` | Chat template from file (env: `LLAMA_ARG_CHAT_TEMPLATE_FILE`) | — |
| `--chat-template-kwargs JSON` | Extra params for template parser (env: `LLAMA_CHAT_TEMPLATE_KWARGS`) | — |
| `--jinja, --no-jinja` | Use Jinja template engine (env: `LLAMA_ARG_JINJA`) | enabled |
| `--prefill-assistant, --no-prefill-assistant` | Prefill if last message is assistant (env: `LLAMA_ARG_PREFILL_ASSISTANT`) | enabled |

## Reasoning / Thinking

| Flag | Description | Default |
|------|-------------|---------|
| `--reasoning-format {none,deepseek,deepseek-legacy}` | How to handle thought tags (env: `LLAMA_ARG_THINK`) | auto |
| `--reasoning-budget N` | Thinking budget (-1=unlimited, 0=disable) (env: `LLAMA_ARG_THINK_BUDGET`) | -1 |

## Monitoring Endpoints

| Flag | Description | Default |
|------|-------------|---------|
| `--metrics` | Enable Prometheus `/metrics` endpoint (env: `LLAMA_ARG_ENDPOINT_METRICS`) | disabled |
| `--props` | Allow POST `/props` to change settings (env: `LLAMA_ARG_ENDPOINT_PROPS`) | disabled |
| `--slots, --no-slots` | Expose `/slots` monitoring (env: `LLAMA_ARG_ENDPOINT_SLOTS`) | enabled |
| `--slot-save-path PATH` | Save slot KV cache to disk | — |
| `--media-path PATH` | Directory for local media files | — |

## Embedding & Reranking

| Flag | Description | Default |
|------|-------------|---------|
| `--embedding, --embeddings` | Enable embedding endpoint (env: `LLAMA_ARG_EMBEDDINGS`) | disabled |
| `--rerank, --reranking` | Enable reranking endpoint (env: `LLAMA_ARG_RERANKING`) | disabled |
| `--pooling {none,mean,cls,last,rank}` | Pooling type for embeddings (env: `LLAMA_ARG_POOLING`) | model default |

## Model Router (Multi-Model)

| Flag | Description | Default |
|------|-------------|---------|
| `--models-dir PATH` | Directory of models for router (env: `LLAMA_ARG_MODELS_DIR`) | — |
| `--models-preset PATH` | INI file with model presets (env: `LLAMA_ARG_MODELS_PRESET`) | — |
| `--models-max N` | Max models loaded simultaneously (env: `LLAMA_ARG_MODELS_MAX`) | 4 |
| `--models-autoload, --no-models-autoload` | Auto-load models (env: `LLAMA_ARG_MODELS_AUTOLOAD`) | enabled |

## Speculative Decoding

| Flag | Description | Default |
|------|-------------|---------|
| `-md, --model-draft FNAME` | Draft model for speculative decoding (env: `LLAMA_ARG_MODEL_DRAFT`) | — |
| `--draft N` | Tokens to draft (env: `LLAMA_ARG_DRAFT_MAX`) | 16 |
| `--draft-min N` | Min draft tokens (env: `LLAMA_ARG_DRAFT_MIN`) | 0 |
| `--draft-p-min P` | Min speculative probability (env: `LLAMA_ARG_DRAFT_P_MIN`) | 0.75 |
| `-ngld, --n-gpu-layers-draft N` | GPU layers for draft model (env: `LLAMA_ARG_N_GPU_LAYERS_DRAFT`) | auto |

## Sleep / Idle

| Flag | Description | Default |
|------|-------------|---------|
| `--sleep-idle-seconds N` | Sleep after N seconds idle (-1 = disabled) | -1 |

---

## Environment Variables

All flags with `(env: ...)` can be set as environment variables. The flag takes precedence over the env var.

For boolean options:

- `LLAMA_ARG_MMAP=true` → enabled (also: `1`, `on`, `enabled`)
- `LLAMA_ARG_MMAP=false` → disabled (also: `0`, `off`, `disabled`)
- `LLAMA_ARG_NO_MMAP` → disabling (presence alone is enough)

!!! tip "Common Server Configuration"
    ```bash
    tinfer-server -m model.gguf --port 8080 -ngl 99 -c 4096 -np 4
    ```
    This starts the server on port 8080 with all layers on GPU, 4K context, and 4 parallel slots.

!!! note "All CLI Options Also Apply"
    The server accepts all flags from the [CLI Reference](cli.md) as well (model, GPU, memory, sampling, etc.). This page only documents server-specific flags.
