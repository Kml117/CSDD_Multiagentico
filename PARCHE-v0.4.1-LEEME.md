# Parche v0.4.0 → v0.4.1 · anti-retrabajo y ruta crítica adaptable

Contiene **solo** los 30 archivos nuevos o modificados (rutas relativas a la carpeta del paquete).

## Aplicarlo
1. Descomprime este zip **encima** de tu carpeta `csdd-multiagent-engineering-system/` v0.4.0 (reemplaza archivos con el mismo nombre).
2. Verifica: `python3 -m unittest discover -s tests -v` → 12 pruebas OK.
3. Regenera las versiones web si las necesitas: `python3 scripts/build_claude_web.py`.
4. En cada repo donde ya instalaste v0.4.0: `bash scripts/install.sh <repo> --force` y luego `MIGRATION-0.3-to-0.4.md` (sección 0.4.0 → 0.4.1, incluye cómo adoptarlo en Corbeta).

## Skills que cambiaron (las otras 5 siguen igual que en v0.4.0)
- `csdd-multiagent-orchestrator` 0.4.1: protocolo de sesión, ruta crítica, `sesion.py`, `ruta_critica.py`, plantillas, guard.
- `csdd-executive-supervisor` 0.4.1: dueño de la ruta crítica y del cierre de sesión.
- `csdd-mentor` 1.6.1: anti-retrabajo como control del Art. IX.

Decisión pendiente tuya: ADR-0004, parte 3 (¿artículo propio "Estado verificable"?).
