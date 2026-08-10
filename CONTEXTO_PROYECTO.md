# CONTEXTO_PROYECTO.md

Ruta destino en el repo: `la-expansion/CONTEXTO_PROYECTO.md`

> Este documento se lee al inicio de cada sesión de trabajo con Claude para recuperar el contexto completo del proyecto, ya que Claude no mantiene memoria entre sesiones.

## Qué es este proyecto

Sitio web oficial de **La Expansión**, un movimiento político (no un partido registrado). Dirigente/secretario general: **Mario Díaz**.

Desarrollador y líder técnico: Ramon (backend, frontend, bases de datos, algo de GitHub Actions).

## Cómo trabajamos (workflow obligatorio)

1. Claude nunca asume el contenido de un archivo existente: lo pide explícitamente y espera a que Ramon lo pegue.
2. Al pedir un archivo nuevo o modificado, Claude siempre da la **ruta completa de destino** en el repo.
3. Al terminar un bloque de trabajo (una funcionalidad, un fix, una fase):
   - Se **prueba** juntos que funcione.
   - Se **actualiza** este documento y/o `ARQUITECTURA.md` si hubo cambios de estructura, stack o decisiones.
   - Se agrega una entrada en `HISTORIAL_MODIFICACIONES.md`.
   - Recién después se hace **commit**.
   - Solo entonces se empieza la siguiente parte.
4. No se avanza a la siguiente fase sin cerrar este ciclo (probar → documentar → commit).

## Estructura de repos

Tres repos independientes en GitHub (usuario `ramndiazg`), todos dentro de la misma carpeta local `la-expansion/` por organización, pero sin relación de git entre ellos:

```
la-expansion/
├── la-expansion-docs/      → github.com/ramndiazg/la-expansion-docs
│   ├── CONTEXTO_PROYECTO.md
│   ├── ARQUITECTURA.md
│   └── HISTORIAL_MODIFICACIONES.md
├── expansion-backend/      → github.com/ramndiazg/expansion-backend
└── expansion-frontend/     → github.com/ramndiazg/expansion-frontend
```

Al pedir archivos o rutas de destino, Claude las va a referir relativas a cada repo (ej. `expansion-backend/src/server.js`), no a la carpeta padre `la-expansion/`.

## Funcionalidades acordadas (v2 — versión movimiento, no partido)

### 1. Institucional

- Inicio
- Sobre el movimiento (historia, misión, visión, valores)
- Liderazgo (Mario Díaz + estructura del movimiento)

### 2. Prensa y noticias (núcleo del sitio)

- CMS de noticias/notas de prensa con panel admin
- Sala de prensa (kit de prensa: logo, fotos, bios, contacto de comunicaciones)
- Buscador y filtros por fecha/categoría
- RSS o suscripción por correo
- Galería multimedia
- "En los medios" (enlaces a cobertura externa)

### 3. Membresía

- Formulario de afiliación en línea
- Voluntariado

### 4. Transparencia

- Eliminado — no aplica a este proyecto.

### 5. Participación ciudadana

- Encuestas
- Eventos próximos
- Contacto (pendiente decidir si vive acá o dentro de Prensa — **por confirmar**)
- Newsletter — **por confirmar si se queda**
- Donaciones — **por confirmar si se queda**

### 6. Redes y difusión

- Integración/enlaces a redes sociales
- Botones de compartir en notas de prensa
- Open Graph bien configurado

### 7. Multi-idioma/accesibilidad

- Eliminado por ahora.

### 8. Técnico/infraestructura

- Frontend: Next.js (SSR/ISR para SEO de noticias) en Vercel
- Backend: Node.js/Express en Render
- Base de datos: MongoDB Atlas
- Panel de administración para publicar noticias sin tocar código
- CI/CD con GitHub Actions
- SEO técnico: sitemap.xml, robots.txt, metadatos dinámicos
- Analytics (Google Analytics o Plausible)
- Seguridad: rate limiting en formularios, captcha, backups de BD

## Restricción de presupuesto

**Todo el proyecto debe mantenerse gratis en la medida de lo posible.** Esto aplica a cada decisión de infraestructura y servicios de terceros:

- MongoDB Atlas: tier **Free** (M0, 512 MB), no M10 ni Flex.
- Vercel: plan gratuito (Hobby).
- Render: plan gratuito para el backend.
- Cualquier servicio adicional (email, analytics, captcha, etc.) debe evaluarse primero en su tier gratuito antes de considerar planes pagos.

## Fase actual

**Fase 0 — Esqueleto técnico**: crear estructura de repos, conexión backend↔MongoDB, deploy mínimo funcionando end-to-end, página de Inicio simple. Objetivo: descubrir problemas de infraestructura temprano antes de construir features de contenido.

## Decisiones pendientes de confirmar con Ramon

- Newsletter: ¿se queda o se va?
- Donaciones: ¿se queda o se va (y si aplica legalmente para un movimiento)?
- Contacto: ¿sección independiente o parte de Prensa?
