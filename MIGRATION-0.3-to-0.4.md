# Migración v0.3.0 → v0.4.0

## Cambios de estructura
| v0.3.0 | v0.4.0 | Por qué |
|---|---|---|
| `orchestration/` (name: csdd-multiagent-orchestrator) | `skills/csdd-multiagent-orchestrator/` | La carpeta debe llamarse como `name` (claude.ai y Agent Skills) |
| `communication/` | `skills/csdd-agent-communication/` | Ídem |
| `roles/<x>/` (name: csdd-<x>) | `skills/csdd-<x>/` (solo 5) | Ídem + consolidación |
| `config/system.yaml` | `assets/system/config/system.toml` | TOML se lee con la librería estándar (sin PyYAML) |
| `templates/authority-matrix.yaml` | `config/roles.toml` | Una sola fuente que además se hace cumplir |
| `templates/agent-message.yaml`, `task-contract.yaml` | `templates/*.json` | Los genera y valida la CLI |
| `templates/approval-record.yaml` | `approval-record.example.toml` + `csdd.py approval` | Ahora tiene efecto real |
| `plans/` | `skills/csdd-multiagent-orchestrator/references/plans/` | Viajan con la skill (también en claude.ai) |
| `scripts/install-skills.sh`, `Install-Skills.ps1` | `scripts/install.sh`, `Install.ps1` | Idempotentes; instalan runtime, hooks, agentes y CI |
| — | `templates/handoff.md`, `csdd-mentor/references/self-critique-prompt.md` | Estaban referenciados y no existían |

## Roles
`csdd-project-analyst` → Jarvis (modo Tráiler) · `csdd-integration-reviewer` → Cortana · `csdd-git-worktree-operator` → Jefe Maestro (`csdd.py`) · `csdd-unblocker-comunista` → protocolo de autorización. Detalle completo en `skills/csdd-multiagent-orchestrator/references/legacy-roles-v0.3.md`. Los IDs viejos se aceptan como alias (`[legacy]` en `roles.toml`).

## Mensajes
14 tipos → 6 (tabla de equivalencias en la Skill `csdd-agent-communication`).

## Pasos para un repo que tenía v0.3.0 instalado
1. Respaldo: `git switch -c chore/csdd-0.4`.
2. Elimina `.agents/skills/{orchestration,communication}` y los roles retirados de `.agents/skills/`.
3. `bash scripts/install.sh . --force` (respalda lo anterior en `.csdd-backup/`).
4. Fusiona a mano `AGENTS.csdd.md` → `AGENTS.md` (si tenías uno propio) y `.claude/settings.csdd.json` → `.claude/settings.json`.
5. Traslada decisiones de tu `project-profile.yaml` v0.3 al nuevo (añade modo de despliegue y nivel de ceremonia).
6. `python3 .csdd/tools/csdd.py doctor` y una tarea piloto.

## claude.ai
Sube `dist/claude-web/csdd-mentor.zip` para **reemplazar** tu `csdd-mentor` 1.4.1 (mismo `name`, se actualiza). Las demás son nuevas en tu cuenta. Ver `dist/claude-web/INSTALAR-EN-CLAUDE-WEB.md`.

---

# Migración v0.4.0 → v0.4.1

1. `bash scripts/install.sh . --force` (respalda `.csdd/` y las skills anteriores en `.csdd-backup/`), o copia encima solo lo que cambió: `.csdd/` completo y las skills `csdd-multiagent-orchestrator`, `csdd-executive-supervisor` y `csdd-mentor`.
2. `python3 .csdd/tools/sync_agents.py .` para regenerar agentes con el bloque de protocolo de sesión.
3. Fusiona `AGENTS.csdd.md`/`CLAUDE.csdd.md` si ya tenías los tuyos (el instalador no los sobrescribe).
4. Ajusta `[session]` en `.csdd/config/system.toml` a tu proyecto (bitácora, verify_command, rutas privadas, migraciones, señales de desfase).
5. `python3 .csdd/tools/csdd.py session start` y, si quieres ruta crítica, copia `.csdd/templates/ruta-critica.toml` a `.project/` y ejecuta `csdd.py ruta`.

## Adoptarlo en Corbeta sin perder lo que ya funciona
- **Puedes mantener** `scripts/estado.sh` y `PROTOCOLO-SESION.md`: `csdd.py estado` es su generalización y puede convivir.
- Para que el sistema lea tu bitácora real: `bitacora_file = "notion-sincronizacion.md"`, `siguiente_file = "SIGUIENTE-SESION.md"`, `hours_since = "2026-11-02"`, y añade esas dos rutas a `write_paths` del Jefe Maestro en `roles.toml` (cambio tuyo, protegido).
- El semáforo de G6 pasa de `F3_HECHA=si` a un hito en `.project/ruta-critica.toml` (`[[hito]] id = "F3" fecha_limite = "2027-01-10"`); los umbrales salen solos (≈ 54/70 h).
- Formato de bloque: el sistema escribe `- Seguimiento:` donde Corbeta usa `- Notion:`; el conteo de horas (`- Horas: X`) es idéntico, así que tus bloques anteriores suman.

---

# Migración v0.4.1 → v0.4.2

1. Reemplaza el paquete (o aplica el parche 0.4.2 encima) y, en cada repo con el sistema instalado, ejecuta `bash scripts/install.sh <repo> --force` (o `Install.ps1 -Force`). Esto actualiza `.csdd/` (guard y CLI con UTF-8) y respalda lo anterior en `.csdd-backup/`.
2. Ya **no** necesitas `PYTHONUTF8=1` ni un acceso directo `python3` en Windows. Si el instalador no encuentra tu Python, usa `CSDD_PYTHON`.
3. Claude Code: el hook de `.claude/settings.json` usará el intérprete detectado. Si tu `settings.json` ya existía, el instalador deja `settings.csdd.json` al lado: revisa que el comando del hook use un Python que funcione.
4. Sube a claude.ai solo `csdd-multiagent-orchestrator.zip` (las demás skills no cambiaron).
