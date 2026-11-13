<!-- ================================================================= -->
<!-- Development By Angel Esquivel (Automation) [UNCENSORE-LLM 2026] -->
<!-- ================================================================= -->
<!-- UNCENSORE-LLM - LLM uncensore - ram-loader y bootstrap -->

# UNCENSORE-LLM — Guía de arranque desde cero (GETTING-STARTED)

Todo el sistema se opera desde el menú `./deploy.sh`. Esta guía cubre las
tres rutas de despliegue: USB, VM Linux y EC2. Al final: verificación
post-arranque y troubleshooting.

Distribución de versiones al cierre de este doc: bin/ incluye llama.cpp
**b11741** (Linux x86_64, GNU 11.4); modelos NO vienen en el repo
(son ~47 GB); el catálogo es `scripts/catalog/catalog-2026.json` con
verificación sha256 en cada descarga.

## 1. Requisitos rápidos

1.1 CPU x86_64 con AVX2 (2013+). Sin GPU; inferencia CPU-only.
1.2 RAM mínima 2 GB (perfil micro); 8+ GB recomendado (standard).
1.3 Disco según perfil: micro 3 GB · lite 12 GB · standard 25 GB · full 55 GB.
1.4 Dependencias base: `curl`, `sha256sum`, `basename`, `dirname`, `readlink`.

    # El gate verifica automáticamente y muestra el remedio por familia:
    apt  install -y curl coreutils util-linux    # ubuntu / debian
    dnf  install -y curl coreutils               # rhel / fedora / rocky
    brew install curl coreutils                  # macOS

Sin Python: el núcleo (bash + curl) funciona de cero.

## 2. Obtención del proyecto

2.1 Vía git (recomendada en VM/EC2; viene con bin/ incluido):

    git clone https://github.com/aentrepreneur/uncensored-llm-local.git uncensore-llm
    cd uncensore-llm

2.2 Vía USB literal (ver 3).
2.3 Los modelos ni vienen en el clone; van por perfil (ver 2.4 o 3).

## 3. Ruta A — USB (extraíble)

3.1 Copiar el proyecto CON modelos (si existen) a la USB exFAT:

    cp -r uncensore-llm /media/usb/

