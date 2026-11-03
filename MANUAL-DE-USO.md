# Manual de uso · CSDD Multi-Agent Engineering System v0.4.2

**Para quién es:** cualquier persona que quiera desarrollar software con varios agentes de IA (Claude Code, Kimi Code) sin perder el control. No necesitas conocer CSDD de antemano.

**Cómo leerlo:** si tienes prisa, lee la sección 1, haz la sección 3 (tu primera tarea) y vuelve aquí cuando algo te bloquee (sección 8). El resto es consulta.

---

## 1. Qué es, en dos minutos

Tú diriges. Varios agentes de IA trabajan, cada uno con un **rol** y **límites reales**:

- Un agente no puede tocar archivos que su rol no permite. Un programa llamado *guard* lo **bloquea**, no solo se lo pide.
- Cada tarea se hace en su propia carpeta de trabajo (un *worktree* de Git), para que nadie pise el trabajo de otro.
- Los agentes se pasan mensajes por un buzón de archivos. Solo el Jefe Maestro reparte esos mensajes.
- Nada se integra al proyecto sin que **otro** agente lo revise y presente evidencia. Nadie se aprueba a sí mismo.
- Antes de editar, cada sesión consulta el estado real del repo, para no rehacer trabajo ni dar por hecho lo que no se comprobó.

> **La regla de oro:** una afirmación sin evidencia (comando y salida) no cuenta. Los estados válidos son `PASS`, `FAIL`, `NOT_RUN`, `NOT_APPLICABLE` y `UNKNOWN`. Lo que no se ejecutó nunca se presenta como `PASS`.

### Los cinco roles

| Nombre | Qué hace | Qué NO puede hacer |
|---|---|---|
| **Jefe Maestro** | Planea, crea tareas y carpetas de trabajo, reparte el buzón, integra y te informa | Tocar código de producto |
| **Jarvis** | Investiga y escribe especificaciones, contratos y decisiones (ADR) | Escribir código, instalar, hacer commits |
| **El Hombre de la Bolsa** | Escribe el código de **una** tarea, con las pruebas primero | Salir de las rutas de su tarea, integrar, subir cambios |
| **Cortana** | Revisa de forma independiente: re-ejecuta, prueba cada requisito, emite PASS o FAIL | Tocar el código fuente, aprobar su propio trabajo |
| **El Portero** | Revisa seguridad y **puede bloquear** la integración | Escribir código |

**Tú** eres la autoridad final. Hay una herramienta, la *autorización temporal* (antes llamada «Comunista»), que solo tú puedes crear y que amplía los permisos de una tarea por un tiempo limitado.

---

## 2. Qué necesitas e instalación

### Requisitos

| Necesitas | Para qué | Cómo comprobarlo |
|---|---|---|
| **Python 3.11 o superior** | Corre el guard y la herramienta `csdd.py` | `python --version` |
| **Git** | Ramas, carpetas de trabajo, integración | `git --version` |
| **Claude Code** (VS Code o terminal) | Jefe Maestro, Jarvis, Cortana y El Portero | — |
| Kimi Code (opcional) | El Hombre de la Bolsa. Se recomienda otro modelo distinto al que revisa | — |
| `semgrep` y `gitleaks` (opcional) | Análisis de seguridad. Sin ellos esos controles quedan en `NOT_RUN` | — |

El paquete **no instala dependencias de Python**: usa solo la librería estándar.

### Instalar en un proyecto

Tu proyecto debe ser un repositorio Git con al menos un commit. Si no lo es: `git init` y haz un primer commit.

**Windows (PowerShell):**
```powershell
cd C:\ruta\a\tu-proyecto
C:\ruta\al\paquete\scripts\Install.ps1 -TargetPath .
git add .csdd .claude .agents AGENTS.md CLAUDE.md .github .gitignore .project
git commit -m "chore: instala CSDD multi-agent"
python .csdd\tools\csdd.py doctor
```

