# HISTORIAL_MODIFICACIONES.md

Ruta destino en el repo: `la-expansion-docs/HISTORIAL_MODIFICACIONES.md`

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

---

### 2026-08-10 — Sesión 7: Rediseño (v2) + noticias multimedia (pendiente de prueba)

**Qué se hizo:**

- Feedback de Ramon sobre el diseño v1: se sentía anticuado. Rediseño completo:
  - Paleta: blanco en vez de crema, `--ink` azul-marino (`#101828`) en vez del tono oscuro anterior, acento `--blue` en vez de ámbar, `--slate` en vez de teal.
  - Tipografía: Space Grotesk reemplaza a Fraunces (100% sans-serif).
  - Se ajustó el azul a un tono más suave (`#4E7FDB`) tras primer feedback.
- Se actualizaron: `globals.css`, `layout.tsx`, `Navbar.tsx`, `Footer.tsx`, `app/page.tsx`, `sobre-el-movimiento/page.tsx`, `liderazgo/page.tsx`, `noticias/page.tsx`, `noticias/[slug]/page.tsx`.
- Se agregó menú hamburguesa responsive en `Navbar.tsx` (el movimiento se consume mayoritariamente desde móvil — principio mobile-first registrado en `ARQUITECTURA.md`).
- Se rediseñó el hero del Home: título más grande con efecto degradado en la palabra "expande", y presencia de Mario Díaz (nombre + cargo + avatar placeholder) fuera de la página de Liderazgo, por pedido explícito de dar más peso personalista al Secretario General.
- Se extendió `Noticia.js` con `imagenesAdicionales` (array de URLs) y `videoUrl` (YouTube), para soportar noticias con fotos y video.
- Se actualizó `noticias/[slug]/page.tsx` para renderizar imagen destacada, galería de imágenes adicionales, y embed de YouTube.

**Qué se probó:**

- Cambios visuales (color, tipografía, hero, menú) confirmados por Ramon: "se ve mucho mejor". ✅
- Campos nuevos de Noticia (`imagenesAdicionales`, `videoUrl`) y su renderizado en el frontend: **NO probados todavía** — queda pendiente para la próxima sesión antes de hacer commit de esa parte específica.

**Qué falta / pendiente para próxima sesión:**

- Probar creación de una noticia con imagen destacada, imágenes adicionales y video de YouTube (vía curl, con URLs de ejemplo) y confirmar que se ve bien en `/noticias/[slug]`.
- Una vez probado: commit de `expansion-backend` (modelo) y `expansion-frontend` (rediseño completo + noticia multimedia) — **todavía sin commitear**, sesión 6 fue el último commit real.
- Nota para el futuro panel admin: el `contenido` de la noticia sigue siendo texto plano — un editor de texto enriquecido (con imágenes/formato dentro del párrafo) es trabajo del panel admin, no de esta fase.
- Confirmar con Ramon: newsletter, donaciones, ubicación de Contacto (pendiente desde sesión 1).
- Sigue pendiente probar Miembro/Voluntario/Evento/Encuesta individualmente en el backend.
- Decidir próximo bloque: panel admin o deploy a producción.

---

### 2026-08-10 — Sesión 8: Auth JWT, roles, panel admin (Noticias)

**Qué se hizo:**

