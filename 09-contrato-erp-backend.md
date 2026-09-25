# 09 · Contrato con el backend del ERP

Versión 1 · 25 de septiembre de 2026. Fuente de verdad: el **OpenAPI 3.0 del backend desplegado** (`GET https://136.243.223.39.sslip.io/docs-json`, 35 rutas, 38 esquemas, descargado el 25-09-2026), contrastado con los tres documentos que entregó el equipo de backend en `backend/` (`endpoints.md`, `guia_pruebas_manuales_trazabilidad.md`, `guia_pruebas_manuales_vino.md`). Donde el catálogo o las guías discrepan del OpenAPI, manda el OpenAPI (ver §8).

Los datos de prueba del ERP en `mocks/erp/` siguen exactamente estos DTO; se generan con `mocks/erp/generate.py`.

## 1. Lo esencial del backend

| Aspecto | Valor |
|---|---|
| Base URL | `https://136.243.223.39.sslip.io` (desarrollo). Swagger en `/docs`, JSON en `/docs-json` |
| Prefijo | `/v1` |
| Envoltorio de éxito | `{ success: true, statusCode, timestamp, path, data }` |
| Envoltorio de error | `{ success: false, statusCode, timestamp, path, error: { code, message, details } }` (verificado con un 404 real: `code: "NOT_FOUND"`) |
| Autenticación | JWT Bearer. `POST /v1/auth/login` → `{ user, tokens: { accessToken, refreshToken, tokenType: "Bearer", expiresIn: 604800 } }`. Renovación con `POST /v1/auth/refresh` |
| Multi-tenant | El token lleva la bodega activa (`wineryId`); las rutas `/my` y las listas del ERP se filtran solas por bodega. `PLATFORM_ADMIN` ve todo |
| Roles de usuario (`userRole`) | `PLATFORM_ADMIN`, `WINERY_ADMIN`, `ENOLOGIST`, `AGRONOMIST`, `CONSUMER`, `POS_OPERATOR` |
| Roles de miembro en la bodega (`memberRole`) | `OWNER`, `ENOLOGIST`, `AGRONOMIST`, `OPERATOR`, `ACCOUNTANT` |
| Paginación | `limit` y `offset` en las listas (no `page`). El OpenAPI no declara el esquema de respuesta de las listas: **por confirmar** si `data` es un array o `{ items, total }` (ver §8) |
| CORS | Permite orígenes `localhost` (verificado con `Origin: http://localhost:3002`) y credenciales; cabeceras `Content-Type, Authorization, X-Correlation-ID, Accept-Language` |
| Archivos | `POST /v1/uploads?folder=…` (multipart `file`, ≤ 15 MB, imágenes y PDF) devuelve `{ url, key, originalName, mimeType, sizeBytes }`; la `url` se pasa luego en los DTO |
| Billeteras | El backend crea una **billetera custodial** (`walletType: CUSTODIAL`, dirección `G…`) a cada usuario al registrarse y una institucional a cada bodega (`stellarPublicKey`, `onchainProducerId`) al aprobarla |
| Cadena | Al embotellar se calcula `blockchainDataHash` (SHA-256) y, cuando se ancla, `blockchainAnchorTxHash` / `isAnchoredOnChain` / `anchoredAt` |

## 2. Modelo real del ERP frente al modelo de los documentos

El backend **no tiene una entidad "Lote"** con máquina de estados. Modela la trazabilidad como una cadena de entidades, una por etapa, enlazadas por id:

```
Terroir ─► HarvestBatch ─► FermentationTank (+ logs, treatments) ─┬─► WineAgingBatch ──┐
                                                                  └─► ProductionBatch ─┴─► BottlingBatch ─► BatchLabAnalysis
                                                                                                   │
                                                                                     GET /traceability/public/:lotCode
```

