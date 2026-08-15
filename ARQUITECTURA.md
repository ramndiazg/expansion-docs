# ARQUITECTURA.md

Ruta destino en el repo: `la-expansion-docs/ARQUITECTURA.md`

> Este documento describe la estructura técnica real del proyecto. Se actualiza cada vez que cambia algo estructural: carpetas, endpoints, esquemas, variables de entorno, dependencias clave. Última reescritura completa: 2026-08-12 (sesión 15 — Sala de Prensa, logo de marca, buscador/Open Graph/SEO técnico, votación abierta en encuestas, afiliación auto-aprobada).

## Stack

| Capa          | Tecnología                                                                                                                            | Deploy                  |
| ------------- | ------------------------------------------------------------------------------------------------------------------------------------- | ----------------------- |
| Frontend      | Next.js (App Router, TypeScript, Tailwind v4)                                                                                         | Vercel                  |
| Backend       | Node.js + Express                                                                                                                     | Render                  |
| Base de datos | MongoDB Atlas (tier Free/M0)                                                                                                          | —                       |
| Auth          | JWT propio (bcryptjs + jsonwebtoken), sin NextAuth — solo para Usuario (panel); Miembro ya no requiere sesión para votar en encuestas | —                       |
| Imágenes      | Cloudinary (tier Free, subida unsigned desde el navegador)                                                                            | —                       |
| CI/CD         | GitHub Actions                                                                                                                        | pendiente de configurar |

## Estructura de repos y carpetas (estado real, 2026-08-12, sesión 15)

Tres repos independientes en GitHub (usuario `ramndiazg`):

```
la-expansion/                          (carpeta local, no es un repo en sí)
├── la-expansion-docs/                 → github.com/ramndiazg/la-expansion-docs
│   ├── CONTEXTO_PROYECTO.md
│   ├── ARQUITECTURA.md
│   └── HISTORIAL_MODIFICACIONES.md
│
├── expansion-backend/                 → github.com/ramndiazg/expansion-backend
│   ├── src/
│   │   ├── models/
│   │   │   ├── Noticia.js
│   │   │   ├── Miembro.js
│   │   │   ├── Voluntario.js
│   │   │   ├── Evento.js
│   │   │   ├── Encuesta.js
│   │   │   ├── Video.js
│   │   │   ├── Usuario.js
│   │   │   └── Comentario.js
│   │   ├── controllers/
│   │   │   ├── noticiaController.js     (incluye búsqueda ?q=)
│   │   │   ├── miembroController.js     (auto-aprueba, valida duplicados)
│   │   │   ├── voluntarioController.js
│   │   │   ├── eventoController.js
│   │   │   ├── encuestaController.js    (votación abierta, sin auth)
│   │   │   ├── videoController.js
│   │   │   ├── authController.js
│   │   │   └── comentarioController.js
│   │   ├── routes/
│   │   │   ├── noticiaRoutes.js
│   │   │   ├── miembroRoutes.js
│   │   │   ├── voluntarioRoutes.js
│   │   │   ├── eventoRoutes.js
│   │   │   ├── encuestaRoutes.js
│   │   │   ├── videoRoutes.js
│   │   │   ├── authRoutes.js
│   │   │   └── comentarioRoutes.js
│   │   ├── middleware/
│   │   │   └── auth.js
│   │   └── server.js
│   ├── scripts/
│   │   └── crearUsuarioAdmin.js
│   ├── .env                          (no versionado)
│   ├── .env.example
│   ├── .gitignore
│   └── package.json
│
└── expansion-frontend/                → github.com/ramndiazg/expansion-frontend
    ├── app/
    │   ├── page.tsx                   (Inicio: logo grande, última noticia, encuesta activa)
    │   ├── layout.tsx                 (metadata raíz + Open Graph por defecto)
    │   ├── sitemap.ts                 (nuevo, sesión 15)
    │   ├── robots.ts                  (nuevo, sesión 15)
    │   ├── icon.svg                   (favicon, nuevo, sesión 15)
    │   ├── globals.css
    │   ├── login/page.tsx
    │   ├── afiliate/page.tsx          (auto-login tras registrarse, ya no "pendiente")
    │   ├── sobre-el-movimiento/page.tsx
    │   ├── liderazgo/page.tsx
    │   ├── prensa/                    (antes "noticias" — renombrado sesión 15)
    │   │   ├── page.tsx                (listado: destacada + miniaturas + buscador/filtro)
    │   │   └── [slug]/page.tsx         (detalle, imagen + galería + compartir + generateMetadata)
    │   ├── encuestas/
    │   │   ├── page.tsx                (listado de encuestas activas)
    │   │   └── [slug]/page.tsx         (detalle público + votar sin cuenta + compartir + generateMetadata)
    │   ├── videos/
    │   │   └── page.tsx                (grilla pública de videos publicados)
    │   ├── cuenta/
    │   │   └── page.tsx                (área de Miembro)
    │   └── admin/
    │       ├── layout.tsx              (protección de rutas + link "Ver sitio")
    │       ├── page.tsx                (dashboard, tarjeta dice "Sala de Prensa")
    │       ├── perfil/page.tsx
    │       ├── noticias/               (rutas internas de admin SIN renombrar — ver nota abajo)
    │       │   ├── page.tsx
    │       │   ├── nueva/page.tsx
    │       │   └── [slug]/editar/page.tsx
    │       ├── encuestas/
    │       │   ├── page.tsx
    │       │   └── nueva/page.tsx
    │       ├── videos/
    │       │   ├── page.tsx
    │       │   ├── nueva/page.tsx
    │       │   └── [id]/editar/page.tsx
    │       ├── comentarios/page.tsx
    │       └── miembros/page.tsx       (ahora es moderación posterior, no aprobación previa)
    ├── components/
    │   ├── Navbar.tsx                 (logo + links a Sala de Prensa/Videos/Encuestas)
    │   ├── Footer.tsx
    │   ├── SiteChrome.tsx
    │   ├── UserMenu.tsx
    │   ├── Comentarios.tsx
    │   ├── ShareButtons.tsx
    │   ├── EncuestaVotacion.tsx        (votación sin cuenta, vía lib/votante.ts)
    │   ├── Logo.tsx                    (nuevo, sesión 15)
    │   ├── ImageUploader.tsx
    │   └── admin/
    │       ├── NoticiaForm.tsx
    │       └── VideoForm.tsx
    ├── lib/
    │   ├── auth.ts                    (sesión de Usuario)
    │   ├── authMiembro.ts             (sesión de Miembro — login sigue existiendo, pero ya no es requisito para votar)
    │   ├── votante.ts                 (nuevo, sesión 15 — ID anónimo en localStorage para encuestas)
    │   └── provinciasMunicipios.ts    (datos geográficos RD)
    ├── .env.local                     (no versionado)
    └── package.json
```

