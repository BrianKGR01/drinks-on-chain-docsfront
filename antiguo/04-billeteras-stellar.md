# 04 · Billeteras digitales sobre Stellar: qué hace el cliente y dónde

Decisión de partida: el ecosistema se construye sobre **Stellar** (posibilidad de financiación vía Stellar Community Fund). Este documento fija qué se resuelve en el frontend, en qué sistema, con qué librerías, y qué le pedimos al backend. Verificado el 24 de septiembre de 2026 contra la documentación pública.

## 1. Lo que cambia por ser Stellar

| Tema | En Stellar | Consecuencia para el frontend |
|---|---|---|
| Tipos de cuenta | **Cuentas clásicas** (`G…`, clave ed25519, requieren reserva base en XLM y una *trustline* por activo) y **cuentas de contrato / smart wallets** (`C…`, contratos Soroban cuyos firmantes pueden ser passkeys WebAuthn, claves ed25519 o contratos de política) | Para la tribu usamos smart wallets: sin frase semilla, sin trustlines, firma con Face ID / huella / PIN del dispositivo |
| Tokens | Activos clásicos (emisor + código) expuestos también como contrato (SAC) y tokens de contrato **SEP-41** en Soroban | El token de lote se implementa como contrato SEP-41 (o un contrato multi-lote); los consumidores no gestionan trustlines |
| Comisiones y reservas | Muy bajas; pueden ser **patrocinadas** (sponsored reserves, fee bump, relayer) | El usuario nunca compra XLM; el backend/relayer paga |
| Firma | Ed25519 o, en smart wallets, secp256r1 (passkey) más políticas | El cliente firma con WebAuthn; el servidor envía por un relayer |
| Identidad Web | SEP-10 (autenticación con desafío firmado), SEP-1 (`stellar.toml`), SEP-7 (URI de pagos, útil para QR) | El pase de retiro puede ser un desafío firmado por la billetera |
| Tooling | `@stellar/stellar-sdk`, Soroban RPC, Horizon, Mercury (indexador), Launchtube (relayer), Freighter, Stellar Wallets Kit, explorador stellar.expert | Todo tiene SDK JavaScript maduro |

## 2. Opciones para la billetera del consumidor (Sistema 2)

| Opción | Cómo se crea la cuenta | Pros | Contras | Encaje |
|---|---|---|---|---|
| **A. Smart wallet con passkey** — `stellar/passkey-kit` (antes kalepail; ahora repositorio oficial de Stellar), contrato de cuenta WebAuthn, Launchtube para enviar sin que el usuario pague, Mercury para indexar firmantes | El usuario pulsa "Crear mi cava" y confirma con biometría; la passkey se sincroniza con Google Password Manager (Android/Chrome) o iCloud Keychain (Apple) | Cero fricción, cero seed phrase, sin trustlines ni XLM, patrón recomendado por la SDF, alineado con el SCF (contratos Soroban con auditoría gratuita), recuperación y multi-dispositivo vía firmantes adicionales y políticas | Requiere navegador con WebAuthn (Android 9+/Chrome, iOS 16+); la recuperación hay que diseñarla (segundo firmante, política de recuperación); Launchtube en mainnet depende de cuota/credenciales | **Recomendada** |
| **B. Login social con clave MPC** — Privy (soporta Stellar), Web3Auth / MetaMask Embedded Wallets (ed25519, Stellar vía conversión de curva) | El usuario entra con Google o correo; el proveedor deriva una clave ed25519 → cuenta clásica `G…` | Exactamente el "login con Google" que pide el cliente; recuperación resuelta por el proveedor | Cuenta clásica: hay que crearla y fondearla (reserva base) y abrir trustlines patrocinadas por cada colección; dependencia de un SaaS de pago; la clave la gestiona un tercero | **Plan B** o complemento (puede añadirse como segundo firmante de la smart wallet) |
| **C. Billetera externa** — Stellar Wallets Kit (Freighter, xBull, Lobstr, Albedo, Hana, WalletConnect…) | El usuario ya tiene una billetera y la conecta | Cero custodia, ideal para coleccionistas y para roles internos | Fricción total para el consumidor medio | **Opcional** para usuarios avanzados y para bodegas/administradores |

Recomendación: A como camino por defecto, C disponible en "Ajustes → Billetera avanzada", y B evaluada en el spike de la Etapa 0 como alternativa si las passkeys dan problemas en el parque de teléfonos boliviano. Las tres se esconden detrás de una misma API (`doc-wallet`), de modo que el Marketplace no sabe cuál hay debajo.

