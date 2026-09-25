# 06 · Decisiones tomadas y preguntas abiertas

Versión 2 · 25 de septiembre de 2026 (v1 en `antiguo/`). Registro vivo: cada decisión con fecha y motivo; las preguntas se cierran moviéndolas a decisiones.

## 1. Decisiones

| Fecha | Decisión | Motivo |
|---|---|---|
| 2026-09-24 | El equipo construye **solo el frontend** de los cuatro sistemas y de los dos sitios públicos; el backend lo hace otro equipo | Alcance acordado |
| 2026-09-24 | Todo se construye primero contra **datos de prueba en JSON** (MSW) con tipos compartidos; el backend se conecta después | Tener el frontend completo antes de integrar |
| 2026-09-24 | Cadena: **Stellar** | Acceso al Stellar Community Fund; UX sin frase semilla |
| 2026-09-24 | Acento de marca **oro líquido**; el rojo solo como color de peligro y semáforo | El rojo leía como error |
| 2026-09-24 | Landing principal B2C en el dominio raíz; el mapa pasa a ser el sitio de bodegas en `bodegas.`; cada sistema en su subdominio con su propia autenticación; las landings no autentican; POS sin página pública; Backoffice sin enlaces públicos | Seguridad por origen y un mensaje claro por público |
| 2026-09-24 | Marketplace con navegación libre; cuenta y billetera al comprar o al pulsar "Entrar"; el escaneo de botella exige registro | Menos fricción antes de la intención de compra |
| 2026-09-24 | ERP y POS solo inicio de sesión; usuarios y puntos provisionados desde el Backoffice | Control de acceso centralizado |
| 2026-09-24 | Un repositorio por sistema bajo `BrianKGR01`; paquetes compartidos `@doc/ui`, `@doc/mocks`, `@doc/wallet` | Ciclos de vida distintos |
| 2026-09-24 | Conventional Commits; trabajo en `dev`; PR `dev → main` por hito | Historial limpio |
| 2026-09-24 | Tipografías libres (Cormorant Garamond, EB Garamond); fotografías con licencia libre y crédito | Licencias |
| **2026-09-25** | **Marca y equipo: Drinks on Chain**, sin ninguna otra razón social en documentos ni interfaces. Los textos que aún nombran otra empresa (pie, contacto, aviso legal de ambos sitios) se corrigen en el primer hito pendiente | Decisión del cliente |
| 2026-09-25 | Billetera del consumidor: **smart account con passkey** (`smart-account-kit`, contratos OpenZeppelin, relayer propio) por defecto; **cuenta gestionada por la plataforma** como respaldo automático cuando el dispositivo no soporta passkeys; billetera externa como opción avanzada; proveedores embebidos (Privy) documentados pero fuera del MVP | `passkey-kit`/Launchtube quedaron obsoletos; coste mínimo y fricción mínima; ver 04 |
| 2026-09-25 | La creación de la passkey y la derivación de la dirección ocurren en el **frontend** (Marketplace); el despliegue, el pago de comisiones y el registro los hace el **backend** (relayer) | WebAuthn solo existe en el dispositivo del usuario; el usuario no tiene XLM |
| 2026-09-25 | Cuentas Stellar de las **bodegas**: las crea el **backend** al dar de alta la bodega desde el Backoffice; custodia en KMS/HSM. El ERP solo muestra un panel de lectura. Ninguna clave institucional en un navegador | Estándar de custodia institucional; no bloquea el flujo productivo |
| 2026-09-25 | **POS y Backoffice no tienen billetera en el cliente**; disparan operaciones que firma el backend y muestran estados de transacción | Fricción cero para cajeros y gestores |
| 2026-09-25 | Pase de retiro **emitido por el backend** bajo la sesión del usuario (QR firmado, caducidad en días) y **quema por clawback del emisor** al confirmar la entrega; la firma con passkey queda como refuerzo opcional | Funciona igual con cuentas passkey y gestionadas; nadie firma en el mostrador |
| 2026-09-25 | Modelo de token propuesto a backend: **activo clásico por lote con clawback**, en poder de smart accounts vía SAC (sin trustlines), metadatos en `stellar.toml`; alternativa SEP-41 documentada | Cero desarrollo de contratos; quema nativa; visible en exploradores |
| 2026-09-25 | Idiomas: **español** en ERP, Backoffice, POS y Marketplace; ES/EN solo en las landings. El Marketplace se construye con diccionario para poder añadir inglés después | Pedido del cliente |
| 2026-09-25 | Hardware del POS: **iPad o tablet Android**; la app es una PWA en modo kiosco que sirve a ambos | Pedido del cliente |
| 2026-09-25 | Pase de retiro válido en **los puntos habilitados para ese lote** (la bodega y sus puntos asociados, configurados en el Backoffice); caducidad en **días** | Pedido del cliente |
| 2026-09-25 | Pasarela de pago: **la del banco del proyecto**; el checkout se construye con un adaptador y datos de prueba hasta recibir su documentación | Pedido del cliente |
| 2026-09-25 | No se planifica nada sobre bodegas socias reales; todos los datos de bodegas son de prueba o de referencia | Pedido del cliente |
| 2026-09-25 | Sistema de diseño con **dos familias** (editorial y operativa) sobre los mismos tokens; maquetas HTML por sistema en `design-system/` | Las apps deben ser herramientas claras sin perder la identidad de las landings |
| 2026-09-25 | Orden de construcción: Etapa 0 (fundaciones) → ERP → Marketplace → POS → Backoffice → integración; los pendientes de los sitios públicos se cierran en paralelo | El backend del ERP ya existe; Marketplace + POS forman la demo para el grant; ver 03 |
| 2026-09-25 | **El OpenAPI del backend desplegado es la fuente de verdad** del contrato del ERP (por encima de `backend/endpoints.md` y de las guías, que discrepan en nombres de campos). Los fixtures del ERP siguen sus DTO exactos; el "Lote" del ERP es una vista derivada en el cliente | Evitar rehacer al integrar; ver 09 |
| 2026-09-25 | La documentación de frontend vive en el repo `drinks-on-chain-docsfront` (esta carpeta) con `index.html` publicado en GitHub Pages para el sistema de diseño; mismo flujo `dev → main` y Conventional Commits | Que backend y cliente vean las decisiones |

