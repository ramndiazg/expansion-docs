# ARQUITECTURA.md

Ruta destino en el repo: `la-expansion/ARQUITECTURA.md`

> Este documento describe la estructura técnica real del proyecto. Se actualiza cada vez que cambia algo estructural: carpetas, endpoints, esquemas, variables de entorno, dependencias clave.

## Stack

| Capa          | Tecnología        | Deploy |
| ------------- | ----------------- | ------ |
| Frontend      | Next.js           | Vercel |
| Backend       | Node.js + Express | Render |
| Base de datos | MongoDB Atlas     | —      |
| CI/CD         | GitHub Actions    | —      |

## Estructura de carpetas (estado real, confirmado 2026-08-09)

```
la-expansion/
├── CONTEXTO_PROYECTO.md
├── ARQUITECTURA.md
├── HISTORIAL_MODIFICACIONES.md
├── expansion-backend/
│   ├── src/
│   │   ├── models/       (vacío aún — esquemas Mongoose)
│   │   ├── controllers/  (vacío aún)
│   │   ├── routes/       (vacío aún)
│   │   ├── middleware/   (vacío aún)
│   │   └── server.js     ✅ creado y funcionando
│   ├── .env              (no versionado — ver .gitignore)
│   ├── .env.example      ✅
│   ├── .gitignore        ✅
│   └── package.json      ✅ (scripts start/dev, main actualizado)
│
└── expansion-frontend/
    └── (scaffold default de create-next-app, aún sin tocar)
```

Nota: `expansion-frontend` usa **App Router**, TypeScript, sin carpeta `src/` (todo vive directo en `app/`). Confirmado 2026-08-10.

## Sistema de diseño (v2 — actualizado 2026-08-10)

**Principio: mobile-first.** El tráfico esperado es mayoritariamente desde móvil — todo bloque nuevo debe diseñarse y probarse primero en viewport angosto.

**Paleta:**
| Token | Hex | Uso |
|---|---|---|
| `--ink` | `#101828` | Texto principal, fondo del hero/footer |
| `--ink-soft` | `#1D2939` | Bloques placeholder, acentos oscuros secundarios |
| `--blue` | `#4E7FDB` | Acento primario (CTAs, labels destacados) — versión suave, no saturado |
| `--slate` | `#64748B` | Acento secundario (bordes, detalles) |
| fondo | `#FFFFFF` (blanco nativo) | Fondo base |

Sin dark mode automático.

**Tipografía**: Space Grotesk (display, títulos) + Inter (body) — 100% sans-serif, reemplazó a Fraunces por sentirse "anticuado" combinado con la paleta anterior.

**Elemento de firma**: motivo de anillos concéntricos (SVG) en el hero del Home.

**Navegación**: menú hamburguesa en móvil (`components/Navbar.tsx`, `"use client"`, estado `open` con `useState`), menú horizontal en desktop (`md:flex`).

**Personalismo del líder**: Mario Díaz debe tener presencia visible fuera de la página de Liderazgo — implementado en el hero del Home (línea "Liderado por Mario Díaz, Secretario General" con avatar placeholder).

## Frontend — páginas y estado

| Página              | Ruta                               | Descripción                                                                                             | Estado         |
| ------------------- | ---------------------------------- | ------------------------------------------------------------------------------------------------------- | -------------- |
| Inicio              | `app/page.tsx`                     | Hero con anillos SVG + fetch a `/api/health` _(ver nota)_ + sección de pilares (placeholder)            | ✅ funcionando |
| Sobre el movimiento | `app/sobre-el-movimiento/page.tsx` | Historia, misión, visión, valores — **todo contenido placeholder**                                      | ✅ funcionando |
| Liderazgo           | `app/liderazgo/page.tsx`           | Bio de Mario Díaz + estructura organizativa — **todo contenido placeholder**                            | ✅ funcionando |
| Noticias (listado)  | `app/noticias/page.tsx`            | Server component, fetch a `/api/noticias?estado=publicado`, tarjetas con categoría/título/resumen/fecha | ✅ funcionando |
| Noticias (detalle)  | `app/noticias/[slug]/page.tsx`     | Server component, fetch a `/api/noticias/:slug`, `notFound()` si no existe                              | ✅ funcionando |