**macOS / Linux / Git Bash:**
```bash
cd /ruta/a/tu-proyecto
bash /ruta/al/paquete/scripts/install.sh .
git add .csdd .claude .agents AGENTS.md CLAUDE.md .github .gitignore .project
git commit -m "chore: instala CSDD multi-agent"
python3 .csdd/tools/csdd.py doctor
```

**Muy importante: haz el commit antes de crear la primera tarea.** Cada carpeta de trabajo hereda el guard desde la rama base. Si `.csdd/` no está commiteado, esa carpeta no tendrá protección.

El instalador es seguro de repetir:

- Lo que ya existe lo **omite**. Con `--force` (`-Force` en PowerShell) lo respalda en `.csdd-backup/` y lo reemplaza.
- Nunca pisa tu `AGENTS.md`, tu `CLAUDE.md`, tu `.claude/settings.json` ni tu workflow de CI. Si ya existen, deja una copia `*.csdd.*` para que las fusiones a mano.
- Los avisos `LF will be replaced by CRLF` que Git muestra en Windows son normales y no afectan.

### Qué queda en tu proyecto

| Carpeta o archivo | Contenido |
|---|---|
| `.csdd/` | El sistema: guard, herramientas, configuración, reglas, plantillas |
| `.claude/` y `.agents/` | Las 8 skills y los agentes generados, para Claude Code y para Kimi |
| `AGENTS.md`, `CLAUDE.md` | Instrucciones que leen los agentes al abrir el proyecto |
| `.github/workflows/csdd-gate.yml` | La verificación automática en GitHub (el control final) |
| `.project/` | Tareas, buzón y registro de eventos (estado de trabajo: el instalador lo deja fuera de Git con un `.gitignore`) y los dos perfiles del proyecto (esos sí se versionan) |
| `docs/` | `SIGUIENTE-SESION.md` y, al cerrar la primera sesión, `BITACORA.md` |

### Comprobar que quedó bien

`doctor` debe mostrar `[PASS]` en todo. Un `[UNKNOWN] aún no hay llamadas al hook registradas` al principio es normal: se resuelve al abrir Claude Code y ejecutar algo.

### Atajo recomendado

Para no escribir la ruta completa cada vez, define un alias.

PowerShell (en tu perfil, o por sesión):
```powershell
function csdd { python .csdd\tools\csdd.py @args }
```
Bash: `alias csdd='python3 .csdd/tools/csdd.py'`

**En el resto del manual escribo `csdd` para abreviar.** Si no creas el alias, escribe el comando completo.

---

## 3. Tu primera tarea (hazla como piloto)

Haz el primer recorrido completo con algo **pequeño y no crítico**. Aquí aprendes qué se siente cuando el guard bloquea y cuando el flujo funciona.

### Paso 0 · Abre la sesión (siempre)

En la carpeta principal del proyecto:
```
csdd session start
```
Lee lo que imprime: es el estado **calculado** del repo. Si dice `ALERTA`, resuélvela o pregúntale al responsable antes de seguir. Sin una sesión abierta, el guard bloquea toda modificación. Leer sí está permitido.

### Paso 1 · Dile al Jefe Maestro qué quieres

Abre Claude Code en la carpeta principal y habla con él, por ejemplo:

> *«Usa csdd-multiagent-orchestrator. Quiero añadir el login con correo y contraseña. Primero pídele a Jarvis la especificación y dime qué tareas propones.»*

Claude Code carga la skill por su nombre. Jarvis responderá con requisitos escritos de forma verificable (cada uno con un identificador `REQ-…`) y un contrato de interfaz.

### Paso 2 · Crea la tarea

Aquí defines los límites. Los pide el sistema; tú los eliges (o el Jefe Maestro los propone y tú los apruebas). En los ejemplos largos uso `\` para partir la línea (bash). En PowerShell escribe todo en una sola línea, o usa la comilla invertida `` ` `` al final de cada línea:

