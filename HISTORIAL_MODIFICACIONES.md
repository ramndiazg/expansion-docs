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

---

### 2026-08-09 — Sesión 2: Esqueleto del backend funcionando

**Qué se hizo:**

- Se creó la estructura de carpetas `src/{models,controllers,routes,middleware}` en `expansion-backend`.
- Se creó `src/server.js`: Express + CORS + ruta `/api/health` + conexión a MongoDB vía Mongoose.
- Se creó `.env.example` y `.gitignore` (excluye `node_modules/` y `.env`).
- Se actualizó `package.json` (scripts `start`/`dev`, `main`, se agregó `nodemon` como devDependency).
- Se creó cluster **MongoDB Atlas** desde cero: tier **Free (M0)**, AWS, `us-east-1`, nombre `Cluster0`. Se agregó restricción de presupuesto ("todo gratis en lo posible") a `CONTEXTO_PROYECTO.md`.
- Se creó usuario de base de datos y se habilitó Network Access `0.0.0.0/0`.
- Se configuró `.env` local con el connection string real (no se comparte en chat por seguridad).

**Qué se probó:**

- `npm run dev` levanta el servidor sin errores, conecta a Atlas correctamente.
- `GET http://localhost:4000/api/health` responde `{"status":"ok","timestamp":"..."}`. ✅

**Qué falta / pendiente para próxima sesión:**

- Hacer commit de este bloque (backend skeleton + docs actualizados).
- Confirmar con Ramon: newsletter, donaciones, ubicación de Contacto (sigue pendiente).
- Siguiente bloque: página de Inicio básica en `expansion-frontend` + primera conexión frontend→backend (fetch a `/api/health` para validar el pipeline completo).
- Deploy a Vercel/Render (después de validar frontend↔backend localmente).

---

### 2026-08-09 — Sesión 3: Modelos de datos del backend

**Qué se hizo:**

- Se crearon los 5 esquemas Mongoose en `expansion-backend/src/models/`: `Noticia.js`, `Miembro.js`, `Voluntario.js`, `Evento.js`, `Encuesta.js`.
- `Noticia` incluye generación automática de `slug` a partir del título (hook `pre('validate')`).
- `Encuesta` usa subdocumentos para las opciones (texto + contador de votos).
- Se documentaron los 5 esquemas completos en `ARQUITECTURA.md`.
- Se movieron los tres documentos de contexto a su propio repo: `la-expansion-docs` (github.com/ramndiazg/la-expansion-docs).

**Qué se probó:**

- Los 5 modelos cargan sin errores de sintaxis vía `node -e "require(...)"`. ✅
- (Pendiente probar inserción/lectura real contra MongoDB Atlas — se hará junto con las rutas/controladores.)

**Qué falta / pendiente para próxima sesión:**

- Hacer commit de este bloque en `expansion-backend`.
- Confirmar con Ramon: newsletter, donaciones, ubicación de Contacto (sigue pendiente desde sesión 1).
- Crear rutas y controladores CRUD básicos (al menos Noticia, para validar un modelo completo end-to-end antes de replicar el patrón a los demás).
- Retomar el bloque de frontend: página de Inicio + fetch a `/api/health`.

---

### 2026-08-10 — Sesión 4: Controladores, rutas y CRUD funcionando

**Qué se hizo:**

- Se crearon los 5 controladores en `expansion-backend/src/controllers/` (CRUD estándar para Noticia, Miembro, Voluntario, Evento; Encuesta incluye endpoint especial `votar`).
- Se crearon las 5 rutas correspondientes en `expansion-backend/src/routes/` y se registraron en `server.js` bajo `/api/noticias`, `/api/miembros`, `/api/voluntarios`, `/api/eventos`, `/api/encuestas`.
- **Bug encontrado y resuelto**: el hook `pre('validate')` de `Noticia.js` (generación de slug) causaba `"next is not a function"` con Mongoose 9.x. Se corrigió quitando el parámetro `next` del callback (ver detalle en `ARQUITECTURA.md`).
- Se documentó la tabla completa de endpoints en `ARQUITECTURA.md`.

**Qué se probó:**

- `POST /api/noticias` — crea correctamente, slug autogenerado (`noticia-de-prueba`). ✅
- `GET /api/noticias` — lista la noticia creada. ✅
- Los demás modelos (Miembro, Voluntario, Evento, Encuesta) siguen el mismo patrón de controlador/ruta pero **no se probaron individualmente todavía** — queda como pendiente antes de darlos por completamente validados.

