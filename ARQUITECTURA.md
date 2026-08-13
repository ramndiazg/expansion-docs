# ARQUITECTURA.md

Ruta destino en el repo: `la-expansion-docs/ARQUITECTURA.md`

> Este documento describe la estructura técnica real del proyecto. Se actualiza cada vez que cambia algo estructural: carpetas, endpoints, esquemas, variables de entorno, dependencias clave. Última reescritura completa: 2026-08-11 (sesión 11). Última actualización parcial: 2026-08-12 (sesión 13 — encuestas completas, compartir en redes, deploy confirmado).

## Stack

| Capa          | Tecnología                                         | Deploy                  |
| ------------- | -------------------------------------------------- | ----------------------- |
| Frontend      | Next.js (App Router, TypeScript, Tailwind v4)      | Vercel                  |
| Backend       | Node.js + Express                                  | Render                  |
| Base de datos | MongoDB Atlas (tier Free/M0)                       | —                       |
| Auth          | JWT propio (bcryptjs + jsonwebtoken), sin NextAuth | —                       |
| CI/CD         | GitHub Actions                                     | pendiente de configurar |

## Estructura de repos y carpetas (estado real, 2026-08-12)

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
│   │   │   ├── Usuario.js
│   │   │   └── Comentario.js
│   │   ├── controllers/
│   │   │   ├── noticiaController.js
│   │   │   ├── miembroController.js
│   │   │   ├── voluntarioController.js
│   │   │   ├── eventoController.js
│   │   │   ├── encuestaController.js
│   │   │   ├── authController.js
│   │   │   └── comentarioController.js
│   │   ├── routes/
│   │   │   ├── noticiaRoutes.js
│   │   │   ├── miembroRoutes.js
│   │   │   ├── voluntarioRoutes.js
│   │   │   ├── eventoRoutes.js
│   │   │   ├── encuestaRoutes.js
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
    │   ├── page.tsx                   (Inicio)
    │   ├── layout.tsx
    │   ├── globals.css
    │   ├── login/page.tsx
    │   ├── afiliate/page.tsx
    │   ├── sobre-el-movimiento/page.tsx
    │   ├── liderazgo/page.tsx
    │   ├── noticias/
    │   │   ├── page.tsx                (listado)
    │   │   └── [slug]/page.tsx         (detalle, incluye compartir)
    │   ├── encuestas/
    │   │   └── [slug]/page.tsx         (detalle público + votar + compartir)
    │   ├── cuenta/
    │   │   └── page.tsx                (área de Miembro)
    │   └── admin/
    │       ├── layout.tsx              (protección de rutas + link "Ver sitio")
    │       ├── page.tsx                (dashboard)
    │       ├── perfil/page.tsx
    │       ├── noticias/
    │       │   ├── page.tsx
    │       │   └── nueva/page.tsx
    │       ├── encuestas/
    │       │   ├── page.tsx
    │       │   └── nueva/page.tsx
    │       ├── comentarios/page.tsx
    │       └── miembros/page.tsx
    ├── components/
    │   ├── Navbar.tsx
    │   ├── Footer.tsx
    │   ├── SiteChrome.tsx
    │   ├── UserMenu.tsx
    │   ├── Comentarios.tsx
    │   ├── ShareButtons.tsx
    │   └── EncuestaVotacion.tsx
    ├── lib/
    │   ├── auth.ts                    (sesión de Usuario)
    │   ├── authMiembro.ts             (sesión de Miembro)
    │   └── provinciasMunicipios.ts    (datos geográficos RD)
    ├── .env.local                     (no versionado)
    └── package.json