## 2. Preguntas cerradas el 25-09-2026 (registro)

| Pregunta | Respuesta |
|---|---|
| ¿Bodegas socias con acuerdo? | No se toca todavía; datos de prueba |
| ¿Pasarela de pago? | La del banco; documentación por pedir (ver §3) |
| ¿Dominio raíz? | Se comprará; subdominios decididos |
| ¿Idiomas de las aplicaciones? | Solo español por ahora |
| ¿Hardware de los puntos de recojo? | iPad o tablet Android |
| ¿Alcance y caducidad del pase? | Puntos habilitados para el lote; caducidad en días |
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
6. **Precio en pantalla**: ¿solo BOB o también referencia en USD?

## 5. Preguntas para el equipo de backend

1. **Contrato del ERP**: recibido e incorporado (09). Quedan los 12 puntos de alineación de 09 §8: esquema de respuesta de las listas, URL del QR configurable, bifurcación y estado del tanque sin `PATCH`, análisis obligatorio en el pesaje, discrepancias del catálogo con el OpenAPI, registro de direcciones `C…`, códigos QR por botella, `traceability/public` realmente público, y **datos de prueba cargados en el servidor de desarrollo** (hoy está vacío).
2. **Contrato del token**: ¿aceptan el activo clásico por lote con clawback vía SAC (04 §6) o prefieren un contrato SEP-41? ¿Emisor por bodega o único?
3. **Relayer**: ¿OpenZeppelin Relayer autoalojado o servicio de terceros? ¿Cómo entrega el cliente la autorización de corta vida?
4. **Indexación** de saldos y eventos: ¿Mercury o servicio propio? ¿Websocket o sondeo para `lot.ready`, `mint.confirmed`, `claim.confirmed`?
5. **Cuentas institucionales**: ¿cuenta clásica con clave en KMS o smart account con firmante de política? ¿Qué devuelve el alta de bodega?
6. **Autenticación**: ¿identidad única (OIDC) para los cuatro sistemas o sesiones separadas? ¿2FA del Backoffice con TOTP? ¿Vinculación de dispositivos POS por código de un solo uso?
7. **Modelo de datos**: ¿aceptan `@doc/mocks` como borrador del contrato para los sistemas que aún no tienen backend?
8. **Archivos de QR** del ERP: ¿los genera el frontend (ZIP con PNG/SVG) o el backend (PDF para imprenta)?
9. **Entornos**: testnet ahora; ¿fecha objetivo para mainnet?

## 6. Supuestos vigentes mientras no haya respuesta

- Token: activo clásico por lote, 1 unidad = 1 botella, quema por clawback.
- Pase de retiro: caducidad de 7 días; válido en los puntos habilitados para el lote.
- Pago en BOB; ningún usuario paga XLM.
- Español en aplicaciones; inglés solo en landings.
- Subdominios bajo un dominio único; hasta la compra, los enlaces entre sitios usan los hosts de Vercel.
- Recuperación de cuenta sugerida (no obligatoria) antes de la primera compra.
