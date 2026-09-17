<div align="center">

# 🧠 uncensore-llm

**LLMs locales portables desde USB — zero-install, sin GPU, sin internet**

[![Version](https://img.shields.io/badge/version-2.1.0-blue?style=flat-square)](https://github.com/aentrepreneur/uncensore-llm)
[![License](https://img.shields.io/badge/license-MIT-green?style=flat-square)](LICENSE)
[![Bash](https://img.shields.io/badge/bash-5.0%2B-4EAA25?style=flat-square&logo=gnubash)](scripts/)
[![Python](https://img.shields.io/badge/python-3.11%2B-blue?style=flat-square&logo=python)](python/)
[![Platform](https://img.shields.io/badge/platform-Linux%20%7C%20macOS%20%7C%20Windows-lightgrey?style=flat-square)]()

</div>

---

## Que es esto

Conectas una USB, ejecutas `./menu.sh` y tienes un LLM funcionando 100% local.
Sin instalar nada. Sin GPU. Sin internet. Sin cuentas.

Ideal para educacion, laboratorios de ciberseguridad (OWASP), prototipado rapido,
privacy-first, air-gapped, y cualquier escenario donde un LLM local sea util sin
la friccion de configurar entornos.

---

## Caracteristicas

| Capability | Description |
|---|---|
| Zero-install | Ejecuta desde USB (exFAT/NTFS/ext4), no requiere apt/pip/system |
| CPU-only | Inferencia 100% CPU via llama.cpp, optimizado para AVX2 |
| Multi-modelo | 5 modelos GGUF incluidos (9.7 GB), 18 en catalogo evaluable |
| Interfaz web | Chat UI en `http://localhost:8080` via llama-server |
| API OpenAI | Endpoint `/v1` compatible con cualquier cliente OpenAI |
| Modo educativo | Labs OWASP Top 10 for Agentic Applications (ASI01-ASI10) |
| RAM Load | Copia a `/dev/shm/` para cargar desde RAM y desconectar USB |
| TOOL_USE | Tool calling mode activable: `--tool-use` flag > env > file |
| Download offline | Descarga modelos via `curl` sin dependencia de huggingface-cli |
| Auto-update | `update.sh` compara el build local de llama.cpp vs el ultimo release b-tag en GitHub |
| Python portable | Python 3.11+ auto-descargable (indygreg builds), sin root |

---

## Quick Start

```bash
# 1. Clona o copia a USB
cp -r uncensore-llm /media/usb/

# 2. Ejecuta el menu
./menu.sh

# 3. Opcion 1 → Iniciar servidor (selecciona modelo, lanza en :8080)
# 4. Abri http://localhost:8080 en el navegador
```

> [!TIP]
> **Modo educativo:** `./menu.sh` → opcion 2. Labs OWASP funcionando contra tu LLM local.

### Probar la API

```bash
curl http://localhost:8080/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{"model":"gguf","messages":[{"role":"user","content":"Hola"}],"max_tokens":50}'
```

```python
import openai
client = openai.OpenAI(base_url="http://localhost:8080/v1", api_key="not-needed")
response = client.chat.completions.create(
    model="gguf",
    messages=[{"role": "user", "content": "Hola"}]
)
print(response.choices[0].message.content)
```

<details>
<summary>📋 Menu completo (8 opciones)</summary>

| Opcion | Funcion |
|--------|---------|
| 1 | Iniciar servidor (seleccionar modelo, lanzar llama-server) |
| 2 | Modo educativo (labs OWASP + curriculum) |
| 3 | Gestionar modelos (validar, descargar, listar catalogo) |
| 4 | Actualizar sistema (scripts + tools + catalogo desde GitHub) |
| 5 | Configuracion (TOOL_USE, runtime llama.cpp/OLLAMA, .env) |
| 6 | Cargar en RAM (tmpfs, permite desconectar USB) |
| 7 | Informacion del sistema (RAM, CPU, espacio, modelos) |
| 0 | Salir |

</details>

---

## Modelos incluidos

| Modelo | Archivo | Tamano | RAM | Contexto | Perfil |
|--------|---------|--------|-----|----------|--------|
| Qwen 3.5 4B (Alibaba) | `Qwen3.5-4B-Q4_K_M.gguf` | 2.55 GB | 4 GB | 256K | **Mejor general 2026** |
| Qwen 3.5 4B Uncensored | `Qwen3.5-4B-Uncensored-HauhauCS-Aggressive-Q4_K_M.gguf` | 2.52 GB | 4 GB | 256K | Sin censura |
| Qwen 3.5 9B (Alibaba) | `Qwen3.5-9B-Q4_K_M.gguf` | 5.29 GB | 8 GB | 256K | Alta calidad |
| Qwen 3.5 2B (Alibaba) | `Qwen3.5-2B-Q4_K_M.gguf` | 1.19 GB | 2 GB | 256K | Ultra-ligero |
| Gemma 4 E4B (Google) | `gemma-4-E4B-it-Q4_0.gguf` | 4.28 GB | 8 GB | 128K | Multimodal |
| Gemma 4 E4B Uncensored | `Gemma-4-E4B-Uncensored-HauhauCS-Aggressive-Q4_K_M.gguf` | 4.97 GB | 8 GB | 128K | Multimodal, sin censura |
| Phi-4 Mini (Microsoft) | `microsoft_Phi-4-mini-instruct-Q4_K_M.gguf` | 2.32 GB | 4 GB | 128K | Solido, Keeper |
| Qwen Coder 3B (Alibaba) | `qwen2.5-coder-3b-instruct-q4_k_m.gguf` | 2.0 GB | 4 GB | 32K | Codigo y debug |
| SmolLM3 3B (HF) | `SmolLM3-3B-Q4_K_M.gguf` | 1.78 GB | 4 GB | - | Ligero, agentes |
| Llama 3.2 1B (Meta) | `llama-3.2-1b-instruct-q4_k_m.gguf` | 0.75 GB | 2 GB | 128K | Chat rapido |

> [!NOTE]
> **13 modelos en catalogo** (`scripts/models/catalog-2026.json`) en 3 tiers:
> ultra-light (<2 GB), light (2-4 GB), medium (4-8 GB).
> 10 se instalan por defecto (~27.7 GB); 3 disponibles bajo demanda
> (Llama 3.2 3B Uncensored, Nanbeige 4.2, Dolphin Qwen 7B).
> Ejecuta `./scripts/download-model.sh --list` para verlos todos.

### Descargar mas modelos

```bash
# Listar disponibles
./scripts/download-model.sh --list

# Descargar un modelo especifico (curl-only, con resume)
./scripts/download-model.sh qwen3.5-4b

# Instalar el set completo por defecto
./scripts/download-model.sh --install-all
```

---

## Como funciona

| Componente | Descripcion |
|---|---|
| `llama.cpp` | Motor de inferencia C/C++, binarios pre-compilados en `bin/` |
| `llama-server` | Servidor HTTP con chat UI + API OpenAI-compatible |
| `menu.sh` | Entry point: detecta RAM, selecciona modelo, lanza servidor |
| `bootstrap.sh` | Verifica integridad: deps, modelos, Python, manifest |
| `ram-loader.sh` | Copia proyecto a `/dev/shm/` y crea symlinks a modelos |
| `.manifest` | SHA256 de integridad (se excluye models/) |

No hay Python, Node, Docker ni GPU requeridos. Todo son binarios C/C++ pre-compilados.

<details>
<summary>📂 Estructura completa</summary>

```
uncensore-llm/
├── menu.sh                  Entry point interactivo
├── bootstrap.sh             Verifica deps, modelos, Python, manifest
├── ram-loader.sh            Carga proyecto a tmpfs (/dev/shm/)
├── start.sh                 Lanza llama-server (detecta RAM + selector)
├── start.bat                Lanzador Windows
├── .env.dist                Template de configuracion
├── .manifest                SHA256 de integridad (excluye models/)
├── .model                   Ultimo modelo seleccionado
├── TOOL_USE                 Flag para activar tool calling
│
├── bin/                     Binarios llama.cpp (build b10936)
│   ├── llama-server         Servidor HTTP principal
│   ├── llama-cli            CLI interactivo
│   └── lib*.so              Librerias compartidas
│
├── models/                  Modelos GGUF (10 instalados, ~27.7 GB)
│   └── *.gguf               Quantizados Q4_K_M (~0.75-5.3 GB c/u)
│
├── scripts/
│   ├── download-model.sh    Descarga modelos (--curl mode sin deps)
│   ├── validate-models.sh   Valida vs catalogo, sugiere por RAM
│   ├── update.sh            Compara build local de llama.cpp vs GitHub
│   ├── ollama-bundle.sh     OLLAMA portable opcional
│   └── models/
│       └── catalog-2026.json  13 modelos en 3 tiers
│
├── python/
│   └── bootstrap.sh         Python portable (indygreg, ~35 MB)
│
├── education/
│   ├── setup.sh             Entry point modo educativo
│   ├── labs/                Labs OWASP practicos
│   │   ├── 01-basic-injection/  ASI-01: Prompt Injection
│   │   └── template/            Template para nuevos labs
│   ├── curriculum/           Plan de estudio + roadmap
│   └── cert-tracker/         Tracker de certificaciones
│
└── docs/
    ├── CHANGELOG.md          Historial de versiones
    ├── TECHNICAL.md          Arquitectura tecnica
    └── ADR/                  Architecture Decision Records
```

</details>

---

## Requisitos

| Componente | Minimo | Recomendado |
|------------|--------|-------------|
| **RAM** | 4 GB | 8 GB+ |
| **USB** | 8 GB | 32 GB+ |
| **CPU** | x86_64 con AVX2 (2013+) | Tiger Lake+ |
| **SO** | Linux, macOS, Windows | Linux (Ubuntu 24.04+) |

---

## Rendimiento

En CPU i7-1165G7 (Tiger Lake, 2021) con AVX2, modelo Q4_K_M, 4096 contexto:

| Modelo | Tokens/s | RAM uso |
|--------|----------|---------|
| Llama 3.2 1B | ~20-35 | ~0.7 GB |
| Qwen 3.5 2B | ~15-28 | ~0.9 GB |
| SmolLM3 3B / Qwen Coder 3B | ~12-20 | ~1.2 GB |
| Qwen 3.5 4B / Phi-4 Mini | ~10-18 | ~1.6 GB |
| Qwen 3.5 9B / Gemma 4 E4B | ~5-10 | ~3.0 GB |

> [!NOTE]
> Contexto por defecto: 4096 tokens. Configurable con `-c` en start.sh.
> Modelos con `--no-jinja` por defecto. TOOL_USE mode activa Jinja templates.

---

## USB — Portabilidad

El proyecto esta disenado para ejecutarse directamente desde USB sin instalacion.

### Preparar USB

```bash
# Copiar proyecto a USB
cp -r uncensore-llm /media/usb/

# Ejecutar
# Linux:
/media/usb/uncensore-llm/menu.sh

# Windows (doble-click):
/media/usb/uncensore-llm/start.bat
```

### Cargar en RAM y desconectar USB

```bash
./menu.sh     # → opcion 6
cd /dev/shm/uncensore-llm && ./menu.sh
# USB ya puede desconectarse. Modelos se quedan en USB.
```

<details>
<summary>📋 Requisitos de almacenamiento por perfil</summary>

| Perfil | Contenido | USB minima |
|--------|-----------|------------|
| Minimo | Gemma 2 2B + binarios | 4 GB |
| Liviano | Gemma 2 2B + Phi-4 Mini + binarios | 8 GB |
| Completo | Todos los modelos + binarios | **16 GB** |

</details>

<details>
<summary>📋 Notas tecnicas para USB</summary>

- **exFAT**: Recomendado para compatibilidad Windows/Linux/macOS. Soporta archivos >4 GB.
- **Permisos**: En exFAT no se preservan `+x`. Ejecutar `chmod +x menu.sh bin/llama-server` al copiar.
- **Modelos**: Descargar modelos adicionales ANTES de copiar a USB con `./scripts/download-model.sh --list`.
- **Sesion**: Seleccion de modelo se guarda en `.model`. En USB solo-lectura, el selector aparece cada vez.

</details>

---

## Licencias

| Componente | Licencia |
|------------|----------|
| Este proyecto | MIT |
| llama.cpp | MIT |
| Modelo Gemma 2 2B | Google Gemma License |
| Modelo Phi-3.5/Phi-4 Mini | MIT (Microsoft) |
| Modelo Qwen Coder 3B | Tongyi Qianwen LICENSE (Alibaba) |
| Modelo Abliterated | Derivado de Gemma 2, misma licencia |

---

## Runtime Options

| Opcion | Descripcion | Prioridad |
|--------|-------------|-----------|
| `LLAMA_TYPE=llama.cpp` | Runtime primario (default) | .env |
| `LLAMA_TYPE=ollama` | OLLAMA portable | .env |
| `TOOL_USE` | Activa tool calling (quita --no-jinja) | file > env > flag |
| `CURL_DOWNLOAD=true` | Usa curl para descargas | env > flag |
| `LLAMA_HOST` | IP del servidor (default: 127.0.0.1) | .env |
| `LLAMA_PORT` | Puerto del servidor (default: 8080) | .env |

---

## Roadmap

- [x] Selector interactivo de modelos con recomendacion por RAM
- [x] Menu interactivo (8 opciones)
- [x] Modo educativo con labs OWASP (ASI-01 operational)
- [x] RAM loader (tmpfs con symlinks para modelos)
- [x] Python portable (indygreg builds)
- [x] Download sin huggingface-cli (curl mode)
- [x] TOOL_USE mode (Jinja templates condicional)
- [x] Bootstrap completo (deps + modelos + Python + manifest)
- [x] Catalogo de 13 modelos con pipeline de evaluacion (2026)
- [x] Llama.cpp build b10936 portable (2026-09)
- [x] Qwen 3.5 4B/9B/2B, Gemma 4 E4B, Phi-4 Mini, SmolLM3 3B
- [x] Update check vs ultimo release b-tag de llama.cpp (sin repo externo)
- [x] Download con curl, resume y gate-check (zero-account, cero residuos)
- [ ] Streaming mejorado en chat UI
- [ ] CI/CD compilacion cruzada Windows/macOS/Linux
- [ ] Script de abliteration automatica
- [ ] Empaquetado ISO booteable
- [ ] Labs OWASP ASI-02 a ASI-10

---

<div align="center">

## Support

[![GitHub](https://img.shields.io/badge/GitHub-aentrepreneur-181717?style=for-the-badge&logo=github)](https://github.com/aentrepreneur)

---

*Creado por Angel Esquivel (CyberSecurity) — 2026*

</div>
