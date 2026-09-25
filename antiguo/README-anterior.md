# Drinks on Chain — Documentación de planificación (frontend)

Carpeta raíz del ecosistema. Aquí viven los documentos base del cliente y los documentos de análisis y planificación que produce el equipo de frontend. Cada sistema tiene su propio repositorio dentro de esta carpeta; esta documentación es transversal.

| Documento | Qué contiene |
|---|---|
| `Drinks On Chain.pdf` | Documento maestro de arquitectura y flujo de producto (fuente, del cliente) |
| `Estructura de pantallas.pdf` | Documento maestro de UI/UX y estructura de pantallas del MVP (fuente, del cliente) |
| [01-analisis-ecosistema.md](01-analisis-ecosistema.md) | Lectura analítica de ambos documentos: actores, sistemas, flujos cruzados, entidades, magnitud en pantallas y dependencias |
| [02-plan-landing-ecosistema.md](02-plan-landing-ecosistema.md) | v2 aprobada: públicos, mapa de subdominios, landing principal B2C con ruta B2B, sitio de bodegas (mapa actual) para bodegas y puntos de recojo, fronteras de seguridad, fases |
| [07-roadmap-landing-principal.md](07-roadmap-landing-principal.md) | Roadmap secuencial de la landing principal (`drinks-on-chain-landing`): hitos M0–M4, alcance por hito, criterios de terminado |
| [03-roadmap-frontend.md](03-roadmap-frontend.md) | Roadmap global por etapas y sub-etapas por sistema, solo frontend con datos mock, dependencias y criterios de terminado |
| [04-billeteras-stellar.md](04-billeteras-stellar.md) | Arquitectura de billeteras sobre Stellar: qué se resuelve en el cliente, en qué sistema, con qué librerías, y qué necesita el backend |
| [05-sistema-de-diseno.md](05-sistema-de-diseno.md) | Tokens, componentes y temas que se extraen de la landing para los cuatro sistemas |
| [06-decisiones-y-preguntas.md](06-decisiones-y-preguntas.md) | Registro de decisiones tomadas y preguntas abiertas para cliente y backend |

Repositorios (todos bajo la cuenta GitHub `BrianKGR01`):

| Carpeta / repo | Sistema | Estado |
|---|---|---|
| `drinks-on-chain-front` | Sitio de las bodegas (`bodegas.`): mapa grabado, bodegas, parcelas, puntos de recojo, acceso a ERP y POS | En desarrollo, rama `dev`, PR #1 abierto |
| `drinks-on-chain-landing` | Landing principal (dominio raíz): B2C primero, sección y ruta B2B hacia `bodegas.`, sin autenticación | M0 + inicio hechos; rama `dev`; GitHub `BrianKGR01/drinks-on-chain-landing` |
| `doc-design-system` | Tokens y componentes compartidos | Por crear (Etapa 0) |
| `doc-mocks` | Tipos, fixtures JSON y handlers MSW compartidos | Por crear (Etapa 0) |
| `doc-erp-web` | Sistema 1 · ERP de trazabilidad | Por crear |
| `doc-marketplace-app` | Sistema 2 · Marketplace + visor QR + cava | Por crear |
| `doc-backoffice-web` | Sistema 3 · Panel de control central | Por crear |
| `doc-claim-pos` | Sistema 4 · Aplicación de claim y entregas | Por crear |

Convenciones: Conventional Commits, trabajo en `dev`, PR `dev → main` por hito. Ver `06-decisiones-y-preguntas.md`.

Última actualización: 24 de septiembre de 2026.
