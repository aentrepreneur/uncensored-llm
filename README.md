# Uncensore-LLM: Modelos Locales GGUF desde USB
# =================================================================

Uncensore-LLM ejecuta modelos de lenguaje locales (GGUF) directamente desde
una memoria USB, sin instalación, sin GPU, sin internet.

## Caracteristicas

- 100% local: inferencia en CPU via llama.cpp, sin dependencias externas
- Zero-install: descarga el proyecto, ejecuta `./start.sh`, usa el modelo
- Portable: funciona desde USB (exFAT), SSD externo o disco local
- Multiplataforma: Linux (x86_64), macOS (Intel y Apple Silicon), Windows (x64)
- Multi-modelo: 5 modelos incluidos, selector interactivo por RAM disponible
- Interfaz web: servidor HTTP con chat UI en http://localhost:8080
- API OpenAI-compatible: http://localhost:8080/v1
- Privado: cero datos salen de tu maquina. Sin telemetria, sin cuentas

## Requisitos minimos

| Componente | Minimo | Recomendado |
|------------|--------|-------------|
| RAM        | 4 GB   | 8 GB        |
| USB        | 8 GB   | 32 GB+      |
| CPU        | x86_64 con AVX2 (2013+) | Cualquier moderno |
| SO         | Linux, macOS, Windows | - |

## Uso rápido

```bash
./start.sh
```

Esto:
1. Detecta RAM disponible
2. Lista los modelos `.gguf` en `models/`
3. Recomienda el modelo optimo segun RAM
4. Muestra menu interactivo con descripciones y fichas tecnicas
5. Lanza llama-server en http://localhost:8080

**Windows:** Doble-click en `start.bat`

```bash
# Probar la API
curl http://localhost:8080/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{"model":"gguf","messages":[{"role":"user","content":"Hola"}],"max_tokens":50}'
```

## Modelos incluidos

Los 5 modelos en `models/` suman ~9.8 GB. El selector recomienda segun RAM:

| Modelo | Archivo | Tamano | RAM | Contexto | Perfil |
|--------|---------|--------|-----|----------|--------|
| Gemma 2 2B (Google) | `gemma-2-2b.gguf` | 1.6 GB | 4 GB | 8K | Ligero, oficial |
| Gemma 2 2B Abliterated | `gemma-2-2b-it-abliterated-Q4_K_M.gguf` | 1.6 GB | 4 GB | 8K | Sin censura |
| Phi-3.5 Mini (Microsoft) | `Phi-3.5-mini-instruct-Q4_K_M.gguf` | 2.3 GB | 4 GB | 4K | Solido generalista |
| Qwen Coder 3B (Alibaba) | `qwen2.5-coder-3b-instruct-q4_k_m.gguf` | 2.0 GB | 4 GB | 32K | Código y debug |
| Phi-4 Mini (Microsoft) | `Phi-4-mini-instruct-Q4_K_M.gguf` | 2.4 GB | 4 GB | 128K | Calidad/precio |

```bash
# Descargar modelos adicionales
./scripts/download-model.sh
```

## Estructura del proyecto

```
uncensore-llm/
├── models/             Modelos GGUF (descargar o colocar aqui)
│   └── *.gguf          Quantizados Q4_K_M (~1.6-2.4 GB c/u)
├── bin/                Binarios llama.cpp pre-compilados
│   ├── llama-server    Servidor HTTP con chat UI
│   ├── llama-cli       Interfaz de línea de comandos
│   ├── llama-bench     Benchmark de rendimiento
│   └── lib*.so         Librerias compartidas
├── scripts/
│   └── download-model.sh  Descarga modelos adicionales
├── start.sh            Lanzador Linux/macOS
├── start.bat           Lanzador Windows
└── README.md           Este archivo
```

## Como funciona