**Nota importante sobre el renombre "Sala de Prensa"**: solo se tocó la cara **pública** (`app/prensa/` en vez de `app/noticias/`, textos en Navbar/Home/sitemap/dashboard admin). Las rutas internas (`/admin/noticias/*`), el modelo `Noticia`, y todos los endpoints `/api/noticias/*` **se mantuvieron sin cambio de nombre** — cambiar eso también hubiera sido mucho más riesgoso sin necesidad, ya que nadie fuera de Ramon ve esas rutas.

**Pendiente de crear**: páginas de gestión de Voluntario/Evento en el panel.

## Sistema de diseño

**Principio: mobile-first.** El tráfico esperado es mayoritariamente desde móvil — todo bloque nuevo debe diseñarse y probarse primero en viewport angosto.

**Paleta** (definida en `app/globals.css`, tokens vía `@theme inline` de Tailwind v4):
| Token | Hex | Uso |
|---|---|---|
| `--ink` | `#101828` | Texto principal, fondo del hero/footer |
| `--ink-soft` | `#1D2939` | Bloques placeholder, acentos oscuros secundarios |
| `--blue` | `#4E7FDB` | Acento primario (CTAs, labels destacados) |
| `--slate` | `#64748B` | Acento secundario (bordes, detalles) |
| fondo | `#FFFFFF` (blanco nativo) | Fondo base |

Sin dark mode automático.

**Tipografía**: Space Grotesk (display, títulos) + Inter (body), 100% sans-serif.

**Logo** (`components/Logo.tsx`, nuevo sesión 15): emblema de tres círculos concéntricos, cada uno con ~20% de la circunferencia faltante (efecto de anillos "abiertos"), en azul (`#4E7FDB`), blanco (`#FFFFFF`) y rojo (`#C1272D`) — colores del movimiento — rotados a distintos ángulos entre sí, sobre un fondo circular azul oscuro (`#101828`) para que el anillo blanco tenga contraste incluso sobre fondo blanco. Coherente con el motivo de "anillos concéntricos" que ya era la firma visual del hero. Variantes: `variant` (`full` con wordmark / `icon` solo), `tone` (`dark`/`light`, para fondo claro u oscuro), `size` (`sm`/`lg`), `showTagline`. Usado en `Navbar` (compacto), Home (grande, con tagline "Movimiento político"), y como favicon (`app/icon.svg`, versión simplificada del mismo diseño).

**Personalismo del líder**: Mario Díaz tiene presencia en el hero del Home ("Liderado por Mario Díaz, Secretario General"), no solo en `/liderazgo`.

