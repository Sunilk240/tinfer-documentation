# Installation

## Install via pip

```bash
pip install tinfer-ai
```

This installs all Tinfer components:

- `tinfer` — CLI for chat and text completion
- `tinfer-server` — HTTP server with WebUI
- `tinfer-bench` — Model benchmarking tool
- `tinfer-quantize` — Model quantization tool

## Prerequisites

| Requirement | Details |
|---|---|
| **Python** | 3.8 or higher |
| **OS** | Windows (x64) |
| **GPU** (optional) | NVIDIA GPU with CUDA support for acceleration |

!!! note "GPU is optional"
    Tinfer works on CPU-only systems. GPU acceleration speeds up inference but is not required. Models will automatically use your GPU if CUDA is detected.

## Verify Installation

After installing, verify that the commands are available:

```bash
# Check CLI version
tinfer --version

# Check server version
tinfer-server --version
```

You should see output similar to:

```
ggml_cuda_init: found 1 CUDA devices:
  Device 0: NVIDIA GeForce ..., compute capability X.X, VMM: yes
version: XXXX (commit hash)
Tinfer build: XXXX (commit) with MSVC XX for x64
```

## Upgrade

```bash
pip install --upgrade tinfer-ai
```

## Uninstall

```bash
pip uninstall tinfer-ai
```

## Next Steps

Once installed, [download a model](models.md) and start using Tinfer!