| Concepto del documento maestro | Entidad del backend | Estado que la gobierna |
|---|---|---|
| Terroir / parcela | `Terroir` | `isActive`, `isDoEligible` |
| Cosecha (temporada) | No existe como entidad. `HarvestBatch.harvestYear` cumple el papel | — |
| Pesaje + análisis preliminar | `HarvestBatch` (pesaje bruto/tara/neto, Brix, pH, acidez, temperatura) | `phytosanitaryStatus`: `PENDING_INSPECTION → APPROVED \| REJECTED \| QUARANTINE` (`PATCH …/phyto-status`) |
| Tanque, bitácora, tratamientos | `FermentationTank` + `POST …/logs` + `POST …/treatments` | `status`: `FILLING → FERMENTING → COMPLETED → TRANSFERRED \| CLEANED` |
| Bifurcación (vino / singani) | `FermentationTank.destinationType`: `WINE_AGING` \| `SINGANI_DIST` (también `BEER_MATURATION`, `SPIRITS_DIST`, `OTHER`) | Se fija al crear el tanque; **no hay endpoint para cambiarla después** (ver §8) |
| Crianza y candado | `WineAgingBatch` con `plannedMonths` y `lockUntilDate` calculado | `agingStatus`: `AGING → READY → BOTTLED \| DISCARDED` |
| Destilación, cortes y reposo | `ProductionBatch` (`processType: SINGANI_DISTILLATION`, `additionalParams` con cabezas/corazón/colas) con `mandatoryRestUntil` = fin + 180 días | `restStatus`: `NOT_REQUIRED \| RESTING → READY → BOTTLED \| DISCARDED`; `GET …/rest-status` devuelve días transcurridos y restantes |
| Embotellado y QR | `BottlingBatch`: `internationalLotCode` (`{BODEGA}-{AÑO}-{TIPO}-{SEQ}`), `qrBatchUrl`, hash, `totalBottlesPackaged`, dilución | `isAnchoredOnChain` |
| Certificado de laboratorio | `BatchLabAnalysis` (ISO 17025 / SENASAG) | `conformsToSenasagStandards` |
| Visor del consumidor | `GET /v1/traceability/public/:lotCode` (público) | — |

**Decisión de frontend**: las pantallas del ERP siguen mostrando un "lote" con etapa y candado, como pide el documento maestro, pero esa fila es una **vista derivada en el cliente** (`LotView`) que se calcula uniendo la cadena anterior a partir del `HarvestBatch`. Está definida en `mocks/erp/lots-view.json` y la calcula `generate.py` (`lot_view`). Etapas de la vista: `pesaje → vendimia → fermentacion → bifurcacion → crianza | reposo → embotellado` más `rechazado`. Así ninguna pantalla depende de una entidad que el backend no tiene.

## 3. Mapa endpoint ↔ pantalla del ERP