**Contenido placeholder pendiente**: historia, misión, visión, valores, bio de Mario Díaz, estructura organizativa, eslogan del hero, pilares del Home — todo marcado con comentarios `{/* PLACEHOLDER: ... */}` en el código, a reemplazar con copy real antes de producción.

## Modelos de datos (Mongoose)

Todos en `expansion-backend/src/models/`. Todos incluyen `timestamps: true`.

**Noticia.js**
| Campo | Tipo | Notas |
|---|---|---|
| titulo | String | requerido |
| slug | String | único, autogenerado del título si no se envía (hook `pre('validate')`, sin `next`) |
| resumen | String | requerido, máx 300 |
| contenido | String | requerido (texto plano) |
| imagenDestacada | String | URL de Cloudinary, subida vía `ImageUploader` |
| imagenesAdicionales | [String] | URLs de galería de Cloudinary |
| categoria | enum | `comunicado`, `actividad`, `declaracion`, `en_los_medios` |
| autor | String | requerido |
| estado | enum | `borrador` \| `publicado` (default `borrador`) |
| fechaPublicacion | Date | |
| tags | [String] | |

(`videoUrl` ya no existe en este modelo — se movió a `Video.js`, ver más abajo)

**Miembro.js** (afiliación) — **cambios de sesión 15**
| Campo | Tipo | Notas |
|---|---|---|
| nombre, apellido | String | requeridos |
| cedula | String | requerido, único, **validado con regex `/^\d{3}-\d{7}-\d{1}$/`** (formato dominicano, ej. `001-1566974-2`) — agregado sesión 15 |
| email | String | requerido, único |
| telefono | String | requerido |
| provincia, municipio | String | requerido/opcional |
| sectorInteres | String | texto libre |
| passwordHash | String | recibido en texto plano, hasheado con bcrypt en hook `pre('save')` |
| estado | enum | `pendiente` \| `aprobado` \| `rechazado` — **default cambiado a `aprobado`** (sesión 15, antes era `pendiente`) |

**Decisión de sesión 15**: la afiliación con aprobación manual previa frenaba el crecimiento — se cambió a auto-aprobación al registrarse. El campo `estado` se mantiene con sus tres valores porque el Admin conserva la capacidad de **desactivar/rechazar después** si detecta un problema (moderación posterior en vez de previa); `/admin/miembros` sigue existiendo para eso.

**Voluntario.js**
| Campo | Tipo | Notas |
|---|---|---|
| nombre, apellido, email, telefono, provincia | String | requeridos |
| areaInteres, disponibilidad | String | opcionales |
| mensaje | String | máx 500 |
| estado | enum | `pendiente` \| `contactado` \| `activo` |

**Evento.js**
| Campo | Tipo | Notas |
|---|---|---|
| titulo, descripcion, lugar | String | requeridos |
| fecha | Date | requerido |
| imagen | String | opcional |
| requiereInscripcion | Boolean | default false |
| cupoMaximo | Number | opcional |
| estado | enum | `proximo` \| `realizado` \| `cancelado` |

**Encuesta.js** — **cambios de sesión 15**
| Campo | Tipo | Notas |
|---|---|---|
| pregunta | String | requerido |
| slug | String | único, autogenerado de la pregunta |
| opciones | [{ texto, votos }] | mínimo 2, subdocumento con `_id` |
| activa | Boolean | default true |
| fechaCierre | Date | opcional |
| creadoPor | ObjectId → Usuario | quién la creó |
| votantes | **[String]** | **cambiado de `[ObjectId → Miembro]` a `[String]`** (sesión 15) — ya no son IDs de cuenta, son identificadores anónimos (`crypto.randomUUID()`) generados por el navegador y guardados en `localStorage`, para permitir votar sin cuenta mientras se previene doble voto desde el mismo dispositivo |

**Decisión de sesión 15**: exigir sesión de Miembro para votar (decisión de sesión 11) frenaba la viralidad de las encuestas compartidas. Se revirtió a votación abierta, protegida por marca en el navegador en vez de cuenta — no es a prueba de alguien que borre `localStorage` a propósito, pero es el estándar razonable para esto y elimina la fricción de afiliación completa solo para votar.

**Video.js**
| Campo | Tipo | Notas |
|---|---|---|
| titulo | String | requerido |
| videoUrl | String | requerido, link de YouTube (cualquier formato) |
| estado | enum | `borrador` \| `publicado` (default `borrador`) |
| fechaPublicacion | Date | |

**Usuario.js** (cuentas del panel — Admin/Publicador)
| Campo | Tipo | Notas |
|---|---|---|
| nombre, email | String | requeridos, email único |
| passwordHash | String | hasheado con bcrypt en hook `pre('save')` |
| rol | enum | `admin` \| `publicador` |
| activo | Boolean | default true — solo Admin puede desactivar (endpoint aún no construido) |

