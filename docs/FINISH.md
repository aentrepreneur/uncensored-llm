# FINISH — uncensore-llm (ancla de cierre DAME)

- **Fecha**: 2026-10-19
- **Version**: 2.3.0 (sin publicar: commit/tag pendientes de peticion explicita)
- **Arquetipo**: infra · **Estado**: VIVO-DONE
- **Rondas de verificacion**: 2 (smoke local + smoke en 3 hosts)
- **DONE tecnico**: tests/smoke.sh verde en local (7/0/1), ubuntu 24 (8/0/0),
  rhel 10.2 (7/0/1); manifest OK; reglas 14 y 16 PASS.
- **Deuda aceptada**: commit/tag NOT Solicitado; ollama ausente en RHEL (SKIP).

## Comandos canonico del dueno

```bash
./deploy.sh                              # TODO opera desde este menu
./system.sh --sync --model auto angelo@<ip>   # sync + modelo por LAN
./scripts/start.sh --stop                # detener daemon
./scripts/update.sh --apply              # auto-actualiza bin/
bash tests/smoke.sh --keep               # pruebas (deja server activo)
```

## Proxima sesion (hint)

1. Commit/tag v2.3.0 si el dueno lo pide (deuda [!] en SESSION-SUMMARY).
2. Validar homogeneidad en ubuntu 26 / rhel 9 cuando existan labs (diseno ya
   cubre por familia).
3. Opcional: streaming en llama-chat.sh (README roadmap).
4. DONE de este tramo: seguridad E2E validada en remote-lab (tunel SSH
   TUNEL-OK; modo expuesto 401/200 + WARN loud; restauracion a LOCAL).
   Referencias: docs/GETTING-STARTED.md, docs/ADR/ADR-002-exposicion-remota.md,
   tests/smoke.sh T6.
