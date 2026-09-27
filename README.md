# Drinks on Chain — Documentación de planificación (frontend)

Versión 3 · 27 de septiembre de 2026.

Repositorio `drinks-on-chain-docsfront` (carpeta `docs-front` del ecosistema). Aquí viven los documentos base del producto y los documentos de análisis y planificación del **frontend** de **Drinks on Chain**. Cada sistema tiene su propio repositorio; esta documentación es transversal. Los documentos sustituidos están en [`antiguo/`](antiguo/README.md) y ya no se editan.

**Cómo encaja con el resto**:
- El **backend** está en el alcance del ecosistema desde el 26-09-2026. Su documentación (estado, visión funcional, decisiones A-01…A-34, catálogo, tokens y cadena, procesos, roadmap) está en [`drinks-on-chain-docsback`](https://github.com/drinks-on-chain/drinks-on-chain-docsback). **Donde se contradice con estos documentos, mandan las decisiones acordadas del backend**; la reconciliación del 27-09 ya está aplicada aquí.
- El calendario común de frontend, backend, contratos y operación es el **plan maestro** (`PLAN-MAESTRO.md`): documento local de coordinación que vive en la carpeta paraguas del ecosistema, fuera de este repositorio, junto a `plan/` (estado inicial, reconciliación R1–R16, flujo de trabajo, calidad, contratos por ola). Ordena el trabajo en **olas** (0 a 6 y F) con hitos H0–H6.

| Documento | Qué contiene |
|---|---|
| `Drinks On Chain.pdf` | Documento maestro de arquitectura y flujo de producto (fuente) |
| `Estructura de pantallas.pdf` | Documento maestro de UI/UX y estructura de pantallas del MVP (fuente) |
| [01-analisis-ecosistema.md](01-analisis-ecosistema.md) | Qué es el ecosistema, actores, sistemas, **estado real de cada repo (27-09)**, magnitud, flujos cruzados, modelo de entidades, **glosario** |
| [02-plan-landing-ecosistema.md](02-plan-landing-ecosistema.md) | Los dos sitios públicos: públicos, mapa de subdominios, estado de la landing principal y del sitio de bodegas, fronteras de seguridad |
| [03-roadmap-frontend.md](03-roadmap-frontend.md) | **Roadmap del frontend v3 por olas** del plan maestro: tareas de frontend por ola con sus IDs, sub-etapas de ERP, Marketplace, POS y Backoffice, casillas de avance |
| [04-billeteras-stellar.md](04-billeteras-stellar.md) | **Sustituido** por `docs-back/06` (A-01…A-05, A-28): propuesta del 25-09 de billeteras con passkey y token por lote, conservada como registro |
| [05-sistema-de-diseno.md](05-sistema-de-diseno.md) | Sistema de diseño: dos familias (editorial y operativa), tokens, tipografía, componentes, layouts, adaptación por sistema |
| [design-system/](design-system/) | `tokens.css` y cinco maquetas HTML navegables: fundamentos, ERP, Marketplace, Backoffice, POS |
| [06-decisiones-y-preguntas.md](06-decisiones-y-preguntas.md) | Decisiones con fecha (las sustituidas, marcadas), **qué pedir al banco para la pasarela**, preguntas abiertas para el cliente, preguntas al backend cerradas con su A-xx, supuestos vigentes |
| [07-roadmap-landing-principal.md](07-roadmap-landing-principal.md) | Hitos de la landing principal con su estado y el orden de lo pendiente |
| [08-datos-de-prueba.md](08-datos-de-prueba.md) | Datos de prueba: repo `doc-mocks`, catálogo, fixtures, esquemas, MSW, escenarios, contrato con el backend |
| [09-contrato-erp-backend.md](09-contrato-erp-backend.md) | **Contrato real del ERP** (OpenAPI del backend desplegado): endpoints ↔ pantallas, DTO, enumeraciones, envoltorio, roles, vista "Lote", 20 puntos de alineación y en qué etapa del backend se resuelve cada uno |
| [10-preparacion-etapas-0-1.md](10-preparacion-etapas-0-1.md) | Qué necesita el equipo del cliente y del backend para arrancar las Etapas 0 y 1, y qué crea el equipo solo |
| [11-billeteras-para-backend.md](11-billeteras-para-backend.md) | **Sustituido** por `docs-back/06` (A-01…A-05, A-28): lo que el frontend pidió al backend para billeteras y tokens el 25-09, conservado como registro |
| [mocks/erp/](mocks/erp/README.md) | Datos de prueba del ERP con las formas exactas del backend, generados por `generate.py` (determinista) |
| [backend/](backend/) | Documentación entregada por el equipo de backend (catálogo de endpoints y guías de prueba manual) |
| [antiguo/](antiguo/README.md) | Versiones sustituidas (primera ronda del 24-09 y roadmap v2) |
| [index.html](index.html) | Portada publicada en GitHub Pages: enlaza el sistema de diseño navegable y los documentos |

## Repositorios

Los repos viven en la organización GitHub [`drinks-on-chain`](https://github.com/drinks-on-chain), son públicos (salvo el backend) y se llaman `drinks-on-chain-<sistema>`. Los paquetes compartidos son `@drinks-on-chain/ui` y `@drinks-on-chain/mocks` y se instalan desde el tarball de su GitHub Release, sin registro ni token. Estado al 27-09-2026:

| Carpeta / repo | Sistema | Estado |
|---|---|---|
| `drinks-on-chain-landing` | Landing principal (dominio raíz) | En producción; pendientes de la Ola 0: rendimiento móvil, textos ES/EN, CI |
| `drinks-on-chain-front` | Sitio de las bodegas (`bodegas.`) | En producción como sitio B2B; pendientes de la Ola 0: variables en Vercel, CI |
| `drinks-on-chain-design-system` | `@drinks-on-chain/ui`: tokens, componentes, shells, Storybook | **0.2.0** publicada (27-09) |
| `drinks-on-chain-mocks` | `@drinks-on-chain/mocks`: esquemas zod, fixtures, MSW | **0.1.0** (solo ERP); 0.2 en la Ola 0 |
| `drinks-on-chain-app-template` | Plantilla de aplicación (Next 16, cliente de API, pruebas, CI) | Hecha en `dev`; pasa a `main` en la Ola 0 |
| `drinks-on-chain-erp` | S1 · ERP de trazabilidad (`erp.`) | **1A–1G hechas contra mocks**, desplegado con mocks; sesiones nuevas e integración en la Ola 0 |
| `drinks-on-chain-backoffice` | S3 · Backoffice (`admin.`) | Por crear (Ola 1) |
| `drinks-on-chain-marketplace` | S2 · Marketplace + visor + cava (`app.`) | Por crear (Ola 2) |
| `drinks-on-chain-pos` | S4 · POS del punto de canje (`pos.`) | Por crear (Ola 4) |
| `drinks-on-chain-e2e` | Pruebas entre aplicaciones | Por crear (Ola 1) |
| `drinks-on-chain-contracts` | Contrato NFT por bodega (Soroban) | Por crear (Ola 1) |
| `drinks-on-chain-back` | Backend (API y `worker`) | Existente (solo ERP); Etapa 0 en curso |
| `drinks-on-chain-docsback` | Documentación del backend | BORRADOR v0.4 |

Despliegue en Vercel: producción desde `main`, previews desde `dev`. Convenciones: Conventional Commits, trabajo en `dev`, PR `dev → main` por hito.

Esta carpeta es el repo `BrianKGR01/drinks-on-chain-docsfront`. El workflow `.github/workflows/pages.yml` publica `index.html` y `design-system/` en GitHub Pages en cada push a `main` (hay que activar *Settings → Pages → Source: GitHub Actions* una vez).

## Glosario breve

- **Pase de canje**: QR temporal que el consumidor genera para canjear un NFT por su botella; caduca en horas (24 h por defecto) y se regenera. Antes "pase de retiro".
- **Punto de canje**: local donde se entrega la botella y se quema el NFT, con cajeros con PIN personal y tabletas vinculadas. Antes "punto de recojo".

Glosario completo en [01 §9](01-analisis-ecosistema.md#9-glosario).

Última actualización: 27 de septiembre de 2026.