```
csdd task new --id T-001 --title "Login" --role csdd-implementation-engineer \
    --paths "src/auth/**" --paths "tests/auth/**" \
    --req REQ-AUTH-01 --dep pydantic --budget-calls 40
```

- `--paths`: **los únicos lugares donde el implementador puede escribir.** Sé estrecho.
- `--req`: el requisito que cubre la tarea.
- `--dep`: dependencias nuevas autorizadas. Cualquier otra se bloquea.
- `--budget-calls`: cuántas acciones puede hacer el agente antes de pararse (por defecto 150).
- `--security`: añádelo si la tarea necesita revisión de El Portero.

Si dos tareas intentan escribir en las mismas rutas, el sistema las rechaza (`choque de rutas`). Es a propósito.

### Paso 3 · Crea el espacio de trabajo

```
csdd worktree new T-001
```
Esto crea una carpeta hermana (algo como `tu-proyecto-wt/T-001`) con su propia rama. Anota la ruta que imprime.

### Paso 4 · El implementador trabaja

Abre **Kimi Code (o Claude Code)** apuntando a la **carpeta de la tarea**, no a la principal. Dentro de ella:
```
csdd session start
csdd task start T-001
```
Luego pídele, por ejemplo:

> *«Eres El Hombre de la Bolsa. Implementa la tarea T-001 con pruebas primero. Haz commits con el formato `[REQ-AUTH-01][Art. II]`. Al terminar, ejecuta el handoff.»*

Al terminar, el agente entrega con:
```
csdd handoff T-001 --summary "Login listo, 1 prueba pasando"
```
Si tocó archivos fuera de `--paths`, el handoff se **rechaza** y le dice cuáles. Debe corregirlo.

### Paso 5 · El Jefe Maestro reparte y prepara la revisión

De vuelta en la carpeta principal:
```
csdd route
csdd worktree review T-001
```
`route` es el único momento en que los mensajes llegan a su destinatario. Si no lo ejecutas, nadie recibe nada.

### Paso 6 · Cortana revisa

Abre Claude Code en la **carpeta de revisión** (`…-wt/T-001-review`) y pídele:

> *«Eres Cortana. Revisa la tarea T-001 de forma independiente: re-ejecuta las pruebas, comprueba cada requisito y que no se haya salido de su alcance. Guarda la evidencia y da el resultado.»*

Cortana guarda su evidencia (salida de las pruebas) y emite el resultado:
```
csdd result T-001 --status PASS --summary "re-ejecutado: 1 passed" \
    --evidence .project/artifacts/evidence/T-001/qa.txt
```
`PASS` **sin evidencia se rechaza**. Si algo falla: `--status FAIL`, y la tarea vuelve al implementador. Si la tarea tiene riesgo de seguridad, El Portero revisa también y puede marcar `--blocking`.

### Paso 7 · Integrar y cerrar

En la carpeta principal:
```
csdd route
csdd merge T-001
csdd verify
csdd session end --horas 1.5 --resultado completado
```
- `merge` **se niega** si no hay un PASS independiente.
- `session end` exige que indiques las horas, y si tocaste código ejecuta la verificación que configuraste.
- El Jefe Maestro te entrega un informe consolidado. Tú solo intervienes en decisiones reservadas (sección 7).

### Prueba el guard a propósito

Antes de usarlo en serio, abre la carpeta de una tarea e intenta que el agente escriba **fuera** de sus rutas, o que instale una dependencia no autorizada. Debes ver `BLOQUEADO`. Así confirmas que la protección funciona en tu equipo.

---

## 4. Aplicarlo a tu proyecto

Elige tu caso:

### Caso A · Proyecto nuevo

