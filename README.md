<!-- ================================================================= -->
<!-- Development By Angel Esquivel (Automation) [UNCENSORE-LLM 2026] -->
<!-- ================================================================= -->
<!-- UNCENSORE-LLM - LLM uncensore - ram-loader y bootstrap -->

<div align="center">

# 🧠 uncensore-llm

**LLMs locales portables desde USB — zero-install, sin GPU, sin internet**

[![Version](https://img.shields.io/badge/version-2.3.0-blue?style=flat-square)](https://github.com/aentrepreneur/uncensore-llm)
[![License](https://img.shields.io/badge/license-MIT-green?style=flat-square)](LICENSE)
[![Bash](https://img.shields.io/badge/bash-5.0%2B-4EAA25?style=flat-square&logo=gnubash)](scripts/)
[![Python](https://img.shields.io/badge/python-3.11%2B-blue?style=flat-square&logo=python)](python/)
[![Platform](https://img.shields.io/badge/platform-Linux%20%7C%20macOS%20%7C%20Windows-lightgrey?style=flat-square)]()

</div>

---

## Que es esto

Conectas una USB, ejecutas `./deploy.sh` y tienes un LLM funcionando 100% local.
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
| Multi-modelo | 10 modelos GGUF uncensored (~50 GB), 4 perfiles por RAM/disco |
| Perfiles | `micro` / `lite` / `standard` / `full` con instalador `scripts/install.sh` |
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

> [!TIP]
> Guia completa desde cero (USB / VM / EC2 con seguridad): `docs/GETTING-STARTED.md`.

```bash
# 1. Clona o copia a USB
cp -r uncensore-llm /media/usb/

# 2. Ejecuta el menu
./deploy.sh

# 3. Opcion 1 → Iniciar servidor (selecciona modelo, lanza en :8080)
# 4. Abri http://localhost:8080 en el navegador
```

> [!TIP]
> **Modo educativo:** `./deploy.sh` → opcion 8. Labs OWASP funcionando contra tu LLM local.

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
<summary>📋 Menu completo (10 opciones)</summary>

| Opcion | Funcion |
|--------|---------|
| 1 | Servidor (dinamico: iniciar daemon / estado / detener / chat) |
| 2 | Chat CLI con el modelo cargado (estilo `ollama run`) |
| 3 | Modelos (validar, descargar, catalogo, enviar al remoto) |
| 4 | Actualizar sistema (verificar / aplicar bin/ nuevo) |
| 5 | Remote-lab (sync, sync+modelo, build, status) |
| 6 | Configuracion (runtime, TOOL_USE, .env, prerrequisitos) |
| 7 | Cargar en RAM (tmpfs, permite desconectar USB) |
| 8 | Educacion (labs OWASP + curriculum) |
| 9 | Info del sistema (host + prerrequisitos + manifest) |

| 0 | Salir |

</details>

---

## Modelos incluidos (100% uncensored)

| Modelo | Archivo | Tamano | RAM | Contexto | Perfil |
|--------|---------|--------|-----|----------|--------|
| Dolphin 3.0 Qwen2.5 1.5B Uncensored | `Dolphin3.0-Qwen2.5-1.5B-Q4_K_M.gguf` | 0.99 GB | 2 GB | 32K | **Piso micro** |
| Qwen 3.5 2B Uncensored | `Qwen3.5-2B-Uncensored-HauhauCS-Aggressive-Q4_K_M.gguf` | 1.27 GB | 4 GB | 256K | Tier minimo |
| Dolphin 3.0 Qwen2.5 3B Uncensored (Coder) | `Dolphin3.0-Qwen2.5-3b.Q4_K_M.gguf` | 2.10 GB | 4 GB | 32K | Coder / pentest |
| Qwen 3.5 4B Uncensored | `Qwen3.5-4B-Uncensored-HauhauCS-Aggressive-Q4_K_M.gguf` | 2.71 GB | 8 GB | 256K | **Default del proyecto** |
| Gemma 4 E4B Uncensored | `Gemma-4-E4B-Uncensored-HauhauCS-Aggressive-Q4_K_M.gguf` | 5.34 GB | 8 GB | 128K | Multimodal, sin censura |
| Phi-4 Mini Uncensored | `phi_4_mini_uncensored.Q4_K_M.gguf` | 2.49 GB | 8 GB | 128K | Razonamiento MIT |
| SmolLM3 3B Uncensored | `SmolLM3-3B-FineTuned-uncensored-Q8_0.gguf` | 3.28 GB | 8 GB | Q8_0 | Calidad maxima 3B |
| Qwen 3.5 9B Uncensored | `Qwen3.5-9B-Uncensored-HauhauCS-Aggressive-Q4_K_M.gguf` | 5.63 GB | 12 GB | 256K | Alta calidad denso |
| Gemma 4 12B QAT Uncensored | `Gemma4-12B-QAT-Uncensored-HauhauCS-Balanced-Q4_K_M.gguf` | 7.38 GB | 16 GB | 128K | Techo denso 16GB |
| Qwen 3.6 35B-A3B Uncensored (MoE) | `Qwen3.6-35B-A3B-Uncensored-HauhauCS-Aggressive-IQ4_XS.gguf` | 18.73 GB | 24 GB | MoE 3B activos | **Flagship**, ~15-25 tok/s CPU |

> [!NOTE]
> **10 modelos en catalogo** (`scripts/catalog/catalog-2026.json`) — set 100% uncensored,
> politica `uncensored-first`. Tiers RAM: 2 / 4 / 8 / 12 / 16 / 24 GB.
> Total ~49.9 GB. Ejecuta `./scripts/download-model.sh --list` para verlos todos.
> Requiere llama.cpp **b11741** (soporta qwen35moe y gemma4 12B).

<details>
<summary>📦 Perfiles de instalacion</summary>

| Perfil | RAM | Disco | Modelos | Tamano |
|--------|-----|-------|---------|--------|
| `micro` | <=2 GB | <=3 GB | Dolphin 1.5B | 0.99 GB |
| `lite` | <=4 GB | <=12 GB | + Qwen3.5 2B, Dolphin 3B coder | 4.36 GB |
| `standard` | <=8 GB | <=25 GB | + Qwen3.5 4B, Phi-4 Mini, SmolLM3, Gemma4 E4B | 18.18 GB |
| `full` | <=24 GB | <=55 GB | + Qwen3.5 9B, Gemma4 12B QAT, Qwen3.6 35B-A3B (MoE) | 49.92 GB |

```bash
./scripts/install.sh --list                  # ver perfiles
./scripts/install.sh --profile standard      # instalar perfil
./scripts/install.sh --for-ram 16 --dry-run  # simular en otro host
```

</details>

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
| `deploy.sh` | Menu dinamico: servidor, chat, modelos, remoto, prerequisitos |
| `scripts/start.sh` | Server en background (daemon default) + `--stop`/`--status`/`-f` |
| `scripts/llama-chat.sh` | Chat en consola via API (el `ollama run` del proyecto) |
| `system.sh` | Entry point de sync/build: replica el proyecto en un remote-lab via rsync/ssh |
| `scripts/bootstrap.sh` | Verifica integridad: deps, modelos, Python, manifest |
| `scripts/ram-loader.sh` | Copia proyecto a `/dev/shm/` y crea symlinks a modelos |
| `.manifest` | SHA256 de integridad (excluye models/, .git/, backups/) |

No hay Python, Node, Docker ni GPU requeridos. Todo son binarios C/C++ pre-compilados.

### Runtime dual: llama.cpp u OLLAMA

El runtime se elige en `deploy.sh` → `6) Configuracion` → `2) Runtime`:

| Runtime | Fuente de modelos | URL |
|---------|-------------------|-----|
| `llama.cpp` (default) | GGUF de `models/` (catalogo) | `http://localhost:8080` |
| `ollama-system` | Service del host (`ollama.service`, puerto 11434) | `http://127.0.0.1:11434` |
| `ollama-project` | Binario portable de `bin/ollama/` | `http://127.0.0.1:11434` |

- Si detecta `ollama.service` activo, el menu **sugiere `ollama-system`** (no interfiere con servicios existentes).
- En modo OLLAMA, `start.sh` lista los tags del service, sugiere un modelo segun RAM (`qwen2.5-coder:1.5b` para 4+ GB) y puede hacer `ollama pull` del sugerido.
- En modo llama.cpp nada cambia respecto a la v2.2.

### Remote-lab (sync a otro host)

```bash
# Estado remoto (solo lectura)
SSHPASS='...' sshpass -e ./system.sh --status user@host

# Copiar proyecto SIN modelos (models/, .env y backups quedan fuera)
SSHPASS='...' sshpass -e ./system.sh --sync user@host

# Bootstrap remoto: checks + manifest base + sugerencia de perfil
SSHPASS='...' sshpass -e ./system.sh --build user@host
```

Destino por defecto configurable en `deploy.conf` (`REMOTE_LAB_HOST` y
`REMOTE_PATH`, ruta RELATIVA a la home del remoto). Sin credenciales en el
repo: usa `sshpass -e`. Guia completa: `docs/GETTING-STARTED.md`.

<details>
<summary>📂 Estructura completa</summary>

```
uncensore-llm/
├── deploy.sh                 Entry point interactivo (todo opera aqui)
├── deploy.conf               Identidad + defaults + guia de verbos
├── system.sh                Entry point sync/build hacia remote-lab
├── start.bat                Lanzador Windows
├── .env.dist                Template de configuracion
├── .manifest                SHA256 de integridad (excluye models/)
├── .model                   Ultimo modelo seleccionado
├── TOOL_USE                 Flag para activar tool calling
│
├── bin/                     Binarios llama.cpp (build b11741)
│   ├── llama-server         Servidor HTTP principal
│   ├── llama-cli            CLI interactivo
│   └── lib*.so              Librerias compartidas
│
├── models/                  Modelos GGUF (set uncensored, ~50 GB total)
│   └── *.gguf               Quantizados Q4_K_M/IQ4_XS/Q8_0
│
├── scripts/
│   ├── local_system.sh      Libreria host/catalogo/perfil/backend/runtime + CLI
│   ├── install.sh           Instalador por perfiles (micro/lite/standard/full)
│   ├── start.sh             Lanza runtime (llama.cpp :8080 u OLLAMA :11434)
│   ├── bootstrap.sh         Verifica deps, modelos, Python, manifest
│   ├── ram-loader.sh        Carga proyecto a tmpfs (/dev/shm/)
│   ├── download-model.sh    Descarga modelos (--curl mode sin deps)
│   ├── validate-models.sh   Valida vs catalogo, sugiere por perfil
│   ├── update.sh            Compara build local de llama.cpp vs GitHub
│   ├── ollama-bundle.sh     OLLAMA portable opcional
│   └── catalog/
│       └── catalog-2026.json  10 modelos en 4 perfiles
│
├── python/
│   └── bootstrap.sh         Python portable (indygreg, ~35 MB)
│
├── education/
│   ├── setup.sh             Entry point modo educativo
│   ├── labs/                Labs OWASP practicos
│   │   ├── 01-basic-injection/  ASI-01: Prompt Injection
│   │   └── template/            Template para nuevos labs
│   └── curriculum/           Plan de estudio + roadmap
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
| **RAM** | 2 GB (perfil micro) | 8 GB+ (standard) |
| **USB** | 3 GB (perfil micro) | 32 GB+ (full) |
| **CPU** | x86_64 con AVX2 (2013+) | Tiger Lake+ |
| **SO** | Linux, macOS, Windows | Linux (Ubuntu 24.04+) |

---

## Rendimiento

En CPU i7-1165G7 (Tiger Lake, 2021) con AVX2, modelo Q4_K_M, 4096 contexto:

| Modelo | Tokens/s | RAM uso |
|--------|----------|---------|
| Dolphin 1.5B | ~20-30 | ~0.7 GB |
| Qwen 3.5 2B | ~15-28 | ~0.9 GB |
| Dolphin 3B (coder) / SmolLM3 3B | ~12-20 | ~1.4 GB |
| Qwen 3.5 4B / Phi-4 Mini | ~10-18 | ~1.6 GB |
| Qwen 3.5 9B / Gemma 4 E4B | ~5-10 | ~3.0 GB |
| Qwen 3.6 35B-A3B (MoE) | ~15-25 | ~20 GB |

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
/media/usb/uncensore-llm/deploy.sh

# Windows (doble-click):
/media/usb/uncensore-llm/start.bat
```

### Cargar en RAM y desconectar USB

```bash
./deploy.sh    # → opcion 7
cd /dev/shm/uncensore-llm && ./deploy.sh
# USB ya puede desconectarse. Modelos se quedan en USB.
```

<details>
<summary>📋 Requisitos de almacenamiento por perfil</summary>

| Perfil | Contenido | USB minima |
|--------|-----------|------------|
| `micro` | Dolphin 1.5B + binarios | 3 GB |
| `lite` | + Qwen3.5 2B + Dolphin 3B coder | 12 GB |
| `standard` | + Qwen3.5 4B, Phi-4 Mini, SmolLM3, Gemma4 E4B | 25 GB |
| `full` | + Qwen3.5 9B, Gemma4 12B QAT, Qwen3.6 35B-A3B (MoE) | **55 GB** |

</details>

<details>
<summary>📋 Notas tecnicas para USB</summary>

- **exFAT**: Recomendado para compatibilidad Windows/Linux/macOS. Soporta archivos >4 GB.
- **Permisos**: En exFAT no se preservan `+x`. Ejecutar `chmod +x deploy.sh bin/llama-server` al copiar.
- **Modelos**: Descargar modelos adicionales ANTES de copiar a USB con `./scripts/download-model.sh --list`.
- **Sesion**: Seleccion de modelo se guarda en `.model`. En USB solo-lectura, el selector aparece cada vez.

</details>

---

## Licencias

| Componente | Licencia |
|------------|----------|
| Este proyecto | MIT |
| llama.cpp | MIT |
| Modelos GGUF | Licencia del modelo base (ver card en Hugging Face) |

---

## Runtime Options

| Opcion | Descripcion | Prioridad |
|--------|-------------|-----------|
| `LLAMA_TYPE=llama.cpp` | Runtime primario (default) | .env |
| `LLAMA_TYPE=ollama` | Runtime OLLAMA (system o project) | .env |
| `OLLAMA_MODE=auto` | system / project / auto (default) | .env |
| `REMOTE_LAB_HOST` | Destino default de `system.sh` | env |
| `REMOTE_LAB_PATH` | Ruta destino en el remoto (default `/opt/uncensore-llm`) | env |
| `TOOL_USE` | Activa tool calling (quita --no-jinja) | file > env > flag |
| `CURL_DOWNLOAD=true` | Usa curl para descargas | env > flag |
| `LLAMA_API_KEY` | Auth del server (`--api-key`; /v1 exige Bearer) | .env |
| `LLAMA_HOST` | IP del servidor (default: 127.0.0.1) | .env |
| `LLAMA_PORT` | Puerto del servidor (default: 8080) | .env |

---

## Roadmap

- [x] Selector interactivo de modelos con recomendacion por RAM
- [x] Menu interactivo (10 opciones, deploy.sh + deploy.conf)
- [x] Modo educativo con labs OWASP (ASI-01 operational)
- [x] RAM loader (tmpfs con symlinks para modelos)
- [x] Python portable (indygreg builds)
- [x] Download sin huggingface-cli (curl mode)
- [x] TOOL_USE mode (Jinja templates condicional)
- [x] Bootstrap completo (deps + modelos + Python + manifest)
- [x] Catalogo v4 de 10 modelos con 4 perfiles (micro/lite/standard/full)
- [x] Libreria unica `scripts/local_system.sh` (host/catalogo/perfil/backend/CLI)
- [x] Instalador por perfiles `scripts/install.sh` (incluye `--dry-run`)
- [x] Llama.cpp build b11741 portable (2026-10)
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

<!-- End Development By Angel Esquivel (Automation) [UNCENSORE-LLM 2026] -->