- Se diseñó el sistema de roles con Ramon: **Publicador** (uso diario — noticias, comentarios, sus propias encuestas) y **Admin** (dashboard, dominio sobre cualquier encuesta, y única acción exclusiva: desactivar cuentas — activar lo puede hacer cualquiera para no crear cuello de botella).
- Se diseñó el sistema de encuestas públicas/virales (compartibles en redes, votación anónima, invitación opcional a "Inscribirse" — término elegido para sonar a menos compromiso que "afiliarse"). **Diseño acordado, código pendiente para próxima sesión.**
- Se diseñó el sistema de comentarios: requieren ser Miembro afiliado y aprobado (no cuenta nueva, no anónimo), con pre-moderación por Publicador/Admin.
- **Backend**: se instalaron `bcryptjs` y `jsonwebtoken`. Se creó `Usuario.js` (cuentas del panel), `Comentario.js`; se actualizaron `Miembro.js` (+ `passwordHash`) y `Encuesta.js` (+ `creadoPor`). Se creó `middleware/auth.js` (`verifyToken`, `requireRolUsuario`, `requireMiembro`). Se creó `authController`/`authRoutes` (login de Usuario y de Miembro). Se creó `comentarioController`/`comentarioRoutes`. Se protegieron `noticiaRoutes` (create/update/delete requieren Usuario admin/publicador) y se actualizó `encuestaController`/`encuestaRoutes` (crear requiere auth, cerrar respeta dueño salvo admin, votar sigue público). Se creó script `scripts/crearUsuarioAdmin.js` (patrón dry-run + `--confirmar`, igual que en Muvo).
- **Frontend**: se creó `lib/auth.ts` (sesión en `localStorage`, sin NextAuth). Se creó `/admin/login`, `/admin/layout.tsx` (protección de rutas), `/admin` (dashboard), `/admin/noticias` (listado + publicar/despublicar), `/admin/noticias/nueva` (crear).

**Qué se probó:**

- Usuario admin creado con el script (`mario@laexpansion.do`). ✅
- Login desde `/admin/login` funcionando, redirección automática a login si no hay sesión. ✅
- Creación de noticia desde el panel — cae correctamente como `borrador` (comportamiento esperado del modelo, no bug). ✅
- Botón Publicar/Despublicar en `/admin/noticias` — cambia estado vía `PUT /api/noticias/:id`, y la noticia publicada aparece en `/noticias` (sitio público). ✅
- Protección de rutas: `POST/PUT/DELETE /api/noticias` ahora exige token de Usuario válido con rol admin/publicador.

**Qué NO se probó todavía:**

- Login de Miembro (`/api/auth/miembro-login`).
- Sistema de comentarios completo (crear, listar aprobados, bandeja de pendientes, moderar).
- Cierre de encuestas respetando dueño (`PUT /api/encuestas/:id/cerrar`).
- Desactivación de cuentas de Usuario (no hay endpoint para esto todavía — falta construirlo).
- Todo el sistema de encuestas públicas/Inscrito — solo quedó diseñado, no hay código.

**Qué falta / pendiente para próxima sesión:**

- Hacer commit de este bloque en `expansion-backend` y `expansion-frontend` (grande — auth, roles, panel).
- Construir: endpoint y UI para activar/desactivar cuentas de Usuario (recordar: solo admin desactiva).
- Construir: registro/login de Miembro desde el frontend público, y flujo de comentarios (form en noticia + bandeja de moderación en el panel).
- Construir: modelo `Inscrito`, página pública de encuesta compartible, votación anónima con protección `localStorage`, flujo de inscripción opcional post-voto.
- Construir en el panel: gestión de Miembro (aprobar/rechazar afiliaciones), Voluntario, Evento — actualmente el panel solo tiene Noticias.
- Agregar edición de noticias existentes (título/contenido) — el panel actual solo permite crear y cambiar estado.
- Sigue pendiente probar Miembro/Voluntario/Evento/Encuesta (CRUD original) individualmente.
- Confirmar con Ramon: newsletter, donaciones, ubicación de Contacto (pendiente desde sesión 1).
- Decidir: seguir con las piezas pendientes del panel, o pasar a deploy de lo que ya existe.

---

### 2026-08-10 — Sesión 9: Fix de bug (setState en useEffect)

**Qué se hizo:**

- Ramon detectó que no hay ningún botón/enlace visible en el sitio hacia `/admin/login` (ni hacia un futuro login de Miembro) — **anotado como pendiente para próximo bloque**, no se resolvió en esta sesión.
- Se corrigió bug real: `app/admin/noticias/page.tsx` llamaba una función async con `setState` síncrono dentro de `useEffect`, disparando el lint `react-hooks/set-state-in-effect`. Se separó el fetch de la actualización de estado (ver detalle en `ARQUITECTURA.md`).