**Nota**: el `app/page.tsx` de esta sesión reemplazó la versión que hacía fetch a `/api/health` (sesión 5) por la versión con hero de marca. El check de salud del backend ya no es visible en Home — si se necesita, se debe reintegrar deliberadamente (ej. en una futura página de estado/diagnóstico), no en la página pública.

**Componentes compartidos**: `components/Navbar.tsx` (logo + enlaces + botón Afíliate), `components/Footer.tsx`. Ambos registrados en `app/layout.tsx`.

**Pendiente conocido**: el botón "Afíliate" enlaza a `/afiliate`, que aún no existe (404 esperado hasta construir el formulario de membresía).

## Panel admin (frontend)

| Ruta                                | Descripción                                                                   | Estado     |
| ----------------------------------- | ----------------------------------------------------------------------------- | ---------- |
| `app/admin/login/page.tsx`          | Login de Usuario, guarda token+datos en `localStorage`                        | ✅ probado |
| `app/admin/layout.tsx`              | Protege todas las rutas `/admin/*` (excepto login), redirige si no hay sesión | ✅ probado |
| `app/admin/page.tsx`                | Dashboard simple, saludo + accesos                                            | ✅ probado |
| `app/admin/noticias/page.tsx`       | Listado de noticias con botón Publicar/Despublicar (`PUT` estado)             | ✅ probado |
| `app/admin/noticias/nueva/page.tsx` | Formulario de creación (crea como `borrador` por diseño)                      | ✅ probado |

`lib/auth.ts`: helpers de sesión (`guardarSesion`, `obtenerToken`, `obtenerUsuario`, `cerrarSesion`) usando `localStorage` — no se usa NextAuth (decisión de arquitectura previa).

**Aún no construido en el panel**: gestión de Miembro/Voluntario/Evento/Encuesta, bandeja de moderación de comentarios, dashboard de estadísticas real, edición de noticias existentes (solo hay crear + publicar/despublicar, no editar título/contenido).

**Todo el contenido de texto (historia, misión, visión, valores, bio de Mario Díaz, estructura organizativa, eslogan del hero, pilares) es placeholder** y debe reemplazarse con copy real antes de producción — buscar comentarios `{/* PLACEHOLDER: ... */}` en el código para ubicarlos todos.

## Variables de entorno

**expansion-backend/.env** (real, valores no documentados aquí por seguridad — solo la forma)

```
PORT=4000
MONGODB_URI=mongodb+srv://<usuario>:<password>@cluster0.xxxxx.mongodb.net/la-expansion?retryWrites=true&w=majority
CORS_ORIGIN=http://localhost:3000
```

**expansion-frontend/.env.local**

```
NEXT_PUBLIC_API_URL=http://localhost:4000
```

## Infraestructura desplegada

- **MongoDB Atlas**: cluster `Cluster0`, tier **Free (M0)**, proveedor AWS, región `us-east-1` (N. Virginia). Network Access: `0.0.0.0/0` (allow anywhere) — necesario porque Render (plan free) no tiene IP fija.
- **Backend**: corriendo localmente en `localhost:4000`, aún no desplegado a Render.
- **Frontend**: corriendo localmente en `localhost:3000`, conectado exitosamente al backend vía `/api/health`. Aún no desplegado a Vercel.

## Modelos de datos (Mongoose)

Todos en `expansion-backend/src/models/`.

**Noticia.js**
| Campo | Tipo | Notas |
|---|---|---|
| titulo | String | requerido |
| slug | String | único, autogenerado del título si no se envía |
| resumen | String | requerido, máx 300 caracteres |
| contenido | String | requerido |
| imagenDestacada | String | URL |
| imagenesAdicionales | [String] | URLs de fotos adicionales — ⚠️ agregado 2026-08-10, sin probar |
| videoUrl | String | Link de YouTube, se convierte a embed en el frontend — ⚠️ agregado 2026-08-10, sin probar |
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
| provincia | String | requerido |
| municipio | String | |
| sectorInteres | String | |
| estado | enum | `pendiente` \| `aprobado` \| `rechazado` |

**Comentario.js**
| Campo | Tipo | Notas |
|---|---|---|
| noticia | ObjectId → Noticia | requerido |
| miembro | ObjectId → Miembro | requerido — solo miembros afiliados y aprobados pueden comentar |
| texto | String | requerido, máx 1000 |
| estado | enum | `pendiente` \| `aprobado` \| `rechazado` (pre-moderación) |