1. Instala (sección 2) y haz el commit.
2. Completa `.project/project-profile.yaml` y `.project/lifecycle-profile.yaml`: objetivo, nivel de complejidad, criticidad, estado del producto (`greenfield`), nivel de ceremonia.
3. Si el proyecto es complejo o maneja datos sensibles, pide al **csdd-mentor** que te ayude a redactar una constitución:
   > *«Usa csdd-mentor. Ayúdame a crear la constitución de este proyecto.»*
   Ratifícala con un commit propio: `RATIFY: Constitution v1.0`. **Ese commit lo haces tú.** Ningún agente ratifica.
4. Pide a Jarvis la especificación y trabaja en tareas pequeñas, como en la sección 3.

### Caso B · Proyecto que ya existe

1. Instala y haz el commit.
2. Pide a Jarvis en **modo Tráiler** un inventario del código actual:
   > *«Eres Jarvis en modo Tráiler. Haz un inventario de este proyecto. Separa lo que encontraste, lo que infieres y lo que no sabes.»*
   Cada afirmación debe quedar como hecho, inferencia o pregunta abierta, y nada se da por sabido.
3. Primera tarea: **pruebas de caracterización** del comportamiento actual. Así tienes una red de seguridad antes de cambiar nada.
4. Para la trazabilidad, usa `python .csdd/tools/check_traceability.py --warn-only` al principio. Quita `--warn-only` cuando el proyecto esté al día.
5. Pide a El Portero que revise la superficie de ataque según la criticidad.
6. **No se hacen migraciones destructivas ni despliegues** sin tu autorización explícita.

### Caso C · Proyecto pequeño o personal

No necesitas todo el aparato:

- Usa solo **Jefe Maestro + Hombre de la Bolsa + Cortana**. Jarvis y El Portero solo cuando haga falta.
- Pon `required = false` en `[session]` de `.csdd/config/system.toml` si el protocolo de sesión te estorba.
- Los cambios triviales (un bug de una línea) no necesitan especificación: basta un commit con el requisito y las pruebas.

### Ajustar el sistema a tu proyecto

Todo se configura en archivos de texto dentro de `.csdd/config/`:

| Archivo | Para qué |
|---|---|
| `system.toml` | Sesión, bitácora, comando de verificación (`verify_command`, p. ej. `pnpm verify` o `pytest -q`), rutas privadas que nunca deben subir a Git, ubicación de migraciones, presupuestos por defecto |
| `roles.toml` | Qué puede escribir, leer y ejecutar cada rol. **Es una zona protegida:** la cambias tú, no un agente |
| `constitution-controls.toml` | Qué controles automáticos verifican cada artículo de tu constitución |

Después de cambiar `roles.toml`, regenera los agentes: `python .csdd/tools/sync_agents.py .`

---

## 5. El día a día

### Cada sesión, siempre

```
csdd session start        ← al empezar: calcula el estado, lee SIGUIENTE-SESION.md
   ...trabajo...
csdd session end --horas X ← al terminar: bitácora, horas y verificación
```

- Aplica a **todos los roles y todas las carpetas**.
- La sesión caduca a las 12 horas (`ttl_hours`).
- Si `session start` muestra una `ALERTA`, solo tú puedes abrir la sesión de todos modos: `csdd session start --acepto-alertas`.
- `session end` no cierra sin `--horas`. Si no sabes cuántas, es una pregunta para ti, no un dato que el agente deba inventar.
- Si la verificación falla al cerrar, la sesión queda como **parcial**.

### Ritmo de una tarea

```
Tú ► Jefe Maestro ► Jarvis (spec)
       │
       ├─ task new ► worktree new ► Hombre de la Bolsa  (tests ► commits ► handoff)
       ├─ route    ► worktree review ► Cortana         (re-ejecuta ► result)
       │                              (+ El Portero si hay riesgo)
       └─ route ► merge ► verify ► informe para ti
```

### Prohibido en el flujo