| Pantalla (05 · 01-erp.html) | Lee | Escribe | Roles |
|---|---|---|---|
| 1.1 Login | — | `POST /v1/auth/login`, `POST /v1/auth/refresh` | todos |
| Perfil | `GET /v1/users/me` | `PATCH /v1/users/me` | todos |
| 1.2 Dashboard | `GET /v1/harvest-batches`, `GET /v1/fermentation-tanks?status=FERMENTING`, `GET /v1/wine-aging`, `GET /v1/production-batches?restStatus=RESTING`, `GET /v1/bottling` | — | miembros |
| Ajustes de bodega / miembros | `GET /v1/wineries/my`, `GET /v1/wineries/my/members` | `PATCH /v1/wineries/my`, `POST /v1/wineries/my/members/create`, `POST /v1/wineries/my/members` | `WINERY_ADMIN` |
| 2.1 Directorio de terroirs | `GET /v1/terroirs?varietyName=&isDoEligible=` | — | `WINERY_ADMIN`, `AGRONOMIST`, `ENOLOGIST` |
| 2.2 Ficha de terroir + badge D.O. | `GET /v1/terroirs/:id` (incluye lotes de vendimia históricos) | `PATCH /v1/terroirs/:id` | `WINERY_ADMIN`, `AGRONOMIST` |
| Alta de terroir | — | `POST /v1/terroirs` (`altitudeMasl`, `varietyName`, `isDoEligible`, `doType`, polígono GeoJSON opcional, `doCertificateUrl` tras `POST /v1/uploads?folder=certificates`) | `WINERY_ADMIN`, `AGRONOMIST` |
| 2.3 Slide-over "Nueva cosecha" | No existe en el backend | Se sustituye por el pesaje (3.1), que ya lleva `harvestYear` e `intakeDate` | — |
| 3.1 Pesaje | — | `POST /v1/harvest-batches` (`grossWeightKg`, `tareWeightKg`; el neto lo calcula el backend; Brix, pH y acidez son **obligatorios** en el mismo DTO) | `WINERY_ADMIN`, `AGRONOMIST`, `ENOLOGIST` |
| 3.2 Análisis y aprobar/rechazar | `GET /v1/harvest-batches/:id` | `PATCH /v1/harvest-batches/:id/phyto-status` (`APPROVED` / `REJECTED` / `QUARANTINE`, PDF de inspección opcional vía `uploads?folder=inspections`) | `AGRONOMIST`, `ENOLOGIST` |
| 4.1 Mapa de tanques | `GET /v1/fermentation-tanks?status=&destinationType=` | `POST /v1/fermentation-tanks` (crea el tanque **y** fija el destino) | `WINERY_ADMIN`, `ENOLOGIST` |
| 4.2 Bitácora | `GET /v1/fermentation-tanks/:id` (con lecturas y tratamientos) | `POST …/:id/logs` (temperatura obligatoria, densidad, pH), `POST …/:id/treatments` (tipo SENASAG, aditivo, dosis, código regulatorio) | logs: `WINERY_ADMIN`, `ENOLOGIST`, `POS_OPERATOR`; tratamientos: `ENOLOGIST`, `WINERY_ADMIN` |
| 4.3 Bifurcación | — | Se decide al crear el tanque (`destinationType`). La pantalla se reubica: el modal aparece **al llenar el tanque**, no al terminar la fermentación (ver §8) | `ENOLOGIST` |
| 5.1A Barricas y cuenta regresiva | `GET /v1/wine-aging`, `GET /v1/wine-aging/:id` (`lockUntilDate`, `agingStatus`) | `POST /v1/wine-aging` (`fermentationTankId`, `containerType`, material, código, ciclo, litros, `plannedMonths`, `startDate`) | `WINERY_ADMIN`, `ENOLOGIST` |
| 5.1B Cortes del alambique | `GET /v1/production-batches/:id` | `POST /v1/production-batches/distillation` (`additionalParams.headDiscardLiters / heartYieldLiters / tailDiscardLiters`, `initialAlcoholPercentage`, `isDoEligible`) | `WINERY_ADMIN`, `ENOLOGIST` |
| 5.2B Candado de reposo | `GET /v1/production-batches/:id/rest-status` (`daysElapsed`, `daysRemaining`, `isRestCompleted`) | — | miembros |
| 6.1 Embotellado | — | `POST /v1/bottling` (`wineAgingBatchId` **o** `productionBatchId`, `productType: WINE \| SINGANI`, `finalAlcoholAbv`, `waterDilutionLiters`, `totalBottlesPackaged`, `packagingFormatCl`, `labelDesignUrl` vía `uploads?folder=labels`); responde **422** si el candado no se cumplió | `WINERY_ADMIN`, `ENOLOGIST` |
| 6.2 Éxito y QR | `GET /v1/bottling/:id` (`internationalLotCode`, `qrBatchUrl`, `blockchainDataHash`, `isAnchoredOnChain`) | Los archivos de QR los genera el frontend a partir de `qrBatchUrl` (ver 06 §5) | miembros |
| Certificado de laboratorio | `GET /v1/lab-analyses/batch/:bottlingBatchId` | `POST /v1/lab-analyses` (PDF vía `uploads?folder=lab-reports`) | `WINERY_ADMIN`, `ENOLOGIST` |
| Trazabilidad del lote (línea de tiempo) | `GET /v1/traceability/dag/:bottlingBatchId` | — | miembros |
| Cuenta Stellar de la bodega | `GET /v1/wineries/my` (`stellarPublicKey`, `onchainProducerId`, `onchainRegisterTxHash`) | — | miembros |

Fuera del ERP pero en la misma API: `POST /v1/auth/signup` (consumidores y solicitantes de bodega), `POST /v1/wineries` (solicitud de alta), `GET /v1/wineries`, `GET /v1/wineries/pending`, `POST /v1/wineries/:id/approve` y `/reject` (Backoffice), `GET /v1/users/me/wallet`, `GET /v1/traceability/public/:lotCode` (visor del Marketplace).

## 4. Enumeraciones (copiar tal cual en los tipos)

