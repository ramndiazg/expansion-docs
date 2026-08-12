# ARQUITECTURA.md

Ruta destino en el repo: `la-expansion-docs/ARQUITECTURA.md`

> Este documento describe la estructura técnica real del proyecto. Se actualiza cada vez que cambia algo estructural: carpetas, endpoints, esquemas, variables de entorno, dependencias clave. Última reescritura completa: 2026-08-11 (sesión 11), para eliminar contenido duplicado/desactualizado acumulado en sesiones anteriores.

## Stack

| Capa          | Tecnología                                         | Deploy                     |
| ------------- | -------------------------------------------------- | -------------------------- |
| Frontend      | Next.js (App Router, TypeScript, Tailwind v4)      | Vercel (aún no desplegado) |
| Backend       | Node.js + Express                                  | Render (aún no desplegado) |
| Base de datos | MongoDB Atlas (tier Free/M0)                       | —                          |
| Auth          | JWT propio (bcryptjs + jsonwebtoken), sin NextAuth | —                          |
| CI/CD         | GitHub Actions                                     | pendiente de configurar    |

## Estructura de repos y carpetas (estado real, 2026-08-11)

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
    │   │   └── [slug]/page.tsx         (detalle)
    │   └── admin/
    │       ├── layout.tsx              (protección de rutas)
    │       ├── page.tsx                (dashboard)
    │       ├── noticias/
    │       │   ├── page.tsx
    │       │   └── nueva/page.tsx
    │       ├── comentarios/page.tsx
    │       └── miembros/page.tsx
    ├── components/
    │   ├── Navbar.tsx
    │   ├── Footer.tsx
    │   ├── SiteChrome.tsx
    │   ├── UserMenu.tsx
    │   └── Comentarios.tsx
    ├── lib/
    │   ├── auth.ts                    (sesión de Usuario)
    │   ├── authMiembro.ts             (sesión de Miembro)
    │   └── provinciasMunicipios.ts    (datos geográficos RD)
    ├── .env.local                     (no versionado)
    └── package.json
