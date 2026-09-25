# 04 · Billeteras sobre Stellar: qué hace el cliente, en qué sistema y por qué

Versión 2 · 25 de septiembre de 2026. Sustituye a la v1 (`antiguo/04-billeteras-stellar.md`). Verificado contra la documentación pública de Stellar y los repositorios oficiales el 25-09-2026.

**Corrección respecto a la v1**: `passkey-kit` quedó como librería de legado y **Launchtube (el relayer hospedado por la SDF) fue retirado**. El camino oficial hoy es `smart-account-kit` (repositorio `stellar/smart-account-kit`), construido sobre los contratos de cuenta inteligente de OpenZeppelin (auditados), con patrocinio de comisiones a través de un relayer que opera el propio proyecto (OpenZeppelin Relayer, código abierto) o un servicio de terceros. Todo lo demás de la v1 se mantiene o se precisa aquí.

## 1. Principios

1. **Sin fricción para el consumidor**: no hay frase semilla, no hay extensión, no hay XLM que comprar, no hay trustlines. La billetera se crea en el momento en que se crea la cuenta (al comprar o al pulsar "Entrar") con la biometría del dispositivo.
2. **El consumidor nunca paga en cripto ni firma compras en el MVP**: compra en moneda local; la billetera solo recibe tokens y, al retirar, el sistema los quema. Eso mantiene la promesa "sin conocimientos Web3" y reduce la superficie regulatoria.
3. **Ninguna clave institucional en un navegador**: las claves de la plataforma (emisora, relayer) y las cuentas de las bodegas viven en el backend, en custodia con KMS/HSM. El frontend muestra estados y enlaces al explorador.
4. **Coste mínimo**: nada de SaaS obligatorio. La opción por defecto solo requiere el relayer propio (un servicio pequeño) y comisiones de red de fracciones de centavo. Las opciones de pago quedan documentadas como alternativas.
5. **Una sola API en el cliente**: el paquete `@doc/wallet` esconde la implementación; el Marketplace no sabe si debajo hay una passkey, una cuenta gestionada o una billetera externa. Tiene modo `mock` para construir todas las pantallas sin red.

## 2. Qué cambia por ser Stellar (lo que el frontend tiene que saber)

| Tema | En Stellar | Consecuencia |
|---|---|---|
| Tipos de cuenta | Cuentas clásicas `G…` (ed25519, reserva base en XLM, trustline por activo) y **cuentas de contrato / smart accounts `C…`** (contratos Soroban cuyos firmantes pueden ser passkeys WebAuthn secp256r1, claves ed25519, cuentas `G…` delegadas o políticas) | La tribu usa smart accounts: firma con Face ID / huella / PIN del dispositivo, sin seed phrase |
| Tokens | Activos clásicos (emisor + código) expuestos a los contratos por el **Stellar Asset Contract (SAC)**, y tokens de contrato SEP-41 | Una smart account puede recibir un activo clásico **sin trustline**: su saldo se guarda en el almacenamiento del contrato (documentación del SAC) |
| Quema controlada | Los activos clásicos con `AUTH_CLAWBACK_ENABLED` (y `AUTH_REVOCABLE`) permiten al emisor **recuperar saldo** (`clawback`) también en saldos de contratos | La entrega física se cierra con un clawback del emisor: el consumidor no tiene que firmar nada en el mostrador |
| Comisiones y reservas | Muy bajas; las paga quien envía la transacción; una smart account no necesita reserva base pero su almacenamiento paga renta (TTL) | El relayer del proyecto paga despliegue y renta; el usuario nunca ve XLM |
| Firma | WebAuthn en el cliente; envío por relayer | La ceremonia biométrica **solo puede ocurrir en el dispositivo del usuario**: por eso la creación de la billetera es trabajo del frontend |
| Identidad web | SEP-10 (desafío firmado), SEP-1 (`stellar.toml` con metadatos de activos), SEP-7 (URI) | `stellar.toml` publica los metadatos de cada lote sin escribir un contrato |
| Herramientas | `@stellar/stellar-sdk`, Soroban RPC, Horizon, `smart-account-kit`, OpenZeppelin Relayer, Mercury (indexador), `@creit.tech/stellar-wallets-kit`, explorador stellar.expert | Todo con SDK JavaScript |

