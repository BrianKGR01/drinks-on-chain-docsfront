# 07 · Roadmap de la landing principal (`drinks-on-chain-landing`)

Versión 2 · 25 de septiembre de 2026 (v1 en `antiguo/`). Sitio del dominio raíz. Público primario: el consumidor. Sin autenticación, sin WebGL, carga en menos de dos segundos en móvil. Trabajo en `dev`, PR a `main` al cerrar cada hito.

## Alcance fijo
- Enlaces salientes por variables de entorno (`NEXT_PUBLIC_URL_APP`, `NEXT_PUBLIC_URL_BODEGAS`); nunca hosts en el código.
- Contenido compartido con el sitio de bodegas copiado hoy desde `drinks-on-chain-front/src/content`; se extrae a `@doc/content` en la Etapa 0 del roadmap global.
- Mismos tokens, tipografías y componentes editoriales que el sitio de bodegas. El mapa es un dibujo SVG generado desde la geometría compartida.
- Barrera de edad en la primera visita. ES por defecto, EN disponible.
- Marca única Drinks on Chain.

## Hitos

| Hito | Contenido | Estado |
|---|---|---|
| **M0 · Fundaciones** | Repo, Next 16, TypeScript estricto, Tailwind 4, tokens, tipografías, `Logo`, `AgeGate`, cabecera con "Entrar", pie, ES/EN, contenido copiado, variables de entorno, CI, `dev`/`main`, README | **Hecho** (24-09-2026) |
| **M1 · Inicio** | Héroe, Cómo funciona, Vinos en la red, Las bodegas, Qué garantizamos, franja B2B, pie | **Hecho** en primera versión (24-09-2026). Pendiente: Lighthouse móvil ≥ 90 medido, fuentes autoalojadas, revisión de textos |
| **M2 · Páginas** | `/vinos`, `/como-funciona`, `/bodegas`, `/tecnologia`, `/historia`, `/contacto`, `/aviso-legal`, `/privacidad`, `/b/[codigo]`, 404 | **Hecho** (25-09-2026): título, descripción, canonical y OG propios por ruta; barrera modal por teclado; marca corregida |
| **M3 · Calidad y despliegue** | `sitemap.xml`, `robots.txt`, OG por defecto, cabeceras de seguridad, analítica sin cookies, Vercel con previews | **Hecho en `dev`** (25-09-2026); pendiente el rendimiento móvil ≥ 90 |
| **M4 · Alineación con el sitio de bodegas** | Se ejecuta en `drinks-on-chain-front` (menú nuevo, `/acceso`, redirecciones, pie común, `/vinos` → landing) | **Hecho en `dev`** (25-09-2026); detalle en `03-roadmap-frontend.md` §Sistema 0 |
| **M5 · Dominio** | Compra, DNS, un proyecto de Vercel por subdominio, redirecciones `www` y `.com` | Pendiente de la compra |

## Orden de trabajo pendiente (una persona, ~1 semana)

Las casillas se marcan cuando el paso está en `dev`; el detalle fino vive en `drinks-on-chain-landing/docs/ROADMAP.md`.

- [x] 0. Correcciones reportadas el 25-09-2026: la barrera de edad no debe dejar desplazar la página ni entrar a mitad de página; la barrera se apoya sobre el mapa del héroe y entra al héroe con una transición (como el sitio de bodegas); la sección "Las bodegas" debe mostrar su botón hacia el sitio de bodegas. · 25-09-2026
- [x] 1. Corrección de marca (pie, contacto, aviso legal) en ambos sitios. · 25-09-2026
- [x] 2. `sitemap.ts`, `robots.ts`, `opengraph-image` por defecto y metadatos por ruta. · 25-09-2026
- [x] 3. Cabeceras de seguridad en `next.config.ts` (CSP sin nonce porque el sitio es estático, `frame-ancestors 'none'`, `Referrer-Policy`, `Permissions-Policy`, HSTS). · 25-09-2026
- [ ] 4. Fuentes autoalojadas y medición de Lighthouse móvil; corregir lo que baje de 90. Fuentes autoalojadas (`next/font`) y accesibilidad 100 hechas; rendimiento móvil 69–77 en build local, pendiente de optimizar (detalle en el `docs/ROADMAP.md` del repo).
- [ ] 5. Revisión de textos ES/EN y de accesibilidad por teclado. Teclado hecho (barrera modal, foco visible, contraste AA); falta revisar los textos existentes.
- [x] 6. Analítica mínima: **Vercel Web Analytics** (decisión del 25-09-2026; sin cookies, no necesita banner). · 25-09-2026 Falta activarla en el panel de Vercel.
- [ ] 7. PR `dev → main` — abierto el 25-09-2026, pendiente de revisión.

## Definición de terminado (todos los hitos)
Móvil y escritorio · estados vacío y error donde aplique · textos ES y EN · accesible por teclado y lector de pantalla · sin errores de consola · Conventional Commits en `dev` · PR a `main` con captura.
