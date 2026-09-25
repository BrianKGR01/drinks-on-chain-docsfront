# 01 · Análisis del ecosistema Drinks on Chain y estado actual

Versión 2 · 25 de septiembre de 2026. Sustituye a la v1 (en `antiguo/`). Lectura de los dos documentos maestros (`Drinks On Chain.pdf`, `Estructura de pantallas.pdf`), del código de los dos sitios ya construidos y de las decisiones tomadas hasta hoy. Alcance del equipo: **solo frontend**. El backend, los contratos y la infraestructura los provee otro equipo; nosotros definimos qué necesita cada pantalla y trabajamos contra datos de prueba hasta conectar.

El proyecto, la marca, el equipo de producto y el equipo de desarrollo son **Drinks on Chain**. No existe ninguna otra razón social en la documentación ni en las interfaces.

## 1. Qué es el ecosistema

Una red que registra la vida de un vino o singani boliviano desde la parcela hasta la botella (ERP), la convierte en un activo digital que el consumidor compra a precio de bodega y guarda en su "cava digital" (Marketplace), la administra un equipo central que da de alta bodegas y aprueba emisiones (Backoffice), y la cierra en el mostrador de una licorería o de la propia bodega, donde el activo se quema al entregar la botella (POS).

Fase 1 (MVP): precio fijo, sin trading, sin KYC biométrico, sin tesorería DeFi, pago en moneda local. Fase 2: preventas con fases de precio, mercado secundario P2P, KYC estricto, tesorería, corchos NFC y API para cadenas de tiendas.

Cadena: **Stellar**. Detalle de billeteras y tokens en `04-billeteras-stellar.md`.

## 2. Actores, sistemas y dispositivos

| Actor | Sistema | Dispositivo dominante | Idioma | Toca billetera en el cliente |
|---|---|---|---|---|
| Consumidor anónimo | Landing principal, sitio de bodegas, catálogo del Marketplace | Móvil | ES / EN | No |
| Miembro de la tribu (consumidor con cuenta) | S2 Marketplace + visor + cava | Móvil (mobile-first, PWA) | ES / EN | **Sí**: la única billetera de usuario final del ecosistema |
| Enólogo / agrónomo | S1 ERP | Escritorio y tablet de planta | ES | No (panel de solo lectura de la cuenta de la bodega) |
| Operario de bodega | S1 ERP | Tablet táctil | ES | No |
| Administrador de bodega | S1 ERP | Escritorio | ES | No (lectura; firmante propio opcional en Fase 2) |
| Equipo gestor de Drinks on Chain | S3 Backoffice | Escritorio | ES | No (dispara operaciones que firma el backend) |
| Cajero / punto de recojo | S4 POS | iPad o tablet Android en mostrador | ES | No |

Un mismo humano puede tener varios roles. Cada sistema vive en su propio subdominio con su propia sesión; si el backend implementa una identidad única (OIDC), no cambia nada en el cliente.

## 3. Mapa de sistemas y repositorios

| Subdominio | Sistema | Repositorio | Estado (25-09-2026) |
|---|---|---|---|
| raíz (`drinksonchain.*`) | Landing principal B2C | `drinks-on-chain-landing` | **Construida y desplegada** (M0, M1 y rutas de M2). Pendientes: calidad, SEO técnico, cabeceras, dominio |
| `bodegas.` | Sitio de las bodegas (mapa grabado, parcelas, vinos) | `drinks-on-chain-front` | **Construido y desplegado** como landing original. Pendiente convertirlo en sitio B2B (menú, `/acceso`, `/unirse`, `/puntos-de-recojo`, perfiles de bodega, capa de bodegas en el mapa) |
| `app.` | S2 Marketplace + visor QR + cava | `doc-marketplace-app` | Por crear |
| `erp.` | S1 ERP de trazabilidad | `doc-erp-web` | Por crear (el backend del ERP ya existe; su documentación de endpoints está por incorporar a `08-datos-de-prueba.md`) |
| `admin.` | S3 Backoffice | `doc-backoffice-web` | Por crear |
| `pos.` | S4 Aplicación de claim | `doc-claim-pos` | Por crear |
| — | Sistema de diseño (`@doc/ui`) | `doc-design-system` | Por crear. Especificación en `05-sistema-de-diseno.md` y maquetas en `design-system/` |
| — | Datos de prueba y tipos (`@doc/mocks`) | `doc-mocks` | Por crear. Plan en `08-datos-de-prueba.md` |
| — | Billetera (`@doc/wallet`) | dentro de `doc-design-system` o repo propio | Por crear tras el spike de la Etapa 0 |

