# docs/ — estructura y rutina

| Archivo | Para qué |
|---|---|
| `PROJECT.md` | Estado actual: versión, stack, módulos, problemas, próximo objetivo |
| `ROADMAP.md` | Backlog: ideas/features/bugs sin ejecutar |
| `CHANGELOG.md` | Historial comprimido por versión (Agregado/Corregido/Mejorado) |
| `daily.md` | Una línea por tarea cerrada, más nuevo arriba |
| `sessions/YYYY-MM-DD.md` | Detalle por fecha: Solución / Cambios / Prueba / Resultado esperado |
| `../TODO.md` | Tareas de la sesión activa |
| `../README.md` | Referencia de la API HTTP (para clientes externos) |

## Rutina

1. Bug/feature/idea nuevo → `ROADMAP.md`.
2. Se ataca ahora → mover a `TODO.md` (sacarlo del roadmap).
3. Si hay decisiones no obvias → documentar en `sessions/<hoy>.md` antes de codear.
4. Implementar → probar manualmente.
5. Cerrar: `[x]` en `TODO.md` + entrada en `sessions/<hoy>.md` + línea en `daily.md`.

## Cierre de versión

1. Comprimir `sessions/` del período → `CHANGELOG.md`.
2. Vaciar `TODO.md`, limpiar `ROADMAP.md`.
3. Actualizar versión en `package.json` (`npm version X.Y.Z --no-git-tag-version`), `PROJECT.md` y `CLAUDE.md`.
4. `npm run make` y probar instalador.
5. Commit + `git tag vX.Y.Z`.