## 3. Opciones para la billetera del consumidor, ordenadas por fricción y coste

| Opción | Cómo se crea | Fricción usuario | Coste | Riesgos | Veredicto |
|---|---|---|---|---|---|
| **A. Smart account con passkey** (`smart-account-kit` + contratos OpenZeppelin + relayer propio) | Al crear la cuenta, el navegador crea una passkey (Face ID / huella / PIN). La dirección `C…` se deriva de forma determinista y el relayer despliega el contrato y paga las comisiones | Mínima: un gesto biométrico. La passkey se sincroniza en iCloud Keychain o Google Password Manager | Infraestructura: un relayer pequeño (código abierto) + XLM para comisiones y renta. Sin SaaS | SDK joven (contratos auditados, kit sin auditar); teléfonos sin WebAuthn (Android < 9, iOS < 16); recuperación hay que diseñarla | **Por defecto** |
| **B. Cuenta gestionada por la plataforma** (el backend custodia una clave por usuario; el usuario solo tiene su login) | Al crear la cuenta, el backend genera y guarda la clave (KMS) y crea la cuenta; el cliente no hace nada | Cero | Custodia y su responsabilidad (KMS, procedimientos); comisiones y reservas las paga la plataforma | El usuario no controla su activo; carga regulatoria de custodia; exportar la clave luego es posible pero incómodo | **Respaldo automático** cuando el dispositivo no soporta passkeys, y modelo de las cuentas institucionales |
| **C. Billetera embebida de un proveedor** (Privy, que soporta Stellar; Web3Auth / MetaMask Embedded Wallets) | El usuario entra con Google o correo; el proveedor deriva la clave (MPC/TEE) | Baja: login social | SaaS de pago por usuario activo; dependencia de un tercero | Cuenta clásica: hay que fondearla y abrir trustlines patrocinadas; la clave la gestiona el proveedor | **Alternativa futura** si la recuperación de passkeys resulta un problema real; no para el MVP |
| **D. Billetera externa** (`@creit.tech/stellar-wallets-kit`: Freighter, xBull, Lobstr, Albedo, Hana, WalletConnect) | El usuario conecta una billetera que ya tiene | Alta para el consumidor medio | Ninguno | Solo para quien ya está en cripto | **Opcional** en "Ajustes → Billetera avanzada" y como firmante añadido de la smart account |

Decisión: **A por defecto, B como respaldo silencioso, D como opción avanzada, C documentada pero fuera del MVP**. Las cuatro caben detrás de `@doc/wallet`.

Nota sobre "login con Google": con la opción A el usuario también ve a Google en pantalla, porque su passkey se guarda en el Gestor de contraseñas de Google y se sincroniza entre sus dispositivos Android/Chrome. La diferencia es que la clave nunca sale de su dispositivo ni pasa por un proveedor.

## 4. Quién hace qué en la creación de la billetera (la pregunta central)

La creación tiene dos partes que no pueden intercambiarse:

| Parte | Dónde | Por qué |
|---|---|---|
| Crear la credencial (passkey) y firmar | **Frontend** (navegador o PWA del usuario) | WebAuthn solo existe en el dispositivo del usuario; el servidor jamás ve la clave privada |
| Derivar la dirección `C…` | Frontend (determinista a partir de la credencial) | Permite mostrar la dirección al instante y registrarla en el perfil |
| Desplegar el contrato de cuenta y pagar comisiones | **Backend** (relayer) | Alguien tiene que pagar; el usuario no tiene XLM. El kit envía `{ función, autorización }` al relayer configurado (`relayerUrl`) |
| Registrar la dirección en el usuario | Backend (`POST /wallets`) | Es dato de perfil |
| Indexar saldos y firmantes | Backend (Mercury o indexador propio) | El cliente lee por la API con caché y ofrece enlace al explorador para verificar |
| Recuperación | Frontend inicia (segunda passkey en otro dispositivo, o firmante de recuperación por correo); backend coordina la política del contrato | Una passkey perdida sin segundo firmante es una cuenta perdida: pedirlo antes de la primera compra |