| Campo | Valores |
|---|---|
| `userRole` | `PLATFORM_ADMIN` `WINERY_ADMIN` `ENOLOGIST` `AGRONOMIST` `CONSUMER` `POS_OPERATOR` |
| `memberRole` | `OWNER` `ENOLOGIST` `AGRONOMIST` `OPERATOR` `ACCOUNTANT` |
| `Winery.beverageCategory` | `WINERY` `BREWERY` `DISTILLERY` `OTHER` |
| `Winery.certificationStatus` | `PENDING` `ACTIVE` `SUSPENDED` `REVOKED` |
| `Wallet.walletType` / `walletPurpose` | `CUSTODIAL` `SELF_CUSTODY` / `CONSUMER_NFT` `PRODUCER_SIGNING` |
| `HarvestBatch.phytosanitaryStatus` | `PENDING_INSPECTION` `APPROVED` `REJECTED` `QUARANTINE` |
| `FermentationTank.destinationType` | `WINE_AGING` `SINGANI_DIST` `BEER_MATURATION` `SPIRITS_DIST` `OTHER` |
| `FermentationTank.status` | `FILLING` `FERMENTING` `COMPLETED` `TRANSFERRED` `CLEANED` |
| `EnologicalTreatment.treatmentType` | `ACIDITY_CORRECTION` `SO2_ADDITION` `CLARIFICATION` `FILTRATION_AID` `NUTRIENT_ADDITION` `ENZYME_ADDITION` `OAK_CHIPS` `FINING_AGENT` `STABILIZATION` `OTHER` |
| `WineAgingBatch.agingStatus` | `AGING` `READY` `BOTTLED` `DISCARDED` |
| `ProductionBatch.processType` | `SINGANI_DISTILLATION` `SPIRITS_DISTILLATION` `BEER_MATURATION` `SECONDARY_FERMENTATION` `OTHER` |
| `ProductionBatch.restStatus` | `NOT_REQUIRED` `RESTING` `READY` `BOTTLED` `DISCARDED` |
| `BottlingBatch.productType` | `WINE` `SINGANI` `BEER` `SPIRITS` `CIDER` `MEAD` `OTHER` |

Reglas que el backend aplica y el cliente debe anticipar (validación en pantalla antes de enviar): altitud ≥ 1.600 m y `isDoEligible` para destilación D.O.; 180 días de reposo desde `processEndDate` antes de embotellar singani; `lockUntilDate` antes de embotellar vino (422 en caso contrario); Brix, pH y acidez obligatorios al registrar el pesaje.

## 5. Datos de prueba del ERP (`mocks/erp/`)

| Archivo | DTO | Contenido |
|---|---|---|
| `wineries.json` | `WineryResponseDto` con `members` | 4 bodegas: Altos de Calamuchita (WINERY, ACTIVE), Cinti Viejo (DISTILLERY, ACTIVE, exportadora), Viñedos del Guadalquivir (PENDING), Casa Uriondo (SUSPENDED) |
| `users.json` | `UserProfileResponseDto` + `_mock.password` | 14 usuarios: 2 gestores, dueño/enóloga/agrónomo/operario por bodega activa, 1 solicitante, 2 consumidores, 1 cajero |
| `wallets.json` | `WalletResponseDto` | Una billetera custodial por usuario (`PRODUCER_SIGNING` para miembros, `CONSUMER_NFT` para consumidores) |
| `auth-login.json` | `AuthResponseDto` por usuario | Respuesta de login con tokens de prueba (`mock.access.<clave>`) |
| `terroirs.json` | `TerroirResponseDto` | 11 parcelas; una no apta D.O. por altitud (El Portillo, 1.540 m) |
| `harvest-batches.json` | `HarvestBatchResponseDto` | 12 lotes de vendimia: aprobados, pendiente de inspección, rechazado, en cuarentena, pesaje de hoy sin laboratorio |
| `fermentation-tanks.json` | `FermentationTankResponseDto` | 14 tanques en todos los estados; uno fermentando con temperatura alta (alerta) |
| `fermentation-logs.json` | lectura de `POST …/logs` | 228 lecturas diarias coherentes (densidad decreciente) |
| `enological-treatments.json` | `CreateEnologicalTreatmentDto` + id | 16 tratamientos (SO₂, nutrientes) |
| `wine-aging.json` | `WineAgingResponseDto` | 4 crianzas: dos bloqueadas (38 y 156 días), una liberada, una embotellada |
| `production-batches.json` / `production-rest-status.json` | `ProductionBatchResponseDto` / respuesta de `rest-status` | 5 destilaciones: dos en reposo (18 y 168 días), una lista, dos embotelladas |
| `bottling.json` | `BottlingBatchResponseDto` | 4 embotellados (`ALT-2026-WINE-001`, `CVJ-2026-SINGANI-001`, …); uno sin anclar en cadena |
| `lab-analyses.json` | `BatchLabAnalysisResponseDto` | 3 certificados (un embotellado sin certificado todavía) |
| `traceability-public.json` | respuesta de `GET /traceability/public/:lotCode` | Un pasaporte por embotellado, indexado por código de lote |
| `lots-view.json` | vista derivada `LotView` (solo cliente) | 12 filas con etapa, tipo, candado y enlaces a cada entidad |

