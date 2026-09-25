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
| [08-datos-de-prueba.md](08-datos-de-prueba.md) | Datos de prueba: repo `doc-mocks`, catálogo, fixtures, esquemas, MSW, escenarios, contrato con el backend |
| [09-contrato-erp-backend.md](09-contrato-erp-backend.md) | **Contrato real del ERP** (OpenAPI del backend desplegado): endpoints ↔ pantallas, DTO, enumeraciones, envoltorio, roles, vista "Lote", 12 puntos de alineación |
| [10-preparacion-etapas-0-1.md](10-preparacion-etapas-0-1.md) | Qué necesita el equipo del cliente y del backend para arrancar las Etapas 0 y 1, y qué crea el equipo solo |
| [mocks/erp/](mocks/erp/README.md) | Datos de prueba del ERP con las formas exactas del backend, generados por `generate.py` (determinista) |
| [backend/](backend/) | Documentación entregada por el equipo de backend (catálogo de endpoints y guías de prueba manual) |
| [index.html](index.html) | Portada publicada en GitHub Pages: enlaza el sistema de diseño navegable y los documentos |

## Repositorios

Desde el 25-09-2026 los repos nuevos viven en la organización GitHub [`drinks-on-chain`](https://github.com/drinks-on-chain), son públicos y se llaman `drinks-on-chain-<sistema>`. Los paquetes compartidos son `@drinks-on-chain/ui` y `@drinks-on-chain/mocks` (antes `@doc/ui` y `@doc/mocks`) y se instalan desde el tarball de su GitHub Release, sin registro ni token.

| Carpeta / repo | Sistema | Estado |
|---|---|---|
| `drinks-on-chain-landing` | Landing principal (dominio raíz) | Construida y desplegada; pendientes de calidad, SEO técnico y marca |
| `drinks-on-chain-front` | Sitio de las bodegas (`bodegas.`) | Construido y desplegado como landing original; pendiente su conversión en sitio B2B |
| `drinks-on-chain-design-system` | `@drinks-on-chain/ui`: tokens, componentes, shells, Storybook | 0.1.0 en PR (Etapa 0.1) |
| `drinks-on-chain-mocks` | `@drinks-on-chain/mocks`: esquemas zod, fixtures, MSW | 0.1.0 en PR (Etapa 0.2) |
| `drinks-on-chain-app-template` | Plantilla de aplicación (Next 16, cliente de API, pruebas, CI) | En construcción (Etapa 0.3) |
| `drinks-on-chain-erp` | S1 · ERP de trazabilidad (`erp.`) | Repo creado, vacío (Etapa 1) |
| `drinks-on-chain-marketplace` | S2 · Marketplace + visor + cava (`app.`) | Por crear (Etapa 2) |
| `drinks-on-chain-pos` | S4 · Aplicación de claim (`pos.`) | Por crear (Etapa 3) |
| `drinks-on-chain-backoffice` | S3 · Backoffice (`admin.`) | Por crear (Etapa 4) |

Despliegue en Vercel: producción desde `main`, previews desde `dev`. Convenciones: Conventional Commits, trabajo en `dev`, PR `dev → main` por hito.

Esta carpeta es el repo `BrianKGR01/drinks-on-chain-docsfront`. El workflow `.github/workflows/pages.yml` publica `index.html` y `design-system/` en GitHub Pages en cada push a `main` (hay que activar *Settings → Pages → Source: GitHub Actions* una vez).

Última actualización: 25 de septiembre de 2026.