**Comentario.js**
| Campo | Tipo | Notas |
|---|---|---|
| noticia | ObjectId → Noticia | requerido |
| miembro | ObjectId → Miembro | requerido — solo Miembros comentan (esto sigue exigiendo cuenta, a diferencia de votar en encuestas) |
| texto | String | requerido, máx 1000 |
| estado | enum | `pendiente` \| `aprobado` \| `rechazado` (pre-moderación) |

**Modelo descartado**: `Inscrito` — se planeó en sesión 8, revertido en sesión 11. No construir (la necesidad que hubiera cubierto quedó resuelta de otra forma: votación anónima directamente en `Encuesta`, sin modelo de cuenta intermedio).

## Sistema de roles y permisos

| Acción                                | Publicador                                   | Admin                                                                     |
| ------------------------------------- | -------------------------------------------- | ------------------------------------------------------------------------- |
| CRUD de noticias                      | ✅                                           | ✅                                                                        |
| CRUD de videos                        | ✅                                           | ✅                                                                        |
| Crear/cerrar sus propias encuestas    | ✅                                           | ✅ (+ cualquier encuesta)                                                 |
| Votar en encuestas                    | Abierto a cualquiera, sin cuenta (sesión 15) | Abierto a cualquiera, sin cuenta (sesión 15)                              |
| Comentar en noticias                  | — (requiere cuenta de Miembro)               | —                                                                         |
| Moderar comentarios                   | ✅                                           | ✅                                                                        |
| Ver/desactivar afiliaciones (Miembro) | ❌ (solo Admin, dato sensible de personas)   | ✅ (moderación posterior — ya no "aprobar", las cuentas ya están activas) |
| Ver dashboard/estadísticas            | —                                            | ✅ (sin construir)                                                        |
| Activar cuentas nuevas del panel      | ✅ (activa por defecto al crear)             | ✅                                                                        |
| **Desactivar** cuentas del panel      | ❌                                           | ✅ (única acción exclusiva, evita cuello de botella en el resto)          |

## Autenticación (JWT)

**Login unificado**: un solo endpoint (`POST /api/auth/login`) y una sola página (`/login`). El backend busca el email primero en `Usuario`, si no existe busca en `Miembro`, responde con `tipo: 'usuario' | 'miembro'`. El frontend redirige: Usuario → `/admin`, Miembro → `/` (o a `?redirect=` si viene con ese parámetro) con sesión activa.

**Nota sesión 15**: la sesión de Miembro sigue existiendo (login, `/cuenta`, comentar en noticias sigue requiriendo cuenta) — lo que cambió es que **votar en encuestas ya no la necesita**. El flujo de `?redirect=` en `/login` y `/afiliate` se mantiene como mecanismo genérico (útil para volver a cualquier página tras loguear/afiliarse), aunque el caso específico que lo motivó (volver a votar tras afiliarse) ya no aplica.

Middleware `expansion-backend/src/middleware/auth.js`:

- `verifyToken`: valida JWT, cuelga payload en `req.auth`
- `requireRolUsuario(...roles)`: exige tipo `usuario` y rol específico
- `requireMiembro`: exige tipo `miembro` — **ya no se usa en `encuestaRoutes.js`** desde sesión 15, sigue usándose en `comentarioRoutes.js` y `app/cuenta`

Token expira en 7 días (`JWT_SECRET` en `.env`).

**Frontend**: dos sistemas de sesión en `localStorage`, independientes — `lib/auth.ts` (Usuario) y `lib/authMiembro.ts` (Miembro). Ambos disparan un evento custom al guardar/cerrar sesión para que `Navbar`/`UserMenu` reaccionen en vivo. `app/admin/layout.tsx` protege `/admin/*`.

**`afiliate/page.tsx` — flujo actualizado sesión 15**: ya no muestra pantalla de "espera aprobación". Al crear la cuenta (que queda `aprobado` automáticamente), hace login inmediato (`POST /api/auth/login` con las mismas credenciales) y redirige a `?redirect=` o al Home. Si el auto-login fallara por algún motivo, cae a `/login` en vez de dejar a la persona atascada.

## Votación en encuestas sin cuenta (`lib/votante.ts`, nuevo sesión 15)

```
obtenerVotanteId()  → genera (o recupera) un crypto.randomUUID() guardado en
                       localStorage bajo la clave expansion_votante_id
yaVoto(slug)         → true si localStorage tiene expansion_voto_<slug>
marcarVotado(slug)   → guarda expansion_voto_<slug> = "1"
```

