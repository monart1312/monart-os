# Cambios pendientes — MONART OS

En orden de prioridad:

## 1. Eliminar boss battle completo
- Borrar sistema boss/combat (~líneas 2163–2510 originales, ahora mezcladas)
- Limpiar `_bossState` de localStorage, referencias CSS y JS
- Ya no hay UI visible del boss — queda código muerto

## 2. Gráfico de progreso diario (sustituto del boss)
- Stacked bar chart por área en página HISTORIAL
- Scroll histórico (ver días anteriores)
- XP system a redefinir

## 3. Misiones diarias
- 3 misiones por día, definición manual
- Aparecen en DASH y RPG día
- Reset a medianoche (distinto de misiones semanales que resetean el lunes)

## 4. Fix tarea con fecha → RPG semana
- Tarea con dueDate debe aparecer en RPG semana en el slot horario correcto
- Ya funciona en RPG día (`renderSchedule`) — falta en vista semana

## 5. Fix edición de eventos existentes
- Poder editar eventos del calendario: título, notas, fecha, área
- Modal `taskDetailSheet` existe pero puede estar incompleto

## 6. Eliminar plus duplicado en TODO
- Dejar solo el FAB global flotante (`.fab-universal`)
- Quitar el + fijo dentro de la página TASKS si existe

---

## Completados ✅

- ~~Fix drag & drop kanban~~ — drag-active 15s, merge defensivo, columna `status` añadida en Supabase
- ~~Fix borrar tarea en móvil~~ — `ttouchStart` ignora botones, área táctil 44px, `ontouchend` handlers
- ~~Confirm dialog nativo~~ — reemplazado por modal custom (`deleteModal`)
- ~~Tarea con hora no aparece en RPG día~~ — `renderSchedule()` interleaves tareas con hábitos
- ~~Google Calendar~~ — integrado via GIS OAuth2, evento creado al añadir tarea con fecha+hora
