# TECHNICAL — uncensore-llm

Documento tecnico de arquitectura, stack y decisiones de diseno.

---

## Arquitectura General

```
┌─────────────────────────────────────────────────────────┐
│                     menu.sh (CLI)                        │
│  1.Servidor  2.Educacion  3.Modelos  4.Update  5.Conf  │
└─────────┬──────────┬──────────┬──────────┬──────────────┘
          │          │          │          │
    ┌─────▼──┐  ┌───▼────┐ ┌──▼──────┐ ┌─▼──────────┐
    │start.sh│  │edu/    │ │validate/│ │update.sh    │
    │        │  │setup.sh│ │download │ │             │
    └────┬───┘  └────┬───┘ └────┬────┘ └──────┬──────┘
         │           │          │              │
    ┌────▼───────────▼──────────▼──────────────▼──────────┐
    │                  llama.cpp (C/C++)                    │
    │  llama-server  │  llama-cli  │  llama-bench          │
    │  :8080 HTTP    │  :terminal  │  :perf                │
    └──────────────────────┬──────────────────────────────┘
                           │
                    ┌──────▼──────┐
                    │  models/    │
                    │  *.gguf     │
                    └─────────────┘
```

---

## Stack Tecnologico

| Capa | Tecnologia | Version | Razon |
|------|-----------|---------|-------|
| Runtime | llama.cpp | b4464 | CPU-only, zero-deps, GGUF nativo, API OpenAI |
| Shell | Bash | 5.0+ | Portable (Git Bash en Windows), universal |
| Python | Portable (indygreg) | 3.11+ | Auto-descargable, sin root, para labs educativos |
| OLLAMA | Binario portable | 0.32.5+ | Opcional, secundario a llama.cpp |
| Binarios | Pre-compilados C/C++ | b4464 | Linux x86_64 AVX2 (Tiger Lake+) |
| Modelos | GGUF Q4_K_M | Varios | Quantizados 4-bit, balance calidad/rendimiento |
| Descargas | curl + HuggingFace API | - | Curl zero-install, fallback a huggingface-cli |
| Integridad | SHA256 manifest | - | `.manifest` con 79 archivos trackeados |

---

## Runtime Decision: llama.cpp vs OLLAMA

### Decision (ADR-001)
**llama.cpp es el runtime primario.** OLLAMA es opcional.

### Justificacion

| Criterio | llama.cpp | OLLAMA |
|----------|-----------|--------|
| Dependencias | Cero (binarios pre-compilados) | Go runtime (~100 MB) |
| Instalacion | Ninguna (copiar bin/) | ollama serve + pull |
| Tamanio | ~50 MB (bin/) | ~1.4 GB (ollama + modelo) |
| API | OpenAI-compatible nativa | OpenAI-compatible |
| Tool calling | --jinja mode (experimental) | Soporte nativo |
| USB-friendly | Excelente | Pobre (daemon, install) |
| Performance CPU | Superior (menos overhead) | Comparable |

---

## TOOL_USE Architecture

El sistema de tool calling se activa en 3 niveles de prioridad:

```
--tool-use flag > TOOL_USE=true env var > TOOL_USE file
```

| Modo | Comportamiento | Jinja |
|------|---------------|-------|
| TOOL_USE OFF (default) | `--no-jinja` activo, max compatibilidad | Desactivado |
| TOOL_USE ON | `--no-jinja` omitido, llama-server usa plantillas | Activado |

El flag `--no-jinja` desabilita el procesamiento de plantillas Jinja en llama-server.
Esto es necesario para modelos que no tienen template configurado (ej: Gemma 2 2B vanilla).
Modelos modernos (Phi-4, Qwen 2.5, Llama 3.2) soportan templates y funcionan mejor con Jinja activo.

---

## Download Pipeline

```
download-model.sh
├── Modo huggingface-cli (default)
│   └── huggingface-cli download {repo} --include {filename}
│
└── Modo curl (--curl flag or CURL_DOWNLOAD=true)
    └── curl -L https://huggingface.co/{repo}/resolve/main/{filename}
```

El modo curl elimina la dependencia de Python/huggingface-hub, permitiendo descargas
en sistemas sin Python instalado (solo `curl` requerido).

---

## RAM Loader Architecture

```
USB (lento)
    │
    ├── models/*.gguf  →  symlink  →  USB (llama.cpp lee directo)
    │
    └── scripts/, edu/, menu.sh  →  copia  →  /dev/shm/uncensore-llm/
                                               │
                                               └──  tmpfs (rapido, en RAM)
```

- **Scripts**: copiados a tmpfs para acceso rapido
- **Modelos**: symlinks desde tmpfs a USB. llama.cpp los carga a RAM al inicio,
  por lo que la velocidad de USB no afecta la inferencia una vez cargados.
- **Python portable**: symlink desde tmpfs a USB si existe en `python/linux/`

---

## Model Validation Pipeline

```
validate-models.sh
├── 1. Detecta modelos en models/*.gguf
├── 2. Compara contra catalog-2026.json
├── 3. Clasifica: EN CATALOGO / NO CATALOGADO
├── 4. Lista disponibles no instalados por tier
├── 5. Recomienda segun RAM del sistema
├── 6. Muestra espacio en disco
└── 7. Verifica conectividad a HuggingFace
```