`EncuestaVotacion.tsx` usa esto para decidir si mostrar el formulario de voto o los resultados, sin ningún fetch de "mi estado" (ese endpoint, `GET /api/encuestas/:id/mi-estado`, se eliminó en sesión 15 — ya no aplica sin cuenta). El backend valida el `votanteId` recibido contra el array `votantes` de la encuesta para bloquear doble voto desde el mismo identificador.

## Frontend — páginas públicas

| Página                   | Ruta                               | Descripción                                                                                                                                                         | Estado                              |
| ------------------------ | ---------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------- |
| Inicio                   | `app/page.tsx`                     | Logo grande, hero con anillos SVG, widget "última noticia" (sesión 15), widget "Encuesta activa", pilares (placeholder)                                             | ✅ funcionando                      |
| Sobre el movimiento      | `app/sobre-el-movimiento/page.tsx` | Historia, misión, visión, valores — placeholder                                                                                                                     | ✅ funcionando                      |
| Liderazgo                | `app/liderazgo/page.tsx`           | Bio de Mario Díaz + estructura — placeholder                                                                                                                        | ✅ funcionando                      |
| Sala de Prensa (listado) | `app/prensa/page.tsx`              | Buscador + filtro de categoría (`?q=` `?categoria=`), tarjeta destacada (más reciente, con imagen grande) + resto en grilla con miniaturas — nuevo diseño sesión 15 | ✅ probado                          |
| Sala de Prensa (detalle) | `app/prensa/[slug]/page.tsx`       | Imagen + galería (Cloudinary), `ShareButtons`, Comentarios, `generateMetadata` con Open Graph                                                                       | ✅ probado                          |
| Encuestas (listado)      | `app/encuestas/page.tsx`           | Fetch `/api/encuestas?activa=true`                                                                                                                                  | ✅ probado                          |
| Encuestas (detalle)      | `app/encuestas/[slug]/page.tsx`    | `EncuestaVotacion` (votar sin cuenta / resultados), `ShareButtons`, `generateMetadata`                                                                              | ✅ probado                          |
| Videos (grilla)          | `app/videos/page.tsx`              | Fetch `/api/videos?estado=publicado`, embeds de YouTube                                                                                                             | ⚠️ pendiente de confirmar por Ramon |
| Login unificado          | `app/login/page.tsx`               | Soporta `?redirect=`, redirige según tipo de cuenta                                                                                                                 | ✅ probado                          |
| Afiliación               | `app/afiliate/page.tsx`            | Auto-login tras registrar (sesión 15), validación de formato de cédula en el input                                                                                  | ✅ probado                          |
| Cuenta (Miembro)         | `app/cuenta/page.tsx`              | Cambiar contraseña, mis comentarios, mis votos (placeholder)                                                                                                        | ✅ funcionando                      |

**Componentes compartidos**: `Navbar.tsx` (logo + `UserMenu` + links), `Footer.tsx`, `SiteChrome.tsx`, `Comentarios.tsx`, `ShareButtons.tsx`, `EncuestaVotacion.tsx`, `ImageUploader.tsx`, `Logo.tsx`.

## Panel admin (`/admin/*`, solo Usuario)

| Ruta                                        | Descripción                                                                                                                                         | Estado                                                     |
| ------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------- |
| `app/admin/layout.tsx`                      | Protección de rutas + header propio + link "Ver sitio"                                                                                              | ✅ probado                                                 |
| `app/admin/page.tsx`                        | Dashboard, tarjeta "Sala de Prensa" (apunta a `/admin/noticias`), Videos, Encuestas, Comentarios, Miembros                                          | ✅ probado                                                 |
| `app/admin/noticias/page.tsx`               | Listado con Publicar/Despublicar/Editar                                                                                                             | ✅ probado                                                 |
| `app/admin/noticias/nueva/page.tsx`         | Crear noticia (`NoticiaForm`)                                                                                                                       | ✅ probado                                                 |
| `app/admin/noticias/[slug]/editar/page.tsx` | Editar noticia existente (`NoticiaForm`)                                                                                                            | ✅ probado                                                 |
| `app/admin/encuestas/page.tsx`              | Listado, cerrar, eliminar encuestas                                                                                                                 | ✅ probado                                                 |
| `app/admin/encuestas/nueva/page.tsx`        | Crear encuesta (pregunta + opciones dinámicas)                                                                                                      | ✅ probado                                                 |
| `app/admin/videos/page.tsx`                 | Listado con Publicar/Despublicar/Editar                                                                                                             | ⚠️ pendiente de confirmar                                  |
| `app/admin/videos/nueva/page.tsx`           | Crear video (título + link de YouTube)                                                                                                              | ⚠️ pendiente de confirmar                                  |
| `app/admin/videos/[id]/editar/page.tsx`     | Editar video existente                                                                                                                              | ⚠️ pendiente de confirmar                                  |
| `app/admin/comentarios/page.tsx`            | Bandeja de moderación — aprobar/rechazar                                                                                                            | ✅ probado                                                 |
| `app/admin/miembros/page.tsx`               | Listado de Miembros — ahora es moderación posterior (las cuentas ya están `aprobado` por defecto), sigue permitiendo cambiar `estado` a `rechazado` | ✅ probado (flujo previo, semántica cambiada en sesión 15) |
| `app/admin/perfil/page.tsx`                 | Cambiar contraseña propia (Usuario)                                                                                                                 | ✅ probado                                                 |

