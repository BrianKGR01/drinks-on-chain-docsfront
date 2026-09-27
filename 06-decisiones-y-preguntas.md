# 06 · Decisiones tomadas y preguntas abiertas

Versión 2.1 · 27 de septiembre de 2026 (v1 en `antiguo/`). Registro vivo: cada decisión con fecha y motivo; las preguntas se cierran moviéndolas a decisiones.

> **Reconciliación del 27-09-2026**: las decisiones del cliente del 27 y 28-09 (A-01…A-34, en [`docs-back/04`](https://github.com/drinks-on-chain/drinks-on-chain-docsback/blob/main/04-decisiones-y-preguntas.md)) mandan sobre este documento. Las filas de §1 que contradicen esas decisiones se marcan **Sustituida** con la decisión que las reemplaza; §5 cierra las preguntas al backend y §6 recoge los supuestos vigentes. La lista completa de contradicciones (R1–R16) está en `plan/02-reconciliacion-documentos.md` (documento local de coordinación junto a `PLAN-MAESTRO.md`, en la carpeta paraguas) y en `docs-back/04` §2.3.

## 1. Decisiones

| Fecha | Decisión | Motivo |
|---|---|---|
| 2026-09-24 | El equipo construye **solo el frontend** de los cuatro sistemas y de los dos sitios públicos; el backend lo hace otro equipo. **Sustituida (26-09)**: el backend entra en el alcance; su documentación vive en `drinks-on-chain-docsback` | Alcance acordado |
| 2026-09-24 | Todo se construye primero contra **datos de prueba en JSON** (MSW) con tipos compartidos; el backend se conecta después | Tener el frontend completo antes de integrar |
| 2026-09-24 | Cadena: **Stellar** | Acceso al Stellar Community Fund; UX sin frase semilla |
| 2026-09-24 | Acento de marca **oro líquido**; el rojo solo como color de peligro y semáforo | El rojo leía como error |
| 2026-09-24 | Landing principal B2C en el dominio raíz; el mapa pasa a ser el sitio de bodegas en `bodegas.`; cada sistema en su subdominio con su propia autenticación; las landings no autentican; POS sin página pública; Backoffice sin enlaces públicos | Seguridad por origen y un mensaje claro por público |
| 2026-09-24 | Marketplace con navegación libre; cuenta y billetera al comprar o al pulsar "Entrar"; el escaneo de botella exige registro. **Sustituida en parte (A-21)**: el escaneo es público, sin cuenta; la cuenta solo hace falta para comprar y para reseñar | Menos fricción antes de la intención de compra |
| 2026-09-24 | ERP y POS solo inicio de sesión; usuarios y puntos provisionados desde el Backoffice | Control de acceso centralizado |
| 2026-09-24 | Un repositorio por sistema bajo `BrianKGR01`; paquetes compartidos `@doc/ui`, `@doc/mocks`, `@doc/wallet`. **Sustituida (25-09 y 27-09)**: repos `drinks-on-chain-<sistema>` en la organización `drinks-on-chain`; paquetes `@drinks-on-chain/ui` y `@drinks-on-chain/mocks`; sin `@doc/wallet` en el MVP (A-28) | Ciclos de vida distintos |
| 2026-09-24 | Conventional Commits; trabajo en `dev`; PR `dev → main` por hito | Historial limpio |
| 2026-09-24 | Tipografías libres (Cormorant Garamond, EB Garamond); fotografías con licencia libre y crédito | Licencias |
| **2026-09-25** | **Marca y equipo: Drinks on Chain**, sin ninguna otra razón social en documentos ni interfaces. Los textos que aún nombran otra empresa (pie, contacto, aviso legal de ambos sitios) se corrigen en el primer hito pendiente | Decisión del cliente |
| 2026-09-25 | Billetera del consumidor: **smart account con passkey** (`smart-account-kit`, contratos OpenZeppelin, relayer propio) por defecto; **cuenta gestionada por la plataforma** como respaldo automático cuando el dispositivo no soporta passkeys; billetera externa como opción avanzada; proveedores embebidos (Privy) documentados pero fuera del MVP. **Sustituida (A-04, A-05, A-28)**: el backend crea direcciones custodiales derivadas (SEP-0005), sin fondear; smart accounts en Fase 2 | `passkey-kit`/Launchtube quedaron obsoletos; coste mínimo y fricción mínima; ver 04 |
| 2026-09-25 | La creación de la passkey y la derivación de la dirección ocurren en el **frontend** (Marketplace); el despliegue, el pago de comisiones y el registro los hace el **backend** (relayer). **Sustituida (A-28)**: sin passkeys ni relayer en el MVP | WebAuthn solo existe en el dispositivo del usuario; el usuario no tiene XLM |
| 2026-09-25 | Cuentas Stellar de las **bodegas**: las crea el **backend** al dar de alta la bodega desde el Backoffice; custodia en KMS/HSM. El ERP solo muestra un panel de lectura. Ninguna clave institucional en un navegador | Estándar de custodia institucional; no bloquea el flujo productivo |
| 2026-09-25 | **POS y Backoffice no tienen billetera en el cliente**; disparan operaciones que firma el backend y muestran estados de transacción | Fricción cero para cajeros y gestores |
| 2026-09-25 | Pase de retiro **emitido por el backend** bajo la sesión del usuario (QR firmado, caducidad en días) y **quema por clawback del emisor** al confirmar la entrega; la firma con passkey queda como refuerzo opcional. **Sustituida (A-07, A-28)**: **pase de canje** con caducidad en horas, regenerable, en la base de datos; quema del NFT por el operador (`redeem_burn`) | Funciona igual con cuentas passkey y gestionadas; nadie firma en el mostrador |
| 2026-09-25 | Modelo de token propuesto a backend: **activo clásico por lote con clawback**, en poder de smart accounts vía SAC (sin trustlines), metadatos en `stellar.toml`; alternativa SEP-41 documentada. **Sustituida (A-01, A-02, A-28)**: un NFT por botella, un contrato por bodega sobre OpenZeppelin | Cero desarrollo de contratos; quema nativa; visible en exploradores |
| 2026-09-25 | Idiomas: **español** en ERP, Backoffice, POS y Marketplace; ES/EN solo en las landings. El Marketplace se construye con diccionario para poder añadir inglés después | Pedido del cliente |
| 2026-09-25 | Hardware del POS: **iPad o tablet Android**; la app es una PWA en modo kiosco que sirve a ambos | Pedido del cliente |
| 2026-09-25 | Pase de retiro válido en **los puntos habilitados para ese lote** (la bodega y sus puntos asociados, configurados en el Backoffice); caducidad en **días**. **Sustituida (A-07, A-25)**: pase de canje con caducidad **en horas** (24 h por defecto, D-15); **puntos de canje** autorizados por la bodega, por soporte o por postulación | Pedido del cliente |
| 2026-09-25 | Pasarela de pago: **la del banco del proyecto**; el checkout se construye con un adaptador y datos de prueba hasta recibir su documentación | Pedido del cliente |
| 2026-09-25 | No se planifica nada sobre bodegas socias reales; todos los datos de bodegas son de prueba o de referencia | Pedido del cliente |
| 2026-09-25 | Sistema de diseño con **dos familias** (editorial y operativa) sobre los mismos tokens; maquetas HTML por sistema en `design-system/` | Las apps deben ser herramientas claras sin perder la identidad de las landings |
| 2026-09-25 | Orden de construcción: Etapa 0 (fundaciones) → ERP → Marketplace → POS → Backoffice → integración; los pendientes de los sitios públicos se cierran en paralelo. **Sustituida (A-34)**: orden del ciclo del MVP por olas del plan maestro (back office y bodegas → ERP → tokenización → Marketplace → POS); ver 03 v3 | El backend del ERP ya existe; Marketplace + POS forman la demo para el grant; ver 03 |
| 2026-09-25 | **El OpenAPI del backend desplegado es la fuente de verdad** del contrato del ERP (por encima de `backend/endpoints.md` y de las guías, que discrepan en nombres de campos). Los fixtures del ERP siguen sus DTO exactos; el "Lote" del ERP es una vista derivada en el cliente | Evitar rehacer al integrar; ver 09 |
| 2026-09-25 | La documentación de frontend vive en el repo `drinks-on-chain-docsfront` (esta carpeta) con `index.html` publicado en GitHub Pages para el sistema de diseño; mismo flujo `dev → main` y Conventional Commits | Que backend y cliente vean las decisiones |
| 2026-09-25 | Analítica de los sitios públicos: **Vercel Web Analytics** (sin cookies, sin banner de consentimiento); se activa en el panel de cada proyecto de Vercel | Decisión del cliente; mínimo dato personal y cero fricción |
| 2026-09-25 | CSP de los sitios públicos **sin nonce** (`script-src 'self' 'unsafe-inline'`, sin orígenes externos, `frame-ancestors 'none'`) | Los sitios son estáticos y sin datos de usuario; el nonce obligaría a renderizar cada página en el servidor. Las aplicaciones con sesión (S1–S4) sí usarán nonce |
| 2026-09-25 | Los sitios públicos muestran la **red de prueba** de `08` (4 bodegas, 5 puntos) con su estado y un aviso visible; los productores reales solo aparecen como contenido editorial de las zonas, nunca como socios | Evitar que una bodega real parezca socia sin acuerdo |
| **2026-09-27** | La documentación de frontend se **alinea con las decisiones acordadas del backend** (A-01…A-34). Donde se contradicen, manda el backend; el roadmap del frontend pasa a la v3 por olas | Reconciliación de la Ola 0 (O0-DOC-1) |
| 2026-09-25 | Sitio de bodegas: fichas de parcela en `/valles/[valle]/[parcela]` (redirección 308 desde `/parcelas`), `/vinos` redirige a la landing, `/acceso` muestra "Disponible pronto" mientras ERP y POS no estén desplegados | Plan de 02 §4 |

## 2. Preguntas cerradas el 25-09-2026 (registro)

| Pregunta | Respuesta |
|---|---|
| ¿Bodegas socias con acuerdo? | No se toca todavía; datos de prueba |
| ¿Pasarela de pago? | La del banco; documentación por pedir (ver §3) |
| ¿Dominio raíz? | Se comprará; subdominios decididos |
| ¿Idiomas de las aplicaciones? | Solo español por ahora |
| ¿Hardware de los puntos de recojo? | iPad o tablet Android |
| ¿Alcance y caducidad del pase? | Puntos habilitados para el lote; caducidad en días (**sustituida el 27-09 por A-07**: horas, configurable) |
| ¿Marca? | Drinks on Chain; identidad visual actual aprobada |

## 3. Pasarela de pago: qué pedir al banco

Sí, el frontend participa: el checkout del Marketplace es la pantalla donde el cliente paga. Lo que **no** hace nunca el frontend es tocar el número de tarjeta en claro (PCI) ni decidir si un pago se aprobó: eso lo hace el banco y lo confirma al backend por webhook. Las integraciones bancarias suelen ser de uno de estos tres tipos, y hay que saber cuál ofrece el banco:

| Tipo | Qué hace el frontend | Qué hace el backend |
|---|---|---|
| **Checkout alojado (redirección)** | Redirige al cliente a la página del banco con un identificador de orden y lo recibe de vuelta en una URL de retorno | Crea la transacción en la API del banco, recibe el webhook, marca la orden pagada |
| **Widget o SDK JavaScript embebido** | Carga el script del banco en el slide-over de checkout; el formulario de tarjeta es del banco (iframe tokenizado) | Igual que arriba, más la creación del token de sesión del widget |
| **QR bancario (QR interoperable del sistema de pagos boliviano)** | Muestra la imagen del QR y el importe; espera la confirmación (sondeo o websocket) y muestra "pago recibido" | Pide la generación del QR a la API del banco, recibe la notificación de pago |

Documentación a solicitar al banco:
1. Manual de integración de la pasarela (API REST y, si existe, SDK JavaScript o checkout alojado), con ejemplos de peticiones y respuestas.
2. Credenciales y entorno de **pruebas** (sandbox), tarjetas de prueba y QR de prueba.
3. Especificación del **webhook / notificación** de resultado de pago (campos, firma, reintentos) y de las URL de retorno.
4. Generación y consulta de **QR de cobro** (importe, glosa, vencimiento, estado).
5. Requisitos de seguridad: 3-D Secure, dominios permitidos, CSP (qué orígenes debe permitir nuestro sitio para cargar su script o iframe).
6. Monedas (BOB), comisiones, tiempos de liquidación, límites, reembolsos y anulaciones por API.
7. Requisitos de marca (logos, textos obligatorios en el checkout) y de cumplimiento (aviso de datos).

Mientras tanto el checkout se construye con `PaymentProvider` en modo mock (ver 08 §7.3) y soporta las tres formas de integración sin cambiar la pantalla.

## 4. Preguntas abiertas para el cliente

1. **Dominio**: extensión definitiva (`.bo`, `.com`) y fecha de compra, para configurar DNS y un proyecto de Vercel por subdominio.
2. **Textos legales**: aviso legal, política de privacidad y textos de consumo responsable definitivos (Ley 259) para ambos sitios y para el registro del Marketplace.
3. **Analítica**: ¿herramienta preferida (Vercel Analytics, Plausible, otra) y necesidad de consentimiento explícito?
4. **Logotipo**: ¿el wordmark tipográfico actual es el definitivo o habrá un símbolo? Afecta al icono de la PWA y al favicon.
5. **Recuperación de cuenta del consumidor**: ¿aceptamos exigir un segundo factor de recuperación (correo o segundo dispositivo) antes de la primera compra?
6. ~~**Precio en pantalla**: ¿solo BOB o también referencia en USD?~~ **Cerrada**: solo bolivianos (A-06); la política de precio se aplaza (A-32).

## 5. Preguntas para el equipo de backend (cerradas el 27-09-2026)

Todas quedan respondidas por las decisiones de `docs-back/04` y por el roadmap del backend (`docs-back/08`). Se conservan con su respuesta como registro.

| # | Pregunta | Respuesta |
|---|---|---|
| 1 | **Contrato del ERP**: puntos de alineación de 09 §8 y datos de prueba en el servidor de desarrollo | Los resuelve el roadmap del backend por etapas (Etapa 0: listas, `details` por campo, organización activa, semilla determinista OPS-09; Etapa 1: recuperar contraseña; Etapa 2: lote, dictamen, URL del QR configurable, códigos por botella). Detalle en la nota de 09 §8. Base: A-33, A-26 |
| 2 | **Contrato del token**: ¿activo clásico por lote o SEP-41? ¿Emisor por bodega o único? | **Un NFT por botella** (A-01), **emisor por bodega** (A-02), un contrato por bodega sobre OpenZeppelin con quema por el operador (A-28) |
| 3 | **Relayer** | No hay *relayer* en el MVP: el usuario nunca firma (A-04, A-28). Toda firma la hace el backend con un firmante aislado y claves en custodio (A-11) |
| 4 | **Indexación** de saldos y eventos | **Indexador propio y mínimo** (A-12). La forma de avisar al cliente (sondeo o eventos) la fija el OpenAPI de las Etapas 3 y 4 |
| 5 | **Cuentas institucionales** | Cuenta de la bodega con clave en custodio, creada al activarse la bodega (A-02, A-11; C6 de `docs-back/04`); el ERP la muestra en solo lectura (1F) |
| 6 | **Autenticación** | Sesiones cortas con renovación rotativa en cookie y **roles por membresía con organización activa** (A-33); contrato en `plan/contratos/o0-sesiones-y-estandares.md`. 2FA TOTP del back office y vinculación de tabletas del POS según el catálogo del backend (PLT-05, IAM-11, IAM-12; Etapas 1 y 5) |
| 7 | **Modelo de datos**: ¿`@doc/mocks` como borrador del contrato? | No: el contrato es el **OpenAPI que publica cada etapa del backend** y `@drinks-on-chain/mocks` se regenera desde él (plan maestro, "contrato primero"; OPS-12). ADR-010 sigue como propuesta en `docs-back/04` |
| 8 | **Archivos de QR** del ERP | **Código y QR por botella** (A-26) generados por el backend al embotellar, con exportación para la imprenta (ERP-15, Etapa 2); el ERP ofrece la descarga (1J) |
| 9 | **Entornos** | Testnet en desarrollo y staging; mainnet al salir a producción (Ola 6), tras la revisión externa del contrato (Pista B, B.4). Sin fecha fija |

## 6. Supuestos vigentes mientras no haya respuesta

Actualizados el 27-09-2026 con las decisiones del backend. Las preguntas que siguen abiertas se citan con su ID de `docs-back/04` §3.

- **Token**: un **NFT por botella** (A-01) en el contrato de su bodega (A-02, A-28); la cava muestra NFT individuales con su número de botella, no saldos por lote. Al canjear, el operador quema el NFT (`redeem_burn`).
- **Emisión en preventa**: la bodega autoriza el lote y la cuota desde el ERP (A-03); el back office aprueba (supuesto de D-20, configurable); el hash del expediente se ancla al certificar el lote (A-22).
- **Billeteras custodiales del backend**: el backend crea para cada consumidor una dirección derivada (SEP-0005) sin fondear y firma todo (A-04, A-28). El cliente solo muestra la dirección, en solo lectura; no hay billetera, passkey ni firma en ninguna app del MVP.
- **Pase de canje**: caducidad **en horas**, **24 h por defecto** (supuesto de D-15), solo configurable a nivel general; se genera otro al caducar; uno activo por NFT (A-07). La **ventana de canje** es de 30 días por defecto (A-19).
- **Puntos de canje**: los habilita la bodega, soporte o una postulación (A-25); la bodega lleva la logística de botellas (A-17). En el POS, tableta vinculada al punto y **PIN personal** de cada cajero. El canje registra el **código de botella** según la configuración (A-26).
- **Acceso del consumidor solo con correo** (A-13): registro con captcha y verificación; sin SMS ni proveedores sociales. Mayoría de edad por declaración (A-18).
- **Reseñas**: una por persona y lote, para cualquiera con sesión, con marca de "verificada" si canjeó (A-21; supuesto de D-19).
- **Pago en bolivianos** (A-06); ningún usuario paga XLM. Precio y pasarela real aplazados (A-14, A-32): adaptador de prueba hasta la Ola F; el precio puede faltar en la colección.
- **Listas y errores**: `data: { items, total, limit, offset }` con `limit` ≤ 100 y `details: [{ field, message }]` (A-33).
- Español en aplicaciones; inglés solo en landings.
- Subdominios bajo un dominio único; hasta la compra, los enlaces entre sitios usan los hosts de Vercel.
- Recuperación de cuenta del consumidor por correo, sin segundo factor obligatorio, hasta que el cliente responda §4.5.