3.2 Recuperar permisos de ejecución (exFAT/NTFS los pierden):

    chmod +x /media/usb/uncensore-llm/deploy.sh
    chmod +x /media/usb/uncensore-llm/bin/llama-server
    chmod +x /media/usb/uncensore-llm/scripts/*.sh

3.3 Arrancar desde la USB:

    /media/usb/uncensore-llm/deploy.sh        # Linux/macOS
    start.bat                                  # Windows (doble-click)

3.4 Descargar modelos ANTES de copiar (para no gastar internet en destino):

    ./deploy.sh   # opcion 3 Modelos → descargar/instalar perfil

3.5 Cargar a RAM y desconectar USB (opcion 7 del menú):

    ./deploy.sh     # opcion 7
    cd /dev/shm/uncensore-llm && ./deploy.sh
    # La USB ya puede desconectarse; los GGUF quedan también en /dev/shm

Modelos: seleccion persistente en `.model` (en USB solo-lectura el selector
aparece cada vez — comportamiento por diseño).

## 4. Ruta B — VM Linux (remote-lab o KVM/VMware)

4.1 Prerrequisitos de la VM: SSH accesible; el usuario destino es un
    usuario normal (no root): el proyecto vive en su home.
4.2 Sincronizar el proyecto sin modelos desde tu PC:

    SSHPASS='<password>' sshpass -e ./system.sh --sync usuario@IP_VM

4.3 Bootstrap remoto (checks + manifest + perfil sugerido):

    SSHPASS='<password>' sshpass -e ./system.sh --build usuario@IP_VM

4.4 Modelo único por LAN (recomendado; evita re-descargas de internet):

    SSHPASS='<password>' sshpass -e ./system.sh --sync --model auto usuario@IP_VM

4.5 En la VM (por SSH): `cd uncensore-llm && ./deploy.sh` → opcion 1.
4.6 Consumo: túnel SSH (5.1) o modo externo (5.2).

Nunca copiar `.env` remoto al PC ni al revés: cada host tiene el suyo.
Credenciales: usar `sshpass -e` (env SSHPASS) jamas en la linea de comandos
ni en el repo (reglas 2/3 del proyecto).

## 5. Ruta C — EC2 (AWS) con seguridad maxima

5.1 Aprovisionamiento de instancia (una vez, desde tu equipo):

    Linux, x86_64, t3.large (8 GB RAM) o superior para perfil standard.
    Disco >= 25 GB (perfil full requiere >= 55 GB).
    Security Group inicial: solo puerto 22 (SSH). El 8080 JAMAS se abre a
    0.0.0.0/0 (ver 5.4).

5.2 SSH key-only (hardening minimo obligatorio):

    ssh-keygen -t ed25519 -f ~/.ssh/uncensore-llm_lab
    # copiar la llave a la instancia y en la instancia:
    sudo sshd -T | grep -E 'PasswordAuthentication|PermitRootLogin'
    # debe quedar: PasswordAuthentication no  ·  PermitRootLogin no

5.3 Proyecto en la instancia: repetir 4.2-4.5 apuntando a la IP publica.

5.4 Modo EXTERNO (decidir entre este y 5.5 tunel):

    aws ec2 authorize-security-group-ingress --group-id <SG-ID> \
      --protocol tcp --port 8080 --cidr TU_IP/32

    TU_IP la ves con: curl -s ifconfig.me   (desde tu laptop)
    Consola AWS → EC2 → Security Groups → Edit inbound rules →
    Type Custom TCP, Port 8080, Source TU_IP/32.

    En la instancia, menu opcion 6 → "Crear/editar .env":
      LLAMA_HOST=0.0.0.0
      LLAMA_API_KEY=<clave larga, >= 8 chars>
    El menú advierte loud si el bind es externo sin key. TOOL_USE = OFF.

5.5 Túnel SSH (RECOMENDADO; no abre 8080):

    # SG no cambia. En la instancia, .env queda con default 127.0.0.1.
    ssh -L 8080:127.0.0.1:8080 ubuntu@TU_SERVER_IP
    # luego en tu laptop: http://127.0.0.1:8080

5.6 Extras de hardening sugeridos en EC2 (fuera del proyecto):

    1) IMDSv2 (metadata): `aws ec2 modify-instance-metadata-options --http-tokens required ...`
    2) fail2ban; firewall: `ufw default deny incoming && ufw allow OpenSSH`
    3) usuario no-root para la app; `sudo` con contraseña.
    4) Volumen EBS cifrado si vas a residir datos sensibles.

## 6. Verificación post-despliegue

6.1 Health (siempre abierto, incluso con API key):

    curl -sf http://127.0.0.1:8080/health && echo OK

6.2 API protegida (modo key activa):

    curl -s -o /dev/null -w '%{http_code}\n' http://127.0.0.1:8080/v1/models
    # con key activa y sin Bearer === 401 (correcto)
    curl -s -H "Authorization: Bearer TU_API_KEY" http://127.0.0.1:8080/v1/models | head -c 200

6.3 Suite de humo (deja el server activo con --keep):

    bash tests/smoke.sh --keep

6.4 Auditoría: `./deploy.sh` opcion 9 (host + prerrequisitos + manifest).

6.5 Chat por CLI: `./scripts/llama-chat.sh` (arranca el server solito).
    Web UI: `http://127.0.0.1:8080` (via tunel o localhost).

## 7. Runtime alternativo (OLLAMA)

7.1 El proyecto soporta `LLAMA_TYPE=ollama` con `OLLAMA_MODE=system|project`
    (seccion 5b de la lib). El default no cambia (llama.cpp). El menú opción
    6.2 muestra el estado del service si existe.
7.2 En remote-lab se valida con el smoke T5 (SKIP si el host no tiene
    ollama.service activo — el nivel de prueba no exige instalarlo).

## 8. Troubleshooting (sin errores silenciosos)

8.1 "FALTA curl/sha256sum" → instalar por familia (ver 1.4); gate bloqueante
    hasta cubrir core.
8.2 "llama-server no pudo iniciar" → `tail -40 logs/llama-server.log`; causa
    tipica: glibc antigua (verificar `./bin/llama-server --version`) o puerto
    ocupado (fuser 8080/tcp).
8.3 "Descarga incompleta" → reintentar: `curl -C -` reanuda solo; sha256
    corrupto -> borra el .gguf y repite la descarga (el hash garantizado)
8.4 "401 al chatear" → la `LLAMA_API_KEY` del `.env` del host no coincide
    con la del server: reinicia (scripts/start.sh --stop && scripts/start.sh)
8.5 Cambios de configuracion que no se aplican → el read-back del menu imprime
    `LLAMA_HOST=... (verificado)`; si ves FAIL es un bug (falta corregir).
8.6 No mutes ni credenciales ni modelos al repo: `.env` está en `.gitignore`.

## 9. Referencias

9.1 README.md (capacidades, modelos, rendimiento).
9.2 docs/TECHNICAL.md (arquitectura + tabla .env completa).
9.3 docs/ADR/ADR-002-exposicion-remota.md (postura de seguridad E2E).
9.4 docs/CHANGELOG.md (historial de capacidades).

<!-- End Development By Angel Esquivel (Automation) [UNCENSORE-LLM 2026] -->
