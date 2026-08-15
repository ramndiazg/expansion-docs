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
5. Ramon prefiere recibir los archivos **completos** (no fragmentos/diffs) para pegar en su editor sin errores de aplicación.

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

## Funcionalidades acordadas (v2 — versión movimiento, no partido)

### 1. Institucional

- Inicio
- Sobre el movimiento (historia, misión, visión, valores)
- Liderazgo (Mario Díaz + estructura del movimiento)

### 2. Sala de Prensa (núcleo del sitio) — renombrada de "Prensa y noticias" en sesión 15

- CMS de noticias/notas de prensa con panel admin (ruta pública `/prensa`, ruta interna de admin sigue siendo `/admin/noticias`)
- Sala de prensa (kit de prensa: logo, fotos, bios, contacto de comunicaciones) — no construido aún
- Buscador y filtros por categoría — **construido sesión 15** (filtro por fecha aún no)
- RSS o suscripción por correo — no construido
- Galería multimedia — separada a su propia sección "Videos" (sesión 14), pendiente renombrar a "Multimedia"
- "En los medios" — sigue siendo una categoría de noticia, no una sección aparte

### 3. Membresía

- Formulario de afiliación en línea — **auto-aprobada desde sesión 15** (antes requería aprobación manual)
- Voluntariado — modelo/CRUD backend existen, sin panel ni protección de rol

### 4. Transparencia

- Eliminado — no aplica a este proyecto.

### 5. Participación ciudadana

- Encuestas — **votación abierta sin cuenta desde sesión 15** (antes solo Miembros)
- Eventos próximos — modelo/CRUD backend existen, sin panel ni protección de rol
- Contacto (pendiente decidir si vive acá o dentro de Prensa — **por confirmar**)
- Newsletter — **por confirmar si se queda**
- Donaciones — **por confirmar si se queda**

### 6. Redes y difusión

- Integración/enlaces a redes sociales
- Botones de compartir en notas de prensa y encuestas — construido (`ShareButtons`, sesión 13)
- Open Graph bien configurado — construido sesión 15

### 7. Multi-idioma/accesibilidad

- Eliminado por ahora.

### 8. Técnico/infraestructura

- Frontend: Next.js (SSR/ISR para SEO) en Vercel
- Backend: Node.js/Express en Render
- Base de datos: MongoDB Atlas
- Imágenes: Cloudinary (tier Free, subida directa desde el navegador)
- Panel de administración para publicar noticias sin tocar código
- CI/CD con GitHub Actions — pendiente de configurar
- SEO técnico: sitemap.xml, robots.txt, metadatos dinámicos — **construido sesión 15**
- Analytics (Google Analytics o Plausible) — no construido
- Seguridad: rate limiting en formularios, captcha, backups de BD — no construido

## Decisiones de arquitectura registradas

- **Autenticación**: NO se usa NextAuth/Auth.js. JWT propio (mismo patrón que Muvo RD Vial). Aplica a Usuario (panel) y Miembro (comentarios, área de cuenta) — **ya no aplica a votar en encuestas** desde sesión 15.
- **Estructura de paneles**: Admin y Publicador comparten un solo panel en `/admin`, con visibilidad de funciones condicionada por rol. Miembro tiene un área liviana propia (`/cuenta`).
- **Cambio de contraseñas por Admin**: pendiente de construir.
- **Recuperación de contraseña por email**: bloqueado hasta tener un servicio de correo gratuito resuelto.
- **Compartir en redes (sesión 13)**: Web Share API en móvil (cualquier app instalada) + fallback WhatsApp/Facebook/X en desktop. Componente `ShareButtons`, usado en Sala de Prensa y encuestas.
- **Video separado de la Sala de Prensa (sesión 14)**: sección propia "Videos" (modelo simple: título + link de YouTube), sin páginas de detalle individuales.
- **Imágenes vía Cloudinary (sesión 14)**: Render (plan free) tiene disco efímero, no puede guardar archivos subidos de forma permanente — Cloudinary (tier Free, preset `unsigned`) resuelve la subida real desde el panel.
- **Descubribilidad de encuestas (sesión 14)**: listado `/encuestas`, link en Navbar, widget en Home — antes solo se llegaba por link compartido directo.
- **Logo e identidad visual (sesión 15)**: emblema de tres círculos concéntricos con corte parcial, en azul/blanco/rojo (colores del movimiento) sobre fondo azul oscuro. Usado en Navbar, Home y favicon.
- **Sala de Prensa — renombre y rediseño (sesión 15)**: "Noticias" no reflejaba el contenido real (comunicados, declaraciones, actividades, cobertura externa). Renombrada a "Sala de Prensa" (solo cara pública, ruta `/prensa`; el admin y el modelo `Noticia` no cambiaron de nombre). Listado rediseñado con tarjeta destacada + miniaturas, antes era solo texto.
- **Votación abierta en encuestas (sesión 15, revierte decisión de sesión 11)**: exigir sesión de Miembro para votar frenaba la viralidad de las encuestas compartidas. Ahora cualquiera vota sin cuenta, protegido por un identificador anónimo en el navegador (`localStorage`) que previene doble voto desde el mismo dispositivo — no es a prueba de alguien que borre esa marca a propósito, pero es el estándar razonable para este caso de uso.
- **Afiliación auto-aprobada (sesión 15, revierte parcialmente decisiones de sesiones 8/11)**: la aprobación manual de cada solicitud no escalaba y frenaba el crecimiento. Ahora las cuentas de Miembro quedan activas de inmediato al registrarse, con **validación de formato de cédula dominicana** (`XXX-XXXXXXX-X`, ej. `001-1566974-2`) y unicidad de cédula/email como control de calidad en el registro. El Admin conserva la capacidad de desactivar/rechazar una cuenta **después** del registro (moderación posterior en vez de previa) — `/admin/miembros` sigue existiendo para eso.

