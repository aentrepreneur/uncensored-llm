# CHANGELOG — uncensore-llm

Todas las versiones notables de este proyecto estan documentadas aqui.

El formato sigue [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
y este proyecto usa [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

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
