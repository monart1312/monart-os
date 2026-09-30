# MONART OS — Arquitectura

App: HTML + CSS + JS puro · Supabase · Cloudflare Pages
Repo: github.com/monart1312/monart-os · Prod: monart-os.pages.dev
**Un solo archivo: `index.html`** (~6400 líneas). Sin build system, sin npm.

## Estructura por secciones

| Líneas aprox. | Contenido |
|--------|-----------|
| 9–2182 | CSS completo (`:root` vars, componentes, páginas, modales) |
| 2183–2564 | HTML DOM estático (sidenav, 7 páginas: DASH, HABITS, TASKS, CAL, HISTORIAL, VISION + modales) |
| 2565– | `<script>` — todo el JS |
| ~2566–2660 | Constantes: `QUOTES`, `LEVELS`, `AREA_COLORS`, `SCHEDULE`, `ALL_HABITS`, `EVENTS` |
| ~2662–2780 | Supabase config + REST wrappers (`sbGetST`, `sbSaveST`, `sbGetAllTasks`, `sbUpsertTask`, `sbDeleteTask`) |
| ~2782–2930 | State management: `ST`, `TASKS`, `save()`, `syncToCloud()`, `loadFromCloud()` |
| ~2930–3010 | Level system, page routing, pixel art canvas, utility fns |
| ~3011–3246 | `renderSchedule()` — horario RPG día (hábitos + tareas con hora intercaladas) |
| ~3247–3310 | `updateAll()` — master re-render |
| ~3310–3550 | Render: hábitos, misiones, calendario, historial, stats |
| ~3552–3700 | Touch drag & drop (`ttouchStart/Move/End`) + keyboard drag |
| ~3700–3930 | Task wizard: `openAddPanel`, `renderAddStep`, `confirmAdd`, `gcalCreateEvent`, `_gcPost` |
| ~3927–4140 | `renderTodoAll()` — vista TODO completa (kanban riding + backlog por área) |
| ~4140–4213 | `renderKanban()` — vista kanban pura |
| ~4213– | Calendar render, history, Vision page, event editor, modales |
| ~6090–6134 | Init/boot (`init()`, `setInterval loadFromCloud 30s`) |

## Páginas

| ID | Nombre | Descripción |
|----|--------|-------------|
| `dash` | DASH | Hub principal: nivel, XP, misiones, tareas, hábitos rápidos |
| `habits` | HABITS | Hábitos del día con checkboxes |
| `tasks` | TASKS | Kanban + TODO por área (tabs: RIDING / BACKLOG / TODAS / KANBAN) |
| `cal` | CALENDAR | Vista mensual + semanal con eventos y tareas |
| `historial` | HISTORIAL | Log diario de XP + gráficos de progreso |
| `vision` | VISION | Timeline roadmap 2026-2027 con tarjetas por bloque |

## Estado

- `ST` — `{ xp, streak, day, completed, history, missions, missionsWeek, _prevLvl, updated_at }` → `localStorage['monart-os']` + Supabase `habits` (row `'sergi'`)
- `TASKS` — array → `localStorage['monart-tasks']` + Supabase `tasks`
- No usar `_bossState` (sistema boss battle eliminado pendiente limpieza)

## Sync

Motor de sync reescrito el 2026-09-30 tras un incidente: una pestaña vieja del móvil subió la fila `habits` entera y machacó las misiones de la semana.

Fila `habits` — merge a 3 bandas por columna (`SYNC_COLS`):
- `_syncBase` (localStorage `monart-sync-base`) = último valor conocido de cada columna en la nube, en JSON canónico (`_canon`, claves ordenadas: jsonb las reordena).
- Columna "sucia" = local ≠ base. `syncToCloud()` hace PATCH SOLO de las columnas sucias, condicionado a `updated_at`; si hay conflicto (0 filas) → baja, fusiona, reintenta (3 veces; el 4º va sin condición).
- `loadFromCloud()` (pull): columna no tocada en local → gana la nube (así el secretario, que escribe directo en Supabase, siempre se respeta). Ambos cambiados → `xp` suma deltas; `completed`/`daily_log`/`roadmap` se fusionan por clave; resto gana local, salvo que el cambio local lleve >6h sin subir (`monart-sync-dirty-at`).
- `save()` no sube nada hasta el primer pull correcto de la sesión (`_cloudReady`).
- Automatismos de fecha (`initMissions`, `initDailyMissions`, `checkAutoClose`) corren SOLO tras cada pull (`_runDateAutomations`). `lastDayClose` se comparte en `roadmap.__lastDayClose` y nunca retrocede.
- Pull + push serializados en `_syncQueue`. Pull al volver a la pestaña (`visibilitychange`). `APP_BUILD` recarga pestañas con código viejo. Al cerrar: `_flushOnExit` sube solo columnas sucias.

Tasks:
- `sbUpsertTask(t)` — pasar SIEMPRE la tarea completa (envía `due`/`duedate`/`notes` aunque sean null).
- Subidas pendientes en `monart-tasks-pending`: el pull no las borra y las reintenta. Borrados fallidos (`monart-deleted-tasks`) se reintentan en cada pull.
- Drag activo: `document.body.classList.contains('drag-active')` bloquea overwrite de tasks durante 15s tras soltar.

**Antes de tocar el sync: leer esta sección. NO modificar missions weekly logic sin confirmar con Sergi.**

## Modales principales

| ID | Función |
|----|---------|
| `todoAddModal` | Wizard añadir tarea (5 pasos) |
| `deleteModal` | Confirmar borrar tarea (`openDeleteModal` / `confirmDelete`) |
| `missionModal` | Editar misión semanal |
| `taskDetailSheet` | Detalle/edición tarea |
| `morningBootModal` | Checklist matutino |
| `weeklyReviewModal` | Revisión semanal |
| `quickCaptureModal` | Captura rápida |
| `catalogarModal` | Catalogar ítem |

## Events

`allEvents()` = `[...EVENTS, ...getUserEvents()]`. Siempre usar `allEvents()`, nunca `EVENTS` directamente.
`saveUserEvents()` / `getUserEvents()` → `localStorage['monart-user-events']`.

## Task wizard

`openAddPanel()` → `renderAddStep()` (Name → Area → Priority → Date → Confirm) → `confirmAdd()`.
Estado en `_addWiz`. Tarea con fecha crea user event. Tarea con fecha+hora llama `gcalCreateEvent(task)`.

## Touch drag

`ttouchStart(e, id)` — ignora toques en `<button>` (`e.target.closest('button')`).
`ttouchMove` — mueve ghost + detecta columna kanban destino.
`ttouchEnd` — hace drop si hay columna bajo el dedo.

## CSS tokens

`:root`. Font display: `Syne` (`--ff`). Mono: `JetBrains Mono` (`--fm`). Fondo: `#04040a`. Accent: `#7b6fff` (`--accent`). Accent2: `#ff4d6d`.

## Diseño

- Mobile-first. Touch events en todos los interactivos. Mínimo 44×44px táctiles.
- Colores calendario: Cyan=Despinsage · Verde=Personal · Rosa=Cumpleaños · Dorado=Trabajo
- Botones `.t-del`: siempre visibles en móvil (`@media max-width:600px { opacity:1 !important }`).
