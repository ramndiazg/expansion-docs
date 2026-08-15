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

- Se analizaron funcionalidades típicas de sitios de partidos/movimientos políticos.
- Se definió el listado de funcionalidades v2, ajustado para movimiento (no partido).
- Se acordó estructura de repos: carpeta `la-expansion/` con `expansion-backend/` y `expansion-frontend/` adentro.
- Se crearon los tres documentos de contexto.

**Qué se probó:** Nada aún — sesión de planificación, sin código.

**Qué falta / pendiente:** Confirmar newsletter/donaciones/Contacto. Scaffolding real de backend/frontend. Conexión a MongoDB Atlas. Deploy mínimo.

---

### 2026-08-09 — Sesión 2: Esqueleto del backend funcionando

**Qué se hizo:** Estructura de carpetas `src/{models,controllers,routes,middleware}`. `server.js` con Express + CORS + `/api/health` + Mongoose. Cluster MongoDB Atlas Free (M0) creado.

**Qué se probó:** `npm run dev` conecta a Atlas. `GET /api/health` responde correctamente. ✅

**Qué falta:** Commit. Página de Inicio + fetch a `/api/health`. Deploy.

---

### 2026-08-09 — Sesión 3: Modelos de datos del backend

**Qué se hizo:** 5 esquemas Mongoose (`Noticia`, `Miembro`, `Voluntario`, `Evento`, `Encuesta`). Slug autogenerado en Noticia. Repo de docs separado.

**Qué se probó:** Los 5 modelos cargan sin errores de sintaxis. ✅

**Qué falta:** Commit. Rutas y controladores CRUD. Frontend básico.

---

### 2026-08-10 — Sesión 4: Controladores, rutas y CRUD funcionando

**Qué se hizo:** 5 controladores + 5 rutas registradas en `server.js`. Bug resuelto: hook `pre('validate')` con `next` incompatible con Mongoose 9.x.

**Qué se probó:** `POST`/`GET /api/noticias` funcionando, slug autogenerado. ✅ Los demás modelos siguen el mismo patrón pero sin probar individualmente.

**Qué falta:** Commit. Probar Miembro/Voluntario/Evento/Encuesta individualmente. Frontend.

---

### 2026-08-10 — Sesión 5: Frontend conectado al backend — Fase 0 completa

**Qué se hizo:** `.env.local`, página de Inicio con fetch a `/api/health`, metadata básica.

**Qué se probó:** Pipeline completo frontend→backend→DB funcionando en local. ✅ **Fase 0 completada.**

**Qué falta:** Commit. Decidir deploy vs. seguir con features.

---

### 2026-08-10 — Sesión 6: Sistema de diseño y páginas institucionales

**Qué se hizo:** Decisiones de auth (JWT propio) y diseño (paleta cream/ink/amber/teal, Fraunces+Inter, anillos concéntricos). `globals.css`, `Navbar`, `Footer`, Home de marca, `sobre-el-movimiento`, `liderazgo` — todo con contenido placeholder.

**Qué se probó:** Las tres páginas cargan y se ven correctamente estilizadas. ✅

**Qué falta:** Commit. Copy real. Botón Afíliate da 404. Próximo bloque: noticias en frontend, panel admin, o deploy.

---

### 2026-08-10 — Sesión 7: Rediseño (v2) + noticias multimedia (pendiente de prueba)

**Qué se hizo:** Rediseño completo (blanco/ink azul-marino/blue/slate, Space Grotesk, menú hamburguesa, hero con Mario Díaz). `Noticia.js` extendido con `imagenesAdicionales` y `videoUrl`.

**Qué se probó:** Cambios visuales confirmados por Ramon. Campos multimedia **NO probados todavía**.

**Qué falta:** Probar noticia con imagen/galería/video antes de commitear esa parte. Editor de contenido enriquecido queda para el panel admin (fuera de esta fase).

---

### 2026-08-10 — Sesión 8: Auth JWT, roles, panel admin (Noticias)

**Qué se hizo:** Roles Publicador/Admin diseñados. Encuestas públicas/virales diseñadas (código pendiente). Comentarios diseñados (requieren Miembro afiliado aprobado). Backend: `bcryptjs`/`jsonwebtoken`, `Usuario.js`, `Comentario.js`, `middleware/auth.js`, `authController`/`authRoutes`, `comentarioController`/`comentarioRoutes`, protección de `noticiaRoutes`, script `crearUsuarioAdmin.js`. Frontend: `lib/auth.ts`, `/admin/login`, `/admin/layout.tsx`, dashboard, listado y creación de noticias.

