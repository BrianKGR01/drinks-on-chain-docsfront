# 11 · Billeteras y tokens: qué necesita el frontend del backend

Versión 1 · 25 de septiembre de 2026. Documento para el equipo de backend. Resume, desde el lado del cliente, qué hay que construir en el servidor para las billeteras y los tokens de las botellas, en qué orden y con qué contrato de API. El razonamiento completo (opciones, costes, librerías) está en `04-billeteras-stellar.md`; aquí va solo lo accionable.

## 1. Resumen

1. **El ERP no toca billeteras.** Solo muestra, en solo lectura, la cuenta institucional de la bodega y el anclaje del hash de cada embotellado. No firma nada ni guarda claves.
2. **La única billetera de usuario final vive en el Marketplace** (Etapa 2 del roadmap). El consumidor no ve criptografía: compra en bolivianos y recibe tokens (1 token = 1 botella) en su "cava".
3. **Toda firma institucional ocurre en el backend** (emisión, transferencias, quema al retirar), con claves en KMS/HSM. Ninguna clave privada llega nunca a un navegador.
4. **Propuesta de token para el MVP**: un activo clásico de Stellar por lote, con `AUTH_CLAWBACK_ENABLED` para quemar al entregar la botella (§4).
5. **Hoy el backend ya crea billeteras custodiales** (`walletType: CUSTODIAL`, `G…`) para cada usuario y una cuenta institucional por bodega al aprobarla. Eso sirve como punto de partida; lo que falta es emisión, transferencia, quema y lectura de saldos.

## 2. Qué existe hoy en el backend y qué usa el frontend

| Dato del backend (OpenAPI 25-09) | Dónde lo usa el frontend | Estado |
|---|---|---|
| `Winery.stellarPublicKey`, `onchainProducerId`, `onchainRegisterTxHash` | ERP · Cuenta Stellar (solo lectura, enlace a stellar.expert) | En uso con mocks |
| `BottlingBatch.blockchainDataHash`, `isAnchoredOnChain`, `blockchainAnchorTxHash`, `anchoredAt` | ERP · Envasado (hash con copia, estado de anclaje, enlace a la transacción) | En uso con mocks |
| `User.primaryWallet` (`GET /v1/users/me`, `GET /v1/users/me/wallet`) | ERP · Perfil (dirección custodial del usuario, solo lectura) | En uso con mocks |
| `GET /v1/traceability/public/:lotCode` (`blockchainIntegrity`) | Marketplace · visor del QR | Etapa 2 |

Nada más del ERP depende de Stellar. Los saldos por lote (emitidas, en circulación, quemadas) aparecen en la pantalla de la cuenta como "disponible cuando el backend exponga los activos" (09 §8, punto 20).

## 3. Decisiones que necesitamos de backend

| # | Decisión | Recomendación del frontend | Por qué |
|---|---|---|---|
| D1 | Billetera del consumidor: custodial `G…` (lo que ya existe) o smart account `C…` con passkey | Empezar con la **custodial que ya existe** y añadir la smart account con passkey como mejora de la Etapa 2 (`walletType: SELF_CUSTODY`) | La custodial funciona hoy y no cambia la experiencia; la passkey quita custodia al proyecto, pero exige relayer. Las dos comparten los mismos endpoints de lectura |
| D2 | Modelo de token | **Activo clásico por lote** con `AUTH_REVOCABLE` + `AUTH_CLAWBACK_ENABLED`; alternativa: contrato SEP-41 | Sin desarrollar ni auditar contratos; la quema por clawback es nativa; visible en exploradores |
| D3 | Emisor | **La cuenta institucional de cada bodega** (ya se crea al aprobarla); alternativa: un emisor de plataforma | La bodega aparece como origen del activo, que es el mensaje del producto |
| D4 | Quién paga comisiones | Backend (cuentas custodiales financiadas; o relayer propio si se adopta la passkey) | El consumidor no tiene XLM |
| D5 | Relayer (solo si D1 = passkey) | **OpenZeppelin Relayer autoalojado**; Launchtube ya no existe | Código abierto, sin coste por uso |
| D6 | Indexador de saldos e historial | API propia que lea Horizon/RPC (o Mercury) y guarde caché | El cliente no consulta la red directamente salvo para "verificar en el explorador" |
| D7 | Red | **Testnet** hasta la Etapa 5; mainnet al integrar | Friendbot fondea gratis |

## 4. Modelo de token propuesto (D2)

- **Un activo por lote embotellado.** Código de hasta 12 caracteres derivado del `internationalLotCode` (por ejemplo `CVJ26SGR001`). Emisor: la cuenta de la bodega (D3).
- **1 unidad = 1 botella.** El activo clásico tiene 7 decimales; el backend emite y transfiere siempre enteros y el frontend muestra enteros.
- **Flags del emisor**: `AUTH_REVOCABLE` y `AUTH_CLAWBACK_ENABLED` (se fijan antes de emitir; no se pueden añadir después a trustlines existentes).
- **Metadatos**: `[[CURRENCIES]]` en el `stellar.toml` del dominio (nombre, imagen, emisor, descripción) y el vínculo lote ↔ activo en la base de datos.
- **Cuentas `G…` custodiales**: necesitan trustline al activo antes de recibirlo (la crea el backend, que controla la cuenta). Con smart accounts `C…` no hace falta trustline (saldo vía SAC).
- **Quema al retirar**: `clawback` desde el emisor por la cantidad entregada. Deja rastro público y reduce la circulación.