### Catalogo Structure

```json
{
  "models": [
    {
      "id": "phi-4-mini",
      "name": "Phi-4 Mini",
      "filename": "Phi-4-mini-instruct-Q4_K_M.gguf",
      "size_gb": 2.4,
      "tier": "light",
      "status": "installed",
      "description": "...",
      "source": "microsoft/Phi-4-mini-instruct-GGUF"
    }
  ]
}
```

### Tiers

| Tier | RAM requerida | Modelos |
|------|--------------|---------|
| ultra-light | 1-2 GB | Llama 3.2 1B, Qwen 3.5 0.8B |
| light | 2-4 GB | Phi-4 Mini, Gemma 2 2B, SmolLM3 3B, Qwen Coder 3B, Llama 3.2 3B, Gemma 4 Nano |
| medium | 4-8 GB | Qwen 2.5 7B, Dolphin Qwen 7B, Gemma 4 4B |

---

## Auto-Update System

```
update.sh
├── 1. Obtiene .manifest remoto (GitHub raw)
├── 2. Compara SHA256 locales vs remotos
├── 3. Detective de archivos modificados/nuevos/eliminados
├── 4. Descarga solo archivos actualizados
└── 5. Reconstruye .manifest local
```

### Manifest Format

```
<SHA256>  <path>
a1b2c3d4...  ./menu.sh
e5f6g7h8...  ./scripts/validate-models.sh
```

---

## Educacion Architecture

```
education/
├── setup.sh          →  Entry point: verifica servidor, lista labs
├── labs/             →  Labs OWASP practicos (ejecutables contra LLM local)
│   ├── 01-basic-injection/
│   │   ├── lab.sh     →  Bash + curl (siempre funciona)
│   │   ├── lab.py     →  Python + openai client (pendiente)
│   │   ├── lab.ipynb  →  Jupyter notebook (pendiente)
│   │   ├── lab.md     →  Documentacion del lab
│   │   └── test.sh    →  Verificacion automatizada
│   ├── template/      →  Template para crear nuevos labs
│   └── ...
├── curriculum/        →  STUDY_PLAN.md, ROADMAP.md, TECHNICAL.md
└── cert-tracker/      →  Tracking de certificaciones
```

Cada lab cubre al menos 1 riesgo OWASP ASI (Agentic System Integration).
Los labs se ejecutan directamente contra el LLM local via API `/v1/chat/completions`.

---

## Configuracion (.env)

```bash
# Runtime
LLAMA_TYPE=llama.cpp        # llama.cpp | ollama
LLAMA_HOST=127.0.0.1
LLAMA_PORT=8080

# Features
TOOL_USE=                   # true = activar tool calling
CURL_DOWNLOAD=              # true = descargar modelos con curl
RAM_LOAD=                   # true = cargar a tmpfs al iniciar

# Paths
MODELS_DIR=models/
PYTHON_DIR=python/linux/
```

Copiar desde `.env.dist`:
```bash
cp .env.dist .env
```

---

## Performance Characteristics

### Benchmarks (CPU: i7-1165G7, 8 threads, 7.6 GB RAM)

| Modelo | Carga (s) | Inferencia (tok/s) | RAM idle | RAM peak |
|--------|-----------|-------------------|----------|----------|
| Gemma 2 2B | ~2 | 15-22 | 0.8 GB | 1.0 GB |
| Phi-3.5 Mini | ~3 | 12-18 | 1.2 GB | 1.5 GB |
| Qwen Coder 3B | ~3 | 10-16 | 1.0 GB | 1.3 GB |
| Phi-4 Mini | ~4 | 10-16 | 1.3 GB | 1.6 GB |

### Optimizaciones activas
- AVX2: detectado y usado automaticamente por llama.cpp
- --no-jinja: reduce overhead de procesamiento de templates
- -c 4096: contexto limitado para conservar RAM
- Q4_K_M: quantizacion balanceada (calidad 95%+ vs FP16)

---

## Security Considerations

- **Air-gapped**: todo funciona sin internet. Ideal para datos sensibles.
- **Sin telemetria**: ni llama.cpp ni los modelos envian datos a servidores externos.
- **Sin cuentas**: no requiere API keys, logins, ni registros.
- **Modelos confiables**: solo se usan modelos de fuentes verificadas (Microsoft, Google, Meta, Alibaba).
- **Local-only**: el servidor escucha en `127.0.0.1:8080`. No expuesto a la red.
- **TOOL_USE**: desactivado por defecto. Requiere activacion explicita en 3 niveles.

---

## Decisiones de Diseno

| Decision | Razon | Ver ADR |
|----------|-------|---------|
| llama.cpp sobre OLLAMA | Zero-deps, portable, menor overhead | ADR-001 |
| GGUF Q4_K_M | Balance calidad/rendimiento para CPU | ADR-001 |
| --no-jinja default | Max compatibilidad con modelos sin template | — |
| curl mode download | Elimina dependencia de Python/huggingface-cli | — |
| USB-first | Estructura plana, sin symlinks internos | ADR-001 |
| Bash sobre Python para scripts core | Universal (Git Bash en Windows), zero-deps | — |
| Python portable (indygreg) | Python sin root, para labs educativos | — |