**Qué se probó:** Usuario admin creado y logueado. ✅ Crear/publicar/despublicar noticia. ✅ Protección de rutas de noticias.

**Qué NO se probó:** Login de Miembro, comentarios, cierre de encuestas, desactivación de cuentas, sistema completo de encuestas públicas.

**Qué falta:** Commit (grande). Activar/desactivar Usuario. Registro/login de Miembro + comentarios. Modelo `Inscrito` + encuestas públicas. Panel de Miembro/Voluntario/Evento. Edición de noticias.

---

### 2026-08-10 — Sesión 9: Fix de bug (setState en useEffect)

**Qué se hizo:** Detectado (sin resolver): no hay acceso visible a login. Corregido: bug de `setState` síncrono en `useEffect` en `admin/noticias/page.tsx`.

**Qué se probó:** Error de lint desapareció, listado sigue funcionando. ✅

**Qué falta:** Commit. Acceso visible a login. Resto de pendientes de sesión 8.

---

### 2026-08-10/11 — Sesión 10: Login unificado, comentarios, datos geográficos reales

**Qué se hizo:** Login unificado (`POST /api/auth/login`, página `/login`). `app/afiliate/page.tsx`, `lib/authMiembro.ts`, `components/Comentarios.tsx` creados (habían quedado pendientes). Mismo fix de `setState` en `admin/layout.tsx` y `Navbar.tsx`. Dataset de provincias/municipios de RD (32/158) integrado con selects dependientes en `/afiliate`. Confirmación de contraseña agregada.

**Qué se probó:** Login de Usuario y Miembro. ✅ Registro completo con selects geográficos. ✅ Comentar en noticia. ✅ Validación de confirmación de contraseña. ✅

**Qué falta:** Commit. UI de moderación de comentarios. Activar/desactivar Usuario. Modelo `Inscrito` + encuestas públicas. Panel de Miembro/Voluntario/Evento. Edición de noticias.

---

### 2026-08-11 — Sesión 11: Gestión de miembros, moderación de comentarios, UserMenu

**Qué se hizo:** Decisiones registradas: encuestas solo-Miembro (descarta `Inscrito`), estructura de paneles aclarada, Admin podrá cambiar contraseñas, recuperación por correo bloqueada hasta tener servicio de email. **Seguridad**: `GET /api/miembros` corregido (estaba público). `admin/comentarios/page.tsx` y `admin/miembros/page.tsx` creados. Bug de color en botón Aprobar corregido. Mismo bug de `setState` en `Navbar.tsx`. `SiteChrome.tsx` creado (Navbar público se superponía con el panel admin). `UserMenu.tsx` creado (reemplaza texto plano). `lib/auth.ts` con sistema de eventos igual a `authMiembro.ts`. Botón Afíliate condicional.

**Qué se probó:** Moderación de comentarios y miembros. ✅ Navbar reactivo. ✅ Panel sin superposición. ✅ Dropdown funcionando en desktop y móvil. ✅

**Qué falta:** Commit. `/cuenta` (da 404). Activar/desactivar Usuario. Encuestas (código, solo Miembro). Panel de Voluntario/Evento. Edición de noticias.

---

### 2026-08-11 — Sesión 12 (parte 1): Limpieza completa de documentos de contexto

**Qué se hizo:** Reescritura completa de `ARQUITECTURA.md` (duplicados eliminados, estructura actualizada). Dos pendientes reales detectados: voto de encuesta seguía público (contradice sesión 11), `/api/voluntarios`/`/api/eventos` sin proteger desde sesión 3-4. `CONTEXTO_PROYECTO.md` corregido (seguía describiendo Fase 0). Rutas de referencia corregidas (`la-expansion/` → `la-expansion-docs/`).

**Qué se probó:** No aplica — limpieza documental.

**Qué falta:** Commit de los tres documentos.

---

### 2026-08-11 — Sesión 12 (parte 2): Cambio de contraseña (Usuario y Miembro) + área de Miembro

**Qué se hizo:** `PUT /api/auth/cambiar-password` (un endpoint para ambos tipos). `GET /api/comentarios/mios`. `app/cuenta/page.tsx` (contraseña, mis comentarios, mis votos placeholder). `app/admin/perfil/page.tsx`. `UserMenu` con múltiples links por tipo. Acceso a "Mi perfil" agregado directo en `admin/layout.tsx` (el de `UserMenu` casi no se veía por estar excluido de `/admin/*`).

**Qué se probó:** Cambio de contraseña (ambos tipos). ✅ `/cuenta` mostrando comentarios con estado. ✅ Acceso a "Mi perfil" desde el panel. ✅