- `git add -A`, `git add .` y `git commit -a`: el guard los bloquea. Añade cada archivo por su nombre. Así no se cuelan archivos de otra sesión ni datos privados.
- Que un agente ratifique la constitución, apruebe un gate o fusione un PR. Eso lo haces tú.

### Si un agente se queda atascado

Tras un número limitado de reintentos (`max_retries`, por defecto 2) la tarea pasa a `STUCK` y se genera un `BLOCKER` para el Jefe Maestro. Él decide: partir la tarea, ampliar el presupuesto con una tarea nueva, o pedirte una decisión.

Cada cierto tiempo ejecuta `csdd watchdog` para ver las tareas que superaron su tiempo máximo.

---

## 6. Qué decirle a cada rol

Estos ejemplos son sugerencias; ajústalos a tu caso.

| Rol | Dónde abrirlo | Frase de ejemplo |
|---|---|---|
| **Jefe Maestro** | Claude Code, carpeta principal | «Usa csdd-multiagent-orchestrator. Estado del proyecto y qué sigue.» |
| **Jarvis** | Claude Code, carpeta principal | «Eres Jarvis. Escribe la especificación de [funcionalidad] con requisitos verificables y su contrato.» |
| **El Hombre de la Bolsa** | Kimi o Claude Code, **carpeta de la tarea** | «Eres El Hombre de la Bolsa. Implementa T-001, pruebas primero, sin salir de tu alcance.» |
| **Cortana** | Claude Code, **carpeta `…-review`** | «Eres Cortana. Verifica T-001 de forma independiente y entrega el resultado con evidencia.» |
| **El Portero** | Claude Code, carpeta de revisión, con `CSDD_ROLE=csdd-security-reviewer` | «Eres El Portero. Revisa T-001: amenazas, dependencias, secretos.» |

El agente sabe qué rol tiene por la carpeta donde lo abres: cada carpeta de tarea trae un archivo `.csdd-task` que se lo indica al guard.

### Los mensajes del buzón

Seis tipos: `TASK`, `HANDOFF`, `QUESTION`, `ANSWER`, `RESULT`, `BLOCKER`.

| Quiero… | Comando |
|---|---|
| Enviar un mensaje | `csdd send --to csdd-software-architect --type QUESTION --task T-001 --summary "¿…?"` |
| Repartir el buzón (solo Jefe Maestro) | `csdd route` |
| Ver mis mensajes | `csdd inbox` |
| Marcar un mensaje como leído | `csdd ack <id>` (con `--done` si ya se atendió) |

Los mensajes inválidos no se pierden: van a `.project/mailbox/rejected/` con el motivo.

---

## 7. Cuándo te toca a ti

El sistema trabaja solo hasta que llega algo que **solo tú** puedes decidir. Estas decisiones nunca las toma un agente:

- Ratificar o cambiar la constitución.
- Aprobar un gate de fase o integrar un PR.
- Producción, borrado de datos, migraciones irreversibles, gastos y compromisos externos.
- Cualquier cosa que use tu identidad o tu firma.
- Crear una **autorización temporal**.

### Autorización temporal (para desbloquear una tarea)

Si una tarea legítima necesita salir de su alcance, por ejemplo tocar `scripts/` una vez, **tú, en tu terminal**:

```
csdd approval new --task T-020 --purpose "mover build" --paths "scripts/**" --deps esbuild --hours 2
csdd approval list
csdd approval revoke <id>
```

- Caduca sola (máximo 8 horas) o cuando la revocas.
- Está limitada a una tarea y un rol.
- **Nunca** abre: la constitución, los hooks, el CI, `roles.toml`, los secretos, los comandos prohibidos, la autoaprobación ni la integración sin PASS.
- Un agente que intente crearla es bloqueado.

---

## 8. Cuando algo te bloquea

El guard imprime `BLOQUEADO` y el motivo. Esto es lo habitual:

| Síntoma | Qué pasó | Qué hacer |
|---|---|---|
| «Protocolo de sesión… antes de modificar nada ejecuta `session start`» | No hay sesión abierta | `csdd session start` |
| «necesita una tarea activa» | El implementador abrió la carpeta equivocada | Abre la carpeta que creó `worktree new` |
| «escritura fuera del repositorio» | Intentó escribir en `.project/` de la carpeta principal | Debe usar `send`, `handoff` o `result`; ellos escriben ahí por él |
| «fuera de allowed_paths» (en el handoff) | Cambió archivos que la tarea no permite | Revertirlos y volver a entregar |
| «choque de rutas» | Dos tareas piden las mismas rutas | Estrechar `--paths` o secuenciar las tareas |
| «posible bucle» | Mismo comando más de 4 veces sin editar nada | Cambiar de estrategia, o enviar un BLOCKER |
| «presupuesto agotado» | Llegó a `--budget-calls` | BLOCKER al Jefe Maestro, que decide partir o ampliar |
| «PASS exige evidencia» | `result` sin `--evidence` | Guardar la salida de las pruebas y referenciarla |
| `merge` se niega | No hay PASS independiente | Falta la revisión de Cortana (o de El Portero) |
| Se bloquea todo con «error interno» | `roles.toml` inválido o Python < 3.11 | `python -c "import tomllib"` y revisa `roles.toml` |
| El guard no bloquea nada | Hook no registrado o carpeta sin `.csdd/` | `csdd doctor`; commitea `.csdd/` en la rama base |
| Falso positivo en un comando de shell | El análisis es heurístico (p. ej. un `>` dentro de un texto) | Reescribe el comando, o pide una autorización con `--commands` |

Primero ejecuta siempre `csdd doctor`: resuelve la mayoría de dudas de instalación.

---

## 9. Comandos esenciales

| Comando | Quién | Qué hace |
|---|---|---|
| `csdd doctor` | cualquiera | Diagnóstico de la instalación |
| `csdd estado` | cualquiera | Estado calculado del repo (rama, cambios, tareas, alertas) |
| `csdd status` | cualquiera | Resumen de tareas y buzón |
| `csdd session start` / `end` / `status` | cualquiera | Abre, cierra y consulta la sesión |
| `csdd task new` / `show` / `list` / `start` | Jefe Maestro crea; el implementador inicia | Gestión de tareas |
| `csdd worktree new` / `review` / `remove` | Jefe Maestro | Carpetas de trabajo y de revisión |
| `csdd send` / `inbox` / `ack` | todos | Buzón |
| `csdd route` | solo Jefe Maestro | Reparte el buzón |
| `csdd handoff T-001 --summary …` | implementador | Entrega para revisión |
| `csdd result T-001 --status … --evidence …` | Cortana, El Portero | Resultado de la revisión |
| `csdd merge T-001` | Jefe Maestro | Integra (exige PASS independiente) |
| `csdd verify` | Jefe Maestro | Verificación global (usa `--strict` para exigirla) |
| `csdd approval new` / `list` / `revoke` | solo tú | Autorizaciones temporales |
| `csdd watchdog` | Jefe Maestro | Detecta tareas pasadas de tiempo |
| `csdd ruta` | Jefe Maestro | Calcula la ruta crítica (opcional) |

Para cualquier comando, `csdd <comando> --help` muestra todas sus opciones.

### Ruta crítica (opcional)

Si tu proyecto tiene fechas límite y poco tiempo semanal, copia `.csdd/templates/ruta-critica.toml` a `.project/` con tus tareas, estimaciones, horas por semana e hitos, y ejecuta `csdd ruta`. Calcula cuándo terminas con tus horas reales y pone un semáforo a cada hito. No busca el orden óptimo: evalúa el orden que le das.

---

## 10. Notas por plataforma

### Windows

