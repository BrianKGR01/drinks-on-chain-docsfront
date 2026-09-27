# 01 · Análisis del ecosistema Drinks on Chain y estado actual

Versión 2.1 · 27 de septiembre de 2026. Sustituye a la v1 (en `antiguo/`). Lectura de los dos documentos maestros (`Drinks On Chain.pdf`, `Estructura de pantallas.pdf`), del código de los repos y de las decisiones tomadas hasta hoy. La v2.1 actualiza el estado real (§3, §4), añade el glosario (§9) y marca lo que sustituyen las decisiones del backend del 27 y 28-09 (A-01…A-34).

**Alcance**: este documento y el resto de `docs-front` cubren el frontend (cuatro aplicaciones, dos sitios públicos, paquetes compartidos). Desde el 26-09-2026 **el backend también está en el alcance del ecosistema**: su análisis, decisiones y roadmap viven en [`drinks-on-chain-docsback`](https://github.com/drinks-on-chain/drinks-on-chain-docsback) y, donde contradicen a estos documentos, mandan las decisiones acordadas allí. El calendario común de frontend, backend, contratos y operación es el **plan maestro** (`PLAN-MAESTRO.md`), documento local de coordinación en la carpeta paraguas, fuera de este repositorio; el roadmap del frontend por olas está en `03-roadmap-frontend.md` (v3).

El proyecto, la marca, el equipo de producto y el equipo de desarrollo son **Drinks on Chain**. No existe ninguna otra razón social en la documentación ni en las interfaces.

## 1. Qué es el ecosistema

Una red que registra la vida de un vino o singani boliviano desde la parcela hasta la botella (ERP), la convierte en un activo digital que el consumidor compra a precio de bodega y guarda en su "cava digital" (Marketplace), la administra un equipo central que da de alta bodegas y aprueba emisiones (Backoffice), y la cierra en el mostrador de una licorería o de la propia bodega, donde el activo se quema al entregar la botella (POS).

Fase 1 (MVP): precio fijo, sin trading, sin KYC biométrico, sin tesorería DeFi, pago en moneda local. Fase 2: preventas con fases de precio, mercado secundario P2P, KYC estricto, tesorería, corchos NFC y API para cadenas de tiendas.

Cadena: **Stellar**. Modelo acordado el 27–28-09: **un NFT por botella** en un contrato por bodega, emitido en **preventa** cuando la bodega autoriza el lote, y billeteras custodiales que crea y gestiona el backend (A-01…A-05, A-28). Detalle vigente en `docs-back/06`; `04-billeteras-stellar.md` y `11-billeteras-para-backend.md` quedan como registro sustituido.

## 2. Actores, sistemas y dispositivos

| Actor | Sistema | Dispositivo dominante | Idioma | Toca billetera en el cliente |
|---|---|---|---|---|
| Consumidor anónimo | Landing principal, sitio de bodegas, catálogo del Marketplace | Móvil | ES / EN | No |
| Miembro de la tribu (consumidor con cuenta) | S2 Marketplace + visor + cava | Móvil (mobile-first, PWA) | ES | No: el backend crea y gestiona su dirección custodial (A-04, A-28); la app la muestra en solo lectura |
| Enólogo / agrónomo | S1 ERP | Escritorio y tablet de planta | ES | No (panel de solo lectura de la cuenta de la bodega) |
| Operario de bodega | S1 ERP | Tablet táctil | ES | No |
| Administrador de bodega | S1 ERP | Escritorio | ES | No (lectura; firmante propio opcional en Fase 2) |
| Equipo gestor de Drinks on Chain | S3 Backoffice | Escritorio | ES | No (dispara operaciones que firma el backend) |
| Cajero de un punto de canje | S4 POS | iPad o tablet Android vinculada al punto, con PIN personal | ES | No |

Un mismo humano puede tener varios roles. Cada sistema vive en su propio subdominio con su propia sesión; si el backend implementa una identidad única (OIDC), no cambia nada en el cliente.

## 3. Mapa de sistemas y repositorios

Repos en la organización GitHub `drinks-on-chain` (públicos, salvo el backend), nombrados `drinks-on-chain-<sistema>`; los dos sitios públicos siguen en la cuenta `BrianKGR01`. Olas según el plan maestro.

| Subdominio | Sistema | Repositorio | Estado (27-09-2026) |
|---|---|---|---|
| raíz (`drinksonchain.*`) | Landing principal B2C | `drinks-on-chain-landing` | **En producción** (`dev` fusionado en `main`, PR #4 del 25-09). Pendientes de la Ola 0: rendimiento móvil ≥ 90, revisión de textos ES/EN, CI y pruebas de humo |
| `bodegas.` | Sitio de las bodegas B2B (mapa grabado, parcelas, red de socios) | `drinks-on-chain-front` | **En producción como sitio B2B** (PR #4 del 25-09). Pendientes de la Ola 0: variables `NEXT_PUBLIC_URL_*` en Vercel, CI; `/unirse` real en la Ola 1 |
| `erp.` | S1 ERP de trazabilidad | `drinks-on-chain-erp` | **1A–1G hechas contra mocks** y desplegado en Vercel con `NEXT_PUBLIC_MOCKS=1`; consume `ui` y `mocks` 0.1.0. Ola 0: sesiones nuevas y organización activa (1H) e integración temprana con el backend de desarrollo |
| `admin.` | S3 Backoffice | `drinks-on-chain-backoffice` | **Por crear** (Ola 1, desde la plantilla) |
| `app.` | S2 Marketplace + visor + cava | `drinks-on-chain-marketplace` | **Por crear** (Ola 2) |
| `pos.` | S4 POS del punto de canje | `drinks-on-chain-pos` | **Por crear** (Ola 4) |
| — | Sistema de diseño (`@drinks-on-chain/ui`) | `drinks-on-chain-design-system` | **0.2.0** publicada (Release `v0.2.0`, 27-09); falta que ERP y plantilla la consuman (O0-PK-1) |
| — | Datos de prueba y tipos (`@drinks-on-chain/mocks`) | `drinks-on-chain-mocks` | **0.1.0** publicada, solo dominio ERP. **0.2** en la Ola 0: listas y errores de A-33, sesión con cookie, prueba de contrato contra el OpenAPI (O0-PK-2) |
| — | Plantilla de aplicación | `drinks-on-chain-app-template` | Hecha en `dev` con CI; pendiente pasar a `main` y añadir `parseDecimal` (O0-PK-1) |
| — | Pruebas entre aplicaciones | `drinks-on-chain-e2e` | **Por crear** (Ola 1) |
| — | Backend (API y `worker`) | `drinks-on-chain-back` (privado) | Existente, solo ERP; Etapa 0 del roadmap del backend en curso |
| — | Contrato NFT por bodega (Soroban) | `drinks-on-chain-contracts` | **Por crear** (Ola 1, pista SC) |
| — | Billetera en el cliente (`@doc/wallet`) | — | **Fuera del MVP** (A-28): sin billetera en el cliente |

Despliegue: Vercel, producción desde `main`, previews desde `dev`. URLs actuales: `drinks-on-chain-landing.vercel.app`, `drinks-on-chain-bodegas.vercel.app`, `drinks-on-chain-erp.vercel.app` (mocks) y `drinks-on-chain-storybook.vercel.app`. El dominio raíz se comprará; los nombres de subdominio ya están decididos.

## 4. Estado detallado de lo construido

### 4.1 Landing principal (`drinks-on-chain-landing`)
Hecho: Next.js 16, TypeScript estricto, Tailwind 4, tokens y tipografías, barrera de edad, cabecera con "Entrar", pie, ES/EN, contenido copiado del sitio de bodegas, enlaces salientes por variables de entorno (`NEXT_PUBLIC_URL_APP`, `NEXT_PUBLIC_URL_BODEGAS`) con valores de producción por defecto, página de inicio completa (héroe con mapa SVG, "Cómo funciona", "Vinos en la red", "Las bodegas", "Qué garantizamos", franja B2B), rutas `/vinos`, `/como-funciona`, `/bodegas`, `/tecnologia`, `/historia`, `/contacto`, `/aviso-legal`, `/privacidad`, `/b/[codigo]` (redirección al visor) y 404. Desplegada en Vercel.

Hecho después (25-09, en `main`): `sitemap.xml` y `robots.txt`, imagen OG, cabeceras de seguridad, fuentes autoalojadas, analítica sin cookies y corrección de marca. Pendiente: Lighthouse móvil ≥ 90 (69–77), revisión de textos ES/EN, CI y pruebas de humo, dominio real.

### 4.2 Sitio de las bodegas (`drinks-on-chain-front`)
Hecho: experiencia WebGL del mapa grabado (React Three Fiber, shaders de grabado, cámara, parcelas, pueblo, nubes, ambiente sonoro), barrera de edad, menú a pantalla completa, navegador de parcelas, 22 zonas reales con fuentes y fotografías con licencia libre, páginas `/parcelas/[valle]/[parcela]`, `/vinos`, `/historia`, `/contacto`, `/aviso-legal`. Desplegado en Vercel.

Hecho después (25-09, en `main`): la fase L2 del plan de sitios públicos (`02-plan-landing-ecosistema.md`): menú nuevo, `/acceso`, `/unirse` (formulario de demostración), `/puntos-de-recojo`, `/bodegas` y `/bodegas/[slug]`, `/parcelas → /valles`, capa de bodegas en el mapa, pie común y corrección de marca. Pendiente: variables `NEXT_PUBLIC_URL_*` en Vercel, CI, trazo del mapa en móvil, `/unirse` real con captcha (Ola 1) y los **puntos de canje** con su postulación (Ola 5).

### 4.3 Aplicaciones y paquetes
- **ERP** (`drinks-on-chain-erp`): 1A–1G hechas el 25-09 contra mocks con las formas exactas del OpenAPI del backend (09): acceso y panel, origen, vendimia y vinificación, crianza y destilación, envasado y QR (códigos por botella provisionales), cuenta de la bodega, calidad (axe, teclado, Playwright del caso "Singani Gran Reserva 2026"). Falta: sesiones cortas y organización activa, integración con el backend real, equipo, lote como entidad, códigos por botella reales y autorizar la tokenización (03 v3, 1H–1L).
- **`@drinks-on-chain/ui`** 0.2.0: tokens, componentes, shells y Storybook, con la revisión de contraste, foco y componentes editoriales del 25-09.
- **`@drinks-on-chain/mocks`** 0.1.0: esquemas zod, fixtures deterministas y handlers MSW del ERP; los dominios `marketplace`, `backoffice` y `pos` se generarán desde el OpenAPI borrador de cada ola (R16).
- **Plantilla** (`drinks-on-chain-app-template`): Next 16, `ui` + `mocks` + MSW, cliente de API, CI con pruebas; el ERP nació de ella.

### 4.4 Documentación
La planificación de frontend vive en este repositorio (`drinks-on-chain-docsfront`). Los documentos de la primera ronda (24-09-2026) están en `antiguo/`; la segunda ronda (25-09) los sustituyó con el estado real, el sistema de diseño, el plan de datos de prueba y el roadmap por sistema. El 27-09 se reconcilió con las decisiones del backend: roadmap v3 por olas, 04 y 11 sustituidos por `docs-back/06`, 06 con las preguntas de backend cerradas.

## 5. Los cuatro sistemas: alcance y magnitud

Conteo de pantallas de `Estructura de pantallas.pdf` más las implícitas (estados vacíos, errores, ajustes). Cada sistema tiene su ficha de diseño en `design-system/` y su plan por olas en `03-roadmap-frontend.md`.

> Las tablas de esta sección reflejan el documento maestro de pantallas. Las decisiones del 27–28-09 cambian varias: la cuenta del Marketplace es **solo con correo** y sin "Splash de bóveda" (A-13, A-28); el pase es de **canje**, en horas y regenerable (A-07); el visor es **público** (A-21); el alta de bodega usa **invitaciones**, no credenciales (A-08, A-10); la emisión es una **solicitud de tokenización** que autoriza la bodega (A-03); el POS usa **PIN personal** del cajero y registra el **código de botella** (A-25, A-26). El alcance vigente de cada sub-etapa está en 03 v3.

### S1 · ERP de gestión y trazabilidad (B2B, escritorio + tablet)
Layout: barra lateral fija con wordmark serif, contenido con migas de pan y acción principal arriba a la derecha. Tema claro.

| Flujo | Pantallas nombradas | Implícitas |
|---|---|---|
| 1 Acceso y panel | Login dividido, Dashboard (widgets + tareas) | Recuperar contraseña, selector de bodega si el usuario pertenece a varias |
| 2 Origen y denominación | Directorio de terroirs, Ficha de terroir con badge D.O., slide-over "Nueva cosecha" | Alta y edición de terroir |
| 3 Vendimia y laboratorio | Pesaje (input gigante), Análisis (Brix, pH, acidez; aprobar/rechazar) | Historial de ingresos |
| 4 Vinificación y bifurcación | Mapa de tanques, Bitácora de tanque, Modal de bifurcación | Añadir registro diario |
| 5A Crianza | Barricas (madera, meses, cuenta regresiva) | Detalle de barrica |
| 5B Destilación y reposo | Cortes del alambique, Candado de reposo | Historial de destilaciones |
| 6 Envasado y sellado | Embotellado final, Éxito y exportación de QR | Descarga del archivo para imprenta |
| Transversal | — | Perfil, ajustes de bodega, notificaciones, cuenta Stellar de la bodega (solo lectura), estados vacíos |

≈ 22 pantallas. Reglas que el cliente valida además del backend: Moscatel de Alejandría y altitud > 1.600 m para D.O. Singani; candados de tiempo (meses de crianza; 6 meses de reposo para Gran Reserva) que bloquean el embotellado; conciliación kilos → litros → botellas.

### S2 · Marketplace + visor QR + cava (B2C, mobile-first, PWA)
Layout: cabecera transparente en escritorio, pestañas inferiores en móvil (Inicio, Escáner, Cava, Perfil). E-commerce de lujo con tono de club. Bottom sheets en móvil.

| Flujo | Pantallas nombradas | Implícitas |
|---|---|---|
| 1 Cuenta ligera | Pantalla única "Entrar" (crear cuenta o iniciar sesión: teléfono + correo, Google/Apple), Splash de bóveda ("Preparando tu cava") | OTP, términos, error, recuperación de acceso, añadir dispositivo de recuperación |
| 2 Catálogo y compra | Escaparate (hero + grid), Ficha de producto con CTA persistente, Checkout en slide-over | Pedido en curso, pago confirmado, pago fallido, historial de pedidos |
| 3 Cava y claim | Mi Cava, Detalle del activo, Generador de pase de retiro (ticket con QR temporal) | Historial de retiros, pase caducado, selector de punto de recojo |
| 4 Visor QR | Scan gate, Viaje del producto (scroll narrativo), Cata y brand story, Reviews | Escáner con cámara, código inválido, entrada manual del código |
| Transversal | — | Perfil, ajustes (idioma, billetera avanzada), notificaciones, ayuda que crea tickets del S3 |

≈ 22 pantallas. Navegación libre sin cuenta; la cuenta (y la billetera) se crea al comprar o al pulsar "Entrar"; el escaneo de una botella física sigue exigiendo registro (scan gate del documento maestro).

### S3 · Backoffice (interno, escritorio, denso)
Layout: barra lateral oscura, cabecera con buscador global y notificaciones, contenido claro con tablas paginadas y modales.

| Flujo | Pantallas nombradas | Implícitas |
|---|---|---|
| 1 Acceso y dashboard | Login con 2FA, Dashboard KPI + alertas del ERP | Usuarios internos y roles |
| 2 Socios | Directorio de bodegas, Perfil de bodega con "Generar credenciales ERP" | Alta de bodega (crea también su cuenta Stellar institucional, la ejecuta el backend), alta de puntos de recojo y dispositivos POS (no documentado, necesario) |
| 3 Tokenización | Pipeline de emisiones (kanban), Configurador de colección (modal de minting) | Detalle de colección, estado de la transacción en cadena, despublicar |
| 4 Operaciones y soporte | Helpdesk, Resolución de disputas con verificación de cava | Auditoría de claims, autorización manual de entrega, reportes |

≈ 17 pantallas.

### S4 · Aplicación de claim (POS, tablet, pantalla completa)
Alto contraste, tipografía gigante, botones masivos, cámara al 80 %. Tema oscuro.

| Flujo | Pantallas nombradas | Implícitas |
|---|---|---|
| 1 Acceso | PIN de sucursal | Vinculación del dispositivo (código de alta desde el Backoffice), selección de sucursal, bloqueo por inactividad |
| 2 Escaneo | Escáner activo, Semáforo verde con deslizador, Semáforo rojo con motivo | Sin cámara o permiso denegado, entrada manual, sin conexión |
| 3 Cierre | Éxito y vuelta automática, Historial y cierre de turno | Detalle de entrega |

≈ 12 pantallas. Modo kiosco; tolera cortes de red.

### Magnitud total
≈ 73 pantallas de aplicación más los dos sitios públicos. Cuatro shells de layout, dos temas, tres dispositivos objetivo. Con sistema de diseño y datos de prueba compartidos: 14–16 semanas para dos personas de frontend; el desglose está en `03-roadmap-frontend.md`.

## 6. Flujos que cruzan sistemas

1. **Alta de bodega**: por formulario público + aprobación en S3 (con reunión opcional) o alta directa desde S3 (A-08) → el backend envía una **invitación** al dueño, que la acepta y entra en S1; al activarse la bodega el backend crea su cuenta en la red. El dueño invita a su equipo (A-10). Nunca se generan ni envían contraseñas.
2. **Tokenización en preventa**: la bodega **autoriza** en S1 la tokenización de un lote y su cuota en cualquier momento del proceso (A-03) → S3 aprueba, pide cambios o rechaza (D-20) → el backend emite **un NFT por botella** en el contrato de la bodega y publica la colección → S2 la muestra con el seguimiento del lote. Al certificar el lote el hash del expediente se ancla (A-22) y los NFT pasan a canjeables.
3. **Compra → cava**: S2 cobra en bolivianos (adaptador de prueba hasta la Ola F) → aviso de "pago recibido" (A-23) → el backend transfiere los NFT a la dirección custodial del consumidor → S2 los muestra en Mi Cava, uno por botella.
4. **Canje → entrega → quema**: S2 genera un **pase de canje** (QR que caduca en horas y se regenera) → S4 lo escanea y valida contra la API → el cajero registra el **código de botella** (según configuración) y confirma con el deslizador → el backend quema el NFT (`redeem_burn`) → S2 actualiza la cava, S4 lo suma al turno.
5. **Escaneo de botella física**: el QR impreso (uno por botella) apunta a la URL configurable `app./b/{código}` → pasaporte público sin cuenta (A-21) con datos del lote de S1 → reseña si la persona tiene sesión.
6. **Soporte**: S2 crea un ticket → S3 lo atiende, consulta la cava del cliente por correo → puede autorizar una entrega manual que S4 registra.

## 7. Modelo de entidades compartido

Base de los tipos y los datos de prueba (`08-datos-de-prueba.md`). C = crea, L = lee, M = modifica.

> Nota del 25-09-2026: para el ERP, el backend real modela la trazabilidad como una cadena de entidades por etapa (`Terroir → HarvestBatch → FermentationTank → WineAgingBatch | ProductionBatch → BottlingBatch → BatchLabAnalysis`) y **no** tiene una entidad "Lote"; el "Lote" con máquina de estados de esta tabla es una vista derivada en el cliente. El detalle está en `09-contrato-erp-backend.md` §2. El resto de la tabla sigue vigente para los sistemas sin backend.

| Entidad | S1 | S2 | S3 | S4 | Notas |
|---|---|---|---|---|---|
| Bodega (`winery`) | L | L | C M | L | Perfil público reutilizado por los sitios públicos; incluye `stellarAccount` (dirección, estado) |
| Usuario y rol (`user`) | L | C | C M | L | Roles: `enologo`, `operario`, `admin_bodega`, `admin_plataforma`, `soporte`, `cajero`, `miembro` |
| Terroir / parcela (`terroir`) | C M | L | L | — | Región, altitud, cepa, geolocalización, aptitud D.O. |
| Cosecha (`harvest`) | C M | — | L | — | Fecha, rendimiento proyectado |
| Lote (`lot`) | C M | L | L M | L | Máquina de estados: `origen → vendimia → fermentacion → crianza \| destilacion → reposo → embotellado → listo → tokenizado` |
| Tanque, barrica, alambique (`vessel`) | C M | — | — | — | Recursos físicos con historial |
| Registro diario (`log`) | C | — | — | — | Temperatura, densidad, cortes |
| Embotellado y QR (`bottling`) | C | L | L | — | Botellas llenadas, archivo de códigos |
| Colección (`collection`) | — | L | C M | — | Lote + precio fijo + metadatos del token + identificador del activo en Stellar |
| Orden y pago (`order`) | — | C | L | — | Estado del pago, método (tarjeta, QR bancario) |
| NFT de botella (`token`, antes `holding`) | — | L | L | L | Un NFT por botella (A-01) con su número; fuente de verdad en la red, caché en la API |
| Billetera (`wallet`) | L | L | L | — | Dirección custodial derivada que crea el backend (A-28); el cliente solo la lee |
| Pase de canje (`claim`) | — | C | L M | L M | QR temporal, caducidad **en horas** (A-07), se regenera al caducar; uno activo por NFT |
| Punto de canje (`pickupPoint`) | L | L | C M | L | Local enlazado a una o varias bodegas y a los lotes que puede entregar; cajeros con PIN personal; tabletas vinculadas (A-25) |
| Dispositivo POS (`device`) | — | — | C M | L | Vinculado a un punto; código de alta |
| Turno y entrega (`shift`, `delivery`) | — | — | L | C | Conciliación diaria |
| Ticket (`ticket`) | — | C | C M | — | Con referencia a claim, orden o lote |
| Review (`review`) | — | C | L | — | Estrellas y comentario |
| Transacción en cadena (`tx`) | L | L | L | L | Hash, estado, enlace al explorador |
| Notificación (`event`) | L | L | L | L | `lot.ready`, `mint.confirmed`, `order.paid`, `claim.confirmed` |

## 8. Decisiones que fijan el análisis (respuestas del 25-09-2026)

- Idiomas: los sistemas operativos (S1, S3, S4) y el Marketplace en **español**; solo las landings en ES/EN. (El Marketplace puede añadir inglés en Fase 2 sin coste estructural si se construye con diccionario desde el día uno.)
- Hardware del POS: **iPad o tablet Android**; la app es una PWA que funciona en ambos.
- Pase de retiro: válido en **los puntos de recojo habilitados para ese lote** (la bodega y sus puntos asociados, configurados desde el Backoffice). Caducidad en **días**, no en minutos: el pase es una intención de retiro, no un ticket de cola. **Sustituido el 27-09 (A-07)**: pase de canje con caducidad en horas, regenerable; la intención de canje la cubre la ventana de canje en días (A-19).
- Pasarela de pago: **la del banco del proyecto**. El frontend del Marketplace la integra en el checkout (ver `06-decisiones-y-preguntas.md` §3). Hasta tener su documentación, el checkout se construye con un adaptador y datos de prueba.
- Bodegas socias: no se planifica nada al respecto todavía; los datos de bodegas son de prueba.
- Marca: **Drinks on Chain** en todos los sistemas; la identidad visual de las dos landings se mantiene y se adapta a las aplicaciones (ver `05-sistema-de-diseno.md`).

## 9. Glosario

| Término | Significado |
|---|---|
| **Pase de canje** | QR temporal que el consumidor genera en el Marketplace para canjear un NFT por su botella en un punto de canje. Caduca **en horas** (24 h por defecto, D-15 abierta), se genera otro al caducar y hay uno activo por NFT (A-07). Sustituye al antiguo "pase de retiro" (caducidad en días) |
| **Punto de canje** | Local (la propia bodega, una licorería u otro comercio) donde se entrega la botella y se quema el NFT. Lo habilita la bodega, soporte o una postulación del propio punto (A-25); tiene cajeros con PIN personal y tabletas vinculadas. Sustituye al antiguo "punto de recojo" |
| **Ventana de canje** | Plazo en días (30 por defecto) para canjear un NFT una vez canjeable (A-19); al vencer se aplica la acción configurada (A-29) |
| **Código de botella** | Código único de 8 caracteres (Crockford) impreso en cada botella y registrado en el canje según la configuración (desactivado, opcional, obligatorio) (A-26) |
| **NFT de botella** | Token no fungible que representa una botella concreta, en el contrato de su bodega (A-01, A-02) |
| **Solicitud de tokenización** | Autorización que la bodega envía desde el ERP para tokenizar un lote con una cuota; el back office la aprueba (D-20) y se emiten los NFT en preventa (A-03) |
| **Organización activa** | Organización (plataforma, bodega o punto de canje) en cuyo nombre actúa una persona con varias membresías; se cambia desde la cabecera de la app (A-33) |
