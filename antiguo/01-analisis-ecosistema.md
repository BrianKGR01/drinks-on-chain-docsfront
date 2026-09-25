# 01 · Análisis del ecosistema Drinks on Chain

Lectura de los dos documentos maestros (`Drinks On Chain.pdf`, `Estructura de pantallas.pdf`) desde la perspectiva del equipo de frontend. Objetivo: entender la magnitud real, los flujos que cruzan sistemas y lo que el cliente tiene que resolver antes de que exista un backend.

## 1. Qué es el ecosistema en una frase

Una red que registra la vida de un vino o singani boliviano desde la parcela hasta la botella (ERP), la convierte en un activo digital que el consumidor compra a precio de distribuidor y guarda en una "cava digital" (Marketplace), la administra un equipo central que aprueba bodegas y emisiones (Backoffice), y la cierra físicamente en el mostrador de una licorería, donde el activo se quema al entregar la botella (POS).

El MVP (Fase 1) evita deliberadamente la especulación: precio fijo, sin trading, sin KYC biométrico, sin tesorería DeFi. La Fase 2 añade preventas dinámicas, mercado secundario P2P, KYC estricto, tesorería y corchos NFC.

## 2. Actores y sus sistemas

| Actor | Sistema que usa | Dispositivo dominante | Relación con billeteras |
|---|---|---|---|
| Enólogo / agrónomo | S1 ERP | Escritorio y tablet de planta | Ninguna en el MVP; opcionalmente firma atestaciones de lote (Fase 2) |
| Operario de bodega | S1 ERP | Tablet táctil industrial | Ninguna |
| Administrador (dueño de bodega) | S1 ERP | Escritorio | Ve la cuenta Stellar de la bodega (saldo, historial), no la opera |
| Entusiasta / miembro de la tribu | S2 Marketplace + visor | Móvil (mobile-first) | Tiene una billetera creada sin fricción; recibe tokens, genera pases de retiro; en Fase 2 firma ventas P2P |
| Equipo gestor (Debro Solutions) | S3 Backoffice | Escritorio | Aprueba emisiones (el backend firma con la clave emisora); ve estados de transacción; en Fase 2 opera tesorería con co-firma |
| Cajero / punto de recojo | S4 POS | Tablet en mostrador | Ninguna; la quema la ejecuta el backend tras la confirmación |

Un mismo humano puede tener varios roles (el dueño de una bodega también es miembro de la tribu). Los sistemas se autentican por separado; conviene un proveedor de identidad único detrás (decisión de backend), pero cada frontend tiene su propio login con la estética de su tema.

## 3. Los cuatro sistemas y su magnitud

Conteo de pantallas según `Estructura de pantallas.pdf`, más las que ese documento implica sin nombrar (estados vacíos, errores, ajustes de perfil).

### Sistema 1 · ERP de gestión y trazabilidad (B2B, escritorio + tablet)
Layout: sidebar fija con logo serif, contenido con migas de pan y acción principal arriba a la derecha. Tema claro "Oro Líquido".

| Flujo | Pantallas nombradas | Implícitas |
|---|---|---|
| 1 Autenticación y panel | Login split-screen, Dashboard | Recuperar contraseña, selector de bodega si el usuario pertenece a varias |
| 2 Origen y denominación | Directorio de terroirs, Ficha de terroir (badge D.O. Singani), Slide-over nueva cosecha | Alta/edición de terroir |
| 3 Vendimia y laboratorio | Pesaje (input gigante), Análisis (Brix, pH, acidez; aprobar/rechazar) | Historial de ingresos |
| 4 Vinificación y bifurcación | Mapa de tanques, Bitácora de tanque, Modal de bifurcación | Añadir registro diario |
| 5A Crianza | Barricas (madera, meses, cuenta regresiva) | Detalle de barrica |
| 5B Destilación y reposo | Cortes del alambique, Candado de reposo | Historial de destilaciones |
| 6 Envasado y sellado | Embotellado final, Éxito y exportación de QR | Descarga del archivo para imprenta |
| Transversal | — | Perfil de usuario, ajustes de bodega, notificaciones, estados vacíos |

Total estimado: 14 nombradas + ~8 implícitas ≈ 22 pantallas. Reglas de negocio que el frontend debe validar en cliente (además del backend): cepa Moscatel de Alejandría y altitud > 1.600 m para D.O. Singani; candados de tiempo (meses de crianza; 6 meses de reposo para Gran Reserva) que bloquean el embotellado; conciliación kilos → litros → botellas.