**Qué se probó:**

- El error de lint/consola desapareció. ✅
- El listado de noticias en `/admin/noticias` sigue cargando y funcionando igual que antes. ✅

**Qué falta / pendiente para próxima sesión:**

- Hacer commit de este fix en `expansion-frontend`.
- **Agregar acceso visible a login** — de Usuario (panel) y de Miembro (comentar) — en el sitio público.
- Construir: endpoint y UI para activar/desactivar cuentas de Usuario.
- Construir: registro/login de Miembro desde el frontend público, y flujo de comentarios.
- Construir: modelo `Inscrito` y encuestas públicas compartibles.
- Construir en el panel: gestión de Miembro, Voluntario, Evento.
- Agregar edición de noticias existentes (título/contenido).
- Sigue pendiente probar Miembro/Voluntario/Evento/Encuesta (CRUD original) individualmente.
- Confirmar con Ramon: newsletter, donaciones, ubicación de Contacto (pendiente desde sesión 1).

---

### 2026-08-10/11 — Sesión 10: Login unificado, comentarios, datos geográficos reales

**Qué se hizo:**

- Se unificó el login: un solo endpoint `POST /api/auth/login` y una sola página `/login`; el backend detecta si el email es de Usuario o Miembro y responde con `tipo`, el frontend redirige a `/admin` o a `/` según corresponda. Se borraron `/admin/login` y `/miembro/login`.
- Se creó `app/afiliate/page.tsx` (no se había creado en el bloque original — desfase detectado y corregido), `lib/authMiembro.ts`, `components/Comentarios.tsx` (mismo caso, faltaban).
- Se corrigió el mismo bug de `setState` síncrono en `useEffect` en `app/admin/layout.tsx` (patrón: envolver en `Promise.resolve().then()` cuando no hay `fetch` de por medio) y en `components/Navbar.tsx`.
- Se integró `Comentarios` en `app/noticias/[slug]/page.tsx`.
- **Datos geográficos**: se descargó (no se escribió a mano) un dataset público de provincias/municipios de RD, se combinó en `lib/provinciasMunicipios.ts` (32 provincias, 158 municipios). Se implementaron selects dependientes provincia→municipio en `/afiliate`. Se decidió dejar `sectorInteres` como texto libre (no geográfico) y no implementar un tercer nivel de sector geográfico por ahora, para no cargar un dataset mucho más grande en un sitio mobile-first.
- Se agregó campo de confirmación de contraseña en `/afiliate`, con validación antes de enviar.

**Qué se probó:**

- Login con email de Usuario (admin) → redirige a `/admin`. ✅
- Login con email de Miembro aprobado → redirige a `/` con sesión activa. ✅
- Registro completo en `/afiliate`, incluyendo selects de provincia/municipio funcionando (municipio se resetea al cambiar provincia, deshabilitado hasta elegir provincia). ✅
- Comentar en una noticia estando logueado como Miembro — guarda como `pendiente`. ✅
- Confirmación de contraseña — rechaza si no coinciden, envía si coinciden. ✅

**Qué falta / pendiente para próxima sesión:**

- Hacer commit de este bloque (backend: login unificado; frontend: todo lo anterior).
- Construir: UI de moderación de comentarios (el dato ya se guarda, falta la bandeja para aprobar/rechazar).
- Construir: endpoint y UI para activar/desactivar cuentas de Usuario.
- Construir: modelo `Inscrito` y encuestas públicas compartibles (diseño ya acordado en sesión 8).
- Construir en el panel: gestión de Miembro (aprobar/rechazar — ahora mismo se hace solo por curl), Voluntario, Evento.
- Agregar edición de noticias existentes.
- Decidir si corregir las inconsistencias de nombres en el dataset geográfico ("Baoruco"/"Bahoruco", "Sanchez"/"Sánchez Ramírez").
- Sigue pendiente probar Voluntario/Evento/Encuesta (CRUD original) individualmente.
- Confirmar con Ramon: newsletter, donaciones, ubicación de Contacto (pendiente desde sesión 1).

