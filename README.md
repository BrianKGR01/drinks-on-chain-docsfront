# Drinks on Chain — Documentación de planificación (frontend)

Carpeta raíz del ecosistema. Aquí viven los documentos base del producto y los documentos de análisis y planificación del equipo de frontend de **Drinks on Chain**. Cada sistema tiene su propio repositorio dentro de esta carpeta; esta documentación es transversal. Los documentos de la primera ronda de planificación (24 de septiembre de 2026) están en `antiguo/` y ya no se editan.

| Documento | Qué contiene |
|---|---|
| `Drinks On Chain.pdf` | Documento maestro de arquitectura y flujo de producto (fuente) |
| `Estructura de pantallas.pdf` | Documento maestro de UI/UX y estructura de pantallas del MVP (fuente) |
| [01-analisis-ecosistema.md](01-analisis-ecosistema.md) | Qué es el ecosistema, actores, sistemas, **estado actual (hecho / pendiente)**, magnitud, flujos cruzados, modelo de entidades, decisiones del 25-09 |
| [02-plan-landing-ecosistema.md](02-plan-landing-ecosistema.md) | Los dos sitios públicos: públicos, mapa de subdominios, estado de la landing principal y del sitio de bodegas, fronteras de seguridad |
| [03-roadmap-frontend.md](03-roadmap-frontend.md) | **Roadmap paso a paso por sistema**: Etapa 0, sitios públicos, ERP, Marketplace, POS, Backoffice, integración; calendario con dos personas |
| [04-billeteras-stellar.md](04-billeteras-stellar.md) | Billeteras sobre Stellar: qué hace el cliente y qué el backend, en qué sistema, opciones por fricción y coste, modelo de token, librerías vigentes |
| [05-sistema-de-diseno.md](05-sistema-de-diseno.md) | Sistema de diseño: dos familias (editorial y operativa), tokens, tipografía, componentes, layouts, adaptación por sistema |
| [design-system/](design-system/) | `tokens.css` y cinco maquetas HTML navegables: fundamentos, ERP, Marketplace, Backoffice, POS |
| [06-decisiones-y-preguntas.md](06-decisiones-y-preguntas.md) | Decisiones con fecha, preguntas cerradas, **qué pedir al banco para la pasarela**, preguntas abiertas para cliente y backend |
| [07-roadmap-landing-principal.md](07-roadmap-landing-principal.md) | Hitos de la landing principal con su estado y el orden de lo pendiente |
| [08-datos-de-prueba.md](08-datos-de-prueba.md) | Datos de prueba: repo `doc-mocks`, catálogo, fixtures, esquemas, MSW, escenarios, contrato con el backend (ERP pendiente de documentación) |

## Repositorios (cuenta GitHub `BrianKGR01`)

| Carpeta / repo | Sistema | Estado |
|---|---|---|
| `drinks-on-chain-landing` | Landing principal (dominio raíz) | Construida y desplegada; pendientes de calidad, SEO técnico y marca |
| `drinks-on-chain-front` | Sitio de las bodegas (`bodegas.`) | Construido y desplegado como landing original; pendiente su conversión en sitio B2B |
| `doc-design-system` | `@doc/ui`: tokens y componentes | Por crear (Etapa 0) |
| `doc-mocks` | `@doc/mocks`: esquemas, fixtures, MSW | Por crear (Etapa 0) |
| `doc-erp-web` | S1 · ERP de trazabilidad (`erp.`) | Por crear (Etapa 1) |
| `doc-marketplace-app` | S2 · Marketplace + visor + cava (`app.`) | Por crear (Etapa 2) |
| `doc-claim-pos` | S4 · Aplicación de claim (`pos.`) | Por crear (Etapa 3) |
| `doc-backoffice-web` | S3 · Backoffice (`admin.`) | Por crear (Etapa 4) |

Despliegue en Vercel: producción desde `main`, previews desde `dev`. Convenciones: Conventional Commits, trabajo en `dev`, PR `dev → main` por hito.

Última actualización: 25 de septiembre de 2026.