### Sistema 2 · Ecosistema integrado: Marketplace + visor QR + cava (B2C, mobile-first)
Layout: header transparente en escritorio, bottom tabs en móvil (Inicio, Escáner, Cava, Perfil). Mezcla de e-commerce de lujo y club privado. Bottom sheets en móvil.

| Flujo | Pantallas nombradas | Implícitas |
|---|---|---|
| 1 Onboarding y registro ligero | Landing/registro (teléfono + correo, Google/Apple), Splash de bóveda (GSAP) | Verificación OTP, términos, error de registro |
| 2 Catálogo y compra | Escaparate (hero + grid), Ficha de producto con CTA persistente y checkout en slide-over | Confirmación de pago, fallo de pago, pedido en curso |
| 3 Cava y claim | Mi Cava, Generador de pase de retiro (ticket con QR temporal) | Detalle del activo, historial de retiros, pase caducado |
| 4 Visor QR | Scan gate, Viaje del producto (scroll narrativo), Cata y brand story, Reviews | Escáner con cámara, código inválido, ya escaneado |
| Transversal | — | Perfil, ajustes, notificaciones, ayuda/soporte (crea tickets del S3) |

Total estimado: 11 nombradas + ~10 implícitas ≈ 21 pantallas. Es el sistema con más superficie visible y el único con billetera de usuario final. PWA instalable.

### Sistema 3 · Panel de control central (Backoffice, escritorio, denso)
Layout: sidebar oscura "Cava Reserva", header con buscador global y notificaciones, contenido claro con tablas paginadas y modales.

| Flujo | Pantallas nombradas | Implícitas |
|---|---|---|
| 1 Acceso y dashboard | Login con 2FA, Dashboard KPI + alertas del ERP | Gestión de usuarios internos y roles |
| 2 Gestión de socios | Directorio de bodegas, Perfil de bodega con "Generar credenciales ERP" | Alta de bodega (formulario), alta de puntos de recojo (necesario para S4) |
| 3 Tokenización | Pipeline de emisiones (kanban), Configurador de colección (modal de minting) | Detalle de colección publicada, estado de transacción en cadena, despublicar |
| 4 Operaciones y soporte | Helpdesk (tabla de tickets), Resolución de disputas (split con verificación de cava) | Auditoría de claims, reportes |

Total estimado: 8 nombradas + ~8 implícitas ≈ 16 pantallas. Nota: el documento no incluye el alta de puntos de recojo (licorerías) aunque el S4 lo requiere ("autorizados desde el Backoffice"); hay que añadirlo.

### Sistema 4 · Aplicación de claim y entregas (POS, tablet, pantalla completa)
Alto contraste, tipografía gigante, botones masivos, la cámara ocupa el 80 %.

| Flujo | Pantallas nombradas | Implícitas |
|---|---|---|
| 1 Acceso operativo | PIN de sucursal | Selección de sucursal si el dispositivo sirve a varias, bloqueo por inactividad |
| 2 Escaneo y validación | Escáner activo, Semáforo verde (swipe para confirmar), Semáforo rojo | Sin cámara / permiso denegado, sin conexión (cola) |
| 3 Confirmación y conciliación | Éxito y quema (invisible), Historial y cierre de turno | Reimpresión/recibo, detalle de entrega |

Total estimado: 6 nombradas + ~5 implícitas ≈ 11 pantallas. Debe funcionar en modo kiosco y tolerar cortes de red.

### Magnitud total
≈ 70 pantallas de aplicación + la landing (hoy 6 rutas, mañana ~12). Cuatro layouts distintos, dos temas, tres dispositivos objetivo. Con un sistema de diseño compartido y datos mock, es un trabajo de 14 a 16 semanas para dos personas de frontend a tiempo completo (ver `03-roadmap-frontend.md`).

## 4. Flujos que cruzan sistemas (los que definen las dependencias)