---

### 2026-08-11 — Sesión 11: Gestión de miembros, moderación de comentarios, UserMenu

**Qué se hizo:**

- Se registraron decisiones: encuestas solo votables por Miembro (se descarta `Inscrito`/voto anónimo de sesión 8); estructura de paneles aclarada (Admin+Publicador comparten `/admin`, Miembro tendrá `/cuenta` propia); Admin podrá cambiar contraseñas; recuperación por correo queda para cuando se resuelva el servicio de email — todo esto anotado en `CONTEXTO_PROYECTO.md`, sin construir todavía salvo lo de encuestas (que solo afecta diseño futuro).
- **Seguridad**: se descubrió que `GET /api/miembros` había quedado público desde sesión 8 — se corrigió, ahora solo Admin.
- Se creó `app/admin/comentarios/page.tsx` (bandeja de moderación) y `app/admin/miembros/page.tsx` (aprobar/rechazar afiliaciones con filtro por estado).
- Se corrigió bug: el botón "Aprobar" de comentarios usaba un token de color (`bg-teal`) que ya no existe en la paleta — quedaba sin estilo visible.
- Se corrigió bug: mismo patrón de `setState` síncrono en `useEffect` en `Navbar.tsx`.
- Se descubrió y corrigió: el Navbar público se renderizaba superpuesto encima del panel admin (ambos vienen del layout raíz). Se creó `components/SiteChrome.tsx` para excluir Navbar/Footer dentro de `/admin/*`.
- Se reemplazó el texto plano "Iniciar sesión"/nombre en el Navbar por `components/UserMenu.tsx`: círculo con iniciales + dropdown (desktop), variante en línea sin dropdown flotante para móvil (el dropdown absoluto anidado rompía el layout del menú hamburguesa). Detecta sesión de Usuario o Miembro y muestra opciones contextuales.
- Se extendió `lib/auth.ts` con el mismo sistema de eventos de sesión que ya tenía `lib/authMiembro.ts`, para que el Navbar reaccione en vivo también a login/logout de Usuario.
- El botón "Afíliate" ahora es condicional: oculto si hay cualquier sesión activa.

**Qué se probó:**

- Bandeja de comentarios pendientes — aprobar/rechazar funcionando, botón con color correcto. ✅
- Gestión de miembros — filtros por estado, aprobar/rechazar. ✅
- Navbar reacciona en vivo a login/logout sin recargar (Usuario y Miembro). ✅
- Panel admin ya no muestra el Navbar público superpuesto. ✅
- Dropdown de cuenta en desktop y variante móvil sin romper layout. ✅
- Botón Afíliate oculto correctamente con sesión activa. ✅

**Qué falta / pendiente para próxima sesión:**

- Hacer commit de este bloque (backend: fix de seguridad en miembros; frontend: todo lo anterior).
- Construir `/cuenta` (área de Miembro): cambiar contraseña, ver encuestas votadas, ver noticias comentadas — el link ya existe en el UserMenu pero da 404.
- Construir: endpoint y UI para activar/desactivar cuentas de Usuario, y cambio de contraseña por Admin.
- Construir: modelo `Inscrito` — **ya no aplica**, decisión revertida (ver arriba). Actualizar cualquier referencia futura a este modelo como descartada.
- Construir: encuestas públicas (ahora solo para Miembros, diseño simplificado respecto a sesión 8).
- Construir en el panel: gestión de Voluntario, Evento.
- Agregar edición de noticias existentes.
- Sigue pendiente probar Voluntario/Evento/Encuesta (CRUD original) individualmente.
- Confirmar con Ramon: newsletter, donaciones, ubicación de Contacto (pendiente desde sesión 1).

---

### 2026-08-11 — Sesión 12 (parte 1): Limpieza completa de documentos de contexto

**Qué se hizo:**