**Qué falta:** Commit. Corregir voto de encuesta (público, debe requerir Miembro). Proteger voluntarios/eventos. Activar/desactivar Usuario. Encuestas completas. Panel de Voluntario/Evento. Edición de noticias.

---

### 2026-08-12 — Sesión 13: Encuestas completas + compartir en redes (noticias y encuestas)

**Qué se hizo:** Backend de encuestas actualizado: `slug` autogenerado, array `votantes` anti-doble-voto (en ese momento ligado a Miembro), voto corregido para requerir sesión de Miembro, `GET /api/encuestas/slug/:slug` y `GET /api/encuestas/:id/mi-estado` agregados. `components/ShareButtons.tsx` (Web Share API + fallback). `components/EncuestaVotacion.tsx`. Página pública `app/encuestas/[slug]/page.tsx`. Panel admin de encuestas. `?redirect=` agregado a `/login` y `/afiliate`. `ShareButtons` agregado a noticias. Link "Ver sitio" en el panel. Cuarto caso del bug de `setState` en `admin/encuestas/page.tsx`. Backend confirmado en Render, frontend en Vercel; `CORS_ORIGIN` corregido (tenía `*`); `NEXT_PUBLIC_SITE_URL` agregado.

**Qué se probó:** Crear encuesta. ✅ Login como Miembro y votación. ✅ (confirmado al inicio de sesión 14).

**Qué falta:** Probar flujo de link compartido sin sesión → afiliarse → aprobar → votar. Probar compartir de noticias. Proteger voluntarios/eventos. Activar/desactivar Usuario. Panel de Voluntario/Evento. Edición de noticias.

---

### 2026-08-12 — Sesión 14: Descubribilidad de encuestas, imágenes reales en noticias (Cloudinary), Videos separado de Noticias

**Qué se hizo:** Confirmado voto de sesión 13. Descubribilidad de encuestas: `app/encuestas/page.tsx` (listado), link en Navbar, widget "Encuesta activa" en Home. Imágenes reales en noticias: decisión de usar Cloudinary (tier Free, preset `unsigned` `la_expansion_noticias`, cloud name `ewzg4kbr`) porque Render tiene disco efímero. `components/ImageUploader.tsx`. Formulario extraído a `components/admin/NoticiaForm.tsx` (compartido crear/editar). `app/admin/noticias/[slug]/editar/page.tsx` (pendiente desde sesión 8). Video separado de Noticias: el embed no se veía (regex limitado); en vez de solo arreglarlo, se decidió separar a un modelo/sección propios — `Video.js`, `videoController.js`, `videoRoutes.js`, `components/admin/VideoForm.tsx`, panel admin de videos, página pública `app/videos/page.tsx` con regex más completo. `videoUrl` quitado de `Noticia`.

**Qué se probó:** Descubribilidad de encuestas. ✅ Noticia con imagen destacada + galería en Cloudinary. ✅ Edición de noticia existente. ✅

**Qué NO se probó:** Sección de Videos completa (código listo, pendiente de que Ramon la pruebe).

**Qué falta:** Confirmar prueba de Videos antes de commitear esa parte. Proteger voluntarios/eventos. Activar/desactivar Usuario. Panel de Voluntario/Evento. Confirmar newsletter/donaciones/Contacto.

---

### 2026-08-12 — Sesión 15: Logo, buscador/Open Graph/SEO, votación abierta en encuestas, afiliación auto-aprobada, Sala de Prensa

**Qué se hizo:**

