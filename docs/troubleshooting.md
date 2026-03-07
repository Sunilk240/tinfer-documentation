# Troubleshooting

Common issues and their solutions when using Tinfer.

---

## Memory Allocation Error

**Error message:**
```
ggml_aligned_malloc: insufficient memory (attempted to allocate 14336.00 MB)
llama_init_from_model: failed to initialize the context: failed to allocate buffer for kv cache
```

**Cause:**
The model's default context window requests more RAM than your system has available. This commonly happens on Google Colab (free tier), GitHub Codespaces, or laptops with limited RAM.

**Fix — limit the context size with `-c`:**

```bash
tinfer -m path/to/model.gguf -p "Hello!" -n 100 -c 512
```

The `-c` flag sets the context window size (in tokens). Reducing it from the model's default (often 128K+) to `512` or `1024` dramatically reduces memory usage:

| `-c` value | Approximate RAM usage | Best for |
|:---:|:---:|---|
| `512` | ~100 MB | Codespaces, Replit, very low RAM |
| `1024` | ~200 MB | Google Colab free tier |
| `2048` | ~400 MB | Standard laptops (8 GB RAM) |
| `4096` | ~800 MB | Desktops (16 GB+ RAM) |

!!! tip "Rule of thumb"
    Start with `-c 1024` and increase if your system can handle it. Larger context = model can remember more of the conversation.

---

## GLIBC Version Error (Linux)

**Error message:**
```
tinfer: /lib/x86_64-linux-gnu/libc.so.6: version `GLIBC_2.38' not found
```

**Cause:**
Your Linux distribution has an older version of GLIBC than the Tinfer binaries require. Tinfer binaries are compiled on Ubuntu 22.04 and require **GLIBC 2.35** or newer.

**Fix:**
Upgrade your Linux distribution to one that ships with GLIBC 2.35+:

| Distribution | GLIBC Version | Supported? |
|:---:|:---:|:---:|
| Ubuntu 24.04 LTS | 2.39 | ✅ |
| Ubuntu 22.04 LTS | 2.35 | ✅ |
| Debian 12 | 2.36 | ✅ |
| Ubuntu 20.04 LTS | 2.31 | ❌ |
| Debian 11 | 2.31 | ❌ |

---

## Missing DLL Error (Windows)

**Error message:**
```
VCRUNTIME140.dll was not found
```

**Cause:**
The Microsoft Visual C++ runtime library is not installed on your system.

**Fix:**
Download and install the [Microsoft Visual C++ Redistributable (v14)](https://learn.microsoft.com/en-us/cpp/windows/latest-supported-vc-redist) from Microsoft's official website.

---

## Minimum System Requirements

| Platform | Requirement |
|---|---|
| **Windows** | Windows 10 (64-bit) or Windows 11 |
| **Linux** | GLIBC 2.35+ (Ubuntu 22.04 LTS or newer) |
| **macOS** | macOS 14 Sonoma or newer |
| **Python** | 3.8 or higher |
| **GPU** (optional) | NVIDIA GPU with CUDA drivers for acceleration |