En la opción B todo ocurre en el backend y el frontend solo muestra "tu cava está lista"; `@doc/wallet` devuelve `kind: 'gestionada'` y la misma dirección.

## 5. Responsabilidades por sistema

### S2 · Marketplace: la única billetera de usuario final

| Momento | Cliente | Backend |
|---|---|---|
| Crear cuenta ("Entrar" o al comprar) | Verifica teléfono/correo; pantalla "Preparando tu cava digital": detecta soporte WebAuthn (`PublicKeyCredential`, `isUserVerifyingPlatformAuthenticatorAvailable`), lanza `createWallet()`; si no hay soporte, pide una cuenta gestionada | Registra la dirección; despliega el contrato vía relayer si hace falta; o crea la cuenta gestionada |
| Comprar | Checkout en moneda local con la pasarela del banco; estado "reservando tus botellas" | Cobra, transfiere N unidades del activo del lote a la dirección, confirma `order.paid` |
| Mi Cava | `getHoldings()` desde la API (caché) con "Verificar en la red" (enlace al explorador) | Indexa saldos por dirección |
| Pase de retiro | Pide `POST /claims` con lote, cantidad y punto de recojo; muestra el ticket con QR (payload firmado por el backend, caducidad en días); opcionalmente añade una firma de la passkey como prueba de posesión (`signChallenge()`) | Emite el pase, lo valida al escanearlo en el POS, ejecuta el clawback al confirmar |
| Recuperación | "Añadir este dispositivo" y "Correo de recuperación" (`addRecoverySigner()`), sugerido antes de la primera compra | Coordina la política del contrato |
| Billetera avanzada (ajustes) | Conectar billetera externa como firmante adicional o como destino de transferencia (Fase 2) | — |
| Fase 2 · P2P y preventas | `signTransaction()` para listar y aceptar ofertas | Casa de órdenes o escrow |

**Cambio respecto a la v1**: el pase de retiro **no depende de una firma de la billetera**. El backend lo emite bajo la sesión del usuario y la quema la ejecuta el emisor por clawback. La firma con passkey queda como refuerzo opcional (anti-fraude) que se activa cuando la opción A está en producción; así el flujo de claim funciona igual para cuentas passkey y gestionadas y no exige tener el teléfono con biometría operativa en el mostrador.

### S1 · ERP: cuenta institucional de la bodega, sin billetera en el cliente

Cada bodega tiene una **cuenta Stellar institucional** para que el emisor del activo de cada lote sea identificable como la bodega y para las liquidaciones de la Fase 2.

- **Se crea desde el backend** cuando el equipo gestor da de alta la bodega en el Backoffice. Motivos: es una clave que debe sobrevivir a cambios de personal, quedar auditada, firmar operaciones automáticas (emisión, clawback) sin una persona delante de un navegador, y custodiarse en KMS/HSM; es el estándar para cuentas de tesorería. Puede ser una cuenta clásica `G…` con la clave en KMS o una smart account con la plataforma como firmante de política; la decisión es del backend.
- **El ERP muestra** un panel de solo lectura: dirección, activos emitidos por lote, historial con enlaces al explorador. No hay clave privada en el navegador. Opcional en Fase 2: "Conectar mi firmante" (`stellar-wallets-kit`) para atestar lotes o añadir un firmante propio a la cuenta.
- Nada de esto bloquea el flujo productivo del ERP; si el backend no expone todavía la cuenta, el panel se oculta con una bandera.

### S3 · Backoffice: dispara operaciones que firma el backend