**Aún no construido en el panel**: gestión de Voluntario/Evento, dashboard de estadísticas real, activar/desactivar cuentas de Usuario.

## Área de Miembro (`/cuenta`)

Sin cambios funcionales en sesión 15. Cambiar contraseña, mis comentarios, mis votos (placeholder — nota: como votar ya no requiere cuenta, esta sección tendría que redefinirse o quitarse más adelante, ya no hay forma de asociar un voto anónimo a un Miembro específico).

## Menú de cuenta (UserMenu)

Sin cambios en sesión 15. Ver sesiones anteriores: detecta Usuario/Miembro, dropdown en desktop, en línea en móvil.

## Encuestas

**Decisión de sesión 15 (revierte la de sesión 11)**: votación completamente abierta, sin necesidad de cuenta. Protegida por un identificador anónimo (`crypto.randomUUID()`) guardado en `localStorage` del navegador — ver sección "Votación en encuestas sin cuenta" arriba.

**Página de detalle** (`app/encuestas/[slug]/page.tsx`): pregunta, opciones o resultados (barra de porcentaje) según si el navegador ya votó (`lib/votante.ts`) o la encuesta está cerrada. `ShareButtons` para compartir. `generateMetadata` para Open Graph.

**Descubribilidad** (sesión 14, sigue vigente): listado `/encuestas`, link en Navbar, widget en Home.

**Panel admin** (`app/admin/encuestas/`): listado con estado/total de votos, crear, cerrar, eliminar — sin cambios, sigue requiriendo sesión de Usuario (esto es distinto de votar, que es lo que se abrió).

**Estado: completo, backend y frontend probados con el nuevo flujo de votación abierta.**

## Imágenes (Cloudinary)

Sin cambios en sesión 15. `components/ImageUploader.tsx`, preset `unsigned` `la_expansion_noticias`, cloud name `ewzg4kbr`. Usado en `NoticiaForm.tsx` (imagen destacada + galería).

## Videos (separado de Sala de Prensa)

Sin cambios funcionales en sesión 15 más allá del rebranding de la sección hermana ("Videos" sigue llamándose así — renombrarlo a "Multimedia" quedó anotado como pendiente para más adelante, no se hizo en esta sesión). Ver detalle en sesiones anteriores: modelo simple (`titulo` + `videoUrl`), sin páginas de detalle individuales, grilla pública `/videos`.

**Estado: código completo, pendiente de que Ramon confirme la prueba end-to-end.**

## Compartir en redes sociales

Sin cambios en sesión 15. `components/ShareButtons.tsx`: Web Share API en móvil, fallback WhatsApp/Facebook/X + copiar link en desktop. Usado en `prensa/[slug]` y `encuestas/[slug]`.

## Buscador y filtros (Sala de Prensa) — nuevo sesión 15

- **Backend**: `noticiaController.getAll` acepta `?q=` además de `?estado=` y `?categoria=`. La búsqueda es un `$or` con regex case-insensitive sobre `titulo`, `resumen` y `contenido` — suficiente para el volumen actual; si crece mucho más adelante, se puede migrar a un índice de texto de MongoDB.
- **Frontend**: `app/prensa/page.tsx` usa un `<form method="GET">` (sin JavaScript del lado del cliente) para mantener la página como server component — mejor para SEO. Input de texto + select de categoría, ambos reflejados en la URL (`?q=...&categoria=...`).

## Open Graph — nuevo sesión 15

- `app/layout.tsx` define `metadataBase` (necesario para que las imágenes relativas se resuelvan como absolutas) y valores por defecto de `openGraph`/`twitter` para todo el sitio.
- `app/prensa/[slug]/page.tsx` y `app/encuestas/[slug]/page.tsx` exportan `generateMetadata()`, que sobreescribe título/descripción/imagen por página con los datos reales de la noticia o encuesta.
- Videos no tiene `generateMetadata` — no tiene página de detalle individual que compartir.

## SEO técnico — nuevo sesión 15