Usa [llama.cpp](https://github.com/ggml-org/llama.cpp) como motor de
inferencia. `llama-server` es un ejecutable unico en C/C++ que:

1. Carga el modelo GGUF en RAM
2. Inicia un servidor HTTP compatible con API de OpenAI
3. Sirve una interfaz web en http://localhost:8080

No hay Python, Node, Docker ni GPU. Todo son binarios pre-compilados en `bin/`.

start.sh gestiona todo: detecta RAM, lista modelos disponibles, recomienda
segun recursos, lanza el servidor con `--no-jinja` y auto-kill del puerto
8080, y muestra las URLs de acceso.

## USB

El proyecto esta disenado para ejecutarse directamente desde una memoria USB.
Compatible con USB 3.0+ formateada como exFAT (recomendado) o ext4.

### Requisitos de almacenamiento

| Perfil | Contenido | USB minima |
|--------|-----------|------------|
| **Minimo** | Gemma 2 2B + binarios | 4 GB |
| **Liviano** | Gemma 2 2B + Phi-4 Mini + binarios | 8 GB |
| **Completo** | Todos los modelos + binarios | **16 GB** |

### Preparar USB

```bash
# 1. Copiar el proyecto completo a la USB
cp -r /opt/uncensore-llm /media/usb/

# 2. Ejecutar desde la USB
/media/usb/uncensore-llm/start.sh
```

### Notas para USB

- **exFAT**: formatear como exFAT para compatibilidad Windows/Linux/macOS y
  soporte de archivos de hasta 2.4 GB. Los binarios en `bin/` ya fueron
  preparados sin symlinks para compatibilidad exFAT.
- **Permisos**: en exFAT no se preservan permisos Unix. `start.sh` requiere
  `+x`. Si usas Linux, ejecuta `chmod +x start.sh bin/llama-server` despues
  de copiar. En Windows/macOS los permisos se manejan automaticamente.
- **Modelos adicionales**: descarga modelos extras con `scripts/download-model.sh`
  ANTES de copiar a la USB. No requiere internet en la maquina destino.
- **Sesion persistente**: la ultima seleccion de modelo se guarda en `.model`.
  En USB solo-lectura, el selector aparecera cada vez.

## Rendimiento

En CPU moderna con AVX2:

| Modelo | Tokens/s | RAM en uso |
|--------|----------|------------|
| Gemma 2 2B | ~15-25 | ~1 GB |
| Phi-3.5 Mini | ~12-20 | ~1.5 GB |
| Qwen Coder 3B | ~10-18 | ~1.3 GB |
| Phi-4 Mini | ~10-18 | ~1.6 GB |

Contexto por defecto: 4096 tokens (configurable con `-c` en start.sh).

## API OpenAI-compatible

```python
import openai
client = openai.OpenAI(base_url="http://localhost:8080/v1", api_key="not-needed")
response = client.chat.completions.create(
    model="gguf",
    messages=[{"role": "user", "content": "Hola"}]
)
print(response.choices[0].message.content)
```

Cualquier herramienta que soporte API de OpenAI puede apuntar a
`http://localhost:8080/v1` y usar el modelo local.

## Roadmap

- [x] Selector interactivo de modelos con recomendación por RAM
- [x] Auto-kill puerto 8080 en lanzamiento
- [x] Modelos adicionales via download-model.sh
- [ ] Web UI mejorada (streaming, markdown, historial)
- [ ] Compilacion cruzada CI/CD para Windows, macOS, Linux
- [ ] Script de abliteration automática para nuevos modelos
- [ ] Empaquetado como ISO booteable

## Licencias

- **Este proyecto**: MIT License
- **llama.cpp**: MIT License
- **Modelos**: Google Gemma License, Microsoft MIT, Alibaba Tongyi Qianwen LICENSE
- **Modelo Abliterated**: Derivado de Gemma 2, misma licencia

Ver archivo `LICENSE` para detalles.

## Disclaimer

Este proyecto se proporciona con fines educativos y de investigación.
Algunos modelos pueden generar contenido sin filtros de seguridad.
El uso responsable es responsabilidad del usuario final.

---

Creado por Angel Esquivel (CyberSecurity) - [aentrepreneur](https://github.com/aentrepreneur)

#End Development By Angel Esquivel (CyberSecurity) [uncensore-llm 2026]