- La **clave emisora** de la plataforma, las claves de las bodegas y el relayer viven en el servidor. El frontend nunca las toca.
- El cliente **dispara**: "Crear bodega" (el backend crea la cuenta institucional y devuelve dirección y estado), "Aprobar, emitir y publicar" (el backend emite el activo del lote y transfiere el inventario a la cuenta de venta), "Anular pase / autorizar entrega manual" (soporte), "Quemar manualmente" (excepcional).
- El cliente **muestra**: estado de cada transacción (`pendiente → enviada → confirmada → fallida`) con hash y enlace a stellar.expert, saldos de las cuentas operativas (XLM del relayer, inventario por lote), alertas de reserva baja.
- Opcional, fuera del MVP: co-firma del administrador con su passkey o Freighter para operaciones sensibles (política 2-de-3). Es la única firma humana posible en S3 y no se planifica hasta Fase 2.

### S4 · POS: ninguna billetera

Escanea el pase, lo valida contra la API (`POST /claims/:id/validate`), el cajero confirma (`POST /claims/:id/confirm`), el backend quema. El POS muestra "Entregado" al instante (confirmación de la API) y, cuando llega, "Confirmado en la red" con el hash. Si la red se demora, la entrega ya está registrada y el POS no espera.

### Sitios públicos: ninguna billetera

La landing principal y el sitio de bodegas no tienen sesión ni billetera. Solo enlazan a `app.`.

## 6. Modelo de token recomendado (para negociar con backend)

**Recomendación MVP: un activo clásico por lote, en poder de smart accounts a través del SAC.**

- Código del activo: hasta 12 caracteres alfanuméricos (por ejemplo `SGR2026CINTI` o un código corto por lote); emisor: la cuenta institucional de la bodega (o de la plataforma si backend prefiere un solo emisor).
- Flags del emisor: `AUTH_REVOCABLE` + `AUTH_CLAWBACK_ENABLED` (para la quema por clawback al entregar); `AUTH_REQUIRED` opcional si se quiere autorizar explícitamente cada cuenta receptora.
- Unidades: 1 unidad = 1 botella; el activo clásico tiene 7 decimales, así que la UI muestra siempre enteros y el backend transfiere cantidades enteras.
- Metadatos: entrada `[[CURRENCIES]]` en el `stellar.toml` del dominio (nombre, descripción, imagen, emisor) más el registro del lote en la API. Sin contrato propio, sin auditoría, visible en exploradores y billeteras.
- Ventajas: cero desarrollo de contratos, quema por clawback nativa, saldos de smart accounts sin trustline, trazabilidad pública inmediata.
- Límite: no hay identidad por botella. Se acepta en el MVP; la Fase 2 (corcho NFC, NFT individual) usaría un contrato Soroban propio.

**Alternativa**: contrato Soroban SEP-41 con `decimals = 0`, un contrato por ecosistema con metadatos por lote y función de quema administrada. Más expresivo, pero exige desarrollar y auditar un contrato. Se documenta por si backend ya tiene esa dirección tomada.

En ambos casos el frontend consume lo mismo: `holdings` por dirección, `tx` por hash y enlaces al explorador. Testnet primero (Friendbot); mainnet al cerrar la integración.

## 7. Paquete `@doc/wallet`

```ts
type WalletKind = 'passkey' | 'gestionada' | 'externa'
type Network = 'testnet' | 'public'

createWallet(displayName: string): Promise<{ address: string; kind: WalletKind }>
connectWallet(kind?: WalletKind): Promise<Wallet>          // restaura la passkey o abre stellar-wallets-kit
getHoldings(address: string): Promise<Holding[]>            // vía API propia; fallback RPC/Mercury
getTx(hash: string): Promise<TxStatus>
signChallenge(payload: Uint8Array): Promise<Signature>      // refuerzo opcional del pase; SEP-10-like
signTransaction(xdr: string): Promise<string>               // Fase 2: P2P, ventas
addRecoverySigner(kind: 'device' | 'email'): Promise<void>
supportsPasskeys(): Promise<boolean>
network: Network
```

