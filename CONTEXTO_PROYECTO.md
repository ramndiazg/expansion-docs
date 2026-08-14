# CONTEXTO_PROYECTO.md

Ruta destino en el repo: `la-expansion-docs/CONTEXTO_PROYECTO.md`

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
- Galería multimedia — **video separado a su propia sección "Videos" (2026-08-12), fuera de Prensa/Noticias**
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
- Imágenes: Cloudinary (tier Free, subida directa desde el navegador) — agregado 2026-08-12
- Panel de administración para publicar noticias sin tocar código
- CI/CD con GitHub Actions
- SEO técnico: sitemap.xml, robots.txt, metadatos dinámicos
- Analytics (Google Analytics o Plausible)
- Seguridad: rate limiting en formularios, captcha, backups de BD

## Decisiones de arquitectura registradas

- **Autenticación**: NO se usa NextAuth/Auth.js. Se implementará JWT propio (mismo patrón que en Muvo RD Vial) para el futuro panel admin. Aplica cuando se construya login/panel de administración.
- **Encuestas — votación (2026-08-11)**: se descarta el diseño original de sesión 8 (votación anónima pública + modelo `Inscrito` ligero para captar gente fuera del movimiento). Ahora **solo Miembros afiliados y aprobados pueden votar**, usando su cuenta existente — no se crea ningún tipo de cuenta liviana solo-para-votar. Prioriza seguridad (un voto por persona real, verificada) sobre alcance viral. El modelo `Inscrito` NO se va a construir.
- **Estructura de paneles**: Admin y Publicador comparten un solo panel en `/admin`, con visibilidad de funciones condicionada por rol (no son paneles separados). Miembro no tiene panel administrativo — tiene un área liviana propia (`/cuenta`) con: cambiar contraseña, ver comentarios propios (votos en encuestas pendiente de mostrar ahí). Simple pero debe existir — no es opcional.
- **Cambio de contraseñas por Admin**: los Admin deben poder cambiar la contraseña de cuentas de Usuario y de Miembro (ej. si alguien la olvida y no hay recuperación por correo todavía). Sin construir aún.
- **Recuperación de contraseña (futuro, depende de correo)**: cuando se implemente envío de correos (aún no decidido cómo — pendiente de servicio gratuito), todos los tipos de cuenta (Usuario y Miembro) deben poder recuperar su contraseña por email. Bloqueado hasta tener el servicio de correo resuelto.
- **Compartir en redes (2026-08-12)**: implementado con Web Share API (`navigator.share`) como opción principal en móvil — permite compartir a cualquier app instalada (WhatsApp, Instagram, X, Facebook, etc.) sin integrarlas una por una. Fallback en desktop: íconos directos de WhatsApp/Facebook/X + "Copiar link". Componente reusable `ShareButtons`, usado en noticias y encuestas.
- **Encuestas — anti-doble-voto (2026-08-12)**: además de exigir sesión de Miembro para votar (sesión 11), se agregó un array `votantes` en el modelo `Encuesta` para impedir que un mismo Miembro vote más de una vez en la misma encuesta.
- **Flujo de link compartido de encuesta sin sesión (2026-08-12)**: quien recibe un link de encuesta sin ser Miembro ve la pregunta y puede afiliarse ahí mismo, pero la afiliación sigue quedando en estado `pendiente` de aprobación (no hay auto-login) — vuelve con el mismo link a votar una vez aprobada.
- **Descubribilidad de encuestas (2026-08-12)**: al probar el flujo de voto se detectó que no había forma de encontrar una encuesta sin recibir el link exacto — no aparecía en Home, Navbar ni ningún listado. Se agregó `/encuestas` (listado), link en el Navbar, y un widget de "Encuesta activa" en el Home.
- **Imágenes en noticias vía Cloudinary (2026-08-12)**: el modelo de noticias ya tenía campos para imagen destacada y galería, pero el formulario del panel nunca los pedía (solo se podían llenar por `curl`). Se resolvió agregando subida real de archivos desde el panel, usando Cloudinary (tier Free, preset `unsigned`) porque Render (plan free) tiene disco efímero y no puede guardar archivos de forma permanente.
- **Video separado de Noticias (2026-08-12)**: se descartó tener `videoUrl` dentro del modelo `Noticia` — mezclaba dos tipos de contenido distintos en un mismo formulario. Se creó un modelo y sección propios, `Video`, con formulario simple (solo título + link de YouTube) y su propia página pública `/videos` (grilla de embeds, sin páginas de detalle individuales por ahora).