Despliegue: Vercel, producción desde `main`, previews desde `dev`. URLs actuales: `drinks-on-chain-landing.vercel.app` y `drinks-on-chain-bodegas.vercel.app`. El dominio raíz se comprará; los nombres de subdominio ya están decididos.

## 4. Estado detallado de lo construido

### 4.1 Landing principal (`drinks-on-chain-landing`)
Hecho: Next.js 16, TypeScript estricto, Tailwind 4, tokens y tipografías, barrera de edad, cabecera con "Entrar", pie, ES/EN, contenido copiado del sitio de bodegas, enlaces salientes por variables de entorno (`NEXT_PUBLIC_URL_APP`, `NEXT_PUBLIC_URL_BODEGAS`) con valores de producción por defecto, página de inicio completa (héroe con mapa SVG, "Cómo funciona", "Vinos en la red", "Las bodegas", "Qué garantizamos", franja B2B), rutas `/vinos`, `/como-funciona`, `/bodegas`, `/tecnologia`, `/historia`, `/contacto`, `/aviso-legal`, `/privacidad`, `/b/[codigo]` (redirección al visor) y 404. Desplegada en Vercel.

Pendiente: `sitemap.xml` y `robots.txt`, imagen OG por defecto, cabeceras de seguridad (CSP, `frame-ancestors 'none'`), Lighthouse móvil ≥ 90 verificado, fuentes autoalojadas, revisión de textos ES/EN, analítica con consentimiento mínimo, dominio real. **Corrección de marca**: el pie ("Hecho por…"), la página de contacto y el aviso legal todavía nombran a otra empresa; deben decir solo Drinks on Chain.

### 4.2 Sitio de las bodegas (`drinks-on-chain-front`)
Hecho: experiencia WebGL del mapa grabado (React Three Fiber, shaders de grabado, cámara, parcelas, pueblo, nubes, ambiente sonoro), barrera de edad, menú a pantalla completa, navegador de parcelas, 22 zonas reales con fuentes y fotografías con licencia libre, páginas `/parcelas/[valle]/[parcela]`, `/vinos`, `/historia`, `/contacto`, `/aviso-legal`. Desplegado en Vercel.

Pendiente (fase L2 del plan de sitios públicos, `02-plan-landing-ecosistema.md`): menú nuevo (Mapa · Bodegas · Puntos de recojo · Unirse · Acceso), `/acceso` con dos tarjetas (ERP y POS), `/unirse`, `/puntos-de-recojo`, `/bodegas` y `/bodegas/[slug]`, redirección `/parcelas → /valles`, relación parcela ↔ bodega en los datos, capa de bodegas en el mapa, pie común con la landing, `/vinos` enlazando a la landing principal. Misma corrección de marca en contacto.

### 4.3 Documentación
La planificación vive en esta carpeta. Los documentos de la primera ronda (24-09-2026) están en `antiguo/`; esta segunda ronda los sustituye con el estado real, la corrección de librerías Stellar, el sistema de diseño por sistema, el plan de datos de prueba y el roadmap por sistema.

## 5. Los cuatro sistemas: alcance y magnitud

Conteo de pantallas de `Estructura de pantallas.pdf` más las implícitas (estados vacíos, errores, ajustes). Cada sistema tiene su ficha de diseño en `design-system/` y su plan paso a paso en `03-roadmap-frontend.md`.

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

