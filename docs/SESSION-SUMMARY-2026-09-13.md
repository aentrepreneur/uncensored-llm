# SESSION-SUMMARY — 2026-09-13 · Recuperacion + Modelos 2026 (v2.1.0)

## Diagnostico (causa raiz)
- `models/` vacio: los LLM se perdieron al cambiar de entorno.
- `bin/` sin permisos +x; llama.cpp viejo (build 9591).
- `update.sh` apuntaba a repos muertos (`angel-esquivel` 404, `aentrepreneur` 301).
- `start.sh` con bug de mapping: `qwen-2.5-7b` → archivo del coder.
- Catalog v1 obsoleto (modelos 2025, gated en HF).

## Solucion aplicada
- Investigacion de GGUFs 2026 accesibles sin cuenta/gate (zero-account): Qwen 3.5
  4B/2B/9B, Gemma 4 E4B (multimodal), Phi-4 Mini, SmolLM3 3B, Llama 3.2 1B,
  + 2 uncensored 4B/E4B (HauhauCS). 3 opcionales bajo demanda.
- `catalog-2026.json` v2: 13 modelos (10 instalados, 27.65 GB) con `arch`,
  `min_build`, `fallback`. Descargas completadas (28 GB total).
- `bin/` actualizado a llama.cpp **b10936** (0.4.0-dev). Backup en
  `backups/bin-pre-b10936/`.
- `download-model.sh` reescrito: curl-only, resume por tamano (>=95%),
  gate-check HEAD con fallback, mensajes a stderr (fix de captura de URL).
- `start.sh`: mapa nuevo de 10 modelos + recomendacion por RAM 2026.
- `update.sh`: check de build local vs release rolling b-tag / semver v0.4.x.
- `.model` default: `Qwen3.5-4B-Q4_K_M.gguf`.
- `.manifest`: 85 archivos, excluye `models/`, `backups/`, `*.backup.*`.

## Pendiente BLOCKED
- `bin/` a b10941 (rolling): opcional, decisivo el dueno cuando quiera.
- `--cleanup-project` de backups real al cerrar proyecto (se ejecuto para sesion).

## Estado de verificacion
- `1119` smoke: bootstrap OK (10 modelos) · validate-models OK (10 EN CATALOGO,
  recomendacion 2026) · update.sh OK (b10936 local, sugiere b10941) ·
  llama-server con Qwen 3.5 4B responde `/v1/chat/completions` y se apaga.
- `bash -n` en todos los scripts: 0 errores. JSON del catalogo: valido.