```

**Pendiente de crear**: `app/cuenta/` (área de Miembro — cambiar contraseña, ver votos/comentarios), páginas de gestión de Voluntario/Evento/Encuesta en el panel, edición de noticias existentes.

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
| opciones | [{ texto, votos }] | mínimo 2, subdocumento con `_id` |
| activa | Boolean | default true |
| fechaCierre | Date | opcional |
| creadoPor | ObjectId → Usuario | quién la creó (para reglas de "propia encuesta") |

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

**Login unificado**: un solo endpoint (`POST /api/auth/login`) y una sola página (`/login`). El backend busca el email primero en `Usuario`, si no existe busca en `Miembro`, responde con `tipo: 'usuario' | 'miembro'`. El frontend redirige: Usuario → `/admin`, Miembro → `/` con sesión activa.

Middleware `expansion-backend/src/middleware/auth.js`:

- `verifyToken`: valida JWT, cuelga payload en `req.auth`
- `requireRolUsuario(...roles)`: exige tipo `usuario` y rol específico
- `requireMiembro`: exige tipo `miembro`

Token expira en 7 días (`JWT_SECRET` en `.env`).

**Frontend**: dos sistemas de sesión en `localStorage`, independientes — `lib/auth.ts` (Usuario) y `lib/authMiembro.ts` (Miembro). Ambos disparan un evento custom al guardar/cerrar sesión (`alCambiarSesionUsuario`, `alCambiarSesionMiembro`) para que componentes como `Navbar`/`UserMenu` reaccionen en vivo sin recargar la página. `app/admin/layout.tsx` protege `/admin/*`, redirige a `/login` si no hay token de Usuario.

## Frontend — páginas públicas

| Página              | Ruta                               | Descripción                                                                     | Estado         |
| ------------------- | ---------------------------------- | ------------------------------------------------------------------------------- | -------------- |
| Inicio              | `app/page.tsx`                     | Hero con anillos SVG, presencia de Mario Díaz, sección de pilares (placeholder) | ✅ funcionando |
| Sobre el movimiento | `app/sobre-el-movimiento/page.tsx` | Historia, misión, visión, valores — placeholder                                 | ✅ funcionando |
| Liderazgo           | `app/liderazgo/page.tsx`           | Bio de Mario Díaz + estructura — placeholder                                    | ✅ funcionando |
| Noticias (listado)  | `app/noticias/page.tsx`            | Server component, fetch `/api/noticias?estado=publicado`                        | ✅ funcionando |
| Noticias (detalle)  | `app/noticias/[slug]/page.tsx`     | Fetch por slug, imagen/galería/video si existen, sección de Comentarios         | ✅ funcionando |
| Login unificado     | `app/login/page.tsx`               | Un form, redirige según tipo de cuenta                                          | ✅ probado     |
| Afiliación          | `app/afiliate/page.tsx`            | Form de Miembro, selects provincia/municipio, confirmación de contraseña        | ✅ probado     |

**Componentes compartidos**: `Navbar.tsx` (con `UserMenu`), `Footer.tsx`, `SiteChrome.tsx` (excluye Navbar/Footer dentro de `/admin/*`), `Comentarios.tsx` (en detalle de noticia — lista aprobados, form solo si hay sesión de Miembro).

**Pendiente conocido**: botón "Afíliate" ya no aparece si hay sesión activa (correcto); si no hay sesión, sigue llevando a `/afiliate` que ya existe y funciona.

## Panel admin (`/admin/*`, solo Usuario)

| Ruta                                | Descripción                                                     | Estado     |
| ----------------------------------- | --------------------------------------------------------------- | ---------- |
| `app/admin/layout.tsx`              | Protección de rutas + header propio (sin Navbar/Footer público) | ✅ probado |
| `app/admin/page.tsx`                | Dashboard simple, accesos a Noticias/Comentarios/Miembros       | ✅ probado |
| `app/admin/noticias/page.tsx`       | Listado con Publicar/Despublicar                                | ✅ probado |
| `app/admin/noticias/nueva/page.tsx` | Crear noticia (queda en `borrador`)                             | ✅ probado |
| `app/admin/comentarios/page.tsx`    | Bandeja de moderación — aprobar/rechazar                        | ✅ probado |
| `app/admin/miembros/page.tsx`       | Solicitudes de afiliación, filtro por estado, aprobar/rechazar  | ✅ probado |
| `app/admin/perfil/page.tsx`         | Cambiar contraseña propia (Usuario)                             | ✅ probado |

**Aún no construido en el panel**: gestión de Voluntario/Evento/Encuesta, dashboard de estadísticas real, edición de noticias existentes (solo crear + cambiar estado), activar/desactivar cuentas de Usuario.

## Área de Miembro (`/cuenta`)

`app/cuenta/page.tsx`: protegida (redirige a `/login` si no hay sesión de Miembro). Tres secciones:

- **Cambiar contraseña** — funcional, mismo endpoint que usa Usuario.
- **Mis comentarios** — lista todos los comentarios propios (cualquier estado: pendiente/aprobado/rechazado) con link a la noticia.
- **Mis votos en encuestas** — placeholder "Próximamente", bloqueado hasta construir el sistema de encuestas.

## Menú de cuenta (UserMenu)

`components/UserMenu.tsx`: detecta sesión de Usuario o Miembro y muestra menú contextual con **múltiples links** por tipo (`linksPara()`):

- Usuario: "Panel admin" (`/admin`) + "Mi perfil" (`/admin/perfil`)
- Miembro: "Mi cuenta" (`/cuenta`)

**Importante**: `UserMenu` vive dentro de `Navbar`, que `SiteChrome` excluye dentro de `/admin/*`. Por eso el link "Mi perfil" para Usuario en la práctica casi no se ve ahí — el usuario ya está dentro del panel la mayoría del tiempo. El acceso real a "Mi perfil" para Usuario se agregó directo en el header propio de `app/admin/layout.tsx` (junto a nombre/rol y "Cerrar sesión"), no depende de `UserMenu` cuando ya estás en `/admin`. `UserMenu` sí es la única vía si un Usuario navega por el sitio público estando logueado.

- **`desktop`**: círculo con iniciales + dropdown flotante, se cierra al hacer clic fuera.
- **`mobile`**: sin dropdown flotante — opciones en línea dentro del menú hamburguesa ya expandido.

## Encuestas (diseño actualizado, código pendiente)

Decisión revertida en sesión 11 respecto al diseño original de sesión 8:

- ~~Votación anónima pública + modelo `Inscrito`~~ → **descartado**
- **Ahora**: solo Miembros afiliados y aprobados pueden votar, usando su cuenta existente. Prioriza seguridad (un voto por persona real verificada) sobre alcance viral.
- Cada encuesta seguirá teniendo una página pública para ver resultados/compartir, pero votar requiere sesión de Miembro.
- **Estado: diseño acordado, código aún no escrito.**

## Datos geográficos (provincias/municipios)

`expansion-frontend/lib/provinciasMunicipios.ts`: 32 provincias (31 + Distrito Nacional), 158 municipios, `Record<string, string[]>`. Fuente: dataset público `DannyFeliz/Datos-Rep-Dom` (GitHub), descargado y combinado vía script — no escrito a mano, para evitar errores de nombres.

**Inconsistencias conocidas del dataset fuente** (sin corregir, a decidir si importan): "Baoruco" en vez de "Bahoruco"; "Sanchez Ramírez" sin tilde.

Usado en `/afiliate`: selects dependientes provincia→municipio, el municipio se resetea al cambiar provincia.

## Endpoints del backend

| Método              | Ruta                                       | Descripción                                                                | Auth                                                                                                         | Estado                                  |
| ------------------- | ------------------------------------------ | -------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------ | --------------------------------------- |
| GET                 | `/api/health`                              | Server + conexión DB activos                                               | pública                                                                                                      | ✅ probado                              |
| GET                 | `/api/noticias`                            | Lista (filtros `?estado=` `?categoria=`)                                   | pública                                                                                                      | ✅ probado                              |
| GET                 | `/api/noticias/:slug`                      | Una noticia por slug                                                       | pública                                                                                                      | ✅ probado                              |
| POST                | `/api/noticias`                            | Crear                                                                      | Usuario (admin/publicador)                                                                                   | ✅ probado                              |
| PUT                 | `/api/noticias/:id`                        | Actualizar                                                                 | Usuario (admin/publicador)                                                                                   | ✅ probado                              |
| DELETE              | `/api/noticias/:id`                        | Eliminar                                                                   | Usuario (admin/publicador)                                                                                   | ⚠️ patrón replicado, sin probar directo |
| POST                | `/api/miembros`                            | Afiliarse                                                                  | pública                                                                                                      | ✅ probado                              |
| GET                 | `/api/miembros`                            | Listar (datos sensibles)                                                   | **Admin únicamente**                                                                                         | ✅ probado                              |
| GET                 | `/api/miembros/:id`                        | Uno                                                                        | Admin                                                                                                        | ⚠️ sin probar directo                   |
| PUT                 | `/api/miembros/:id`                        | Aprobar/rechazar/editar                                                    | Admin                                                                                                        | ✅ probado                              |
| DELETE              | `/api/miembros/:id`                        | Eliminar                                                                   | Admin                                                                                                        | ⚠️ sin probar                           |
| GET/POST/PUT/DELETE | `/api/voluntarios`, `/api/voluntarios/:id` | CRUD voluntariado                                                          | — (sin proteger todavía)                                                                                     | ⚠️ sin probar                           |
| GET/POST/PUT/DELETE | `/api/eventos`, `/api/eventos/:id`         | CRUD eventos                                                               | — (sin proteger todavía)                                                                                     | ⚠️ sin probar                           |
| GET/POST/DELETE     | `/api/encuestas`, `/api/encuestas/:id`     | CRUD encuestas                                                             | POST: Usuario                                                                                                | ⚠️ sin probar                           |
| POST                | `/api/encuestas/:id/votar/:opcionId`       | Votar                                                                      | pública en el código actual — **desactualizado**, debe pasar a requerir Miembro por la decisión de sesión 11 | ⚠️ pendiente actualizar código          |
| PUT                 | `/api/encuestas/:id/cerrar`                | Cerrar (propia o cualquiera si admin)                                      | Usuario                                                                                                      | ⚠️ sin probar                           |
| POST                | `/api/auth/login`                          | Login unificado                                                            | pública                                                                                                      | ✅ probado                              |
| PUT                 | `/api/auth/cambiar-password`               | Cambiar contraseña propia (Usuario o Miembro, detecta tipo por `req.auth`) | Usuario o Miembro (cualquiera autenticado)                                                                   | ✅ probado (ambos tipos)                |
| GET                 | `/api/comentarios/noticia/:noticiaId`      | Comentarios aprobados                                                      | pública                                                                                                      | ✅ probado                              |
| GET                 | `/api/comentarios/mios`                    | Comentarios propios (cualquier estado)                                     | Miembro                                                                                                      | ✅ probado                              |
| POST                | `/api/comentarios`                         | Crear comentario                                                           | Miembro                                                                                                      | ✅ probado                              |
| GET                 | `/api/comentarios/pendientes`              | Bandeja de moderación                                                      | Usuario (admin/publicador)                                                                                   | ✅ probado                              |
| PUT                 | `/api/comentarios/:id/moderar`             | Aprobar/rechazar                                                           | Usuario (admin/publicador)                                                                                   | ✅ probado                              |

**Nota importante**: `POST /api/encuestas/:id/votar/:opcionId` en el código actual sigue siendo público (como se diseñó en sesión 8), pero la decisión de sesión 11 dice que debe requerir sesión de Miembro. Este endpoint necesita actualizarse para reflejar la decisión — anotado como pendiente real, no solo diseño.

## Variables de entorno

**expansion-backend/.env** (valores reales no documentados aquí por seguridad, solo la forma):

```
PORT=4000
MONGODB_URI=mongodb+srv://<usuario>:<password>@cluster0.xxxxx.mongodb.net/la-expansion?retryWrites=true&w=majority
CORS_ORIGIN=http://localhost:3000
JWT_SECRET=<valor largo y aleatorio, no compartido en el chat>
```

**expansion-frontend/.env.local**

```
NEXT_PUBLIC_API_URL=http://localhost:4000
```

## Infraestructura desplegada

- **MongoDB Atlas**: cluster `Cluster0`, tier Free (M0), AWS, `us-east-1`. Network Access `0.0.0.0/0` (necesario para Render free, sin IP fija).
- **Backend**: solo local (`localhost:4000`), sin desplegar a Render todavía.
- **Frontend**: solo local (`localhost:3000`), sin desplegar a Vercel todavía.

## Notas técnicas / bugs conocidos

**Resueltos:**

- **Mongoose 9.x, hooks síncronos sin `next()`**: el hook `pre('validate')` de `Noticia.js` (autogenerar slug) causaba `"next is not a function"` con el parámetro `next` que Mongoose 9.x ya no invoca así. Se quitó el parámetro.
- **`setState` síncrono dentro de `useEffect`**: patrón repetido en varios archivos (`admin/noticias/page.tsx`, `admin/layout.tsx`, `Navbar.tsx`). Fix: separar el fetch/lectura de estado externo de la actualización de estado — la actualización debe ir dentro de un `.then()` o `Promise.resolve().then()`, nunca como primera línea síncrona del cuerpo del efecto. Aplicar este patrón en cualquier código nuevo.
- **`GET /api/miembros` sin proteger**: quedó público al armar roles (sesión 8), exponía cédulas/teléfonos/emails. Corregido: solo Admin.
- **Navbar no reaccionaba a login/logout sin recargar**: vive en el layout raíz, no se remonta en navegación SPA. Fix: sistema de eventos custom que `guardarSesion`/`cerrarSesion` (ambos tipos) disparan.
- **Dropdown de cuenta roto en móvil**: `position: absolute` anidado dentro del menú hamburguesa. Fix: variante `mobile` de `UserMenu` sin dropdown flotante.

**Pendientes (no resueltos):**

- **Zona horaria en fechas**: `toLocaleDateString` corre un día hacia atrás fechas guardadas a medianoche UTC en horario de RD (UTC-4). Resolver cuando se construya selector de fecha real en el panel.
- **`POST /api/encuestas/:id/votar/:opcionId` sigue público**: debe actualizarse para requerir Miembro (ver Endpoints arriba).
- **`/api/voluntarios` y `/api/eventos` sin protección de rol todavía** — a diferencia de noticias/miembros/comentarios, estos dos CRUD quedaron sin `verifyToken`/`requireRolUsuario` desde que se crearon en sesión 3-4. Revisar si deben protegerse igual antes de construir su UI en el panel.
