# HISTORIAL_MODIFICACIONES.md

Ruta destino en el repo: `la-expansion/HISTORIAL_MODIFICACIONES.md`

## Regla de uso (leer antes de cada sesión)

Después de cada bloque de trabajo:

1. Probar que funcione (juntos, Ramon + Claude).
2. Actualizar `CONTEXTO_PROYECTO.md` y/o `ARQUITECTURA.md` si hubo cambios de alcance, stack o estructura.
3. Agregar una entrada abajo con: fecha, qué se hizo, qué se probó, qué falta.
4. Hacer commit.
5. Recién ahí empezar la siguiente parte.

No saltarse pasos aunque el cambio parezca pequeño — el objetivo es que cualquier sesión futura pueda retomar el proyecto sin perder contexto.

---

## Entradas

### 2026-08-09 — Sesión 1: Definición de alcance y documentos de contexto

**Qué se hizo:**

- Se analizaron funcionalidades típicas de sitios de partidos/movimientos políticos (ejemplos: PRM, PRI, PLD en RD).
- Se definió el listado de funcionalidades v2, ajustado para movimiento (no partido): se eliminó Transparencia, Estatutos, Directorio de estructuras, Multi-idioma.
- Se acordó estructura de repos: carpeta `la-expansion/` con `expansion-backend/` y `expansion-frontend/` adentro.
- Se crearon los tres documentos de contexto (este, `CONTEXTO_PROYECTO.md`, `ARQUITECTURA.md`).

**Qué se probó:** Nada aún — sesión de planificación, sin código.

**Qué falta / pendiente para próxima sesión:**

- Confirmar con Ramon: newsletter, donaciones, ubicación de Contacto.
- Scaffolding real de `expansion-backend` y `expansion-frontend` (Fase 0).
- Conexión backend↔MongoDB Atlas.
- Deploy mínimo (Vercel + Render) funcionando end-to-end.