## Restricción de presupuesto

**Todo el proyecto debe mantenerse gratis en la medida de lo posible.**

- MongoDB Atlas: tier **Free** (M0, 512 MB).
- Vercel: plan gratuito (Hobby).
- Render: plan gratuito para el backend.
- Cloudinary: plan gratuito (25 GB combinados de almacenamiento/ancho de banda al mes).
- Cualquier servicio adicional debe evaluarse primero en su tier gratuito.

## Fase actual (actualizado 2026-08-12, sesión 15)

**Fase 0 — Esqueleto técnico: completada** (sesión 5).

**Fase 1 — Construcción de features: en curso.** Completado hasta ahora:

- Páginas institucionales (Inicio, Sobre el movimiento, Liderazgo) con contenido placeholder
- **Sala de Prensa** (antes "Noticias"): CRUD backend, imagen destacada + galería (Cloudinary), buscador y filtro por categoría, listado con tarjeta destacada + miniaturas, panel de creación/edición
- Sistema de auth JWT (Usuario y Miembro por el mismo login, pero votar en encuestas ya no lo requiere)
- Roles Admin/Publicador con permisos diferenciados
- **Afiliación auto-aprobada**, con validación de formato de cédula y unicidad
- Comentarios en noticias (sigue requiriendo cuenta de Miembro) + moderación desde el panel
- Menú de cuenta (UserMenu) dinámico
- Cambio de contraseña (Usuario y Miembro)
- Área de Miembro (`/cuenta`) — nota: "mis votos" quedó desactualizado porque votar ya no requiere cuenta
- **Encuestas con votación abierta sin cuenta** (protegida por marca en el navegador), descubribles desde Home/Navbar/listado, panel admin, compartibles
- Sección de Videos (separada de Sala de Prensa) — código completo, pendiente de que Ramon confirme la prueba
- Compartir en redes sociales (`ShareButtons`) en Sala de Prensa y encuestas
- **Open Graph** (vista previa al compartir) y **SEO técnico** (sitemap.xml, robots.txt, metadatos dinámicos)
- **Logo e identidad visual** (Navbar, Home, favicon)
- Backend en Render, frontend en Vercel, todas las variables de entorno correctamente configuradas en ambos (incluyendo `NEXT_PUBLIC_SITE_URL`, que faltaba y causaba links de compartir rotos)
- Panel admin: link "Ver sitio"

**Pendiente para continuar la Fase 1**:

- Confirmar prueba end-to-end de la sección de Videos
- Proteger `/api/voluntarios` y `/api/eventos` con roles
- Paneles de Voluntario y Evento (backend ya existe)
- Activar/desactivar cuentas de Usuario
- Redefinir o quitar "mis votos en encuestas" en `/cuenta` (ya no aplica sin cuenta para votar)
- Renombrar Videos a "Multimedia" y seguir potenciando esa sección y Sala de Prensa
- Contenido real reemplazando los placeholders
- Confirmar newsletter/donaciones/Contacto

**Fase 2 (no iniciada)**: dominio propio, contenido final.

## Decisiones pendientes de confirmar con Ramon

- Newsletter: ¿se queda o se va?
- Donaciones: ¿se queda o se va?
- Contacto: ¿sección independiente o parte de Sala de Prensa?