- `app/sitemap.ts`: genera `/sitemap.xml` dinámicamente — páginas estáticas (Home, Sala de Prensa, Encuestas, Videos, Sobre el movimiento, Liderazgo, Afíliate) + una entrada por cada noticia publicada y cada encuesta, consultando la API en el momento del build/request.
- `app/robots.ts`: genera `/robots.txt` — permite todo excepto `/admin`, `/cuenta`, `/login`, `/afiliate`; apunta al sitemap.
- Ambos usan la convención de nombre de archivo de Next.js App Router — no requieren tocar `server.js` ni ninguna configuración adicional.

## Datos geográficos (provincias/municipios)

Sin cambios. `expansion-frontend/lib/provinciasMunicipios.ts`: 32 provincias, 158 municipios. Usado en `/afiliate`.

## Endpoints del backend

| Método              | Ruta                                       | Descripción                                                                                                                                        | Auth                                        | Estado                                   |
| ------------------- | ------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------- | ---------------------------------------- |
| GET                 | `/api/health`                              | Server + conexión DB activos                                                                                                                       | pública                                     | ✅ probado                               |
| GET                 | `/api/noticias`                            | Lista (filtros `?estado=` `?categoria=` `?q=` — búsqueda nueva sesión 15)                                                                          | pública                                     | ✅ probado                               |
| GET                 | `/api/noticias/:slug`                      | Una noticia por slug                                                                                                                               | pública                                     | ✅ probado                               |
| POST                | `/api/noticias`                            | Crear                                                                                                                                              | Usuario (admin/publicador)                  | ✅ probado                               |
| PUT                 | `/api/noticias/:id`                        | Actualizar                                                                                                                                         | Usuario (admin/publicador)                  | ✅ probado                               |
| DELETE              | `/api/noticias/:id`                        | Eliminar                                                                                                                                           | Usuario (admin/publicador)                  | ⚠️ patrón replicado, sin probar directo  |
| POST                | `/api/miembros`                            | Afiliarse — **queda `aprobado` automáticamente** (sesión 15); valida formato de cédula y devuelve 409 con mensaje claro si cédula/email ya existen | pública                                     | ✅ probado                               |
| GET                 | `/api/miembros`                            | Listar (datos sensibles)                                                                                                                           | **Admin únicamente**                        | ✅ probado                               |
| GET                 | `/api/miembros/:id`                        | Uno                                                                                                                                                | Admin                                       | ⚠️ sin probar directo                    |
| PUT                 | `/api/miembros/:id`                        | Editar / cambiar `estado` (ahora moderación posterior, no aprobación previa)                                                                       | Admin                                       | ✅ probado                               |
| DELETE              | `/api/miembros/:id`                        | Eliminar                                                                                                                                           | Admin                                       | ⚠️ sin probar                            |
| GET/POST/PUT/DELETE | `/api/voluntarios`, `/api/voluntarios/:id` | CRUD voluntariado                                                                                                                                  | — (sin proteger todavía)                    | ⚠️ sin probar                            |
| GET/POST/PUT/DELETE | `/api/eventos`, `/api/eventos/:id`         | CRUD eventos                                                                                                                                       | — (sin proteger todavía)                    | ⚠️ sin probar                            |
| GET/POST/DELETE     | `/api/encuestas`, `/api/encuestas/:id`     | CRUD encuestas                                                                                                                                     | POST: Usuario                               | ✅ probado                               |
| GET                 | `/api/encuestas/slug/:slug`                | Una encuesta por slug (sin `votantes`)                                                                                                             | pública                                     | ✅ probado                               |
| POST                | `/api/encuestas/:id/votar/:opcionId`       | Votar — **ya no requiere Miembro** (sesión 15), recibe `{ votanteId }` en el body, un voto por `votanteId`                                         | pública                                     | ✅ probado                               |
| PUT                 | `/api/encuestas/:id/cerrar`                | Cerrar (propia o cualquiera si admin)                                                                                                              | Usuario                                     | ⚠️ sin probar                            |
| GET/POST/PUT/DELETE | `/api/videos`, `/api/videos/:id`           | CRUD videos                                                                                                                                        | POST/PUT/DELETE: Usuario (admin/publicador) | ⚠️ patrón replicado, pendiente de probar |
| POST                | `/api/auth/login`                          | Login unificado                                                                                                                                    | pública                                     | ✅ probado                               |
| PUT                 | `/api/auth/cambiar-password`               | Cambiar contraseña propia                                                                                                                          | Usuario o Miembro (cualquiera autenticado)  | ✅ probado (ambos tipos)                 |
| GET                 | `/api/comentarios/noticia/:noticiaId`      | Comentarios aprobados                                                                                                                              | pública                                     | ✅ probado                               |
| GET                 | `/api/comentarios/mios`                    | Comentarios propios (cualquier estado)                                                                                                             | Miembro                                     | ✅ probado                               |
| POST                | `/api/comentarios`                         | Crear comentario                                                                                                                                   | Miembro                                     | ✅ probado                               |
| GET                 | `/api/comentarios/pendientes`              | Bandeja de moderación                                                                                                                              | Usuario (admin/publicador)                  | ✅ probado                               |
| PUT                 | `/api/comentarios/:id/moderar`             | Aprobar/rechazar                                                                                                                                   | Usuario (admin/publicador)                  | ✅ probado                               |

