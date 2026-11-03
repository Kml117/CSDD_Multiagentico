# Changelog

## 0.4.2 — 2026-11-04
**Funciona en Windows sin ajustes manuales; versiones alineadas.** Origen: pruebas de Claude Code en Windows (3-nov-2026).
- **UTF-8:** `guard.py` y `csdd.py` fuerzan UTF-8 en stdin/stdout/stderr, decodifican bien los argumentos y las llamadas a `git`/verificación usan UTF-8. Ya no hace falta `PYTHONUTF8=1`. Los scripts `tools/*.py` también.
- **Instaladores:** `install.sh` e `Install.ps1` prueban cada candidato de Python (descartan el alias vacío de la Tienda de Microsoft y los Python < 3.11) y aceptan `CSDD_PYTHON`; error claro y accionable si no hay ninguno. El hook de `settings.json` usa el intérprete detectado. `Install.ps1` escribe `settings.json` **sin BOM** y se guarda con BOM propio para que PowerShell 5.1 lea las tildes.
- **Pruebas (24: 14 pasan, 10 brechas esperadas; en Windows `test_14` se omite porque usa enlaces simbólicos y un PATH POSIX, así que allí salen 13 + 1 omitida + 10 esperadas):** `test_13` (consola sin UTF-8) y `test_14` (instalador con alias roto); `run()` fuerza UTF-8; `CSDD_TEST_LEGACY_CONSOLE=1` emula una consola heredada solo en los procesos hijos. La prueba nueva encontró un fallo real (argumentos con tildes bajo `LANG=C`) que ya está corregido.
- **Versiones alineadas:** `csdd_core.VERSION`, `roles.toml`, `system.toml`, `constitution-controls.toml`, plantillas y `MANIFEST` → 0.4.2; el orquestador apunta a `csdd-mentor@1.6.1`. Las menciones de "v0.4.0"/"v1.6.0" en encabezados de documentos de referencia son históricas (versión en que se escribió el documento) y no se tocan, para no invalidar los zips web de skills sin cambios de contenido.
- **Skills:** solo `csdd-multiagent-orchestrator` cambia (0.4.2). `csdd-mentor` sigue en 1.6.1 y las otras seis no cambian.
- **Roadmap:** estado de Windows y de WP-V2/WP-V3 actualizado con lo comprobado.

## 0.4.1 — 2026-11-03 (entrega para desarrollo)
- Nuevo `docs/ROADMAP-DESARROLLO.md`: hoja de ruta v0.5 → v1.0 para quien continúe el desarrollo (enrutador vivo, runner, dispatcher, seguridad, costos, evaluación), con criterios de aceptación, decisiones abiertas y qué no construir.
- Nuevo `tests/test_known_gaps.py`: 10 brechas conocidas como pruebas `expectedFailure` (backlog ejecutable). Suite total: 22 pruebas, 12 pasan y 10 fallan a propósito.
- Demostrado y documentado: el guard no detiene `python -c` / `node -e` (puede escribir fuera de `allowed_paths` y reescribir `.csdd-task`); `csdd.py merge` no comprueba la rama activa.
**Anti-retrabajo y ruta crítica adaptable (práctica de Corbeta, generalizada). ADR-0004.**
- Nuevo `tools/sesion.py` + comandos `csdd.py estado`, `session start|end|status`: estado del repo calculado (rama, cambios, origin/main con firmas, PR y CI vía gh, tareas y worktrees, migraciones, archivos sensibles rastreados, documentos con señales de desfase, bitácora y horas, semáforo, qué sigue).
- Guard: sin sesión vigente bloquea escrituras y comandos que modifican; prohíbe `git add -A`, `git add .`, `git commit -a`; `--acepto-alertas` solo para el propietario.
- `session end`: verify si se tocó código (si falla, sesión "parcial"), horas obligatorias, bloque de bitácora ≤ 10 líneas con acumulado.
- Nuevo `tools/ruta_critica.py` + `csdd.py ruta`: CPM (cota inferior), programación con recurso limitado, relojes de calendario, tramos, hitos con semáforo y probabilidad PERT, desvíos > 50 %, horizonte rodante. Reproduce las cifras de Corbeta (prueba de regresión).
- `system.toml`: secciones `[session]` y `[critical_path]`. `constitution-controls.toml`: controles `session-protocol` y `no-blind-add` en Art. IX.
- Plantillas: `SIGUIENTE-SESION.md`, `RUTA-CRITICA.md`, `ruta-critica.toml`, `protocolo-sesion-core.md` (se inyecta en todos los agentes generados). `AGENTS.md`/`CLAUDE.md` incluyen el protocolo.
- Skills: `csdd-multiagent-orchestrator` 0.4.1 (+ `references/protocolo-sesion.md`, `references/ruta-critica.md`, planes), `csdd-executive-supervisor` 0.4.1 (dueño de la ruta y del cierre), `csdd-mentor` 1.6.1 (control del Art. IX). Los otros cinco no cambian.
- Pruebas: 12 end-to-end (3 nuevas).

