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

Nota: `expansion-frontend` todavía no se decidió App Router vs Pages Router — create-next-app ya lo generó, se documentará qué usa en el próximo bloque.

## Variables de entorno

**expansion-backend/.env** (real, valores no documentados aquí por seguridad — solo la forma)

```
PORT=4000
MONGODB_URI=mongodb+srv://<usuario>:<password>@cluster0.xxxxx.mongodb.net/la-expansion?retryWrites=true&w=majority
CORS_ORIGIN=http://localhost:3000
```

**expansion-frontend/.env** — pendiente, se completa en el próximo bloque (conexión frontend→backend).

## Infraestructura desplegada

- **MongoDB Atlas**: cluster `Cluster0`, tier **Free (M0)**, proveedor AWS, región `us-east-1` (N. Virginia). Network Access: `0.0.0.0/0` (allow anywhere) — necesario porque Render (plan free) no tiene IP fija.
- **Backend**: corriendo localmente en `localhost:4000`, aún no desplegado a Render.
- **Frontend**: aún no desplegado a Vercel.

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

| Método              | Ruta                                       | Descripción                                                    | Estado                                            |
| ------------------- | ------------------------------------------ | -------------------------------------------------------------- | ------------------------------------------------- |
| GET                 | `/api/health`                              | Verifica que el servidor y la conexión a MongoDB estén activos | ✅ funcionando                                    |
| GET                 | `/api/noticias`                            | Lista noticias (filtros opcionales `?estado=` `?categoria=`)   | ✅ probado                                        |
| GET                 | `/api/noticias/:slug`                      | Obtiene una noticia por slug                                   | ✅ (patrón replicado, no probado individualmente) |
| POST                | `/api/noticias`                            | Crea noticia (slug autogenerado)                               | ✅ probado                                        |
| PUT                 | `/api/noticias/:id`                        | Actualiza noticia                                              | ✅ (patrón replicado, no probado individualmente) |
| DELETE              | `/api/noticias/:id`                        | Elimina noticia                                                | ✅ (patrón replicado, no probado individualmente) |
| GET/POST/PUT/DELETE | `/api/miembros`, `/api/miembros/:id`       | CRUD de afiliación                                             | ⚠️ mismo patrón que Noticia, aún sin probar       |
| GET/POST/PUT/DELETE | `/api/voluntarios`, `/api/voluntarios/:id` | CRUD de voluntariado                                           | ⚠️ mismo patrón, aún sin probar                   |
| GET/POST/PUT/DELETE | `/api/eventos`, `/api/eventos/:id`         | CRUD de eventos                                                | ⚠️ mismo patrón, aún sin probar                   |
| GET/POST/DELETE     | `/api/encuestas`, `/api/encuestas/:id`     | CRUD de encuestas                                              | ⚠️ mismo patrón, aún sin probar                   |
| POST                | `/api/encuestas/:id/votar/:opcionId`       | Incrementa el voto de una opción                               | ⚠️ aún sin probar                                 |

## Notas técnicas / bugs conocidos y resueltos

- **Mongoose 9.x — hooks síncronos no usan `next()`**: el hook `pre('validate')` de `Noticia.js` originalmente incluía un parámetro `next` que causaba `"next is not a function"` al no ser invocado correctamente por esta versión de Mongoose. Se corrigió quitando el parámetro `next` del callback (hook síncrono, sin callback). Si se agregan más hooks `pre`/`post` en otros modelos, verificar si necesitan ser async o síncronos según corresponda, en vez de asumir el estilo clásico de Mongoose 6/7.

## Decisiones técnicas registradas

- (vacío por ahora — cada decisión importante de arquitectura se anota aquí con fecha y motivo breve)
