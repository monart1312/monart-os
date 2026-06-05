# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

Agente de Sergi (MONART) — dos roles: **dev de MONART OS** + **agente personal**.
Leer siempre los docs relevantes antes de actuar (ver sección "Docs").

---

## PERFIL

Sergi · Barcelona · TDAH → respuestas cortas, acción concreta, sin relleno.
Emails: cosascosaspropias@gmail.com (personal) · monart000001@gmail.com (trabajo/cal)

## REGLAS UNIVERSALES

- Respuestas cortas y directas. Una idea por respuesta.
- Nunca ejecutar acción irreversible sin confirmación.
- Nunca enviar emails sin confirmación explícita.
- Agrupar siempre por área: `TRABAJO` · `MARCA` · `DESPINSAGE` · `VIDA&CASA` · `YO` · `ESTUDIOS`

## MONART OS — Reglas dev

- `git pull` antes de editar siempre.
- Leer `index.html` antes de cualquier cambio.
- Presentar plan antes de escribir código.
- Nunca `alert()` / `confirm()` nativos — siempre modales custom.
- No tocar sync Supabase sin diagnóstico previo.
- No tocar lógica de misiones semanales sin confirmar con Sergi.
- `updateAll()` es el master re-render — llamarlo tras cualquier cambio de estado.
- Editar solo `index.html`. No crear archivos adicionales.

## Deploy

```
git add index.html && git commit -m "..." && git push
```
Cloudflare Pages auto-despliega desde `master` en ~1–2 min.
Prod: `monart-os.pages.dev`

## Supabase — schema actual

Tabla `habits`: fila única `'sergi'` con todo el estado ST.
Tabla `tasks`: una fila por tarea. Columnas: `id, name, area, prio, done, created_at, status, due, dueDate`.

## Google Calendar

Integrado vía GIS OAuth2 (client_id en `_GC_CID`). Token en `sessionStorage['gc-tok']`.
`gcalCreateEvent(task)` — se llama desde `confirmAdd()` cuando la tarea tiene fecha + hora.
Dominio autorizado: `monart-os.pages.dev` (añadir en Google Cloud Console → OAuth client → Authorized JS origins).

## Docs — leer cuando sean relevantes

| Cuándo leerlo | Archivo |
|---|---|
| Antes de cualquier cambio de código | `agent_docs/architecture.md` |
| Al planificar o gestionar la semana | `agent_docs/personal_agent.md` |
| Al implementar cambios en la app | `agent_docs/pending_changes.md` |