**Usuario.js** (cuentas del panel — Admin/Publicador)
| Campo | Tipo | Notas |
|---|---|---|
| nombre, email | String | requeridos, email único |
| passwordHash | String | recibido en texto plano, hasheado con bcrypt en hook `pre('save')` |
| rol | enum | `admin` \| `publicador` |
| activo | Boolean | default true — solo **admin** puede desactivar |

**Miembro.js** — actualizado: se agregó `passwordHash` (mismo patrón de hash que Usuario) para permitir login de miembros y así comentar.

**Encuesta.js** — actualizado: se agregó `creadoPor` (ref a Usuario).

## Sistema de roles y permisos

| Acción                             | Publicador                                                  | Admin                                                                             |
| ---------------------------------- | ----------------------------------------------------------- | --------------------------------------------------------------------------------- |
| CRUD de noticias                   | ✅                                                          | ✅                                                                                |
| Crear/cerrar sus propias encuestas | ✅                                                          | ✅ (+ cualquier encuesta, no solo las propias)                                    |
| Moderar comentarios                | ✅                                                          | ✅                                                                                |
| Ver dashboard/estadísticas         | —                                                           | ✅                                                                                |
| Activar cuentas nuevas del panel   | ✅ (cualquiera puede dar de alta, queda activa por defecto) | ✅                                                                                |
| **Desactivar** cuentas del panel   | ❌                                                          | ✅ (única acción exclusiva de Admin, para no crear cuello de botella en el resto) |

## Autenticación (JWT)

Dos tipos de sesión distintos, mismo mecanismo de token (`JWT_SECRET`, expira en 7 días):

- **Usuario** (panel): `POST /api/auth/login` → `{ id, tipo: 'usuario', rol }`
- **Miembro** (comentar): `POST /api/auth/miembro-login` → `{ id, tipo: 'miembro' }` — requiere `estado: 'aprobado'`

Middleware en `expansion-backend/src/middleware/auth.js`:

- `verifyToken`: valida el JWT, cuelga el payload en `req.auth`
- `requireRolUsuario(...roles)`: exige tipo `usuario` y rol específico
- `requireMiembro`: exige tipo `miembro`

**Frontend**: token + datos de usuario guardados en `localStorage` (`lib/auth.ts`), sin usar NextAuth (decisión previa). `app/admin/layout.tsx` protege todas las rutas `/admin/*` excepto `/admin/login`, redirigiendo si no hay token.

## Encuestas públicas (diseño acordado, pendiente de construir)

- Cada encuesta tendrá una página pública (`/encuestas/[id]` o `[slug]`) compartible en redes.
- Votación **anónima**, sin cuenta — protección básica contra voto múltiple vía `localStorage` (no infalible, es el estándar para este tipo de encuesta abierta).
- Tras votar, invitación opcional a **"Inscribirse"** (no "afiliarse" — término elegido deliberadamente para sugerir menos compromiso): nombre, email, teléfono opcional.
- Requiere modelo nuevo `Inscrito`, distinto de `Miembro` (afiliación formal) y `Voluntario`.
- **Estado: diseño acordado, código aún no escrito** — construir en próxima sesión.

## Endpoints del backend (actualizado, agregados en esta sesión)

| Método | Ruta                                  | Descripción                                        | Auth                       | Estado        |
| ------ | ------------------------------------- | -------------------------------------------------- | -------------------------- | ------------- |
| POST   | `/api/auth/login`                     | Login de Usuario (panel)                           | pública                    | ✅ probado    |
| POST   | `/api/auth/miembro-login`             | Login de Miembro (comentar)                        | pública                    | ⚠️ sin probar |
| GET    | `/api/comentarios/noticia/:noticiaId` | Comentarios aprobados de una noticia               | pública                    | ⚠️ sin probar |
| POST   | `/api/comentarios`                    | Crear comentario                                   | Miembro                    | ⚠️ sin probar |
| GET    | `/api/comentarios/pendientes`         | Bandeja de moderación                              | Usuario (admin/publicador) | ⚠️ sin probar |
| PUT    | `/api/comentarios/:id/moderar`        | Aprobar/rechazar comentario                        | Usuario (admin/publicador) | ⚠️ sin probar |
| PUT    | `/api/encuestas/:id/cerrar`           | Cierra encuesta (propia, o cualquiera si es admin) | Usuario (admin/publicador) | ⚠️ sin probar |