1. **Alta de bodega**: S3 crea la bodega y genera credenciales → S1 permite el primer login. Dependencia: S1 necesita un contrato de "usuario invitado por bodega".
2. **Lote listo → emisión**: S1 marca embotellado y exporta QR → S3 recibe alerta, revisa trazabilidad, fija precio, mintea y publica → S2 muestra la colección. Dependencia: el modelo de Lote es el mismo en los tres frontends; el evento `lot.ready` y el estado de la transacción de minteo tienen que existir en los mocks desde el día uno.
3. **Compra → cava**: S2 cobra (pasarela fiat) → backend transfiere tokens a la billetera del usuario → S2 muestra el activo en Mi Cava. Dependencia: billetera creada en el onboarding (ver `04-billeteras-stellar.md`).
4. **Claim → entrega → quema**: S2 genera pase QR temporal (firmado) → S4 escanea y valida → cajero confirma con swipe → backend quema el token → S2 actualiza la cava, S4 lo suma al turno. Dependencia: formato del pase (payload firmado, caducidad) acordado entre S2, S4 y backend.
5. **Escaneo de botella física**: QR de S1 impreso en etiqueta → S2 scan gate exige registro → viaje del producto con datos del lote (S1) → review. Dependencia: la URL del QR físico apunta al dominio del Marketplace; el visor lee un "lote público".
6. **Soporte**: S2 crea ticket → S3 lo atiende y consulta la cava del cliente → S4 puede entregar con autorización manual. Dependencia: S3 necesita un endpoint de "activos por correo".

## 5. Modelo de entidades compartido (para los mocks y los tipos)

Entidades de negocio y el sistema que las crea (C), lee (L) o modifica (M):

| Entidad | S1 | S2 | S3 | S4 | Notas |
|---|---|---|---|---|---|
| Bodega (winery) | L | L | C M | L | Perfil público reutilizado por la landing |
| Usuario y rol | L | C | C M | L | Roles: enologo, operario, admin_bodega, admin_debro, soporte, cajero, miembro |
| Terroir / parcela | C M | L | L | — | Región, altitud, cepa, año, geolocalización, badge D.O. |
| Cosecha (harvest) | C M | — | L | — | Fecha, rendimiento proyectado |
| Lote (lot) | C M | L | L M | L | Máquina de estados: origen → vendimia → fermentación → crianza \| destilación → reposo → embotellado → listo → tokenizado |
| Tanque, barrica, alambique | C M | — | — | — | Recursos físicos con historial |
| Registro diario (log) | C | — | — | — | Temperatura, densidad, cortes |
| Embotellado y QR | C | L | L | — | Cantidad de botellas, archivo de códigos |
| Colección (collection) | — | L | C M | — | Lote + precio fijo + metadatos del token + dirección del contrato |
| Orden y pago (order) | — | C | L | — | Estado del pago, método (tarjeta, QR bancario) |
| Activo (holding) | — | L | L | L | Tokens por usuario; fuente de verdad en cadena, cache en API |
| Pase de retiro (claim) | — | C | L M | L M | QR temporal firmado, caducidad, sucursal opcional |
| Punto de recojo (branch) | — | L | C M | L | Sucursal, PIN, cajeros |
| Turno y entrega (shift, delivery) | — | — | L | C | Conciliación diaria |
| Ticket de soporte | — | C | C M | — | Con referencia a claim o lote |
| Review | — | C | L | — | Estrellas y comentario |
| Notificación / evento | L | L | L | L | `lot.ready`, `mint.confirmed`, `claim.confirmed`, `order.paid` |

Este modelo es el índice de `doc-mocks` (tipos TypeScript + fixtures JSON + handlers MSW). Cuando llegue el backend, los tipos se reconcilian con su OpenAPI; el frontend no cambia de forma, cambia de origen de datos.

## 6. Lo que los documentos no dicen y hay que decidir

- Cómo se identifica una **botella individual** frente a un **lote**: el QR impreso es "por lote" según S1, pero el claim entrega "N botellas" y la Fase 2 habla de corchos NFC por botella. Propuesta MVP: token fungible por lote (1 token = 1 botella), QR de etiqueta por lote; identidad por botella solo en Fase 2.
- **Pasarela de pago** en Bolivia (tarjeta y QR bancario). Impacta el checkout de S2.
- **Alta de puntos de recojo** en S3 (no documentado).
- **Idiomas**: la landing es ES/EN; los sistemas operativos pueden ser solo ES en el MVP.
- **Multi-bodega por usuario** en S1.
- **Caducidad y alcance del pase de retiro** (¿cualquier sucursal o una elegida?).
- **Cadena**: el documento no fija cadena; la decisión del equipo es Stellar (ver `04-billeteras-stellar.md`), lo que cambia el modelo de token (SEP-41 / Soroban) y las billeteras (passkeys).
