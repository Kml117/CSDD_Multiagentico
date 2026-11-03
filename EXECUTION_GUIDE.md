# Guía de ejecución — CSDD Multi-Agent v0.4.2

## 1. Qué entrega esta versión
Skills (instrucciones por rol) **más** un runtime local mínimo que las hace cumplir: guard de herramientas, CLI de coordinación, verificadores y CI. No hay procesos autónomos: tú abres las sesiones de Claude Code y Kimi Code; el sistema decide qué puede hacer cada una y registra todo.

## 2. Jerarquía de entrada
1. Tú formulas objetivo y restricciones.
2. La sesión principal de Claude Code carga `csdd-multiagent-orchestrator` (y actúa como Jefe Maestro).
3. Lee `AGENTS.md` / `CLAUDE.md`, `.csdd/config/` y el Project Profile.
4. Consulta `csdd-mentor` para los artículos aplicables (sin convertir cada tarea en una clase).
5. Jarvis especifica; Jefe Maestro crea contratos y worktrees.
6. Los agentes se comunican por el buzón; los artefactos versionados son la fuente de verdad.
7. Cortana (y El Portero según riesgo) verifican de forma independiente.
8. Jefe Maestro integra y te entrega un único informe.

## 3. Instalación
```bash
bash scripts/install.sh /ruta/repo              # --force para actualizar (respalda en .csdd-backup/)
# Windows: .\scripts\Install.ps1 -TargetPath C:\ruta\repo
cd /ruta/repo && git add -A && git commit -m "chore: CSDD v0.4.0"
python3 .csdd/tools/csdd.py doctor
```
Qué instala: Skills en `.claude/skills/` y `.agents/skills/`; `.csdd/` (guard, CLI, config, reglas, plantillas, integraciones); `.claude/settings.json` (hook + deny); `.claude/agents/` y `.agents/agents/` generados; `AGENTS.md`, `CLAUDE.md`; `.github/workflows/csdd-gate.yml`; `.project/` (coordinación, no versionado salvo perfiles y artefactos); `core.hooksPath=.csdd/git-hooks` (si no tenías uno).

**Kimi Code:** copia `.csdd/integrations/kimi/hooks.example.toml` a `~/.kimi-code/config.toml` **después de verificar el esquema** en la documentación oficial.

**Opcional pero recomendado:** instala `semgrep` (`pipx install semgrep`) y `gitleaks` para que `csdd.py verify` y el pre-commit no reporten NOT_RUN.

## 4. Arranque de un proyecto nuevo (greenfield)
1. Completa `.project/project-profile.yaml` y `.project/lifecycle-profile.yaml` (modo de despliegue A/B, runtime por rol, nivel de ceremonia).
2. Si aplica, crea la constitución con `csdd-mentor` (Modo A) y ajusta `.csdd/config/constitution-controls.toml`. Ratifica con commit `RATIFY: Constitution v1.0`.
3. Jarvis: spec EARS con REQ-IDs y contratos.
4. Jefe Maestro: tareas pequeñas con `allowed_paths` estrechos → worktrees → ejecución → revisión → merge.
5. Empieza con **una tarea piloto no crítica** y valida todo el ciclo antes de paralelizar.

## 5. Proyecto existente (retrofit/brownfield)
1. Jarvis en modo Tráiler: inventario con hechos/inferencias/preguntas; nada se asume.
2. Primera tarea: pruebas de caracterización del comportamiento actual.
3. El Portero: superficie de ataque según criticidad; `cybersecurity` para escaneo amplio si el código es grande.
4. `check_traceability.py --warn-only` al principio; quita `--warn-only` en CI cuando estés al día.
5. Nada de migraciones destructivas ni despliegues sin tu autorización específica.

## 6. Selección de ciclo de vida, enfoque y ceremonia
Registra por separado: ciclo de vida (predictivo, iterativo, incremental, adaptativo, híbrido), enfoque de desarrollo (predictivo, adaptativo, híbrido), cadencia, fases y gates, estado del producto (greenfield, retrofit, brownfield, mantenimiento) y nivel de ceremonia (trivial, normal, arquitectónico). La selección considera estabilidad de requisitos, incertidumbre, modularidad, riesgo, regulación, dependencias, costo del cambio, feedback disponible, cadena de entrega y madurez del equipo. No deduzcas el enfoque solo por las palabras "ágil" o "cascada".

## 7. Flujo diario (Modo A, recomendado)