**Qué falta / pendiente para próxima sesión:**

- Hacer commit de este bloque en `expansion-backend`.
- (Opcional) Probar al menos un endpoint de cada modelo restante para confirmar que no hay bugs similares al de Noticia.
- (Opcional) Borrar la noticia de prueba de la base de datos, o dejarla como dato de ejemplo.
- Confirmar con Ramon: newsletter, donaciones, ubicación de Contacto (sigue pendiente desde sesión 1).
- Retomar el bloque de frontend: página de Inicio + fetch a `/api/health`.

---

### 2026-08-10 — Sesión 5: Frontend conectado al backend — Fase 0 completa

**Qué se hizo:**

- Se confirmó estructura del frontend: App Router, TypeScript, sin `src/`.
- Se creó `.env.local` con `NEXT_PUBLIC_API_URL`.
- Se reemplazó `app/page.tsx` (boilerplate de create-next-app) por una página que hace fetch a `/api/health` del backend y muestra el estado de conexión.
- Se actualizó `metadata` en `app/layout.tsx` (title/description de "La Expansión").

**Qué se probó:**

- `npm run dev` en frontend, con backend corriendo en paralelo. ✅
- `http://localhost:3000` muestra "Estado del backend: ok" con timestamp real. ✅ Pipeline completo (frontend → backend → MongoDB Atlas) validado end-to-end.

**Fase 0 — Esqueleto técnico: COMPLETADA.** Los tres repos (`expansion-backend`, `expansion-frontend`, `la-expansion-docs`) están creados, vinculados a GitHub, y el flujo de datos frontend↔backend↔DB funciona en local.

**Qué falta / pendiente para próxima sesión:**

- Hacer commit de este bloque en `expansion-frontend`.
- Decidir siguiente prioridad: (a) deploy a Vercel/Render para tener el sitio accesible públicamente, o (b) seguir construyendo features (páginas institucionales, CRUD real de noticias en el frontend, panel admin).
- Sigue pendiente probar Miembro/Voluntario/Evento/Encuesta individualmente en el backend (desde sesión 3-4).
- Confirmar con Ramon: newsletter, donaciones, ubicación de Contacto (pendiente desde sesión 1).

---

### 2026-08-10 — Sesión 6: Sistema de diseño y páginas institucionales

**Qué se hizo:**

- Se registraron dos decisiones de arquitectura en `CONTEXTO_PROYECTO.md`: (1) autenticación será JWT propio, no NextAuth, mismo patrón que Muvo; (2) sin identidad visual previa, se propuso paleta y tipografía desde cero.
- Se definió el sistema de diseño: paleta cream/ink/amber/teal, tipografía Fraunces (display) + Inter (body), motivo de anillos concéntricos como elemento de firma.
- Se actualizó `app/globals.css` con los tokens de color vía `@theme inline` (Tailwind v4), se quitó el dark mode automático del boilerplate.
- Se actualizó `app/layout.tsx`: carga de fuentes, se agregó `<Navbar />` y `<Footer />` globales.
- Se crearon `components/Navbar.tsx` y `components/Footer.tsx`.
- Se reemplazó `app/page.tsx` con el Home de marca (hero + sección de pilares) — esto reemplaza la versión de sesión 5 que mostraba el estado de `/api/health`.
- Se crearon `app/sobre-el-movimiento/page.tsx` y `app/liderazgo/page.tsx`.
- Todo el contenido de texto es placeholder explícito, marcado con comentarios `{/* PLACEHOLDER */}`.

**Qué se probó:**

- Las tres páginas cargan sin errores. ✅
- Confirmado visualmente que los estilos (colores, tipografías, tamaños) se aplican correctamente — Tailwind v4 tomó bien los tokens custom. ✅

**Qué falta / pendiente para próxima sesión:**

- Hacer commit de este bloque en `expansion-frontend`.
- Reemplazar todo el contenido placeholder con copy real cuando Ramon/el equipo lo tenga listo.
- El botón "Afíliate" da 404 (esperado) — construir esa página cuando se aborde el bloque de membresía.
- El check de `/api/health` visible en Home se perdió al reemplazar la página — decidir si se quiere en algún lado (ej. página de diagnóstico aparte).
- Confirmar con Ramon: newsletter, donaciones, ubicación de Contacto (pendiente desde sesión 1).
- Sigue pendiente probar Miembro/Voluntario/Evento/Encuesta individualmente en el backend.
- Decidir próximo bloque: noticias visibles en frontend, panel admin, o deploy a producción.
