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

## Estructura de carpetas (objetivo Fase 0)

```
la-expansion/
├── expansion-backend/
│   ├── src/
│   │   ├── models/       (esquemas Mongoose)
│   │   ├── controllers/
│   │   ├── routes/
│   │   ├── middleware/
│   │   └── server.js
│   ├── .env.example
│   └── package.json
│
└── expansion-frontend/
    ├── app/ (o pages/, según se decida)
    ├── components/
    ├── public/
    ├── .env.example
    └── package.json
```

> Estado real: **aún no creada.** Se actualizará esta sección en cuanto Ramon confirme la estructura definitiva de cada repo (App Router vs Pages Router en Next.js, por ejemplo).

## Variables de entorno (placeholder, se completa en Fase 0)

**expansion-backend/.env**

```
PORT=
MONGODB_URI=
CORS_ORIGIN=
```

**expansion-frontend/.env**

```
NEXT_PUBLIC_API_URL=
```

## Modelos de datos (Mongoose)

> Pendiente — se documentará cada esquema aquí a medida que se cree (Noticia, Miembro/Afiliación, Voluntario, Evento, Encuesta).

## Endpoints del backend

> Pendiente — se documentará cada endpoint aquí a medida que se cree.

## Decisiones técnicas registradas

- (vacío por ahora — cada decisión importante de arquitectura se anota aquí con fecha y motivo breve)