**Noticias**: `POST`/`PUT`/`DELETE` ahora requieren token de Usuario con rol admin o publicador — confirmado con la prueba end-to-end del panel (crear + publicar/despublicar).

**Voluntario.js**
| Campo | Tipo | Notas |
|---|---|---|
| nombre, apellido | String | requeridos |
| email, telefono | String | requeridos |
| provincia | String | requerido |
| areaInteres | String | |
| disponibilidad | String | |
| mensaje | String | máx 500 |
| estado | enum | `pendiente` \| `contactado` \| `activo` |

**Evento.js**
| Campo | Tipo | Notas |
|---|---|---|
| titulo, descripcion | String | requeridos |
| fecha | Date | requerido |
| lugar | String | requerido |
| imagen | String | |
| requiereInscripcion | Boolean | default false |
| cupoMaximo | Number | opcional |
| estado | enum | `proximo` \| `realizado` \| `cancelado` |

**Encuesta.js**
| Campo | Tipo | Notas |
|---|---|---|
| pregunta | String | requerido |
| opciones | [{ texto, votos }] | mínimo 2 opciones, subdocumento con `_id` |
| activa | Boolean | default true |
| fechaCierre | Date | opcional |

Todos incluyen `timestamps: true` (createdAt/updatedAt automáticos).

## Endpoints del backend

| Método              | Ruta                                       | Descripción                                                    | Estado                                                |
| ------------------- | ------------------------------------------ | -------------------------------------------------------------- | ----------------------------------------------------- |
| GET                 | `/api/health`                              | Verifica que el servidor y la conexión a MongoDB estén activos | ✅ funcionando                                        |
| GET                 | `/api/noticias`                            | Lista noticias (filtros opcionales `?estado=` `?categoria=`)   | ✅ probado                                            |
| GET                 | `/api/noticias/:slug`                      | Obtiene una noticia por slug                                   | ✅ (patrón replicado, no probado individualmente)     |
| POST                | `/api/noticias`                            | Crea noticia (slug autogenerado)                               | ✅ probado                                            |
| PUT                 | `/api/noticias/:id`                        | Actualiza noticia                                              | ✅ probado (usado para publicar la noticia de prueba) |
| DELETE              | `/api/noticias/:id`                        | Elimina noticia                                                | ✅ (patrón replicado, no probado individualmente)     |
| GET/POST/PUT/DELETE | `/api/miembros`, `/api/miembros/:id`       | CRUD de afiliación                                             | ⚠️ mismo patrón que Noticia, aún sin probar           |
| GET/POST/PUT/DELETE | `/api/voluntarios`, `/api/voluntarios/:id` | CRUD de voluntariado                                           | ⚠️ mismo patrón, aún sin probar                       |
| GET/POST/PUT/DELETE | `/api/eventos`, `/api/eventos/:id`         | CRUD de eventos                                                | ⚠️ mismo patrón, aún sin probar                       |
| GET/POST/DELETE     | `/api/encuestas`, `/api/encuestas/:id`     | CRUD de encuestas                                              | ⚠️ mismo patrón, aún sin probar                       |
| POST                | `/api/encuestas/:id/votar/:opcionId`       | Incrementa el voto de una opción                               | ⚠️ aún sin probar                                     |

## Notas técnicas / bugs conocidos y resueltos

- **Mongoose 9.x — hooks síncronos no usan `next()`**: el hook `pre('validate')` de `Noticia.js` originalmente incluía un parámetro `next` que causaba `"next is not a function"` al no ser invocado correctamente por esta versión de Mongoose. Se corrigió quitando el parámetro `next` del callback (hook síncrono, sin callback). Si se agregan más hooks `pre`/`post` en otros modelos, verificar si necesitan ser async o síncronos según corresponda, en vez de asumir el estilo clásico de Mongoose 6/7.
- **Zona horaria en fechas (conocido, no resuelto)**: `toLocaleDateString` en las páginas de noticias interpreta fechas guardadas como medianoche UTC y las corre un día hacia atrás al mostrarlas en horario de RD (UTC-4). Ej: `fechaPublicacion: "2026-08-10T00:00:00.000Z"` se muestra como "9 de agosto". Pendiente de resolver cuando se construya el selector de fecha en el panel admin (probablemente forzando la hora a mediodía UTC, o formateando explícitamente en UTC en el frontend).

## Decisiones técnicas registradas

- (vacío por ahora — cada decisión importante de arquitectura se anota aquí con fecha y motivo breve)