Nota sobre "login con Google": con la opción A el usuario también ve Google en pantalla, porque la passkey se guarda en su Gestor de contraseñas de Google y se sincroniza entre sus dispositivos Android/Chrome. La diferencia es que la clave nunca sale del dispositivo ni pasa por un proveedor.

## 3. Paquete `doc-wallet` (frontend, compartido)

```ts
createWallet(displayName): Promise<{ address, kind: 'passkey' | 'social' | 'external' }>
connectWallet(kind?): Promise<Wallet>
getBalances(address): Promise<Holding[]>            // vía API propia (cache) con fallback a RPC/Mercury
signChallenge(payload): Promise<SignedEnvelope>      // SEP-10-like para pases de retiro y login
signTransaction(xdr): Promise<xdr>                   // Fase 2: P2P, ventas
addRecoverySigner(): Promise<void>                   // segundo dispositivo / correo de recuperación
network: 'testnet' | 'public'
```
Implementaciones: `passkey` (passkey-kit + Launchtube token del servidor), `social` (Privy o Web3Auth), `external` (Stellar Wallets Kit). Hooks React: `useWallet()`, `useHoldings()`, `useSignChallenge()`. Vive en el monorepo de paquetes junto a `@doc/ui` y `@doc/mocks`, y tiene un modo `mock` para las pantallas sin red.

## 4. Responsabilidades por sistema

### Sistema 2 · Marketplace (el único con billetera de usuario final)
| Momento | Qué hace el cliente | Qué hace el backend |
|---|---|---|
| Registro ligero | Tras verificar teléfono/correo, la pantalla "Preparando tu cava digital" llama a `createWallet()`: ceremonia WebAuthn, dirección `C…` determinista | Registra la dirección en el perfil; si hace falta desplegar el contrato de cuenta, lo envía por el relayer |
| Compra | Checkout fiat (tarjeta / QR bancario); muestra el estado "acuñando tus botellas" | Cobra, transfiere N tokens del lote a la dirección del usuario, confirma |
| Mi Cava | `getBalances()` desde la API (cache) y verificación opcional en cadena con enlace al explorador | Indexa balances por dirección (Mercury o propio) |
| Pase de retiro | Construye el payload `{claimId, tokenId, cantidad, sucursal?, caducidad}` y lo firma con `signChallenge()`; renderiza el QR con la firma | Emite el `claimId`, verifica la firma al validar en el POS y ejecuta la quema (o transferencia + quema) con la autorización del usuario |
| Recuperación | "Añadir este dispositivo" con `addRecoverySigner()`; correo de recuperación como firmante de política | Coordina la política de recuperación del contrato |
| Fase 2 · P2P | `signTransaction()` para listar y aceptar ofertas | Casa de órdenes o contrato de escrow |

Decisión de diseño importante: en el MVP el usuario **no paga en cripto ni firma compras**; la billetera solo recibe y luego autoriza la salida (claim). Eso mantiene la promesa "sin conocimientos Web3" y reduce la superficie regulatoria.

### Sistema 1 · ERP (bodegas)
Las bodegas no necesitan operar cripto en el MVP: el Backoffice emite, el consumidor compra en fiat y la liquidación a la bodega es bancaria. Aun así, cada bodega tiene una **cuenta Stellar institucional** (para que el emisor del token sea identificable como la bodega y para la Fase 2 de liquidaciones y regalías):
- Se crea desde el Backoffice al dar de alta la bodega, como smart wallet con **firmantes de política**: Debro como operador inicial y, cuando la bodega quiera, un firmante propio (Freighter/xBull vía Stellar Wallets Kit o passkey del administrador).
- En el ERP el cliente muestra un panel de solo lectura (dirección, saldos, historial con enlaces al explorador) y un botón opcional "Conectar mi firmante" para atestar lotes ("este lote fue embotellado por nosotros") firmando un desafío. Nada de esto bloquea el flujo productivo.
- No hay claves privadas de la bodega en el navegador del ERP.

### Sistema 3 · Backoffice (Debro)
- La **clave emisora** de los tokens y la clave del relayer viven en el servidor (HSM o proveedor tipo Turnkey); el frontend nunca las toca.
- El cliente muestra: estado de cada transacción (pendiente → enviada → confirmada → fallida) con hash y enlace a stellar.expert, saldos de las cuentas operativas (XLM del relayer, crédito de Launchtube) y alertas de reserva baja.
- Operaciones sensibles (mintear, quemar manualmente, cambiar precio) pueden exigir **co-firma** del administrador con su passkey o Freighter (política 2-de-3 en el contrato). Es la única firma de usuario en S3 y es opcional en el MVP.

