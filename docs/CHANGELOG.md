# CHANGELOG — uncensore-llm

Todas las versiones notables de este proyecto estan documentadas aqui.

El formato sigue [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
y este proyecto usa [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

---

## [2.1.0] — 2026-09-13 — Modelos 2026 + Llama.cpp b10936

### Added
- `scripts/models/catalog-2026.json` v2 — 13 modelos evaluados 2026, fuente unica
  de verdad con campos `arch`, `min_build`, `fallback`, `fallback_file`.
  10 se instalan por defecto (~27.7 GB); 3 bajo demanda.
- Modelos nuevos: **Qwen 3.5 4B/2B/9B** (256K contexto, 201 idiomas),
  **Gemma 4 E4B** (multimodal texto/imagen/audio, MMLU-Pro 69.4%),
  **Phi-4 Mini** (128K contexto), **SmolLM3 3B**, cuantizados Q4_K_M/Q4_0.
- `--install-all` en download-model.sh — instala el set completo por defecto.
- Gate-check con HEAD y `fallback` de repo ante repos gated (HTTP 401)
  sin cuentas ni tokens (zero-account).
- Resume parcial con verificacion de tamano (>=95% del esperado) en vez de
  tratar archivos incompletos como descargados.

### Changed
- **bin/**: llama.cpp actualizado de build 9591 a **b10936**
  (0.4.0-dev, commit 790cf51aa). Assets b-tag rolling de ggml-org.
- **download-model.sh**: reescrito, curl-only con `-C -` resume, sin
  huggingface-cli ni Python. Los mensajes de estado van a stderr; la URL solo
  a stdout (fix de captura con command substitution).
- **start.sh**: mapa de modelos actualizado a 10 instalados +
  recomendacion por RAM (>=16 → gemma-4-e4b, >=8 → qwen3.5-9b,
  >=4 → qwen3.5-4b, else qwen3.5-2b). Fix al mapping `qwen-2.5-7b` que
  apuntaba al archivo coder.
- **update.sh**: eliminado el repo remoto muerto; ahora compara el build local
  de llama.cpp contra el ultimo release b-tag rolling en GitHub (o la version
  estable semver v0.4.x si no hay b-tags). Zero dependencias de hosting.
- **validate-models.sh**: recomendaciones 2026 (Qwen 3.5 9B/4B, Gemma 4 E4B,
  Phi-4 Mini).
- **.model** default: `Phi-3.5-mini-instruct-Q4_K_M.gguf` → `Qwen3.5-4B-Q4_K_M.gguf`.
- **.manifest**: 85 archivos, excluye `models/`, `backups/` y `*.backup.*`.
- **.env.dist**: removidas variables del repo muerto; agregado `LLAMA_BUILD_CHECK`.

### Security
- Repos oficiales gated (Qwen/Google/Microsoft, `failspy` abliterated)
  declarados sin cuenta en catalogo: no se puede descargar sin token.

---

## [2.0.0] — 2026-07-30 — Fusion + Modo Educativo

### Added
- `menu.sh` — Entry point interactivo con 8 opciones:
  iniciar servidor, modo educativo, gestionar modelos, actualizar sistema,
  configuracion, carga a RAM, info sistema, salir
- `bootstrap.sh` — Verificacion de integridad: dependencias, modelos, Python, manifest
- `ram-loader.sh` — Carga proyecto a `/dev/shm/` (tmpfs) con symlinks a modelos
- `education/` — Modo educativo completo:
  - `setup.sh` — Entry point: lista labs, verifica servidor activo
  - `labs/01-basic-injection/` — Lab OWASP ASI-01: Prompt Injection (lab.sh + test.sh + lab.md)
  - `labs/template/` — Template reutilizable para nuevos labs
  - `curriculum/` — STUDY_PLAN.md, ROADMAP.md, TECHNICAL.md, CHANGELOG.md migrados desde learning-ai
  - `cert-tracker/` — Directorio para tracking de certificaciones
- `scripts/validate-models.sh` — Valida modelos instalados vs catalogo, recomienda por RAM
- `scripts/update.sh` — Auto-update contra manifest remoto (GitHub raw)
- `scripts/ollama-bundle.sh` — Descarga OLLAMA portable (opcional)
- `python/bootstrap.sh` — Python portable (indygreg builds, ~35 MB)
- `scripts/models/catalog-2026.json` — 18 modelos GGUF en 3 tiers
- `.env.dist` — Template de configuracion (LLAMA_HOST, PORT, TOOL_USE, etc.)
- `.manifest` — SHA256 de integridad (79 archivos)
- `docs/CHANGELOG.md` — Este archivo
- `docs/TECHNICAL.md` — Arquitectura tecnica del proyecto
- `docs/ADR/ADR-001-usb-first.md` — Decision de arquitectura USB-first
- `education/curriculum/ROADMAP.md` — Roadmap de fusion

### Changed
- **start.sh**: agregado `TOOL_USE` — flag `--tool-use` > env `TOOL_USE` > file `TOOL_USE`.
  Condicionalmente omite `--no-jinja` para activar tool calling via Jinja templates.
- **download-model.sh**: agregado `CURL_MODE` — flag `--curl` > env `CURL_DOWNLOAD=true`.
  Descarga directa desde HuggingFace via curl sin depender de huggingface-cli.
- **README.md**: rebuild completo con estandar visual AGENTS.md (shields.io, details/summary, icon tables, GitHub alerts, hero centrado)

### Fixed
- `validate-models.sh`: bug con `grep -c` + `set -e` en busqueda de catalogo. Corregido con `grep -cF` + `|| true`.
- `download-model.sh`: error handling mejorado para modo curl (verifica `curl` disponible).

---

## [1.0.0] — 2026-07-09 — Release Inicial

### Added
- `start.sh` — Lanzador con selector interactivo de modelos, deteccion de RAM,
  recomendacion automatica, auto-kill puerto 8080, cleanup con SIGINT
- `start.bat` — Lanzador Windows
- `scripts/download-model.sh` — Descarga modelos GGUF via huggingface-cli
- `bin/` — Binarios llama.cpp pre-compilados para Linux x86_64 (b4464)
- `models/` — 5 modelos GGUF:
  - Phi-3.5 Mini (Microsoft, 2.3 GB)
  - Phi-4 Mini (Microsoft, 2.4 GB)
  - Gemma 2 2B (Google, 1.6 GB)
  - Gemma 2 2B Abliterated (failspy, 1.6 GB)
  - Qwen 2.5 Coder 3B (Alibaba, 2.0 GB)
- `README.md` — Documentacion inicial: caracteristicas, uso, modelos, rendimiento, USB
- `.gitignore` — Excluye models/, .env, backups
- `.model` — Tracking de ultimo modelo seleccionado

### Design Decisions
- **llama.cpp como runtime primario**: zero-deps (solo `curl` en shell), soporte nativo GGUF,
  CPU-only optimizado con AVX2
- **--no-jinja por defecto**: compatibilidad maxima con modelos sin template de chat.
  TOOL_USE mode planeado para siguiente version.
- **USB-first**: estructura plana, sin symlinks internos, compatibilidad exFAT
- **Curated models**: solo modelos sub-4GB Q4_K_M para garantizar funcionalidad en CPU con 4-8 GB RAM

---

## Version Matrix

| Version | Fecha | Enfoque |
|---------|-------|---------|
| 1.0.0 | 2026-07-09 | Runtime base: llama.cpp, 5 modelos, USB-first |
| 2.0.0 | 2026-07-30 | Fusion learning-ai: menu, labs OWASP, educacion, manifest, TOOL_USE, curl download |