- Reescritura completa de `ARQUITECTURA.md`: eliminadas dos secciones duplicadas de "Endpoints del backend", modelos de datos consolidados en un solo lugar (antes repartidos en dos), estructura de carpetas actualizada (reflejaba sesión 1, congelada con `models/`, `controllers/`, etc. "vacío aún").
- Se detectaron dos pendientes reales durante la limpieza (no solo de documentación):
  - `POST /api/encuestas/:id/votar/:opcionId` sigue público en el código, pero la decisión de sesión 11 exige sesión de Miembro — desactualizado, falta corregir el código.
  - `/api/voluntarios` y `/api/eventos` nunca se protegieron con roles (a diferencia de noticias/miembros/comentarios) — quedaron así desde sesión 3-4.
- Se corrigió la sección "Fase actual" de `CONTEXTO_PROYECTO.md`, que seguía describiendo la Fase 0 (sesión 1) sin reflejar nada de lo construido después.
- Se corrigió la ruta de referencia al inicio de los tres documentos (decían `la-expansion/`, debía ser `la-expansion-docs/` desde que se separó el repo de docs en sesión 1).

**Qué se probó:** No aplica — sesión de limpieza documental, sin cambios de código.

**Qué falta:** Commit de los tres documentos en `la-expansion-docs`.

---

### 2026-08-11 — Sesión 12 (parte 2): Cambio de contraseña (Usuario y Miembro) + área de Miembro

**Qué se hizo:**

- Se creó `PUT /api/auth/cambiar-password` (backend): un solo endpoint sirve para Usuario y Miembro, detecta el tipo por `req.auth.tipo`, valida contraseña actual con bcrypt antes de permitir el cambio.
- Se creó `GET /api/comentarios/mios` (Miembro): lista los comentarios propios en cualquier estado (no solo aprobados).
- Se creó `app/cuenta/page.tsx`: área de Miembro con tres secciones — cambiar contraseña (funcional), mis comentarios (funcional, con estado visible), mis votos en encuestas (placeholder "Próximamente", bloqueado hasta construir encuestas).
- Se creó `app/admin/perfil/page.tsx`: cambiar contraseña para Usuario (Admin/Publicador), mismo endpoint que Miembro.
- Se actualizó `UserMenu.tsx` para soportar múltiples links por tipo de cuenta (antes uno solo): Usuario ahora ve "Panel admin" + "Mi perfil"; Miembro ve "Mi cuenta".
- Se descubrió que el link "Mi perfil" en `UserMenu` casi nunca es visible para un Usuario en la práctica, porque `UserMenu` vive en el `Navbar` público, que `SiteChrome` excluye dentro de `/admin/*` (donde el Usuario pasa la mayoría del tiempo). Se agregó el acceso real a "Mi perfil" directo en el header propio de `app/admin/layout.tsx`.

**Qué se probó:**

- Cambio de contraseña como Miembro, cierre de sesión y login con la nueva contraseña. ✅
- Cambio de contraseña como Usuario (Mario) desde `/admin/perfil`. ✅
- `/cuenta` muestra comentarios propios con su estado correctamente. ✅
- Acceso a "Mi perfil" desde el header del panel admin. ✅

**Qué falta / pendiente para próxima sesión:**

- Hacer commit de este bloque completo (backend: cambiar-password + comentarios/mios; frontend: /cuenta, /admin/perfil, UserMenu, admin/layout).
- Corregir código: `POST /api/encuestas/:id/votar/:opcionId` debe requerir Miembro (detectado en la limpieza de docs, sigue sin corregir).
- Proteger `/api/voluntarios` y `/api/eventos` con roles, o decidir explícitamente que se quedan abiertos.
- Construir: endpoint y UI para activar/desactivar cuentas de Usuario.
- Construir: encuestas (código completo, solo Miembros votan).
- Construir en el panel: gestión de Voluntario, Evento.
- Agregar edición de noticias existentes.
- Sigue pendiente probar Voluntario/Evento/Encuesta (CRUD original) individualmente.
- Confirmar con Ramon: newsletter, donaciones, ubicación de Contacto (pendiente desde sesión 1).

---

### 2026-08-12 — Sesión 13: Encuestas completas + compartir en redes (noticias y encuestas)

**Qué se hizo:**