**Cada sesión, de cualquier rol, empieza con `csdd.py session start` y termina con `csdd.py session end --horas X`** (protocolo anti-retrabajo; sin sesión el guard no deja modificar nada). La ruta crítica se arma después de elegir ciclo de vida y enfoque: `.project/ruta-critica.toml` → `csdd.py ruta` → `docs/RUTA-CRITICA.md` aprobado por ti. Ver las referencias `protocolo-sesion.md` y `ruta-critica.md` del orquestador.

| Paso | Quién | Dónde | Comando |
|---|---|---|---|
| 1 | Jefe Maestro | Claude Code, worktree principal | `csdd.py status` · `route` · `watchdog` |
| 2 | Jefe Maestro | — | `task new …` · `worktree new T-001` |
| 3 | El Hombre de la Bolsa | Kimi Code abierto en `../repo-wt/T-001` | `task start T-001` → tests → commits → `handoff T-001 …` |
| 4 | Jefe Maestro | — | `route` · `worktree review T-001` |
| 5 | Cortana | Claude Code abierto en `../repo-wt/T-001-review` | re-ejecuta → evidencia → `result T-001 --status …` |
| 6 | El Portero (si aplica) | misma carpeta, `CSDD_ROLE=csdd-security-reviewer` | escáneres → `result … [--blocking]` |
| 7 | Jefe Maestro | worktree principal | `route` · `merge T-001` · `verify` · informe |

**Modo B** (subagentes en una sesión): solo si `csdd.py doctor` confirma que el hook recibe la identidad del subagente.

## 8. Buzón
Ver la Skill `csdd-agent-communication`. Resumen: `send` escribe en tu outbox; **solo Jefe Maestro** ejecuta `route`; `inbox` y `ack` para leer y cerrar. Seis tipos: TASK, HANDOFF, QUESTION, ANSWER, RESULT, BLOCKER. Lo inválido va a `.project/mailbox/rejected/` con su motivo.

## 9. Autorizaciones (protocolo Comunista)
Solo tú, en tu terminal: `csdd.py approval new --task … --role … --purpose … --paths … --deps … --commands … --hours …`. Amplía permisos de una tarea y un rol hasta que caduca o la revocas (`approval revoke`). Nunca abre constitución, hooks, CI, configuración de agentes, secretos ni comandos prohibidos, ni permite autoaprobación o merge sin PASS. Producción, destrucción de datos, migraciones irreversibles, gastos, compromisos externos e identidad o firma personal requieren tu aprobación específica fuera de este protocolo.

## 10. Gates y entrega
- Cada tarea: responsable único, criterios, dependencias y presupuesto.
- Estados de verificación: `PASS`, `FAIL`, `NOT_RUN`, `NOT_APPLICABLE`, `UNKNOWN`. Nunca presentar `NOT_RUN` o `UNKNOWN` como `PASS` (la CLI exige evidencia para PASS).
- Reintentos limitados (`max_retries`): después, STUCK y BLOCKER.
- Recibes un informe consolidado; solo intervienes en decisiones reservadas o riesgos no cubiertos.

## 11. Runtime e integraciones
- **Agent Skills:** instrucciones por rol; por sí solas no son runtime.
- **Guard + CLI + CI:** el runtime mínimo de esta versión.
- **MCP:** solo cuando se conozcan servidor, permisos y acciones; no hace falta para el buzón.
- **A2A:** solo si hay agentes en máquinas o runtimes distintos.
- **LangGraph u otro motor:** decisión futura, si la persistencia, la concurrencia o la observabilidad lo exigen.
- **Worktrees:** aíslan ramas y archivos, no bases de datos, puertos, procesos, secretos ni servicios.

## 12. Checklist de instalación
- [ ] `csdd.py doctor` sin FAIL.
- [ ] `.csdd/`, `.claude/`, `.agents/`, `AGENTS.md`, `CLAUDE.md`, `.github/` commiteados en la rama base.
- [ ] Hook de Kimi configurado y verificado (o asumido NOT_RUN, con CI como gate).
- [ ] Semgrep y gitleaks instalados (o NOT_RUN aceptado conscientemente).
- [ ] `semgrep --test .csdd/policy/semgrep/` en verde.
- [ ] Tarea piloto completa: task → worktree → handoff → review → result → merge.
- [ ] Tú identificado como autoridad final; nadie más crea autorizaciones.

## 13. Primera ejecución recomendada
Proyecto piloto no crítico, una tarea pequeña. Valida intake, contrato, worktree, buzón, bloqueo del guard (intenta a propósito escribir fuera de `allowed_paths`), handoff, revisión, merge e informe. No actives despliegue productivo ni permisos privilegiados durante el piloto.
