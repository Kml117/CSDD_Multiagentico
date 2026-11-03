# CSDD Multi-Agent Engineering System · v0.4.2

**Qué es:** un sistema para desarrollar software con varios agentes de IA (Claude Code y Kimi Code en VS Code) bajo **Constitutional Spec-Driven Development**. Cada agente tiene un rol con **permisos reales**, se comunica por un **buzón que enruta el líder**, trabaja en su propio **Git worktree** y nada se integra sin **verificación independiente**.

**Qué cambió frente a v0.3.0:** la v0.3 describía todo eso en documentos; la v0.4 lo **hace cumplir**. Trae un guard que bloquea lo que un rol no puede hacer, una CLI para tareas, buzón y merge, controles ejecutables por artículo y un workflow de CI. Está probado con 14 pruebas de extremo a extremo (más 10 brechas conocidas que fallan a propósito).

**Nuevo en v0.4.1 (desde Corbeta):** protocolo de sesión anti-retrabajo (el estado se calcula con `csdd.py estado`; sin `session start` nadie modifica nada; `session end` deja bitácora con horas) y ruta crítica adaptada al ciclo de vida y al enfoque (`csdd.py ruta`: recurso limitado, relojes, tramos, semáforo).

> **¿Primera vez? Empieza por [MANUAL-DE-USO.md](MANUAL-DE-USO.md):** instalación, una tarea piloto paso a paso, cómo aplicarlo a un proyecto nuevo o existente y qué hacer cuando algo se bloquea. Si vas a **continuar el desarrollo**, lee después [docs/ROADMAP-DESARROLLO.md](docs/ROADMAP-DESARROLLO.md) (empieza por su §0).

## En 5 minutos

```bash
cd tu-repo                                              # repo git (ej. Corbeta)
bash /ruta/csdd-multiagent-engineering-system/scripts/install.sh .
git add -A && git commit -m "chore: CSDD multi-agent v0.4.2"
python3 .csdd/tools/csdd.py doctor
```
Windows: `.\scripts\Install.ps1 -TargetPath C:\ruta\tu-repo`. Requisitos: Python 3.11+ **real** (no el alias vacío de la Tienda de Microsoft) y git; para las pruebas y `install.sh` en Windows, Git para Windows (trae `bash`). Si tu Python está en otra ruta: `CSDD_PYTHON=/ruta/python bash scripts/install.sh .` (en PowerShell: `$env:CSDD_PYTHON = "C:\ruta\python.exe"`).

## Los cinco roles

| Alias | ID estable | Hace | Puede escribir | No puede |
|---|---|---|---|---|
| **Jefe Maestro** | `csdd-executive-supervisor` | Planea, crea tareas y worktrees, **enruta el buzón, integra**, informa | `.project/`, `docs/`, `specs/` | tocar código de producto, verificar |
| **Jarvis** (+ modo Tráiler) | `csdd-software-architect` | Descubre, especifica en EARS con REQ-IDs, contratos, ADR | `specs/`, `docs/adr/`, `docs/architecture/` | código, instalar, commitear |
| **El Hombre de la Bolsa** | `csdd-implementation-engineer` | Implementa Test-First **una tarea** | solo `allowed_paths` de su tarea | salir de su alcance, merge, push |
| **Cortana** (+ integración) | `csdd-qa-verification` | Re-ejecuta, prueba cada REQ, revisa alcance e integración, **RESULT** | tests de aceptación/propiedad/integración, evidencia | tocar `src/`, autoaprobar |
| **El Portero** | `csdd-security-reviewer` | Amenazas, escáneres, dependencias, cadena de suministro de agentes; **puede bloquear** | `docs/security/`, evidencia | escribir código, commitear |

**Comunista** ya no es un agente: es una **autorización temporal y acotada** que solo tú creas (`csdd.py approval new`) y que el guard aplica.

## Cómo fluye una tarea