- Backend de encuestas actualizado (modelo/controlador/rutas existían desde sesión 3-4 pero desactualizados): se agregó `slug` autogenerado al modelo `Encuesta` (mismo patrón que `Noticia`), se agregó array `votantes` para anti-doble-voto, se corrigió `POST /api/encuestas/:id/votar/:opcionId` para requerir sesión de Miembro (cerraba el pendiente detectado en sesión 12), se agregaron `GET /api/encuestas/slug/:slug` (público) y `GET /api/encuestas/:id/mi-estado` (Miembro).
- Se creó componente reusable `components/ShareButtons.tsx`: Web Share API en móvil (cualquier app instalada), fallback WhatsApp/Facebook/X + copiar link en desktop.
- Se creó `components/EncuestaVotacion.tsx`: maneja estado de sesión de Miembro, si ya votó, votar, y mostrar resultados con barra de porcentaje.
- Se creó página pública `app/encuestas/[slug]/page.tsx`.
- Se creó panel admin de encuestas: `app/admin/encuestas/page.tsx` (listado, cerrar, eliminar) y `app/admin/encuestas/nueva/page.tsx` (crear).
- Se agregó tarjeta "Encuestas" al dashboard `app/admin/page.tsx`.
- Se actualizaron `app/login/page.tsx` y `app/afiliate/page.tsx` para soportar `?redirect=` — permite que alguien sin sesión de Miembro, al intentar votar desde un link compartido, se afilie o loguee y vuelva directo a la encuesta. La afiliación sigue quedando `pendiente` de aprobación (no hay auto-login).
- Se agregó `ShareButtons` a `app/noticias/[slug]/page.tsx` (compartir de noticias, pendiente desde el diseño original).
- Se agregó link "Ver sitio" en `app/admin/layout.tsx` (pendiente detectado antes de esta sesión: el panel admin no tenía forma de volver al sitio público sin cerrar sesión).
- Se corrigió bug de `setState` síncrono en `useEffect` en `app/admin/encuestas/page.tsx` (cuarta ocurrencia del mismo patrón ya documentado).
- Infraestructura: backend confirmado desplegado en Render (`expansion-backend-8pk9.onrender.com`), frontend confirmado desplegado en Vercel (`expansion-frontend.vercel.app`). Se corrigió `CORS_ORIGIN` en Render (tenía `*`, ahora apunta al dominio real de Vercel) y se agregó `NEXT_PUBLIC_SITE_URL` en Vercel y en `.env.local`.

**Qué se probó:**

- Crear encuesta desde el panel admin. ✅
- Login como Miembro y votación (pendiente de confirmar por Ramon antes del commit — ver nota abajo).

**Qué falta / pendiente para próxima sesión:**

- Confirmar que el flujo completo de votación como Miembro funciona de punta a punta en producción (Ramon estaba por probarlo al cierre de esta sesión).
- Probar el flujo de link compartido sin sesión → afiliarse → aprobar → volver a votar, de punta a punta.
- Proteger `/api/voluntarios` y `/api/eventos` con roles, o decidir explícitamente que se quedan abiertos.
- Construir: endpoint y UI para activar/desactivar cuentas de Usuario.
- Construir en el panel: gestión de Voluntario, Evento.
- Agregar edición de noticias existentes.
- Confirmar con Ramon: newsletter, donaciones, ubicación de Contacto (pendiente desde sesión 1).
- Los miembros no pueden ver las encuestas creadas, nos falto mostrar las encuestas en el flujo de las encuestas, deberian aparecer en la pagina de inicio de una forma vistoza pero tambien en el navbar deberia apareer un boton para acceder al area de encuestas donde tendriamos las encuestas creadas desde la ams reciente a la mas antigua y las cerradas con sus resultados. tambien hay una parte en el panel de miembro que deberia mostrar los votos en las encuestas de ese miembro, ese seria nuestro proximo flujo de trabajo. Tambien es buen momento para modificar la pagina de inicio y darle mas funcionalidad, puede ser integrar contenido ya existente.