Reglas de coherencia: los ids son UUID v5 deterministas; las fechas son relativas al 25-09-2026; cada tanque `TRANSFERRED` tiene una crianza o una destilación; cada embotellado apunta a una sola fuente; `netWeightKg = grossWeightKg − tareWeightKg`; los códigos de lote siguen `{BODEGA}-{AÑO}-{TIPO}-{SEQ}`; las URL de archivos apuntan a `/mocks/uploads/...` (servidas por la app en desarrollo). Credenciales de demo: cualquier usuario de `users.json` con contraseña `demo1234`.

Handlers MSW del ERP (a escribir en `doc-mocks`): imitan las 35 rutas con el envoltorio de §1, filtran por el `wineryId` del token de prueba, aplican `limit`/`offset` y devuelven los 422/404 documentados.

## 6. Capa de acceso a datos en `doc-erp-web`

- `src/lib/api/client.ts`: `fetch` con base URL (`NEXT_PUBLIC_API_URL`), Bearer, refresco automático con `POST /v1/auth/refresh` al recibir 401, cabecera `Accept-Language: es`, desempaquetado de `data` y conversión del `error` a una excepción tipada.
- `src/lib/api/schemas/*.ts`: esquemas zod generados desde el OpenAPI (herramienta candidata: `openapi-zod-client` u `orval` con salida zod) para validar respuestas en desarrollo.
- `src/lib/api/erp/*.ts`: un módulo por recurso (`terroirs`, `harvestBatches`, `tanks`, `aging`, `production`, `bottling`, `lab`, `traceability`, `winery`) que expone hooks (`useTerroirs`, `useLotViews`, …).
- `src/lib/api/lot-view.ts`: la función que deriva `LotView` de la cadena (puerto a TypeScript de `lot_view` en `generate.py`).
- Sesión: tokens en memoria + `refreshToken` en cookie `HttpOnly` si el backend lo permite; si no, `sessionStorage` con expiración.

## 7. Cómo se integra (Etapa 1G y Etapa 5)

1. Apagar MSW y apuntar `NEXT_PUBLIC_API_URL` a `https://136.243.223.39.sslip.io`.
2. Login con un usuario de bodega real creado por el backend (ver 10 §2).
3. Los adaptadores no deberían cambiar: los fixtures ya tienen la forma real. Lo único pendiente de confirmar es el envoltorio de las listas (§8).

## 8. Puntos de alineación con el backend (para llevar a la reunión)