Implementaciones: `passkey` (`smart-account-kit` con `relayerUrl` del backend y `IndexedDBStorage`), `gestionada` (solo lecturas por la API), `externa` (`@creit.tech/stellar-wallets-kit`), `mock` (fixtures de `@doc/mocks`, latencias simuladas, direcciones de prueba). Hooks: `useWallet()`, `useHoldings()`, `useTx(hash)`. Vive junto a `@doc/ui` y `@doc/mocks`.

## 8. Librerías y servicios (verificados el 25-09-2026)

| Uso | Paquete / servicio | Notas |
|---|---|---|
| SDK base, XDR, Horizon, Soroban RPC | `@stellar/stellar-sdk` (≥ 16.3) | — |
| Smart accounts con passkeys | `smart-account-kit` (`stellar/smart-account-kit`) + `smart-account-kit-bindings` | Sucesor de `passkey-kit`; contratos OpenZeppelin auditados; el kit se declara "software de integración sin auditar": fijar versiones y envolver en `@doc/wallet` |
| Relayer (patrocinio de comisiones) | OpenZeppelin Relayer (código abierto, autoalojado) o un servicio de terceros | Launchtube fue retirado; el relayer lo opera backend |
| Indexación de billeteras y saldos | Mercury (integrado en el kit) o servicio propio | Decisión de backend |
| Billeteras externas | `@creit.tech/stellar-wallets-kit`, `@stellar/freighter-api` | Solo opción avanzada |
| Billetera embebida (alternativa) | Privy (soporta Stellar) | Fuera del MVP |
| QR | `qrcode` (generar), `@zxing/browser` o `BarcodeDetector` nativo (leer) | S1 genera, S2 y S4 leen |
| Explorador | stellar.expert | Enlaces por hash y por dirección |

## 9. Grants: qué debemos poder enseñar

Stellar Community Fund, Build Award: hasta 150.000 USD en XLM por proyecto, rondas cada seis semanas, tres pistas (abierta, integración, RFP), entrada por formulario de interés. Para una candidatura sólida el frontend debe mostrar en testnet, con código abierto: creación de cuenta con passkey y cava lista en menos de un minuto, activo de lote visible en Mi Cava y en el explorador, pase de retiro validado desde el POS y quema confirmada, y la trazabilidad pública del lote. Es el recorrido S2 (cuenta, compra, cava, pase) → S4 (validar, confirmar), y por eso el roadmap coloca el POS inmediatamente después del Marketplace.

## 10. Lo que necesitamos del backend

- `POST /wallets` (registrar dirección y tipo), `GET /me/holdings`, `GET /tx/:hash`.
- Relayer: URL y autenticación de corta vida para el cliente (`relayerUrl` del kit), o un proxy propio.
- `POST /claims` (devuelve `claimId`, payload firmado y caducidad), `POST /claims/:id/validate` (POS), `POST /claims/:id/confirm` (dispara el clawback), `POST /claims/:id/void` (soporte).
- Alta de bodega que crea la cuenta institucional y devuelve `{ address, status }`; emisión que devuelve `{ txHash, status }`.
- Activos e issuer en testnet y `stellar.toml` publicado en el dominio.
- Decisión sobre el indexador.

## 11. Riesgos

| Riesgo | Mitigación |
|---|---|
| Teléfonos sin passkeys | Detección y cuenta gestionada automática (opción B) con la misma pantalla de éxito |
| Pérdida de la passkey | Segundo firmante (dispositivo o correo) sugerido antes de la primera compra y obligatorio para retirar más de N botellas |
| Kit joven sin auditar | Versiones fijadas, todo detrás de `@doc/wallet`, spike en testnet en la Etapa 0, plan B sin cambiar pantallas |
| Relayer caído | El relayer solo interviene al crear la cuenta; compras y claims siguen funcionando por la API. Cuenta gestionada como respaldo |
| Renta del almacenamiento (TTL) de las smart accounts | Tarea de backend: extender TTL periódicamente; alerta en el Backoffice |
| Regulación de tokens de consumo | El token representa una botella pagada en moneda local y no se comercia en el MVP; la Fase 2 requiere asesoría legal y KYC |