## 5. Trabajo del backend por sistema, en el orden en que lo necesitamos

| Etapa del frontend | Sistema | Qué necesita del backend |
|---|---|---|
| 1 (ahora) | ERP | Nada nuevo para funcionar. Deseable: `GET /v1/wineries/my/assets` con los activos por lote (código, emitidas, en circulación, quemadas, última tx) para completar la pantalla de la cuenta |
| 2 | Marketplace | Colecciones publicadas con su activo; compra que, al confirmarse el pago, transfiere los tokens a la billetera del comprador; lectura de la cava (`holdings`); estado de transacciones; pase de retiro |
| 3 | POS | Validar el pase y confirmar la entrega, que ejecuta el clawback |
| 4 | Backoffice | Emitir (mint) el activo de un lote aprobado y publicarlo como colección; ver estado de emisión; soporte (anular pase, autorizar entrega manual) |

## 6. Endpoints propuestos (borrador de contrato)

Mismo envoltorio `{ success, data | error }` y JWT del backend actual. Rutas orientativas; los nombres finales los decide backend.

| Método y ruta | Quién llama | Cuerpo → respuesta |
|---|---|---|
| `GET /v1/wineries/my/assets` | ERP | → `[{ lotCode, assetCode, issuer, issued, circulating, burned, lastTxHash, lastTxStatus }]` |
| `POST /v1/collections/:lotCode/mint` | Backoffice | `{ price, currency: "BOB", quantity }` → `{ assetCode, issuer, txHash, status }` |
| `GET /v1/me/holdings` | Marketplace | → `[{ assetCode, issuer, lotCode, quantity, productName, imageUrl }]` |
| `GET /v1/tx/:hash` | Todos | → `{ hash, status: PENDING \| SUBMITTED \| CONFIRMED \| FAILED, ledger, createdAt, explorerUrl }` |
| `POST /v1/wallets` | Marketplace (solo si D1 = passkey) | `{ address: "C…", kind: "passkey", credentialId }` → `WalletResponseDto` |
| `POST /v1/claims` | Marketplace | `{ assetCode, quantity, pickupPointId }` → `{ claimId, code, expiresAt, status }` (reserva las unidades) |
| `POST /v1/claims/:code/validate` | POS | `{ pickupPointId, deviceId }` → `{ valid, reason?, product, quantity, customerName }` |
| `POST /v1/claims/:code/confirm` | POS | `{ deviceId }` → `{ deliveryId, txHash, status }` (dispara el clawback) |
| `POST /v1/claims/:code/void` | Backoffice | `{ reason }` → `{ status }` |

Motivos de rechazo que el POS necesita distinguir: `ALREADY_REDEEMED`, `EXPIRED`, `WRONG_PICKUP_POINT`, `VOIDED`, `NOT_FOUND`.

## 7. Flujos

**Compra (Marketplace)**: pago aprobado por la pasarela → backend crea la trustline si hace falta (custodial) → transfiere `quantity` unidades desde la cuenta de distribución o el emisor a la billetera del comprador → guarda `txHash` → el cliente consulta `GET /v1/tx/:hash` y `GET /v1/me/holdings`.

**Retiro (Marketplace → POS)**: el consumidor genera un pase (`POST /v1/claims`, caduca en días, solo en puntos habilitados para el lote) → el POS escanea y valida → el cajero confirma → backend hace `clawback` de las unidades y devuelve `txHash` → el pase queda `REDEEMED` y el POS muestra "Confirmado en la red" cuando la tx pasa a `CONFIRMED`.

## 8. Seguridad

- Claves de emisor, distribución y cuentas custodiales en KMS/HSM; firma solo en el servidor.
- El frontend nunca recibe XDR para firmar con claves institucionales. Con passkey (D1), el usuario solo firma autorizaciones de su propia cuenta.
- Endpoints de emisión y quema restringidos por rol (`PLATFORM_ADMIN`, `POS_OPERATOR` del punto habilitado) y con idempotencia (`Idempotency-Key`) para no duplicar transferencias si el cliente reintenta.
- Registro de auditoría de cada operación en cadena con el usuario que la originó.

## 9. Qué hará el frontend y cuándo

- **Etapa 2 (Marketplace)**: paquete `@drinks-on-chain/wallet` con modos `mock`, `gestionada` (custodial, solo lecturas por la API) y `passkey` (smart account en testnet). Hasta que existan los endpoints, todo corre contra mocks con estas mismas formas.
- **Etapa 0.4 (spike)**: prueba en testnet de smart account con passkey, recepción de un activo con clawback y quema desde el emisor, para confirmar tiempos y compatibilidad de dispositivos antes de que backend decida D1.

## 10. Preguntas para la reunión

1. ¿Confirmáis la billetera custodial como punto de partida del consumidor (D1)?
2. ¿Emisor por bodega o de plataforma (D3)? ¿Hay ya código de emisión en el backend?
3. ¿Cómo se ancla hoy `blockchainDataHash` (qué operación, qué cuenta firma)?
4. ¿Qué indexador vais a usar para saldos e historial (D6)?
5. ¿Plazo estimado para `GET /v1/wineries/my/assets` y para los endpoints de claims?
