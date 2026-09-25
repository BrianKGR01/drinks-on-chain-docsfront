# 10 · Preparación de las Etapas 0 y 1: qué necesito del cliente y qué crea el equipo

Versión 1 · 25 de septiembre de 2026. Lista de verificación para arrancar la Etapa 0 (fundaciones) y la Etapa 1 (ERP) del `03-roadmap-frontend.md`. Cada punto dice quién lo hace. Lo marcado como **cliente** requiere una cuenta, un permiso o una decisión que el equipo de frontend no puede tomar solo.

## 1. Cuentas y accesos

| # | Qué | Quién | Para qué | Estado |
|---|---|---|---|---|
| 1.1 | **Repositorios en GitHub** `doc-design-system`, `doc-mocks`, `doc-erp-web` bajo `BrianKGR01` (o una organización `drinks-on-chain` si se prefiere; ver §4) | Cliente crea (vacíos, privados o públicos) o autoriza a que los cree el equipo con `gh repo create` desde su sesión | Etapa 0.1, 0.2 y 1 | Pendiente |
| 1.2 | **GitHub Packages**: un token personal (PAT clásico) con `read:packages` para instalar `@doc/ui` y `@doc/mocks` en las apps, y `write:packages` para publicar desde local si hace falta. En CI basta el `GITHUB_TOKEN` del propio repositorio | Cliente genera el PAT y lo guarda como secreto `NPM_TOKEN` en cada repo consumidor | Publicar y consumir los paquetes | Pendiente |
| 1.3 | **Vercel**: proyectos `drinks-on-chain-erp` (y luego los demás) en el equipo `briankgr01s-projects`, conectados a los repos, con producción desde `main` y previews desde `dev` | Cliente (importa el repo en Vercel) o el equipo con el MCP de Vercel ya conectado | Previews de cada PR | Pendiente |
| 1.4 | **Storybook publicado**: se despliega como un proyecto estático más en Vercel (`doc-design-system`), sin cuenta extra. Chromatic (pruebas visuales) es opcional y de pago; no se necesita ahora | Equipo | Revisar componentes | — |
| 1.5 | **GitHub Pages** del repo `drinks-on-chain-docsfront`: activar *Settings → Pages → Source: GitHub Actions* | Cliente (un clic) | Publicar `index.html` y el sistema de diseño con el workflow ya incluido | Pendiente |

## 2. Backend del ERP

| # | Qué | Quién | Estado |
|---|---|---|---|
| 2.1 | **Usuarios de prueba en el servidor de desarrollo** (`https://136.243.223.39.sslip.io`): una bodega `ACTIVE` con dueño, enólogo y agrónomo, y al menos un recorrido completo (terroir → vendimia → tanque → crianza o destilación → embotellado) para poder probar el ERP real en la Etapa 1G. Alternativa: credenciales de `PLATFORM_ADMIN` para ejecutar las guías de `backend/` nosotros mismos | Backend | Pendiente (hoy el servidor está vacío) |
| 2.2 | Respuestas a los 12 puntos de alineación de `09-contrato-erp-backend.md` §8 (envoltorio de listas, URL del QR, bifurcación, estado del tanque, análisis en el pesaje) | Backend | Pendiente |
| 2.3 | CORS para los dominios de Vercel (`*.vercel.app`) y, cuando exista, el dominio real. Hoy solo se ha verificado `localhost` | Backend | Pendiente |
| 2.4 | Confirmar si el `refreshToken` puede entregarse en cookie `HttpOnly` o solo en el cuerpo (afecta a cómo guarda la sesión el ERP) | Backend | Pendiente |

## 3. Stellar (Etapa 0.4, spike)

| # | Qué | Quién | Estado |
|---|---|---|---|
| 3.1 | Testnet: no hace falta ninguna cuenta; Friendbot fondea cuentas de prueba | Equipo | — |
| 3.2 | `smart-account-kit` en testnet usa por defecto un desplegador compartido y un relayer de prueba, suficientes para el spike. Para producción hará falta un **relayer propio** (OpenZeppelin Relayer, autoalojado) que opera backend | Equipo (spike); backend (producción) | — |
| 3.3 | **Mercury** (indexador de billeteras que usa el kit): acceso público de lectura para el spike; si hace falta clave, el cliente crea la cuenta en mercurydata.app | Cliente, solo si el spike lo pide | Por ver |
| 3.4 | Un teléfono Android y un iPhone reales para probar la creación de passkeys | Cliente presta o el equipo usa los suyos | Por confirmar |

## 4. Decisiones que solo el cliente puede tomar

| # | Decisión | Recomendación |
|---|---|---|
| 4.1 | **Dónde viven los repos**: cuenta personal `BrianKGR01` (como hoy) o una organización `drinks-on-chain` en GitHub | Crear la organización ahora, antes de que haya seis repos más: permisos por equipo, Packages con ámbito `@drinks-on-chain`, y los colegas de backend pueden verse invitados. Transferir los tres repos actuales es un clic |
| 4.2 | **Ámbito de los paquetes**: `@doc/ui` y `@doc/mocks` (como en los documentos) o `@drinks-on-chain/ui` | Depende de 4.1: con organización, `@drinks-on-chain/*` |
| 4.3 | **Visibilidad**: repos privados o públicos | Privados ahora; abrir `doc-design-system` y `doc-wallet` cuando se presente la candidatura al Stellar Community Fund (pide código abierto) |
| 4.4 | **Tipografías**: seguir cargando Google Fonts en las apps operativas (ERP, Backoffice, POS) o autoalojarlas en `@doc/ui` | Autoalojar en el paquete (sin dependencia de red en el mostrador ni en la planta) |
| 4.5 | **Analítica en las apps**: ninguna en el MVP salvo errores (Sentry) | Sentry gratuito; el cliente crea la cuenta cuando llegue la Etapa 5 |

## 5. Lo que el equipo crea sin pedir nada

- Repo `doc-design-system`: tokens desde `design-system/tokens.css`, componentes base, shells, Storybook, CI, publicación.
- Repo `doc-mocks`: esquemas zod, catálogo, seed en TypeScript (porta `mocks/erp/generate.py`), handlers MSW, escenarios, página `/__mocks`.
- Plantilla de aplicación y repo `doc-erp-web` desde ella, con MSW y los fixtures de `mocks/erp/` desde el primer día.
- Spike Stellar en testnet e informe.
- `@doc/wallet` 0.1 con modos `mock` y `passkey`.

## 6. Orden sugerido para la próxima sesión

1. Cliente: 1.1 (repos o permiso), 1.2 (PAT de Packages), 1.5 (activar Pages), 4.1–4.3 (organización y ámbito).
2. Backend: 2.1 (datos de prueba en el servidor) y 2.2 (respuestas de alineación).
3. Equipo: Etapa 0.1 y 0.2 en paralelo (design system y mocks), luego 0.3 (plantilla) y 0.4 (spike). La Etapa 1 empieza en cuanto exista la plantilla, sin esperar a que el backend cargue datos: el ERP nace contra `mocks/erp/`.