- Necesitas un **Python 3.11+ real** y Git en el PATH. Si tu Git es portable, agrega su carpeta `cmd` (y `bin`, si usarás `install.sh`) al PATH.
- **Desde la 0.4.2 no hace falta ningún ajuste manual.** `Install.ps1` y `install.sh` prueban cada Python candidato (`python3.13`, `python3.12`, `python3.11`, `python3`, `python`, y `py -3` en PowerShell) y **descartan** el alias vacío de la Microsoft Store, que existe pero no corre. El hook de Claude Code usa el intérprete que se detectó (`python` o `py -3`), no `python3`. El guard y la CLI escriben en UTF-8, así que las tildes no se rompen.
- **Si tu Python está en otra ruta o el instalador no lo encuentra:** indícalo con la variable `CSDD_PYTHON`.
  - PowerShell: `$env:CSDD_PYTHON = "C:\ruta\python.exe"`
  - Bash: `CSDD_PYTHON=/ruta/al/python bash scripts/install.sh <repo>`
- **Solo si usas una versión anterior a la 0.4.2:** activa UTF-8 (`$env:PYTHONUTF8 = "1"`) para que los mensajes del guard no salgan con caracteres rotos, y desactiva el alias de Python de la Tienda (Configuración → Aplicaciones → Configuración avanzada → Alias de ejecución de aplicaciones) o pon un `python3` real en el PATH.
- Los comandos de este manual usan `python`. En macOS/Linux usa `python3`.

### Probar el paquete

Desde la carpeta del paquete: `python -m unittest discover -s tests -v`. Resultado esperado en la 0.4.2:

- **Linux/macOS:** `Ran 24 tests … OK (expected failures=10)`.
- **Windows:** `Ran 24 tests … OK (skipped=1, expected failures=10)`. La prueba omitida usa enlaces simbólicos y un PATH tipo POSIX.

Las 24 se reparten así: 14 pruebas de extremo a extremo (13 pasan en Windows y 1 se omite allí) y 10 **brechas conocidas** que fallan a propósito, en `tests/test_known_gaps.py`. Esas 10 son el trabajo pendiente (ver `docs/ROADMAP-DESARROLLO.md`): cuando una brecha se cierra, su prueba pasa de «falla esperada» a «pasa» y hay que quitarle la marca. Tardan unos 2,5 minutos.

### Kimi Code

Los hooks de Kimi son una **hipótesis sin verificar**: la plantilla está en `.csdd/integrations/kimi/hooks.example.toml`. Antes de confiar en ella, compara su esquema con la documentación oficial de Kimi Code y cópiala a `~/.kimi-code/config.toml`.

Los hooks de Kimi dejan pasar todo si fallan (*fail-open*). Por eso: usa a Kimi **solo dentro de las carpetas de tarea**, y trata el CI y la revisión de Cortana como el control real.

### Claude.ai (web)

En la web no hay hooks, carpetas de trabajo ni buzón. Las skills sirven para **planear, redactar, auditar y entregar comandos**, no para ejecutarlos. Los zips están en `dist/claude-web/`; la guía de subida es `dist/claude-web/INSTALAR-EN-CLAUDE-WEB.md`. Si ya tenías la 0.4.1, solo necesitas volver a subir `csdd-multiagent-orchestrator` (0.4.2); `csdd-mentor` sigue en 1.6.1. Sube siempre `csdd-mentor`, `csdd-multiagent-orchestrator` y `csdd-agent-communication`. De los roles, rinden más Jarvis y El Portero.

### Actualizar a una versión nueva

```
bash scripts/install.sh /ruta/repo --force      # Windows: Install.ps1 -TargetPath … -Force
python3 .csdd/tools/sync_agents.py .
```
`--force` respalda lo anterior en `.csdd-backup/`. Revisa después la guía de migración (`MIGRATION-0.3-to-0.4.md`).

---

## 11. Lo que el sistema NO hace

Conviene saberlo antes de confiar:

