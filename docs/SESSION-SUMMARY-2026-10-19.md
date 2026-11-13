# SESSION-SUMMARY — 2026-10-19

## Diagnostico (causas raiz del tramo)

1. **Re-descarga de modelos ya existentes**: `--sync` excluye `models/` por
   diseno y `download-model.sh` solo salta archivos completos; no habia puente
   para reutilizar los GGUF del local en el remote-lab.
2. **"Cargando eterno" en CLI**: `start.sh` corrria llama-server en PRIMER
   PLANO y bloqueaba la terminal (web-ui wait) — el usuario esperaba una
   consola de chat estilo `ollama run`.
3. **Bloque `IMIZE` en respuestas** (sintoma "no funciona"): server con
   `--no-jinja` fijo ignoraba `chat_template_kwargs`; Qwen3.5 (thinking)
   antepone `IZE` crudo.
4. **Dilema ollama en remote-lab**: `ollama.service` instalado por un tercero
   en el lab ubuntu — resuelto con runtime dual y sugerencia no-invasiva.
5. **"Cuelgue" de 7 min en --sync al RHEL**: no era bug; era rsync transferiendo
   ~800 MB (GGUF parcial de 473 MB del remoto). El timeout del tool mato el
   transfer (rc=255).

## Solucion aplicada

- **`--sync --model <id|auto>`** en `system.sh` (auto = recomendado segun RAM
  del REMOTO via la lib); skip si el tamano remoto es identico.
- **`start.sh` daemon por defecto** (setsid + logs/ + pidfile, devuelve el
  prompt), `--stop`, `--status`, `-f` (foreground); liberacion de puerto
  INCONDICIONAL (pidfile + fuser + sleep 1) — fix de T3a del smoke.
- **`scripts/llama-chat.sh`**: REPL `>>>` con historial y auto-arranque;
  habla con el modelo YA cargado (instantaneo, sin duplicar RAM).
- **Server con `--jinja --chat-template-kwargs {"enable_thinking":false}`**
  (TOOL_USE -> thinking on) + fallback automatico a `--no-jinja`.
- **Runtime dual** llama.cpp | ollama-system | ollama-project (seccion 5b de
  la lib: `runtime_*`); el menu SUGIERE ollama-system si hay service activo,
  sin cambiar el default.
- **`menu.sh` -> `deploy.sh`** (`git mv`) + **`deploy.conf`** nuevo (identidad,
  defaults remotos y guia de verbos embebida; se sourcea antes de .env).
- **Prerrequisitos oficiales** (`prereq_*` en la lib): niveles core|sync|apply,
  reporte con paquete y remedio por familia (`apt`/`dnf`/`brew`), gate
  BLOQUEANTE en menu 1-5, system.sh, download-model, install, start,
  update --apply y bootstrap (reporte remoto).
- `system.sh --apply` de `update.sh`: b11719 -> b11741 con backup + rollback +
  manifest (idempotente). `ConnectTimeout=8` en ssh/rsync.
- Curriculum y labs saneados (refs a learning-ai/ollama/dstoken-docker eliminadas).

## Estado de verificacion (G4)

| Host | Suite (`tests/smoke.sh --keep`) |
|------|--------------------------------|
| Local dev (7 GB) | 7 pass / 0 fail / 1 skip |
| remote-lab-ubuntu 24 (172.16.5.190) | 8 pass / 0 fail / 0 skip |
| remote-lab-rhel 10.2 (172.16.5.115, glibc 2.39 + SELinux Enforcing) | 7 pass / 0 fail / 1 skip |

Regla 16: CHANGELOG 277 lineas (PASS; historico en `docs/history/`).
Regla 14: header+footer en los 5 archivos nuevos (PASS). Cero credenciales en
repo (PASS). Manifest local OK (84 entradas).

## Pendiente BLOCKED / deuda aceptada

- `[!]` Commit + tag `v2.3.0`: NO solicitado por el dueno; desbloqueo: peticion
  explicita (`git add -A && git commit && git tag v2.3.0` mas push si aplica).
- `[!]` Ollama en remote-lab-rhel10: sin service (T5 = SKIP por decision del
  dueno). Desbloqueo: `sudo dnf install ollama` y habilitar service.
- `[ ]` Upstream retiro assets vulkan del rolling; el pin b11719 local queda
  referenciado. Monitorear si el proyecto necesita Vulkan en el futuro.

## Ancla

Estado persistido en `docs/FINISH.md`. Memoria completa en MemPalace
(wing `uncensore-llm`).