### Sistema 4 · POS
- Ninguna billetera. Escanea el pase, lo valida contra la API, el cajero confirma, la API quema. El POS solo necesita mostrar "confirmado en cadena" con el hash cuando llegue.

## 5. Modelo de token propuesto (a negociar con backend)

- Un contrato Soroban **por ecosistema** que gestiona lotes como tokens fungibles con `decimals = 0` (1 unidad = 1 botella), con metadatos por lote (bodega, valle, año, hash del QR, URI de metadatos), cumpliendo SEP-41 en las funciones de usuario para que las billeteras y exploradores lo entiendan. Alternativa: un contrato por lote (más simple de razonar, más despliegues).
- Emisión ("mint") solo por la clave emisora de Debro tras la aprobación en S3; quema por el contrato al confirmar la entrega, con la autorización del titular (firma del pase).
- Identidad por botella (NFT individual, corcho NFC) se pospone a la Fase 2, cuando exista hardware.
- Testnet primero (Friendbot para fondos), mainnet al cerrar la Etapa 6.

## 6. Librerías y servicios

| Uso | Paquete / servicio |
|---|---|
| SDK base, XDR, Horizon, Soroban RPC | `@stellar/stellar-sdk` |
| Smart wallets con passkeys | `passkey-kit` (cliente y servidor), Launchtube (relayer y patrocinio de comisiones), Mercury (indexador y descubrimiento de firmantes) |
| Billeteras externas | `@creit.tech/stellar-wallets-kit` (Freighter, xBull, Lobstr, Albedo, Hana, WalletConnect…), `@stellar/freighter-api` |
| Login social (plan B) | Privy (Stellar soportado) o Web3Auth / MetaMask Embedded Wallets (ed25519) |
| QR | `qrcode` (generar), `@zxing/browser` (leer), SEP-7 para URIs si se usan pagos en cadena |
| Explorador | stellar.expert (enlaces por hash y por dirección) |

## 7. Grants: qué debemos poder enseñar

Stellar Community Fund, Build Award: hasta 150.000 USD en XLM por proyecto (~3 meses), en 4 tramos por hitos, rondas cada 6 semanas, entrada por formulario de interés; proyectos con Soroban acceden a auditorías de seguridad gratuitas; el panel evalúa el valor para el ecosistema Stellar/Soroban.

Para una candidatura sólida el frontend debe mostrar en testnet, con código abierto: onboarding con passkey y creación de la cava en menos de un minuto, token de lote SEP-41 visible en Mi Cava y en el explorador, pase de retiro firmado y quema confirmada desde el POS, y la trazabilidad pública del lote. Es exactamente el recorrido de las sub-etapas 3A → 3C → 5B del roadmap, así que conviene ordenar el trabajo para tener esa demo antes de la primera ronda a la que se aplique.

## 8. Lo que necesitamos del backend (contrato mínimo)

- `POST /wallets` (registrar dirección por usuario), `GET /me/holdings`, `POST /claims` (devuelve `claimId` y payload a firmar), `POST /claims/:id/validate` (POS), `POST /claims/:id/confirm` (dispara la quema), `GET /tx/:hash` (estado).
- Endpoint de sesión para Launchtube (token de relayer de corta vida para el cliente) o proxy propio de envío.
- Cuenta emisora y contrato desplegados en testnet con ABI/`spec` publicado para generar los bindings TypeScript.
- Decisión sobre el indexador de balances (Mercury frente a propio).

## 9. Riesgos específicos

| Riesgo | Mitigación |
|---|---|
| Teléfonos sin soporte de passkeys | Detectar `PublicKeyCredential` y ofrecer el camino social (plan B) o una passkey en el navegador de escritorio |
| Pérdida de la passkey | Segundo firmante obligatorio (correo de recuperación como política) antes de la primera compra |
| Disponibilidad/cuota de Launchtube en mainnet | Relayer propio como respaldo (fee bump desde una cuenta de Debro) |
| Regulación de tokens de consumo | En el MVP el token representa una botella pagada en fiat y no se comercia; la Fase 2 (P2P, preventas) requiere asesoría legal y KYC |
| Cambios de API en librerías jóvenes (passkey-kit) | Fijar versiones, envolver todo en `doc-wallet` |
