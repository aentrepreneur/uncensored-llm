<!-- ================================================================= -->
<!-- Development By Angel Esquivel (Automation) [UNCENSORE-LLM 2026] -->
<!-- ================================================================= -->
<!-- UNCENSORE-LLM - LLM uncensore - ram-loader y bootstrap -->

# CHANGELOG — uncensore-llm

Todas las versiones notables de este proyecto estan documentadas aqui.

El formato sigue [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
y este proyecto usa [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

---

## [2.3.0] — 2026-10-19 — Perfiles + Libreria Unica

### Added
- **Catalogo v4** (`scripts/catalog/catalog-2026.json`): bloque `profiles`
  con presupuestos de RAM/disco (`micro` 2/3, `lite` 4/12, `standard`
  8/25, `full` 24/55), campo `use` (general/coder/moe) y `uncensored`.
  Tier nuevo `micro` y 2 entradas Dolphin (1.5B general, 3B coder).
- **`scripts/local_system.sh`**: libreria unica + CLI offline
  (`--host`, `--profile`, `--recommend`, `--min-build`, `--where`,
  `--check`). Dominios `host_*`/`path_*`/`catalog_*`/`profile_*`/
  `backend_*`/`integ_*`; no usa `set -euo pipefail`.
- **`scripts/install.sh`**: instalador por perfiles (`--profile`,
  `--for-ram`/`--disk`, `--dry-run`, `--list`).
- **Runtime dual llama.cpp / OLLAMA**: seccion 5b en `local_system.sh`
  (`runtime_*`: tipo, modo system/project/auto, health, tag sugerido por
  RAM); `start.sh` con branch real para OLLAMA (lista tags del service,
  sugiere modelo segun RAM, `ollama pull` opt-in, seleccion persistente
  en `.model.ollama`).
- **`system.sh`** (raiz): entry point de sync/build hacia remote-lab
  (`--sync` sin modelos por defecto, `--build` con bootstrap + manifest
  base + sugerencia de perfil remotos, `--status` solo lectura,
  `--dry-run` sin red). Destino via `REMOTE_LAB_HOST`/`REMOTE_LAB_PATH`;
  sin credenciales en el repo (usa `sshpass -e`).

### Changed
- Los 8 scripts (`download-model`, `validate-models`, `update`, `start`,
  `menu`, `ram-loader`, `bootstrap`, `ollama-bundle`) consumen la
  libreria: se elimina la deteccion de RAM y el parseo de catalogo
  duplicados.
- Ruta del catalogo `scripts/models/` -> `scripts/catalog/`.
- Recomendacion de modelo unificada en `catalog_recommend_for_ram`:
  mayor modelo general/moe que cabe en RAM y disco. Cambio de
  comportamiento: `rec(8)=gemma-4-e4b` (antes 4B), `rec(4)=qwen3.5-2b`.
- `profile_recommend` alineado a los presupuestos: `full` desde 24 GB
  (antes 16 GB), `standard` desde 8 GB.
- `deploy.sh`: vista de catalogo via libreria; banner dinamico.
- `bootstrap.sh`: verifica el manifest con `integ_manifest_check`
  (antes solo comprobaba existencia).
- **Raiz consolidada en 2 entry points**: `deploy.sh` (menu
  interactivo dinamico, ex-menu.sh) + `deploy.conf` (identidad,
  defaults y guia de verbos embebida); `start.sh`, `bootstrap.sh` y
  `ram-loader.sh` movidos a `scripts/` (`git mv`).
- `deploy.sh`: selector de runtime real (llama.cpp / ollama-system /
  ollama-project) en opcion 6; si detecta `ollama.service` activo lo
  sugiere en la barra de estado sin cambiar el default. Opciones 1 y 5
  dinamicas (servidor y remote-lab como sub-menus con estado).
- `.env.dist`: `OLLAMA_MODE` (system|project|auto) documentado.
- `update.sh --apply`: auto-actualizacion de `bin/` (descarga asset
  ubuntu-x64 del b-tag, verificacion de build, backup previo en
  `backups/bin-b<old>-<ts>`, rollback si el binario nuevo no reporta
  el build esperado, rebuild de `.manifest`). Sin el flag, sigue siendo
  solo informativo.
- Banner del menu alineado y con version dinamica desde
  `project_version` (nueva en la libreria; `system.sh` la reutiliza).
- **Prerrequisitos oficiales** (dominio `prereq_*` en la lib): niveles
  `core`/`sync`/`apply`, reporte con paquete faltante y remedio por
  familia de SO (`apt`/`dnf`/`brew`; cubre ubuntu/ debian y rhel/fedora/
  rocky por familia). Gate BLOQUEANTE en menu opciones 1-5,
  `system.sh` (local), `download-model.sh`, `install.sh`, `start.sh`,
  `update.sh` (--apply exige `tar`) y `bootstrap.sh` (reporte en [1/4]).
- `prereq_backend`: `ldd bin/llama-server` detecta binario incompatible
  (glibc/SELinux) con mensaje claro.
- **menu.sh -> deploy.sh** (`git mv`) + **`deploy.conf`** nuevo
  (identidad, defaults remotos y guia de verbos embebida; deploy.sh lo
  sourcea antes de .env). Menú renumerado: 1 servidor dinamico,
  2 chat, 3 modelos (+enviar al remoto), 4 actualizar (+aplicar),
  5 remote-lab (sync/build/status), 6 config (+prerequisitos),
  7 RAM, 8 educación, 9 info (+prerequisitos+manifest).
- **Seguridad de exposicion**: dominio `env_*`/`expose_*` en la lib
  (`env_ensure`, `env_set` con read-back verificado, `env_validate`,
  `expose_status`/`expose_warn` con comando de tunel SSH listo). `LLAMA_API_KEY`
  end-to-end: `start.sh` pasa `--api-key` (los 3 modos de arranque),
  `llama-chat.sh` envia `Authorization: Bearer` y distingue 401 de caida,
  smoke **T6** exige 401 sin key / 200 con key. Menu opcion 6 ampliada
  (crear/editar `.env` + estado de exposicion); configuracion jamas silenciosa.
- **Integridad de descargas**: campo `sha256` por modelo en el catalogo v4,
  verificacion automatica tras cada descarga y en el branch "ya existe";
  preflight de disco en `download-model.sh` y plan acumulado en
  `install.sh` (margen 2 GB, fallo loud).
- `.env` protegido en `.gitignore` y en el exclude del mirror (jamás en git).
- `system.sh`: `ConnectTimeout=8` en ssh/rsync; `sync_excludes` ya no
  filtra con sed.
- **`scripts/llama-chat.sh`**: chat en consola estilo `ollama run`
  (`>>>`), historial de sesion, auto-arranca el server si esta inactivo;
  habla con el modelo YA cargado (sin recargar el GGUF).
- **`start.sh` daemon por defecto**: server en background (setsid, log
  en `logs/`, pidfile), devuelve el prompt; `--stop`, `--status`,
  `-f` para primer plano. Server con `--jinja --chat-template-kwargs
  {"enable_thinking":false}` (respuestas sin bloque `<think>`),
  fallback automatico a `--no-jinja` si el GGUF no trae template.
- **`system.sh --sync --model <id|auto>`**: lleva UN GGUF del local al
  remoto (auto = recomendado por RAM del remoto); skip si ya existe
  con el mismo tamano. `download-model.sh` ahora reporta
  "Reanudando desde X%".
- Menu: opcion 8 (chat CLI); opcion 1 deja de bloquear (daemon).

### Removed
- `education/cert-tracker/` (sin contenido activo).
- Duplicados `detect_ram`, `count_models`, `recommend_by_ram` y el
  parser ad-hoc de catalogo en el menu.
- 2 modelos censored huerfanos (2.91 GB): `qwen2.5-coder-3b-instruct`
  y `llama-3.2-1b-instruct`, eliminados tras verificar sus twins
  uncensored (`dolphin-3b-coder`, `dolphin-1.5b`).

### Verified
- `bash -n` OK en los 8 scripts; CLI de la libreria probada en host
  (7 GB RAM) y presupuestos simulados 2/4/8/12/16/24/32 GB.
- `install.sh --dry-run` para `micro` y `standard`; `--list` correcto.
- Dolphin 1.5B (986,051,648 B) y Dolphin 3B (2,104,936,192 B)
  descargados al 100% del `content-length`; cabecera `GGUF` valida.
  Catalogo 10/10 `installed`, 10 GGUF en disco.
- `.manifest` reconstruido (84 entradas, excluye `models/`, `.git/`,
  `backups/`, `.manifest`); `local_system.sh --check` reporta
  integridad OK.
- Sync real validado a remote-lab (`angelo@172.16.5.190`:
  `/home/angelo/uncensore-llm`, sin modelos): `--status` previo,
  `--sync` completo, `--build` con bootstrap + manifest base (84) +
  sugerencia de perfil (lite). Tag OLLAMA sugerido por la logica interna
  (`qwen2.5-coder:1.5b`) coincide con el ya presente en el service
  remoto (v0.24.0, enabled+active).
- `update.sh --apply` en remote-lab: `b11719 -> b11741` con backup y
  verificacion; segundo `--apply` reporta "al dia" (idempotente).
- **Homogeneidad multi-SO** (`tests/smoke.sh --keep`):
  - Ubuntu 24 (172.16.5.190): 8 pass / 0 fail / 0 skip.
  - RHEL 10.2 (172.16.5.115, glibc 2.39 + SELinux Enforcing):
    7 pass / 0 fail / 1 skip (T5 sin ollama).
  - Local dev: 7 pass / 0 fail / 1 skip.
- `--sync --model auto` al RHEL transferio el GGUF por LAN (el remoto
  tenia un parcial de 473 MB de una descarga HF interrumpida).
- Chat E2E en remote-lab: arranque daemon como usuario, respuestas sin
  `<think>`, segunda sesion instantanea, `--stop` limpio. Modelo remoto
  verificado (tamano exacto, GGUF valido) y reposeido angelo:angelo.
- **`tests/smoke.sh`**: suite de humo (T1 sintaxis, T2 auditoria,
  T3 ciclo de vida daemon, T4 chat sin `<think>`, T5 branch OLLAMA
  con skip automatico). `--keep` deja el server activo. Resultado:
  local 6 pass / 0 fail; remote-lab 7 pass / 0 fail.
- `start.sh`: liberacion de puerto incondicional (pidfile + fuser)
  antes de lanzar; evita HTTP server error por holders previos.

---

## [2.2.0] — 2026-10-18 — Set 100% Uncensored + Llama.cpp b11719

### Added
- **Set 100% uncensored** (catalog v3): 8 modelos sin filtro de rechazo,
  ~46.8 GB total, tiers RAM 4/8/12/16/24 GB. Politica `uncensored-first`:
  los modelos censored se retiran del set default.
- Modelos nuevos: **Qwen 3.5 2B/9B Uncensored** (HauhauCS Aggressive,
  Q4_K_M), **Phi-4 Mini Uncensored** (arzaan789, Q4_K_M, MIT),
  **SmolLM3 3B Uncensored** (BossDefender, Q8_0), **Gemma 4 12B QAT
  Uncensored** (HauhauCS Balanced, Q4_K_M, requiere b11719).
- **Flagship MoE: Qwen 3.6 35B-A3B Uncensored** (HauhauCS Aggressive,
  IQ4_XS, 18.73 GB) — 35B params, solo 3B activos, ~15-25 tok/s en CPU.
  Requiere llama.cpp b11719 (arquitectura `qwen35moe`). RAM >= 24 GB.
- Seccion `retired` en catalog v3 con trazabilidad de los 8 modelos
  censored retirados (fecha + razon).

### Changed
- `llama.cpp` **b10936 -> b11719** (0.7.0-dev, commit 5aeac37a3).
  Binarios CPU-only ubuntu-x64 pre-compilados (regla 8). Soporta las
  arquitecturas nuevas: `qwen35moe`, `gemma4` 12B.
- `catalog-2026.json` v2 -> **v3**: 8 entradas uncensored; tiers
  `ultra-light` (4GB), `light` (8GB), `medium` (16GB), `heavy` (24GB).
- `start.sh`: `recommend_model` con tiers 24/16/12/8/4 GB apuntando al
  set uncensored; default del proyecto ahora es el 4B Uncensored.
- `validate-models.sh`: tiers y recomendacion por RAM actualizados al
  set uncensored.
- `.model` default: `Qwen3.5-4B-Uncensored-HauhauCS-Aggressive-Q4_K_M.gguf`.

### Removed
- **8 modelos censored** (~21.6 GB) retirados del set default:
  Qwen 3.5 2B/4B/9B (censored), Gemma 4 E4B it (censored),
  Phi-4 Mini instruct (censored), SmolLM3 3B (censored),
  Qwen 2.5 Coder 3B (censored + obsoleto 2024, sin twin uncensored),
  Llama 3.2 1B (censored, el mas debil; uncensored gated).
  Los archivos se eliminan del disco solo cuando su twin uncensored
  esta verificado en `models/`.

### Verified
- Los 6 modelos nuevos descargados y verificados **byte-a-byte** contra
  `content-length` remoto (Hugging Face). Catalog v3: 8/8 `installed`.
- `llama-server` b11719 smoke en CPU: 2B/4B/9B Uncensored generan
  correctamente (`enable_thinking:false` para respuesta directa, sin
  refusal). El flagship MoE (`qwen35moe`) es reconocido por b11719
  (parseo de GGUF OK, cero errores de arquitectura); en el host de
  desarrollo (7.6 GB RAM) aborta por memoria y requiere RAM >= 24 GB.
- `update.sh` detecta build local b11719 y catalogo 8 modelos / 8
  instalados; rolling remoto disponible en b11724.

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


## v<major> - Releases anteriores archivadas (2026-10)


<!-- End Development By Angel Esquivel (Automation) [UNCENSORE-LLM 2026] -->