## Restricción de presupuesto

**Todo el proyecto debe mantenerse gratis en la medida de lo posible.** Esto aplica a cada decisión de infraestructura y servicios de terceros:

- MongoDB Atlas: tier **Free** (M0, 512 MB), no M10 ni Flex.
- Vercel: plan gratuito (Hobby).
- Render: plan gratuito para el backend.
- Cloudinary: plan gratuito (25 GB combinados de almacenamiento/ancho de banda al mes).
- Cualquier servicio adicional (email, analytics, captcha, etc.) debe evaluarse primero en su tier gratuito antes de considerar planes pagos.

## Fase actual (actualizado 2026-08-12, sesión 14)

**Fase 0 — Esqueleto técnico: completada** (sesión 5).

**Fase 1 — Construcción de features: en curso.** Completado hasta ahora:

- Páginas institucionales (Inicio, Sobre el movimiento, Liderazgo) con contenido placeholder
- Sistema de noticias completo: CRUD backend, listado/detalle público, panel de creación y edición, con imagen destacada y galería reales (Cloudinary)
- Sistema de auth JWT unificado (Usuario y Miembro por el mismo login)
- Roles Admin/Publicador con permisos diferenciados
- Afiliación de miembros (con selects geográficos reales) + aprobación desde el panel
- Comentarios en noticias (solo Miembros aprobados) + moderación desde el panel
- Menú de cuenta (UserMenu) dinámico en el sitio público
- Cambio de contraseña (Usuario y Miembro, mismo endpoint)
- Área de Miembro (`/cuenta`): contraseña + comentarios propios
- Sistema de encuestas completo: modelo con anti-doble-voto, endpoints protegidos (solo Miembro vota), página pública de detalle y de listado (`/encuestas`), panel admin (crear/cerrar/eliminar), descubrible desde Home/Navbar/listado
- Sección de Videos (separada de Noticias): modelo propio, panel admin, grilla pública `/videos` — **código completo, pendiente de que Ramon confirme la prueba**
- Compartir en redes sociales (WhatsApp, Facebook, X, Web Share API nativo) en noticias y encuestas, vía componente reusable `ShareButtons`
- Flujo de redirect a login/afiliación cuando alguien sin sesión de Miembro intenta votar desde un link compartido
- Backend desplegado en Render, frontend desplegado en Vercel (dominio: `https://expansion-frontend.vercel.app`)
- Panel admin: link "Ver sitio" para volver al sitio público sin cerrar sesión

**Pendiente para continuar la Fase 1** (ver detalle completo en `ARQUITECTURA.md` → "Aún no construido"):

- Confirmar prueba end-to-end de la sección de Videos (crear, publicar, ver embed)
- Activar/desactivar cuentas de Usuario
- Gestión de Voluntario/Evento en el panel (y decidir si proteger esas rutas del backend)
- Contenido real reemplazando los placeholders
- Confirmar newsletter/donaciones/Contacto (ver más abajo, sigue sin decidir)

**Fase 2 (no iniciada)**: dominio propio (hoy corre en subdominios gratuitos de Render/Vercel), contenido final.

## Decisiones pendientes de confirmar con Ramon

- Newsletter: ¿se queda o se va?
- Donaciones: ¿se queda o se va (y si aplica legalmente para un movimiento)?
- Contacto: ¿sección independiente o parte de Prensa?
