<div align="center">

# Uncensored LLM

Portable local LLM runtime — GGUF models from USB, zero install, zero internet

![Status](https://img.shields.io/badge/status-Stable-28a745?style=flat-square)
[![License](https://img.shields.io/badge/license-MIT-blue?style=flat-square)](LICENSE)
![Models](https://img.shields.io/badge/models-5_GGUF-6526d3?style=flat-square)
![Platform](https://img.shields.io/badge/platform-Linux%20%7C%20macOS%20%7C%20Windows-lightgrey?style=flat-square)
![Updated](https://img.shields.io/github/last-commit/aentrepreneur/uncensored-llm?style=flat-square)

</div>

## Overview

Run GGUF language models directly from a USB drive — no installation, no GPU, no internet. Powered by llama.cpp with a user-friendly RAM-aware model selector, OpenAI-compatible API, and web chat interface.

## Features

- **100% local**: CPU inference via llama.cpp, zero external dependencies
- **Zero-install**: download the project, run `./start.sh`, use the model
- **Portable**: works from USB (exFAT), external SSD, or local disk
- **Cross-platform**: Linux (x86_64), macOS (Intel & Apple Silicon), Windows (x64)
- **Multi-model**: 5 models included, interactive selector based on available RAM
- **Web UI**: HTTP server with chat interface at `http://localhost:8080`
- **OpenAI-compatible API**: `http://localhost:8080/v1`
- **Private**: zero data leaves your machine. No telemetry, no accounts

## Minimum Requirements

| Component | Minimum | Recommended |
|-----------|---------|-------------|
| RAM | 4 GB | 8 GB |
| USB | 8 GB | 32 GB+ |
| CPU | x86_64 with AVX2 (2013+) | Any modern |
| OS | Linux, macOS, Windows | — |

## Quick Start

```bash
./start.sh
```

This will:
1. Detect available RAM
2. List `.gguf` models in `models/`
3. Recommend optimal model based on RAM
4. Launch llama-server at `http://localhost:8080`

**Windows:** Double-click `start.bat`

```bash
# Test the API
curl http://localhost:8080/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{"model":"gguf","messages":[{"role":"user","content":"Hello"}],"max_tokens":50}'
```

## Included Models

5 models totaling ~9.8 GB. The selector recommends based on RAM:

| Model | Size | RAM | Context | Profile |
|-------|------|-----|---------|---------|
| Gemma 2 2B (Google) | 1.6 GB | 4 GB | 8K | Lightweight, official |
| Gemma 2 2B Abliterated | 1.6 GB | 4 GB | 8K | Uncensored variant |
| Phi-3.5 Mini (Microsoft) | 2.3 GB | 4 GB | 4K | Solid generalist |
| Qwen Coder 3B (Alibaba) | 2.0 GB | 4 GB | 32K | Code & debugging |
| Phi-4 Mini (Microsoft) | 2.4 GB | 4 GB | 128K | Best quality/price |

```bash
# Download additional models
./scripts/download-model.sh
```

## Performance

On modern CPU with AVX2:

| Model | Tokens/s | RAM Usage |
|-------|----------|-----------|
| Gemma 2 2B | ~15-25 | ~1 GB |
| Phi-3.5 Mini | ~12-20 | ~1.5 GB |
| Qwen Coder 3B | ~10-18 | ~1.3 GB |
| Phi-4 Mini | ~10-18 | ~1.6 GB |

Default context: 4096 tokens (configurable via `-c` in start.sh).

## OpenAI-Compatible API

```python
import openai
client = openai.OpenAI(base_url="http://localhost:8080/v1", api_key="not-needed")
response = client.chat.completions.create(
    model="gguf",
    messages=[{"role": "user", "content": "Hello"}]
)
print(response.choices[0].message.content)
```

Any tool supporting the OpenAI API can point to `http://localhost:8080/v1` and use the local model.

## USB Setup

```bash
cp -r /opt/uncensored-llm /media/usb/
cd /media/usb/uncensored-llm && ./start.sh
```

| Profile | Content | Min USB |
|---------|---------|---------|
| **Minimal** | Gemma 2 2B + binaries | 4 GB |
| **Light** | Gemma 2 2B + Phi-4 Mini | 8 GB |
| **Complete** | All models | **16 GB** |

## License

- This project: **MIT License**
- llama.cpp: MIT License
- Models: Google Gemma License, Microsoft MIT, Alibaba Tongyi Qianwen LICENSE

See `LICENSE` file for details.

## Author

Angel Esquivel — [@aentrepreneur](https://github.com/aentrepreneur)