```

**Pendiente de crear**: páginas de gestión de Voluntario/Evento en el panel, edición de noticias existentes.

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

**Elemento de firma**: motivo de anillos concéntricos (SVG) en el hero del Home.

**Personalismo del líder**: Mario Díaz tiene presencia en el hero del Home ("Liderado por Mario Díaz, Secretario General"), no solo en `/liderazgo`.

**Contenido placeholder pendiente**: historia, misión, visión, valores, bio de Mario Díaz, estructura organizativa, eslogan del hero, pilares del Home — todo marcado con comentarios `{/* PLACEHOLDER: ... */}` en el código, a reemplazar con copy real antes de producción.

## Modelos de datos (Mongoose)

Todos en `expansion-backend/src/models/`. Todos incluyen `timestamps: true`.

**Noticia.js**
| Campo | Tipo | Notas |
|---|---|---|
| titulo | String | requerido |
| slug | String | único, autogenerado del título si no se envía (hook `pre('validate')`, sin `next` — ver notas técnicas) |
| resumen | String | requerido, máx 300 |
| contenido | String | requerido (texto plano; editor enriquecido queda para más adelante) |
| imagenDestacada | String | URL |
| imagenesAdicionales | [String] | URLs de galería — sin probar en la práctica más allá del diseño |
| videoUrl | String | link de YouTube, se convierte a embed en el frontend — sin probar en la práctica |
| categoria | enum | `comunicado`, `actividad`, `declaracion`, `en_los_medios` |
| autor | String | requerido |
| estado | enum | `borrador` \| `publicado` (default `borrador`) |
| fechaPublicacion | Date | |
| tags | [String] | |

**Miembro.js** (afiliación)
| Campo | Tipo | Notas |
|---|---|---|
| nombre, apellido | String | requeridos |
| cedula | String | requerido, único |
| email | String | requerido, único |
| telefono | String | requerido |
| provincia, municipio | String | requerido/opcional — llenados vía selects dependientes en `/afiliate` |
| sectorInteres | String | texto libre, temático (no geográfico) |
| passwordHash | String | recibido en texto plano, hasheado con bcrypt en hook `pre('save')` |
| estado | enum | `pendiente` \| `aprobado` \| `rechazado` |

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

**Encuesta.js**
| Campo | Tipo | Notas |
|---|---|---|
| pregunta | String | requerido |
| slug | String | único, autogenerado de la pregunta (mismo patrón hook `pre('validate')` async sin `next`, ver Noticia.js) — agregado 2026-08-12 |
| opciones | [{ texto, votos }] | mínimo 2, subdocumento con `_id` |
| activa | Boolean | default true |
| fechaCierre | Date | opcional |
| creadoPor | ObjectId → Usuario | quién la creó (para reglas de "propia encuesta") |
| votantes | [ObjectId → Miembro] | agregado 2026-08-12, evita doble voto del mismo Miembro |

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
| miembro | ObjectId → Miembro | requerido — solo Miembros afiliados y aprobados comentan |
| texto | String | requerido, máx 1000 |
| estado | enum | `pendiente` \| `aprobado` \| `rechazado` (pre-moderación) |

**Modelo descartado**: `Inscrito` — se planeó en sesión 8 para votación anónima pública, revertido en sesión 11 (ver Decisiones en `CONTEXTO_PROYECTO.md`). No construir.

## Sistema de roles y permisos

| Acción                                  | Publicador                                 | Admin                                                            |
| --------------------------------------- | ------------------------------------------ | ---------------------------------------------------------------- |
| CRUD de noticias                        | ✅                                         | ✅                                                               |
| Crear/cerrar sus propias encuestas      | ✅                                         | ✅ (+ cualquier encuesta)                                        |
| Votar en encuestas                      | — (usa su cuenta como Miembro si aplica)   | —                                                                |
| Moderar comentarios                     | ✅                                         | ✅                                                               |
| Aprobar/rechazar afiliaciones (Miembro) | ❌ (solo Admin, dato sensible de personas) | ✅                                                               |
| Ver dashboard/estadísticas              | —                                          | ✅ (sin construir)                                               |
| Activar cuentas nuevas del panel        | ✅ (activa por defecto al crear)           | ✅                                                               |
| **Desactivar** cuentas del panel        | ❌                                         | ✅ (única acción exclusiva, evita cuello de botella en el resto) |

**Nota**: solo Miembros afiliados y aprobados pueden votar en encuestas (decisión sesión 11) — ver sección de Encuestas abajo.

## Autenticación (JWT)

**Login unificado**: un solo endpoint (`POST /api/auth/login`) y una sola página (`/login`). El backend busca el email primero en `Usuario`, si no existe busca en `Miembro`, responde con `tipo: 'usuario' | 'miembro'`. El frontend redirige: Usuario → `/admin`, Miembro → `/` (o a `?redirect=` si viene de un link compartido) con sesión activa.

Middleware `expansion-backend/src/middleware/auth.js`:

- `verifyToken`: valida JWT, cuelga payload en `req.auth`
- `requireRolUsuario(...roles)`: exige tipo `usuario` y rol específico
- `requireMiembro`: exige tipo `miembro`

Token expira en 7 días (`JWT_SECRET` en `.env`).

**Frontend**: dos sistemas de sesión en `localStorage`, independientes — `lib/auth.ts` (Usuario) y `lib/authMiembro.ts` (Miembro). Ambos disparan un evento custom al guardar/cerrar sesión (`alCambiarSesionUsuario`, `alCambiarSesionMiembro`) para que componentes como `Navbar`/`UserMenu` reaccionen en vivo sin recargar la página. `app/admin/layout.tsx` protege `/admin/*`, redirige a `/login` si no hay token de Usuario.

**Redirect tras login/afiliación (2026-08-12)**: `/login` y `/afiliate` aceptan `?redirect=<ruta>`. Si un Miembro loguea con ese parámetro, se redirige ahí en vez de a `/`. La afiliación (`/afiliate`) sigue quedando en estado `pendiente` de aprobación — no hay auto-login — pero la pantalla de confirmación muestra un link directo de vuelta a `/login?redirect=<ruta>` para cuando el Admin la apruebe. Usado hoy por el flujo de votación en encuestas compartidas.

## Frontend — páginas públicas

| Página              | Ruta                               | Descripción                                                                                    | Estado         |
| ------------------- | ---------------------------------- | ---------------------------------------------------------------------------------------------- | -------------- |
| Inicio              | `app/page.tsx`                     | Hero con anillos SVG, presencia de Mario Díaz, sección de pilares (placeholder)                | ✅ funcionando |
| Sobre el movimiento | `app/sobre-el-movimiento/page.tsx` | Historia, misión, visión, valores — placeholder                                                | ✅ funcionando |
| Liderazgo           | `app/liderazgo/page.tsx`           | Bio de Mario Díaz + estructura — placeholder                                                   | ✅ funcionando |
| Noticias (listado)  | `app/noticias/page.tsx`            | Server component, fetch `/api/noticias?estado=publicado`                                       | ✅ funcionando |
| Noticias (detalle)  | `app/noticias/[slug]/page.tsx`     | Fetch por slug, imagen/galería/video si existen, `ShareButtons`, sección de Comentarios        | ✅ funcionando |
| Encuestas (detalle) | `app/encuestas/[slug]/page.tsx`    | Fetch por slug, `EncuestaVotacion` (votar/resultados), `ShareButtons`                          | ✅ probado     |
| Login unificado     | `app/login/page.tsx`               | Un form, soporta `?redirect=`, redirige según tipo de cuenta                                   | ✅ probado     |
| Afiliación          | `app/afiliate/page.tsx`            | Form de Miembro, selects provincia/municipio, confirmación de contraseña, soporta `?redirect=` | ✅ probado     |
| Cuenta (Miembro)    | `app/cuenta/page.tsx`              | Cambiar contraseña, mis comentarios, mis votos (placeholder)                                   | ✅ funcionando |

**Componentes compartidos**: `Navbar.tsx` (con `UserMenu`), `Footer.tsx`, `SiteChrome.tsx` (excluye Navbar/Footer dentro de `/admin/*`), `Comentarios.tsx` (en detalle de noticia — lista aprobados, form solo si hay sesión de Miembro), `ShareButtons.tsx` (compartir, ver sección propia abajo), `EncuestaVotacion.tsx` (lógica de votación).

**Pendiente conocido**: botón "Afíliate" ya no aparece si hay sesión activa (correcto); si no hay sesión, sigue llevando a `/afiliate` que ya existe y funciona.

## Panel admin (`/admin/*`, solo Usuario)

| Ruta                                 | Descripción                                                                        | Estado     |
| ------------------------------------ | ---------------------------------------------------------------------------------- | ---------- |
| `app/admin/layout.tsx`               | Protección de rutas + header propio + link "Ver sitio" (sin Navbar/Footer público) | ✅ probado |
| `app/admin/page.tsx`                 | Dashboard simple, accesos a Noticias/Encuestas/Comentarios/Miembros                | ✅ probado |
| `app/admin/noticias/page.tsx`        | Listado con Publicar/Despublicar                                                   | ✅ probado |
| `app/admin/noticias/nueva/page.tsx`  | Crear noticia (queda en `borrador`)                                                | ✅ probado |
| `app/admin/encuestas/page.tsx`       | Listado, cerrar, eliminar encuestas                                                | ✅ probado |
| `app/admin/encuestas/nueva/page.tsx` | Crear encuesta (pregunta + opciones dinámicas)                                     | ✅ probado |
| `app/admin/comentarios/page.tsx`     | Bandeja de moderación — aprobar/rechazar                                           | ✅ probado |
| `app/admin/miembros/page.tsx`        | Solicitudes de afiliación, filtro por estado, aprobar/rechazar                     | ✅ probado |
| `app/admin/perfil/page.tsx`          | Cambiar contraseña propia (Usuario)                                                | ✅ probado |

**Aún no construido en el panel**: gestión de Voluntario/Evento, dashboard de estadísticas real, edición de noticias existentes (solo crear + cambiar estado), activar/desactivar cuentas de Usuario.

## Área de Miembro (`/cuenta`)

`app/cuenta/page.tsx`: protegida (redirige a `/login` si no hay sesión de Miembro). Tres secciones:

- **Cambiar contraseña** — funcional, mismo endpoint que usa Usuario.
- **Mis comentarios** — lista todos los comentarios propios (cualquier estado: pendiente/aprobado/rechazado) con link a la noticia.
- **Mis votos en encuestas** — placeholder "Próximamente" (a construir: listar encuestas donde el Miembro ya votó, usando el array `votantes`).

## Menú de cuenta (UserMenu)

`components/UserMenu.tsx`: detecta sesión de Usuario o Miembro y muestra menú contextual con **múltiples links** por tipo (`linksPara()`):

- Usuario: "Panel admin" (`/admin`) + "Mi perfil" (`/admin/perfil`)
- Miembro: "Mi cuenta" (`/cuenta`)

**Importante**: `UserMenu` vive dentro de `Navbar`, que `SiteChrome` excluye dentro de `/admin/*`. Por eso el link "Mi perfil" para Usuario en la práctica casi no se ve ahí — el usuario ya está dentro del panel la mayoría del tiempo. El acceso real a "Mi perfil" para Usuario se agregó directo en el header propio de `app/admin/layout.tsx` (junto a nombre/rol, "Ver sitio" y "Cerrar sesión"), no depende de `UserMenu` cuando ya estás en `/admin`. `UserMenu` sí es la única vía si un Usuario navega por el sitio público estando logueado.

- **`desktop`**: círculo con iniciales + dropdown flotante, se cierra al hacer clic fuera.
- **`mobile`**: sin dropdown flotante — opciones en línea dentro del menú hamburguesa ya expandido.

## Encuestas

Decisión revertida en sesión 11 respecto al diseño original de sesión 8:

- ~~Votación anónima pública + modelo `Inscrito`~~ → **descartado**
- Solo Miembros afiliados y aprobados pueden votar, usando su cuenta existente. Prioriza seguridad (un voto por persona real verificada) sobre alcance viral.
- Anti-doble-voto vía array `votantes` en el modelo (2026-08-12).

**Página pública** (`app/encuestas/[slug]/page.tsx`): pregunta, opciones u resultados (barra de porcentaje) según si el Miembro ya votó o la encuesta está cerrada. `EncuestaVotacion` maneja el estado (consulta `/mi-estado`, envía el voto). `ShareButtons` para compartir.

**Sin sesión de Miembro**: se muestra CTA "Afiliarme" / "Ya soy miembro, iniciar sesión", ambos con `?redirect=/encuestas/[slug]` para volver directo a la encuesta tras loguear (la afiliación sigue requiriendo aprobación de Admin, no hay auto-login).

**Panel admin** (`app/admin/encuestas/`): listado con estado/total de votos, crear (`nueva/`), cerrar, eliminar.

**Estado: completo, backend y frontend probados.**

## Compartir en redes sociales

Componente reusable `components/ShareButtons.tsx`, usado en `noticias/[slug]` y `encuestas/[slug]`. Recibe `url`, `titulo`, `texto` — quien lo usa arma la URL absoluta (server component con acceso a `NEXT_PUBLIC_SITE_URL`).

- **Móvil** (con `navigator.share` disponible): un solo botón "Compartir" que abre el selector nativo del sistema — incluye cualquier app instalada (WhatsApp, Instagram, X, Facebook, Mensajes, etc.), sin integrarlas una por una.
- **Desktop** (sin Web Share API): íconos directos de WhatsApp, Facebook y X vía intent-links.
- **Ambos casos**: botón "Copiar link" siempre visible como fallback universal.
- **Nota**: Instagram no tiene forma pública de recibir "compartir por URL" como WhatsApp/X/Facebook — solo aparece si el usuario tiene la app instalada y usa el selector nativo del Web Share API.

**Estado: completo, en producción en noticias y encuestas.**

## Datos geográficos (provincias/municipios)

`expansion-frontend/lib/provinciasMunicipios.ts`: 32 provincias (31 + Distrito Nacional), 158 municipios, `Record<string, string[]>`. Fuente: dataset público `DannyFeliz/Datos-Rep-Dom` (GitHub), descargado y combinado vía script — no escrito a mano, para evitar errores de nombres.

**Inconsistencias conocidas del dataset fuente** (sin corregir, a decidir si importan): "Baoruco" en vez de "Bahoruco"; "Sanchez Ramírez" sin tilde.

Usado en `/afiliate`: selects dependientes provincia→municipio, el municipio se resetea al cambiar provincia.

## Endpoints del backend

| Método              | Ruta                                       | Descripción                                                                | Auth                                       | Estado                                  |
| ------------------- | ------------------------------------------ | -------------------------------------------------------------------------- | ------------------------------------------ | --------------------------------------- |
| GET                 | `/api/health`                              | Server + conexión DB activos                                               | pública                                    | ✅ probado                              |
| GET                 | `/api/noticias`                            | Lista (filtros `?estado=` `?categoria=`)                                   | pública                                    | ✅ probado                              |
| GET                 | `/api/noticias/:slug`                      | Una noticia por slug                                                       | pública                                    | ✅ probado                              |
| POST                | `/api/noticias`                            | Crear                                                                      | Usuario (admin/publicador)                 | ✅ probado                              |
| PUT                 | `/api/noticias/:id`                        | Actualizar                                                                 | Usuario (admin/publicador)                 | ✅ probado                              |
| DELETE              | `/api/noticias/:id`                        | Eliminar                                                                   | Usuario (admin/publicador)                 | ⚠️ patrón replicado, sin probar directo |
| POST                | `/api/miembros`                            | Afiliarse                                                                  | pública                                    | ✅ probado                              |
| GET                 | `/api/miembros`                            | Listar (datos sensibles)                                                   | **Admin únicamente**                       | ✅ probado                              |
| GET                 | `/api/miembros/:id`                        | Uno                                                                        | Admin                                      | ⚠️ sin probar directo                   |
| PUT                 | `/api/miembros/:id`                        | Aprobar/rechazar/editar                                                    | Admin                                      | ✅ probado                              |
| DELETE              | `/api/miembros/:id`                        | Eliminar                                                                   | Admin                                      | ⚠️ sin probar                           |
| GET/POST/PUT/DELETE | `/api/voluntarios`, `/api/voluntarios/:id` | CRUD voluntariado                                                          | — (sin proteger todavía)                   | ⚠️ sin probar                           |
| GET/POST/PUT/DELETE | `/api/eventos`, `/api/eventos/:id`         | CRUD eventos                                                               | — (sin proteger todavía)                   | ⚠️ sin probar                           |
| GET/POST/DELETE     | `/api/encuestas`, `/api/encuestas/:id`     | CRUD encuestas                                                             | POST: Usuario                              | ✅ probado                              |
| GET                 | `/api/encuestas/slug/:slug`                | Una encuesta por slug (sin `votantes`)                                     | pública                                    | ✅ probado                              |
| GET                 | `/api/encuestas/:id/mi-estado`             | Indica si el Miembro logueado ya votó                                      | Miembro                                    | ✅ probado                              |
| POST                | `/api/encuestas/:id/votar/:opcionId`       | Votar (un voto por Miembro, valida `activa`)                               | Miembro                                    | ✅ probado                              |
| PUT                 | `/api/encuestas/:id/cerrar`                | Cerrar (propia o cualquiera si admin)                                      | Usuario                                    | ⚠️ sin probar                           |
| POST                | `/api/auth/login`                          | Login unificado                                                            | pública                                    | ✅ probado                              |
| PUT                 | `/api/auth/cambiar-password`               | Cambiar contraseña propia (Usuario o Miembro, detecta tipo por `req.auth`) | Usuario o Miembro (cualquiera autenticado) | ✅ probado (ambos tipos)                |
| GET                 | `/api/comentarios/noticia/:noticiaId`      | Comentarios aprobados                                                      | pública                                    | ✅ probado                              |
| GET                 | `/api/comentarios/mios`                    | Comentarios propios (cualquier estado)                                     | Miembro                                    | ✅ probado                              |
| POST                | `/api/comentarios`                         | Crear comentario                                                           | Miembro                                    | ✅ probado                              |
| GET                 | `/api/comentarios/pendientes`              | Bandeja de moderación                                                      | Usuario (admin/publicador)                 | ✅ probado                              |
| PUT                 | `/api/comentarios/:id/moderar`             | Aprobar/rechazar                                                           | Usuario (admin/publicador)                 | ✅ probado                              |

## Variables de entorno

**expansion-backend/.env** (valores reales no documentados aquí por seguridad, solo la forma):

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
```

En Vercel (producción): `NEXT_PUBLIC_API_URL=https://expansion-backend-8pk9.onrender.com` y `NEXT_PUBLIC_SITE_URL=https://expansion-frontend.vercel.app`.

## Infraestructura desplegada

- **MongoDB Atlas**: cluster `Cluster0`, tier Free (M0), AWS, `us-east-1`. Network Access `0.0.0.0/0` (necesario para Render free, sin IP fija).
- **Backend**: desplegado en Render (`expansion-backend`, plan free) — `https://expansion-backend-8pk9.onrender.com`. `CORS_ORIGIN` apuntando al frontend de Vercel (corregido 2026-08-12, antes era `*`).
- **Frontend**: desplegado en Vercel (plan Hobby) — `https://expansion-frontend.vercel.app`.

## Notas técnicas / bugs conocidos

**Resueltos:**

- **Mongoose 9.x, hooks síncronos sin `next()`**: el hook `pre('validate')` de `Noticia.js` (autogenerar slug) causaba `"next is not a function"` con el parámetro `next` que Mongoose 9.x ya no invoca así. Se quitó el parámetro. Mismo patrón aplicado luego en `Encuesta.js`.
- **`setState` síncrono dentro de `useEffect`**: patrón repetido en varios archivos (`admin/noticias/page.tsx`, `admin/layout.tsx`, `Navbar.tsx`, `admin/encuestas/page.tsx`). Fix: envolver la llamada async en `Promise.resolve().then(() => { ... })` dentro del efecto — la actualización de estado nunca debe quedar como primera línea síncrona del cuerpo. Aplicar este patrón en cualquier código nuevo con fetch en `useEffect`.
- **`GET /api/miembros` sin proteger**: quedó público al armar roles (sesión 8), exponía cédulas/teléfonos/emails. Corregido: solo Admin.
- **Navbar no reaccionaba a login/logout sin recargar**: vive en el layout raíz, no se remonta en navegación SPA. Fix: sistema de eventos custom que `guardarSesion`/`cerrarSesion` (ambos tipos) disparan.
- **Dropdown de cuenta roto en móvil**: `position: absolute` anidado dentro del menú hamburguesa. Fix: variante `mobile` de `UserMenu` sin dropdown flotante.
- **`POST /api/encuestas/:id/votar/:opcionId` público**: quedó sin protección desde el diseño original de sesión 8. Corregido 2026-08-12: ahora requiere `requireMiembro`, valida `activa` y previene doble voto vía `votantes`.
- **`useSearchParams` en `/login` y `/afiliate`**: Next.js App Router exige envolver en `<Suspense>` cualquier componente que use `useSearchParams` para no romper el build estático. Aplicado al agregar soporte de `?redirect=`.

**Pendientes (no resueltos):**

- **Zona horaria en fechas**: `toLocaleDateString` corre un día hacia atrás fechas guardadas a medianoche UTC en horario de RD (UTC-4). Resolver cuando se construya selector de fecha real en el panel.
- **`/api/voluntarios` y `/api/eventos` sin protección de rol todavía** — a diferencia de noticias/miembros/comentarios/encuestas, estos dos CRUD quedaron sin `verifyToken`/`requireRolUsuario` desde que se crearon en sesión 3-4. Revisar si deben protegerse igual antes de construir su UI en el panel.