```
Tú ─► Jefe Maestro ─► Jarvis (spec EARS + contrato)
          │
          ├─ csdd.py task new  (allowed_paths, REQ, artículos, presupuesto)
          ├─ csdd.py worktree new T-001 ─► Kimi (El Hombre de la Bolsa) en ../repo-wt/T-001
          │                                   └─ tests primero → commits → csdd.py handoff
          ├─ csdd.py route  (solo él enruta el buzón)
          ├─ csdd.py worktree review T-001 ─► Claude (Cortana) en ../repo-wt/T-001-review
          │                                   └─ re-ejecuta → csdd.py result PASS/FAIL + evidencia
          │                                   (+ El Portero si hay riesgo)
          └─ csdd.py merge T-001  (se niega sin PASS independiente) ─► informe a ti
```

## Qué hay en el paquete

```
skills/                         8 skills (nombre de carpeta = name; listas para Claude Code, Kimi y claude.ai)
  csdd-mentor/                  constitución CSDD v1.6.0 (SKILL.md + references/)
  csdd-multiagent-orchestrator/ entrada del sistema + references/ (planes, runtime, Comunista)
    assets/system/              TODO lo que se instala como .csdd/ en tu repo (código, config, CI, plantillas)
  csdd-agent-communication/     protocolo del buzón
  csdd-executive-supervisor/ … csdd-security-reviewer/   los cinco roles
scripts/                        install.sh, Install.ps1, build_claude_web.py
tests/test_system.py            14 pruebas end-to-end (python3 -m unittest discover -s tests -v)
tests/test_known_gaps.py        10 brechas conocidas como pruebas que fallan a propósito (backlog del roadmap)
docs/adr/                       ADR-0002 (CSRF/TLS), ADR-0003 (cinco roles, enrutador, Comunista), ADR-0004 (sesión y ruta crítica)
dist/claude-web/                una .zip por skill para subir a claude.ai
docs/ROADMAP-DESARROLLO.md      hoja de ruta v0.5 → v1.0 para quien continúe el desarrollo (empieza por su §0)
EXECUTION_GUIDE.md              guía paso a paso
MIGRATION-0.3-to-0.4.md         qué cambió y cómo migrar
```

## Estado honesto (qué está verificado y qué no)

| Componente | Estado |
|---|---|
| Guard, CLI, buzón, merge con gate, Comunista, presupuestos, bucles, verificadores, instalador, protocolo de sesión, ruta crítica, UTF-8 | **PASS**: 14 pruebas end-to-end en repos git temporales (la ruta crítica reproduce las cifras de Corbeta) |
| Windows | **PARCIAL**: instalador, `doctor` y `session start` probados por Claude Code (3-nov); la corrección de UTF-8 de 0.4.2 está validada en emulación y falta reconfirmarla en Windows real |
| Reglas Semgrep | **NOT_RUN**: escritas con fixtures (`semgrep --test`), sin Semgrep en el entorno de construcción |
| Hook de Claude Code | Formato documentado (exit 2 bloquea); probado invocando el guard como lo hace el hook |
| Identidad de subagente en el hook (Modo B) | **HIPÓTESIS**: `csdd.py doctor` te lo dice en tu versión |
| Hooks y agentes de Kimi Code | **HIPÓTESIS**: plantilla; verifica el esquema en la documentación oficial |
| Workflow de CI | Escrito; se valida en tu primer PR |

## Referencias
- PMI, PMBOK Guide — Eighth Edition: https://www.pmi.org/standards/pmbok · Project Life Cycle: https://www.pmi.org/about/what-is-a-project · Spectrum of Approaches: https://www.pmi.org/learning/thought-leadership/series/achieving-greater-agility/spectrum-of-approaches · Tailoring: https://www.pmi.org/-/media/pmi/documents/public/pdf/pmbok-standards/pmi-tailoring.pdf · Agile Practice Guide: https://www.pmi.org/standards/agile (adaptación operativa; no es reproducción del PMBOK ni certificación PMI).
- Marri (2026), *Constitutional Spec-Driven Development*, arXiv:2602.02584 · Cemri et al., MAST (NeurIPS 2025), arXiv:2503.13657 · Gloaguen et al., *Evaluating AGENTS.md* (ETH Zúrich, 2026), arXiv:2602.11988 · OWASP Top 10 for Agentic Applications (dic 2025) · MITRE/CISA CWE Top 25 (dic 2025) · Documentación de Claude Code (agent teams, hooks) y Kimi Code (agentes, hooks), consultadas en nov 2026.
