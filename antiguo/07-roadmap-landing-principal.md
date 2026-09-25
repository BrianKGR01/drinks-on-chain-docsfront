# 07 · Roadmap de la landing principal (`drinks-on-chain-landing`)

Sitio del dominio raíz. Público primario: el consumidor. Sin autenticación, sin WebGL pesado, carga en menos de dos segundos en móvil. Se construye en secuencia, un hito detrás de otro, en la rama `dev` con PR a `main` al cerrar cada hito.

## Alcance fijo

- **Enlaces salientes** por variables de entorno: `NEXT_PUBLIC_URL_APP` (Marketplace), `NEXT_PUBLIC_URL_BODEGAS` (sitio de bodegas). Mientras no exista el dominio apuntan a valores de desarrollo; nunca se escriben en el código.
- **Contenido compartido** con el sitio de bodegas (bodegas, zonas, vinos, imágenes, textos editoriales): hoy se copia desde `drinks-on-chain-front/src/content`; en la Etapa 0 del roadmap global se extrae al paquete `@doc/content` y los dos sitios lo consumen.
- **Diseño**: mismos tokens, tipografías y componentes editoriales que el sitio de bodegas (papel, tinta, oro). Sin escena 3D: el mapa aparece como un dibujo SVG generado desde la misma geometría de valles y parcelas.
- **Barrera de edad** en la primera visita (recordada en el navegador).
- ES por defecto, EN disponible.

## Hitos

### M0 · Fundaciones (día 1–2) — hecho el 24-09-2026
Repositorio creado desde la plantilla (Next.js 16, TypeScript estricto, Tailwind 4, ESLint), tokens y tipografías, `Logo`, `AgeGate` con ornamentos, cabecera con navegación y botón "Entrar", pie común, diccionario ES/EN, contenido copiado, variables de entorno, CI (lint, tsc, build), `dev` y `main`, README con el flujo de trabajo.
Terminado cuando: `pnpm build` pasa, la home muestra cabecera, barrera de edad y pie en móvil y escritorio, y el repo está en GitHub con `dev` protegida por PR.

### M1 · Inicio (semana 1) — primera versión hecha el 24-09-2026; pendientes Lighthouse, fuentes autoalojadas y revisión de textos
Secciones de la página de inicio en este orden: héroe (dibujo del mapa detrás de un velo, tesis, CTA "Explorar los vinos" → `app.`, enlace "Escaneé una botella"); "Cómo funciona" (Escanea · Descubre · Adquiere · Retira, con la dinámica de preventa y retiro explicada en una frase cada una); "Vinos en la red" (tarjetas desde el contenido compartido, "Ver todo en el Marketplace"); "Las bodegas" (vista previa del mapa + tres bodegas + "Conocer las bodegas" → `bodegas.`); "Qué garantizamos" (trazabilidad, edición limitada, retiro seguro); franja B2B discreta; pie.
Terminado cuando: Lighthouse móvil ≥ 90 en rendimiento y accesibilidad, todos los CTA llevan a su destino por variable de entorno, ES/EN completos.

### M2 · Páginas (semana 2)
`/vinos` (vitrina trasladada desde el sitio de bodegas, con "Adquirir" → `app.`), `/como-funciona` (recorrido paso a paso con capturas mock del Marketplace y preguntas frecuentes: qué es un token de botella, qué pasa si no retiro, dónde retiro, cómo se protege mi cuenta), `/bodegas` (vista previa del mapa, lista breve y enlace a `bodegas.`), `/tecnologia` (trazabilidad, Stellar, billetera sin frase semilla, qué datos guardamos), `/historia`, `/contacto`, `/aviso-legal`, `/privacidad`, `/b/[codigo]` (redirección al visor de `app.`), página 404.
Terminado cuando: cada ruta tiene título, descripción y OG propios; navegación por teclado completa; sin errores de consola.

### M3 · Calidad y despliegue (semana 3, primera mitad)
`sitemap.xml` y `robots.txt`, OG por defecto (dibujo del mapa), analítica con consentimiento mínimo, cabeceras de seguridad (CSP, `frame-ancestors 'none'`), proyecto de Vercel con previews por PR, variables de entorno por entorno, redirección `www` y `.com → raíz` cuando exista el dominio.
Terminado cuando: despliegue de vista previa público para revisión del cliente.

### M4 · Alineación con el sitio de bodegas (semana 3, segunda mitad)
En `drinks-on-chain-front`: menú nuevo (Mapa · Bodegas · Puntos de recojo · Unirse · Acceso), `/acceso`, redirección `/parcelas → /valles`, `/vinos` pasa a enlazar a la landing principal, pie común. Es la fase L2.2 del documento 02 y se ejecuta después de M3 para que ambos sitios se enlacen entre sí con URLs reales.
Terminado cuando: navegar de un sitio al otro y volver funciona en móvil y escritorio.

### Después
Extracción de `@doc/ui` y `@doc/content` (Etapa 0 del roadmap global) y sustitución de las copias por los paquetes; perfiles de bodega y capa de bodegas en el mapa (L2.2 restante); formularios B2B contra el backend.

## Definición de terminado (todos los hitos)
Móvil y escritorio · estados vacío y error donde aplique · textos ES y EN · accesible por teclado y lector de pantalla · sin errores de consola · Conventional Commits en `dev` · PR a `main` con captura de pantalla.