- **No es autónomo.** No lanza agentes ni corre en segundo plano. Tú abres cada sesión de Claude Code o Kimi.
- **El análisis de comandos de shell es heurístico.** Un `python -c "open('x','w')"` se le puede escapar. Por eso existen la verificación de alcance de Cortana (`git diff`) y el CI: son el filtro real; el guard avisa temprano.
- **Las carpetas de trabajo aíslan archivos y ramas, no** bases de datos, puertos, secretos ni servicios externos.
- **Las reglas Semgrep no se han ejecutado** en el entorno de construcción. Corre `semgrep --test .csdd/policy/semgrep/` en tu máquina.
- **No es una auditoría ni una certificación.** Reduce errores y deja evidencia; no garantiza que el código sea correcto o seguro.

---

## 12. Lista de comprobación del primer día

- [ ] `python --version` ≥ 3.11 y `git --version` responden.
- [ ] Instalado y **commiteado** (`.csdd/`, `.claude/`, `.agents/`, `AGENTS.md`, `CLAUDE.md`, `.github/`).
- [ ] `csdd doctor` sin `FAIL`.
- [ ] `verify_command` configurado en `system.toml` (o aceptado como `NOT_RUN`).
- [ ] Piloto completo: `session start` → `task new` → `worktree new` → `handoff` → `worktree review` → `result PASS` → `merge` → `session end`.
- [ ] Probaste que el guard bloquea una escritura fuera de las rutas permitidas.
- [ ] Tú eres la única persona que crea autorizaciones y ratifica la constitución.

---

## 13. Glosario

| Término | Significado |
|---|---|
| **CSDD** | Constitutional Spec-Driven Development: se trabaja desde una especificación y unas reglas (la constitución) que se pueden comprobar |
| **Constitución** | Las reglas del proyecto, en 9 artículos. La mantiene el skill `csdd-mentor` |
| **Guard** | Programa que se ejecuta antes de cada acción del agente y la permite o la bloquea |
| **Worktree** | Carpeta de trabajo de Git con su propia rama, para aislar cada tarea |
| **`allowed_paths`** | Las únicas rutas donde el implementador puede escribir en una tarea |
| **REQ-ID** | Identificador de un requisito (`REQ-AUTH-01`) que une especificación, código, pruebas y evidencia |
| **EARS** | Forma estándar de redactar requisitos verificables |
| **ADR** | Registro de una decisión de arquitectura |
| **Handoff** | Entrega del implementador a la revisión |
| **Gate** | Punto de control que debe pasar antes de avanzar |
| **Evidencia** | Comando y salida que demuestran un resultado; sin ella, `PASS` no se acepta |
| **Autorización temporal** | Permiso extra, acotado y con caducidad, que solo crea el propietario (antes «Comunista») |
| **Skill** | Paquete de instrucciones que un agente carga por su nombre |

---

## 14. Dónde seguir leyendo

| Documento | Para qué |
|---|---|
| `README.md` | Visión general del paquete y su estado de verificación |
| `docs/ROADMAP-DESARROLLO.md` | Hoja de ruta v0.5 → v1.0 para quien continúe el desarrollo (empieza por su §0) |
| `EXECUTION_GUIDE.md` | Guía técnica del flujo y los modos de ejecución |
| `CHANGELOG.md` | Qué cambió en cada versión |
| `MIGRATION-0.3-to-0.4.md` | Migrar desde versiones anteriores |
| `docs/adr/` | Decisiones de diseño (y las que esperan tu respuesta) |
| `skills/csdd-multiagent-orchestrator/references/` | Planes, protocolo de sesión, ruta crítica, runtime |

*Manual redactado sobre la v0.4.2. La instalación (`Install.ps1` e `install.sh`), `doctor`, `session start` y las 24 pruebas se ejecutaron en Windows (3-nov-2026); el recorrido completo con agentes reales (Claude Code y Kimi) y las reglas Semgrep no se ejecutaron.*