## 0.4.0 — 2026-11-03
**Permisos reales, constitución ejecutable, cinco roles.**
- Nuevo runtime local mínimo sin dependencias (Python 3.11+): `guard.py` (hook PreToolUse para Claude Code y Kimi Code), `csdd.py` (tareas, worktrees, buzón, gates, merge, autorizaciones, watchdog, verify, doctor), `check_constitution_controls.py`, `check_traceability.py`, `sync_agents.py`.
- `roles.toml`: fuente única de permisos por rol (rutas, shell, herramientas, modelo sugerido).
- Cinco roles (ADR-0003): Jefe Maestro, Jarvis (+ modo Tráiler), El Hombre de la Bolsa, Cortana (+ integración), El Portero. Retirados: project-analyst, integration-reviewer, git-worktree-operator, unblocker-comunista (con alias en `[legacy]`).
- Buzón con enrutador explícito (Alternativa B): solo Jefe Maestro enruta; 6 tipos de mensaje; validación, rechazo y deduplicación.
- Protocolo Comunista: autorización temporal y acotada creada por el propietario y aplicada por el guard.
- Presupuesto por tarea, detector de bucles, watchdog y bitácora `events.jsonl`.
- `constitution-controls.toml` + 14 reglas Semgrep por artículo + gitleaks + workflow `csdd-gate.yml` + pre-commit.
- `csdd-mentor` 1.6.0: SKILL.md + references/ sin eliminar contenido (verificado línea a línea); controles por artículo; corrección del mapeo SEC (ADR-0002); se crea `references/self-critique-prompt.md` que faltaba.
- Correcciones de v0.3.0: referencias rotas (`templates/agent-handoff.yaml`, `operations/`, `references/self-critique-prompt.md`), versión 1.4.1/1.5.0 inconsistente, carpetas de skills con nombre distinto a su `name`, instalador que abortaba en la segunda ejecución.
- Versiones para claude.ai (web) en `dist/claude-web/`, una por skill, generadas desde las mismas fuentes.
- Fix detectado por las pruebas: el guard ya no genera `__pycache__` dentro de `.csdd/`.

## 0.3.0 — 2026-11-02
- Integrada la Skill `csdd-mentor` v1.5.0 con el sistema multiagente; los nueve artículos se preservan y se añade guía de ejecución.
- Añadidos `AGENTS.md`, `EXECUTION_GUIDE.md`, `MANIFEST.yaml`, configuración de sistema y scripts de instalación.
- Formalizado el buzón inicial basado en archivos, con estados, idempotencia, trazabilidad y límites de concurrencia.
- Añadido registro de autorizaciones acotadas para Comunista y matriz de autoridad reforzada.
- Actualizados los nombres operativos proporcionados; integración y operador Git quedan pendientes de nombre.
- Añadida política de tailoring para ciclo de vida, enfoque de desarrollo, fases y cadencia.
- Aclarado que el paquete no conecta ni despliega un runtime externo.

## 0.2.0
- Planes de dirección, enfoque de desarrollo, orquestación y organización; comunicación y plantillas iniciales.