- **Bugfix de producción**: `NEXT_PUBLIC_SITE_URL` faltaba por completo en Vercel (solo estaban `NEXT_PUBLIC_CLOUDINARY_CLOUD_NAME`, `NEXT_PUBLIC_CLOUDINARY_UPLOAD_PRESET`, `NEXT_PUBLIC_API_URL`) — causaba que los links compartidos por WhatsApp salieran como `undefined/noticias/...` con ruta duplicada. Se agregó la variable en Vercel + redeploy manual (recordatorio documentado: las `NEXT_PUBLIC_*` se hornean en el build, no alcanza con solo agregarlas).
- **Logo e identidad visual**: se diseñó primero con la Visualizer (dos iteraciones) antes de tocar código, a pedido de Ramon. Diseño final: tres círculos concéntricos con ~20% de la circunferencia faltante cada uno, en azul/blanco/rojo (colores del movimiento), rotados a distintos ángulos, sobre fondo azul oscuro. Se creó `components/Logo.tsx` (variantes `full`/`icon`, `dark`/`light`, `sm`/`lg`) y `app/icon.svg` (favicon). Integrado en `Navbar.tsx` y `app/page.tsx` (hero, grande con tagline).
- **Buscador y filtros en Sala de Prensa**: `noticiaController.getAll` acepta `?q=` (regex sobre título/resumen/contenido). Formulario `GET` sin JavaScript en el listado.
- **Open Graph**: `app/layout.tsx` con `metadataBase` y valores por defecto. `generateMetadata()` agregado a `prensa/[slug]/page.tsx` y `encuestas/[slug]/page.tsx` con título/descripción/imagen específicos.
- **SEO técnico**: `app/sitemap.ts` (dinámico, incluye noticias y encuestas publicadas) y `app/robots.ts` (bloquea `/admin`, `/cuenta`, `/login`, `/afiliate`), por convención de nombre de Next.js.
- **Renombre a "Sala de Prensa"**: decidido con Ramon (entre opciones, eligió "Sala de Prensa" en vez de "Prensa" o "Actualidad") porque "Noticias" no reflejaba el contenido real. Se renombró solo la cara pública: `app/noticias/` → `app/prensa/`, textos en Navbar/Home/sitemap/dashboard admin. El admin (`/admin/noticias`) y el modelo `Noticia` se dejaron sin cambiar de nombre a propósito. Listado rediseñado con tarjeta destacada (más reciente, imagen grande) + miniaturas en el resto de la grilla — antes era solo texto. Widget de "última noticia" agregado al Home, mismo patrón que el de encuesta destacada.
- **Votación abierta en encuestas (revierte decisión de sesión 11)**: Ramon identificó que exigir sesión de Miembro para votar frenaba el crecimiento. `Encuesta.votantes` cambiado de `[ObjectId → Miembro]` a `[String]` (IDs anónimos). `encuestaController.votar` ya no requiere `requireMiembro`, recibe `votanteId` en el body. `GET /api/encuestas/:id/mi-estado` eliminado (ya no aplica). Nuevo `lib/votante.ts` en el frontend (genera y persiste un `crypto.randomUUID()` en `localStorage`, y marca por separado si ya se votó cada encuesta por su slug). `EncuestaVotacion.tsx` reescrito para usar esto en vez de sesión.
- **Afiliación auto-aprobada (revierte parcialmente sesiones 8/11)**: Ramon identificó que la aprobación manual no escalaba. `Miembro.estado` default cambiado de `pendiente` a `aprobado`. Se agregó validación de formato de cédula dominicana (regex `/^\d{3}-\d{7}-\d{1}$/`, ej. `001-1566974-2`) en el modelo. `miembroController.create`/`update` manejan el error de duplicado (`code 11000`) de Mongo con mensajes claros ("Esa cédula ya está registrada" / "Ese email ya está registrado") en vez del error crudo. `app/afiliate/page.tsx` reescrito: ya no muestra pantalla de "espera aprobación", hace login automático tras registrar y redirige a `?redirect=` o al Home; input de cédula con `pattern`/`placeholder` para validar en el navegador antes de enviar.

**Qué se probó:**

- Favicon y logo en Navbar/Home. ✅
- Fix de `setState` síncrono en `EncuestaVotacion.tsx` (quinto caso del mismo patrón). ✅
- Afiliación con cédula mal formateada bloqueada por el navegador; afiliación válida con auto-login; mensaje claro al repetir cédula/email. ✅
- Votación en encuesta sin sesión iniciada, y verificación de que `localStorage` previene votar dos veces desde el mismo navegador. ✅

**Qué NO se probó todavía:**

- Sección de Videos (sigue pendiente desde sesión 14).
- Buscador/filtros de Sala de Prensa, Open Graph (vista previa real en WhatsApp/Facebook), y sitemap/robots — construidos en esta sesión pero no se confirmó la prueba explícitamente en el chat antes de pasar al siguiente bloque (encuestas/afiliación).

**Qué falta / pendiente para próxima sesión:**

- Confirmar pruebas de Videos, buscador/filtros, Open Graph y sitemap/robots.
- Proteger `/api/voluntarios` y `/api/eventos` con roles.
- Paneles de Voluntario y Evento.
- Activar/desactivar cuentas de Usuario.
- Redefinir o quitar "mis votos en encuestas" en `/cuenta` (ya no aplica sin cuenta para votar).
- Renombrar Videos a "Multimedia" y seguir potenciando esa sección y Sala de Prensa (idea de Ramon, pendiente para más adelante).
- Confirmar con Ramon: newsletter, donaciones, ubicación de Contacto.