| # | Tema | Qué dice hoy el backend | Qué necesita el frontend / propuesta |
|---|---|---|---|
| 1 | **Esquema de las listas** | `GET` de colecciones sin esquema de respuesta declarado; `limit`/`offset` | Confirmar si `data` es `T[]` o `{ items: T[], total, limit, offset }`. Los mocks asumen `{ items, total, limit, offset }` hasta que se confirme |
| 2 | **URL del QR** | `qrBatchUrl` fija `https://drinksonchain.com/trace/batch/{lotCode}` | Debe ser configurable por entorno y apuntar a `app.{dominio}/b/{lotCode}` (plan 02). Mientras tanto el Marketplace también responderá en `/trace/batch/:lotCode` |
| 3 | **Bifurcación** | El destino se fija al crear el tanque (`destinationType`); no hay `PATCH` de tanque | O se añade `PATCH /v1/fermentation-tanks/:id` (destino y estado), o el ERP muestra la decisión al crear el tanque. El frontend se construye con la segunda opción y cambia si aparece el endpoint |
| 4 | **Estado del tanque** | `status` solo se envía al crear | ¿Cómo pasa un tanque a `COMPLETED` o `TRANSFERRED`? ¿Lo hace el backend al crear crianza/destilación? Confirmar |
| 5 | **Cosecha como entidad** | No existe (`harvestYear` en el pesaje) | Aceptado: la pantalla 2.3 desaparece y se funde con el pesaje |
| 6 | **Análisis en el pesaje** | Brix, pH y acidez son obligatorios en `POST /v1/harvest-batches` | El documento maestro separa pesaje (operario) y análisis (enólogo). Propuesta: hacerlos opcionales en el alta y editables en `phyto-status`, o aceptar que el operario los capture. Hasta entonces el ERP pide los tres valores en la pantalla de pesaje |
| 7 | **Discrepancias catálogo ↔ OpenAPI** | `endpoints.md` usa `legalBusinessName`, `tankIdentifier`, `productType: VINO`, `woodType`, `phValue` en el pesaje; el OpenAPI usa `legalName`, `tankCode`, `WINE`, `containerMaterial`, `initialPh` | Actualizar `endpoints.md` (los mocks siguen el OpenAPI) |
| 8 | **Alta de bodega** | La bodega se autoinscribe (`signup` + `POST /v1/wineries`) y el gestor aprueba | Encaja con el sitio de bodegas (`/unirse` puede ser este flujo real) y con el Backoffice (`pending` + `approve`). Se documenta en 02 y 03 en la próxima revisión |
| 9 | **Billeteras** | Custodial por defecto para todos (`walletType: CUSTODIAL`) | Coincide con la opción B de 04. La smart account con passkey (`SELF_CUSTODY`) se añadiría después; pedir que `POST /wallets` acepte registrar una dirección `C…` |
| 10 | **Archivos de QR** | Solo `qrBatchUrl`, un código por lote | El ERP genera los PNG/SVG en el cliente (una imagen por lote; si se quiere una por botella, hace falta un endpoint de códigos individuales) |
| 11 | **`GET /traceability/public`** | Marcado con candado en Swagger pero el catálogo lo declara público; responde 404 sin token (no 401) | Confirmar que es público en producción |
| 12 | **Usuarios de prueba** | El servidor está vacío (los códigos de ejemplo de las guías devuelven 404) | Pedir que el backend cargue un juego de datos de prueba o nos dé un usuario `PLATFORM_ADMIN` para ejecutar las guías (ver 10) |
| 13 | **Bodega activa** | El token lleva una sola `wineryId`; no hay endpoint para cambiarla | Un usuario con varias membresías no puede elegir bodega en el ERP. Proponer `POST /v1/auth/switch-winery` o `wineryId` en el refresh |
| 14 | **Recuperar contraseña** | No existe | Proponer `POST /v1/auth/forgot-password` y `/reset-password`. Hoy el ERP muestra un aviso para pedirlo a la administración de la bodega |
| 15 | **Numeración del código de lote** | Los mocks usan un contador por bodega y año para todos los productos (tras `CVJ-2026-WINE-003` viene `CVJ-2026-SINGANI-004`) | Confirmar si el `SEQ` es por producto o global |
| 16 | **Crianza sin fecha de inicio** | `WineAgingResponseDto` no devuelve `startDate` | El candado se dibuja con `lockUntilDate − plannedMonths`; pedir `startDate` en la respuesta |
| 17 | **Permisos del agrónomo** | Puede listar crianzas y destilaciones pero recibe 403 en su detalle | Confirmar si es intencional |
| 18 | **`phyto-status` pisa las notas** | `PATCH …/phyto-status` sobrescribe `notes` del lote | Separar notas del dictamen o concatenarlas; exponer el nombre de quien dictamina, no solo `certifiedByMemberId` |
| 19 | **Errores de validación** | `details` llega como cadenas ("campo: mensaje") | Proponer `details: [{ field, message }]` para marcar el campo exacto |
| 20 | **Activos por lote y códigos por botella** | No se exponen saldos de tokens ni códigos individuales | La cuenta Stellar del ERP y la exportación de QR por botella quedan provisionales hasta tenerlos |