**Eliminado en sesión 15**: `GET /api/encuestas/:id/mi-estado` (ya no aplica sin cuenta de Miembro).

## Variables de entorno

**expansion-backend/.env**:

```
PORT=4000
MONGODB_URI=mongodb+srv://<usuario>:<password>@cluster0.xxxxx.mongodb.net/la-expansion?retryWrites=true&w=majority
CORS_ORIGIN=https://expansion-frontend.vercel.app
JWT_SECRET=<valor largo y aleatorio, no compartido en el chat>
```

**expansion-frontend/.env.local**

```
NEXT_PUBLIC_API_URL=http://localhost:4000
NEXT_PUBLIC_SITE_URL=http://localhost:3000
NEXT_PUBLIC_CLOUDINARY_CLOUD_NAME=ewzg4kbr
NEXT_PUBLIC_CLOUDINARY_UPLOAD_PRESET=la_expansion_noticias
```

En Vercel (producción): las cuatro variables, con `NEXT_PUBLIC_SITE_URL=https://expansion-frontend.vercel.app` — **esta faltaba por completo en Vercel hasta sesión 15** (causaba links de compartir rotos tipo `undefined/noticias/...`), ya corregida.

## Infraestructura desplegada

- **MongoDB Atlas**: cluster `Cluster0`, tier Free (M0), AWS, `us-east-1`. Network Access `0.0.0.0/0`.
- **Backend**: Render (`expansion-backend`, plan free) — `https://expansion-backend-8pk9.onrender.com`.
- **Frontend**: Vercel (plan Hobby) — `https://expansion-frontend.vercel.app`.
- **Cloudinary**: cuenta Free, cloud name `ewzg4kbr`, upload preset `la_expansion_noticias` (unsigned).

## Notas técnicas / bugs conocidos

**Resueltos:**

- **Mongoose 9.x, hooks síncronos sin `next()`**: hook `pre('validate')` sin el parámetro `next`. Aplicado en `Noticia.js` y `Encuesta.js`.
- **`setState` síncrono dentro de `useEffect`**: patrón repetido en múltiples archivos (`admin/noticias/page.tsx`, `admin/layout.tsx`, `Navbar.tsx`, `admin/encuestas/page.tsx`, `EncuestaVotacion.tsx`). Fix constante: envolver en `Promise.resolve().then(() => { ... })` dentro del efecto — aplica incluso cuando no hay `fetch` de por medio (ej. lectura de `localStorage`).
- **`GET /api/miembros` sin proteger**: corregido, solo Admin.
- **Navbar no reaccionaba a login/logout sin recargar**: corregido con eventos custom.
- **Dropdown de cuenta roto en móvil**: corregido con variante `mobile` sin dropdown flotante.
- **Encuestas sin forma de descubrirse**: corregido con listado, Navbar y widget en Home (sesión 14).
- **Formulario de noticias sin campos de imagen/video**: corregido con `ImageUploader` + Cloudinary; video separado a su propia sección (sesión 14).
- **`NEXT_PUBLIC_SITE_URL` ausente en Vercel**: causaba links de compartir rotos (`undefined/...`, con doble ruta por interpretarse como relativo). Corregido agregando la variable en Vercel + redeploy — recordatorio: las variables `NEXT_PUBLIC_*` se hornean en el build, agregarlas sola no alcanza sin redeploy.
- **Aprobación manual de afiliación y votación solo-Miembro limitaban el crecimiento**: ambas revertidas en sesión 15 — ver secciones de Miembro y Encuestas arriba.

**Pendientes (no resueltos):**

- **Zona horaria en fechas**: `toLocaleDateString` corre un día hacia atrás fechas guardadas a medianoche UTC en horario de RD (UTC-4).
- **`/api/voluntarios` y `/api/eventos` sin protección de rol todavía**.
- **Sección de Videos sin probar por Ramon todavía** — código completo, pendiente confirmación end-to-end.
- **`app/cuenta` "mis votos en encuestas"**: sigue como placeholder, y ahora es más difícil de resolver — como votar ya no requiere cuenta, no hay forma de asociar un voto anónimo a un Miembro específico. Probablemente esta sección deba quitarse o redefinirse.
- **Renombrar Videos a "Multimedia"**: anotado como idea para más adelante, no se hizo en sesión 15.
