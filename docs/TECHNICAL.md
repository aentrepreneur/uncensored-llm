<!-- ================================================================= -->
<!-- Development By Angel Esquivel (Automation) [UNCENSORE-LLM 2026] -->
<!-- ================================================================= -->
<!-- UNCENSORE-LLM - LLM uncensore - ram-loader y bootstrap -->

# TECHNICAL — uncensore-llm

Documento tecnico de arquitectura, stack y decisiones de diseno.

---

## Arquitectura General

```
┌─────────────────────────────────────────────────────────┐
│                    deploy.sh (CLI)                       │
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
| Runtime | llama.cpp | b11741 | CPU-only, zero-deps, GGUF nativo, API OpenAI |
| Shell | Bash | 5.0+ | Portable (Git Bash en Windows), universal |
| Python | Portable (indygreg) | 3.11+ | Auto-descargable, sin root, para labs educativos |
| OLLAMA | Binario portable | 0.32.5+ | Opcional, secundario a llama.cpp |
| Binarios | Pre-compilados C/C++ | b11741 | Linux x86_64 AVX2 (Tiger Lake+) |
| Modelos | GGUF Q4_K_M/IQ4_XS/Q8_0 | Varios | 10 uncensored en 4 perfiles (0.99-18.73 GB) |
| Descargas | curl + HuggingFace API | - | Curl zero-install, fallback a huggingface-cli |
| Integridad | SHA256 manifest | - | `.manifest` (excluye `models/`, `.git/`, `backups/`, `.manifest`) |

---

## Libreria Unica (`scripts/local_system.sh`)

Todos los entrypoints sourcean la misma libreria. Evita divergencia de
deteccion de hardware, rutas de catalogo y recomendacion de modelo.

| Dominio | Funciones | Responsabilidad |
|---------|-----------|-----------------|
| `host_*` | `host_ram_gb`, `host_disk_free_gb`, `host_cpu_*` | Hardware real |
| `path_*` | `path_models_dir`, `path_backend_dir` | Rutas canonicas |
| `catalog_*` | `catalog_get_field`, `catalog_list_ids`, `catalog_in_profile`, `catalog_id_for_file`, `catalog_recommend_for_ram`, `catalog_count_installed` | Lectura del catalogo v4 |
| `profile_*` | `profile_list`, `profile_budget`, `profile_resolve`, `profile_resolve_for`, `profile_recommend`, `profile_explain` | Perfiles RAM/disco |
| `backend_*` | `backend_find`, `backend_build`, `backend_min_build` | llama.cpp local |
| `integ_*` | `integ_manifest_check` | Integridad SHA256 |

CLI: `--host`, `--profile [name|--ram N --disk M]`, `--recommend`,
`--min-build`, `--where`, `--check`.

La libreria no activa `set -euo pipefail`: los entrypoints deciden su
propio modo de error.

## Perfiles de Instalacion

| Perfil | RAM | Disco | Modelos | Tamano |
|--------|-----|-------|---------|--------|
| `micro` | <=2 GB | <=3 GB | 1 | 0.99 GB |
| `lite` | <=4 GB | <=12 GB | 3 | 4.36 GB |
| `standard` | <=8 GB | <=25 GB | 7 | 18.18 GB |
| `full` | <=24 GB | <=55 GB | 10 | 49.92 GB |

`profile_recommend` mapea RAM del host al perfil mayor que cabe:
>=24 `full`, >=8 `standard`, >=4 `lite`, resto `micro`.
`install.sh` materializa el perfil; `download-model.sh <id>` sigue siendo
el descargador de bajo nivel.

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
Esto es necesario para modelos que no tienen template configurado.
Modelos modernos (Qwen 3.5/3.6, Gemma 4, Phi-4, SmolLM3) soportan templates
y funcionan mejor con Jinja activo.

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
    └── scripts/, edu/, deploy.sh → copia → /dev/shm/uncensore-llm/
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

### Tiers (set 100% uncensored)

| Tier | RAM requerida | Modelos |
|------|--------------|---------|
| ultra-light | 4 GB | Qwen 3.5 2B Uncensored |
| light | 8 GB | Qwen 3.5 4B Uncensored, Gemma 4 E4B Uncensored, Phi-4 Mini Uncensored, SmolLM3 3B Uncensored |
| medium | 12-16 GB | Qwen 3.5 9B Uncensored, Gemma 4 12B QAT Uncensored |
| heavy | 24 GB | Qwen 3.6 35B-A3B Uncensored (MoE, flagship) |

---

## Auto-Update System

```
update.sh
├── 1. Conecta a GitHub (check internet)
├── 2. Lee build local: bin/llama-server --version (bXXXXX)
├── 3. Obtiene ultimo release b-tag via GitHub API
├── 4. Compara: si local < remoto, sugiere descargar assets
└── 5. Reporta estado del catalogo local (modelos instalados)
```

### Manifest Format

```
<SHA256>  <path>
a1b2c3d4...  ./deploy.sh
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
LLAMA_HOST=127.0.0.1        # 127.0.0.1 = modo local (recomendado)
LLAMA_PORT=8080
LLAMA_API_KEY=              # vacio = sin auth (modo local). En bind externo: OBLIGATORIA

# Features
TOOL_USE=                   # true = activar tool calling (OFF si bind externo)
CURL_DOWNLOAD=              # true = descargar modelos con curl
RAM_LOAD=                   # true = cargar a tmpfs al iniciar

# Paths
MODELS_DIR=models/
PYTHON_DIR=python/linux/
```

Copiar desde `.env.dist` (o menu deploy.sh opcion 6, con read-back
verificado por `env_set` y validacion por `env_validate`). Postura de
exposicion: `docs/ADR/ADR-002-exposicion-remota.md` y `docs/GETTING-STARTED.md`
(tunel SSH recomendado; externo solo con Security Group TU_IP/32 + API key).

---

## Performance Characteristics

### Benchmarks (CPU: i7-1165G7, 8 threads, 7.6 GB RAM)

| Modelo | Carga (s) | Inferencia (tok/s) | RAM idle | RAM peak |
|--------|-----------|-------------------|----------|----------|
| Dolphin 1.5B | ~2 | 20-30 | 0.7 GB | 0.9 GB |
| Qwen 3.5 2B | ~3 | 15-28 | 0.9 GB | 1.1 GB |
| Dolphin 3B (coder) | ~3 | 12-20 | 1.2 GB | 1.4 GB |
| Phi-4 Mini | ~4 | 10-16 | 1.3 GB | 1.6 GB |
| Qwen 3.5 9B | ~6 | 5-10 | 3.0 GB | 3.6 GB |
| Qwen 3.6 35B-A3B (MoE) | ~10 | 15-25 | ~20 GB | ~22 GB |

> El flagship MoE requiere >=24 GB RAM: el host de desarrollo (7.6 GB) no lo
> ejecuta; la fila es referencia de hardware objetivo, no una medicion local.

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

<!-- End Development By Angel Esquivel (Automation) [UNCENSORE-LLM 2026] -->