1. **Alta de bodega**: S3 crea la bodega, el backend crea su cuenta Stellar institucional y genera credenciales → S1 permite el primer login. El frontend de S3 solo muestra el resultado (dirección, estado).
2. **Lote listo → emisión**: S1 marca embotellado y exporta QR → S3 recibe la alerta, revisa trazabilidad, fija precio, aprueba → el backend emite los tokens del lote y publica la colección → S2 la muestra. El estado de la transacción (pendiente → enviada → confirmada → fallida) aparece en S3.
3. **Compra → cava**: S2 cobra en moneda local a través de la pasarela del banco → el backend transfiere N tokens a la billetera del usuario → S2 muestra el activo en Mi Cava.
4. **Claim → entrega → quema**: S2 pide un pase de retiro (QR temporal firmado por el backend) → S4 lo escanea y valida contra la API → el cajero confirma con el deslizador → el backend ejecuta la quema → S2 actualiza la cava, S4 lo suma al turno.
5. **Escaneo de botella física**: el QR impreso apunta a `app./b/{código}` → scan gate → viaje del producto con datos del lote de S1 → review.
6. **Soporte**: S2 crea un ticket → S3 lo atiende, consulta la cava del cliente por correo → puede autorizar una entrega manual que S4 registra.

## 7. Modelo de entidades compartido

Base de los tipos y los datos de prueba (`08-datos-de-prueba.md`). C = crea, L = lee, M = modifica.

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
| Activo (`holding`) | — | L | L | L | Tokens por usuario; fuente de verdad en cadena, caché en la API |
| Billetera (`wallet`) | L | C L | L | — | Dirección `C…`, tipo (`passkey`, `gestionada`, `externa`), firmantes de recuperación |
| Pase de retiro (`claim`) | — | C | L M | L M | QR temporal, caducidad en días, punto de recojo elegido |
| Punto de recojo (`pickupPoint`) | L | L | C M | L | Sucursal enlazada a una o varias bodegas y a los lotes que puede entregar; PIN; dispositivos |
| Dispositivo POS (`device`) | — | — | C M | L | Vinculado a un punto; código de alta |
| Turno y entrega (`shift`, `delivery`) | — | — | L | C | Conciliación diaria |
| Ticket (`ticket`) | — | C | C M | — | Con referencia a claim, orden o lote |
| Review (`review`) | — | C | L | — | Estrellas y comentario |
| Transacción en cadena (`tx`) | L | L | L | L | Hash, estado, enlace al explorador |
| Notificación (`event`) | L | L | L | L | `lot.ready`, `mint.confirmed`, `order.paid`, `claim.confirmed` |

## 8. Decisiones que fijan el análisis (respuestas del 25-09-2026)

- Idiomas: los sistemas operativos (S1, S3, S4) y el Marketplace en **español**; solo las landings en ES/EN. (El Marketplace puede añadir inglés en Fase 2 sin coste estructural si se construye con diccionario desde el día uno.)
- Hardware del POS: **iPad o tablet Android**; la app es una PWA que funciona en ambos.
- Pase de retiro: válido en **los puntos de recojo habilitados para ese lote** (la bodega y sus puntos asociados, configurados desde el Backoffice). Caducidad en **días**, no en minutos: el pase es una intención de retiro, no un ticket de cola.
- Pasarela de pago: **la del banco del proyecto**. El frontend del Marketplace la integra en el checkout (ver `06-decisiones-y-preguntas.md` §3). Hasta tener su documentación, el checkout se construye con un adaptador y datos de prueba.
- Bodegas socias: no se planifica nada al respecto todavía; los datos de bodegas son de prueba.
- Marca: **Drinks on Chain** en todos los sistemas; la identidad visual de las dos landings se mantiene y se adapta a las aplicaciones (ver `05-sistema-de-diseno.md`).
