# Python SDK

The Tinfer Python SDK lets you manage servers and make API calls programmatically.

```bash
pip install tinfer-ai
```

---

## Quick Example

```python
from tinfer import Server, chat

# Start server and chat in one block
with Server("model.gguf", port=8080, n_gpu_layers=-1) as s:
    response = chat("What is artificial intelligence?")
    print(response)
# Server automatically stops when the block exits
```

---

## Server Class

The `Server` class manages the `tinfer-server` process lifecycle.

### Constructor

```python
from tinfer import Server

server = Server(
    model_path="path/to/model.gguf",  # Required: path to GGUF file
    port=8080,                         # Server port (default: 8080)
    host="127.0.0.1",                  # Bind address (default: 127.0.0.1)
    n_gpu_layers=None,                 # GPU layers (-1 = all, None = auto)
    ctx_size=None,                     # Context window size
    n_parallel=None,                   # Parallel request slots
    extra_args=None                    # Additional CLI flags as list
)
```

| Parameter | Type | Description | Default |
|---|---|---|---|
| `model_path` | str | Path to GGUF model file | required |
| `port` | int | Port for the server | 8080 |
| `host` | str | Host/IP to bind to | 127.0.0.1 |
| `n_gpu_layers` | int | Layers to offload to GPU (-1=all) | None (auto) |
| `ctx_size` | int | Context window size | None (model default) |
| `n_parallel` | int | Number of parallel slots | None (auto) |
| `extra_args` | list | Additional command-line flags | [] |

### Methods

#### `start(timeout=30)`

Start the server and wait until it's healthy.

```python
server = Server("model.gguf", port=8080)
server.start(timeout=30)  # Waits up to 30 seconds
print(f"Server ready at {server.base_url}")
```

!!! warning
    Raises `RuntimeError` if the server fails to start or doesn't become healthy within the timeout.

#### `stop()`

Stop the server process.

```python
server.stop()
```

#### `is_running()`

Check if the server is running and healthy.

```python
if server.is_running():
    print("Server is ready")
```

### Context Manager

The recommended way to use the server — it auto-starts and auto-stops:

```python
from tinfer import Server, chat

with Server("model.gguf", port=8080) as s:
    print(s.base_url)  # http://127.0.0.1:8080
    response = chat("Hello!")
    print(response)
# Server automatically stopped here
```

### Extra Args Example

Pass any additional `tinfer-server` flags:

```python
server = Server(
    "model.gguf",
    port=8080,
    extra_args=["--flash-attn", "on", "-c", "8192", "--api-key", "my-key"]
)
```

---

## Client Functions

### `set_server(url)`

Set the default server URL for all client functions.

```python
from tinfer import set_server

set_server("http://192.168.1.100:8080")  # Remote server
set_server("http://localhost:8080")       # Local (default)
```

---

### `chat()`

Send a chat completion request. Returns the assistant's response as a string.

```python
from tinfer import chat

# Simple string prompt
response = chat("What is the capital of France?")
print(response)

# Full message format
response = chat([
    {"role": "system", "content": "You are a pirate."},
    {"role": "user", "content": "What's the weather?"}
])

# With parameters
response = chat(
    messages="Explain quantum computing",
    max_tokens=200,
    temperature=0.5,
    top_p=0.9
)
```

| Parameter | Type | Description | Default |
|---|---|---|---|
| `messages` | str or list | Message string or list of `{role, content}` dicts | — |
| `prompt` | str | Shorthand for a single user message | — |
| `model` | str | Model name (optional) | — |
| `max_tokens` | int | Max tokens to generate | 512 |
| `temperature` | float | Sampling temperature | 0.7 |
| `top_p` | float | Nucleus sampling | 0.95 |
| `stream` | bool | Stream response | False |
| `base_url` | str | Override server URL | — |

!!! tip
    Either `messages` or `prompt` must be provided. If `messages` is a string, it's automatically wrapped as `[{"role": "user", "content": "..."}]`.

---

### `complete()`

Send a text completion request (non-chat format).

```python
from tinfer import complete

text = complete(
    prompt="The capital of France is",
    max_tokens=50,
    temperature=0.3
)
print(text)
```

| Parameter | Type | Description | Default |
|---|---|---|---|
| `prompt` | str | Text to complete | required |
| `model` | str | Model name | — |
| `max_tokens` | int | Max tokens | 512 |
| `temperature` | float | Temperature | 0.7 |
| `top_p` | float | Top-P | 0.95 |
| `base_url` | str | Override server URL | — |

---

### `models()`

List models loaded on the server.

```python
from tinfer import models

for m in models():
    print(m["id"])
```

Returns a list of model info dictionaries from the `/v1/models` endpoint.

---

## Error Handling

All client functions raise clear errors:

```python
from tinfer import chat

try:
    response = chat("Hello")
except ConnectionError:
    print("Server is not running! Start it with:")
    print("  tinfer-server -m model.gguf --port 8080")
except RuntimeError as e:
    print(f"Server error: {e}")
```

| Exception | When |
|---|---|
| `ConnectionError` | Server is not running or unreachable |
| `RuntimeError` | Server returned an HTTP error |
| `FileNotFoundError` | Model file not found (Server.start) |
| `ValueError` | Neither messages nor prompt provided (chat) |

---

## Complete Example

```python
from tinfer import Server, chat, complete, models

# Start server
with Server(
    r"C:\models\Llama-3.2-3B-Instruct-Q4_K_M.gguf",
    port=8080,
    n_gpu_layers=-1
) as s:
    # List models
    print("Loaded models:", [m["id"] for m in models()])

    # Chat
    print(chat("What is Python?"))

    # Chat with system prompt
    print(chat([
        {"role": "system", "content": "Answer in exactly 10 words."},
        {"role": "user", "content": "What is machine learning?"}
    ]))

    # Complete
    print(complete("def fibonacci(n):", max_tokens=100))
```
