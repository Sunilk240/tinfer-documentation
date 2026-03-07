# Installation

## Step 1: Install via pip

```bash
pip install tinfer-ai
```

This installs the Tinfer Python wrapper and the following commands:

- `tinfer` — CLI for chat and text completion
- `tinfer-server` — HTTP server with WebUI
- `tinfer-bench` — Model benchmarking tool
- `tinfer-quantize` — Model quantization tool
- `tinfer-setup` — Inference engine installer

## Step 2: Setup the Inference Engine

After installing the pip package, you need to download the inference engine binary for your system. There are two options:

### Option A: Automatic Setup (Recommended)

Run the setup command — it will automatically detect your OS, CPU, and GPU, then download the correct engine:

```bash
tinfer-setup
```

You will see output like:

```
=======================================================
  Tinfer Setup -- Inference Engine Installer
=======================================================

[Tinfer] OS:   Windows (win)
[Tinfer] Arch: AMD64 (x64)
[Tinfer] GPU:  NVIDIA GeForce RTX 3060

[Tinfer] Selected Engine: win-x64-cuda
[Tinfer] Downloading: tinfer-v0.2.0-win-x64-cuda.zip
[########################################] 100% (505.5/505.5 MB)

[Tinfer] Extracting to C:\Users\you\.tinfer\bin...

[Tinfer] Done! Setup complete!
```

The engine is installed to `~/.tinfer/bin/` and all tinfer commands will automatically use it.

### Option B: Manual Download (For Developers)

If you prefer to download the engine manually:

**1. Download the correct archive for your platform from the [GitHub Releases](https://github.com/Sunilk240/tinfer-ai/releases) page:**

| Platform | GPU | Download |
|----------|-----|----------|
| Windows x64 | NVIDIA CUDA | [tinfer-v0.2.0-win-x64-cuda.zip](https://github.com/Sunilk240/tinfer-ai/releases/download/v0.2.0/tinfer-v0.2.0-win-x64-cuda.zip) |
| Windows x64 | CPU only | [tinfer-v0.2.0-win-x64-cpu.zip](https://github.com/Sunilk240/tinfer-ai/releases/download/v0.2.0/tinfer-v0.2.0-win-x64-cpu.zip) |
| Linux x64 | NVIDIA CUDA | [tinfer-v0.2.0-linux-x64-cuda.tar.gz](https://github.com/Sunilk240/tinfer-ai/releases/download/v0.2.0/tinfer-v0.2.0-linux-x64-cuda.tar.gz) |
| Linux x64 | CPU only | [tinfer-v0.2.0-linux-x64-cpu.tar.gz](https://github.com/Sunilk240/tinfer-ai/releases/download/v0.2.0/tinfer-v0.2.0-linux-x64-cpu.tar.gz) |
| Linux ARM64 | CPU only | [tinfer-v0.2.0-linux-arm64-cpu.tar.gz](https://github.com/Sunilk240/tinfer-ai/releases/download/v0.2.0/tinfer-v0.2.0-linux-arm64-cpu.tar.gz) |
| macOS ARM64 (Apple Silicon) | Metal | [tinfer-v0.2.0-macos-arm64-metal.tar.gz](https://github.com/Sunilk240/tinfer-ai/releases/download/v0.2.0/tinfer-v0.2.0-macos-arm64-metal.tar.gz) |
| macOS x64 (Intel) | CPU only | [tinfer-v0.2.0-macos-x64-cpu.tar.gz](https://github.com/Sunilk240/tinfer-ai/releases/download/v0.2.0/tinfer-v0.2.0-macos-x64-cpu.tar.gz) |

**2. Extract the downloaded archive to a folder of your choice.**

**3. Set the `TINFER_ENGINE_PATH` environment variable to the extracted folder path:**

=== "Windows (PowerShell)"

    ```powershell
    $env:TINFER_ENGINE_PATH = "D:\path\to\extracted\folder"
    ```

=== "Linux / macOS"

    ```bash
    export TINFER_ENGINE_PATH="/path/to/extracted/folder"
    ```

After setting the variable, all tinfer commands will use the engine from that custom path.

## Prerequisites

| Requirement | Details |
|---|---|
| **Python** | 3.8 or higher |
| **OS** | Windows (x64), Linux (x64, ARM64), macOS (x64, ARM64) |
| **GPU** (optional) | NVIDIA GPU with CUDA drivers for GPU acceleration |

!!! note "GPU is optional"
    Tinfer works on CPU-only systems. GPU acceleration speeds up inference but is not required. When a CUDA-capable GPU is detected, `tinfer-setup` will automatically download the GPU-accelerated engine.

## Verify Installation

After installing and running setup, verify everything works:

```bash
tinfer --help
```

## Upgrade

```bash
pip install --upgrade tinfer-ai
tinfer-setup  # Re-download the engine for the new version
```

## Uninstall

```bash
pip uninstall tinfer-ai
```

## Next Steps

Once installed, [download a model](models.md) and start using Tinfer!
