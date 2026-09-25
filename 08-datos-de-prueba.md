# 08 · Datos de prueba y contrato con el backend

Versión 1 · 25 de septiembre de 2026. Define cómo se construye todo el frontend sin backend: un único juego de datos de prueba en JSON, coherente entre los seis sitios y aplicaciones, tipado, generado por script y servido por un interceptor HTTP en desarrollo. Cuando llegue cada backend, se cambia el origen de datos, no las pantallas.

> **Pendiente**: la documentación de los endpoints del backend del ERP (la única que existe hoy) todavía no se ha incorporado. Cuando se comparta, la sección 7 se completa con el mapeo endpoint ↔ pantalla y los tipos se ajustan a sus DTO antes de escribir los fixtures del ERP.

## 1. Principios

1. **Una sola fuente**: los datos viven en el repo `doc-mocks` (paquete `@doc/mocks`). Las bodegas del sitio de bodegas son las del ERP y del Backoffice; los lotes del ERP son las colecciones del Marketplace; los pases del Marketplace aparecen en el historial del POS.
2. **JSON legible y versionado**: los fixtures son archivos `.json` que cualquiera puede abrir y editar; no se generan al azar en cada arranque.
3. **Generación determinista**: un script (`pnpm seed`) produce los JSON a partir de un catálogo pequeño escrito a mano (bodegas, terroirs, productos) y de una semilla fija; volver a ejecutarlo da el mismo resultado. Así se regeneran cuando cambia el modelo.
4. **Tipado con esquemas**: cada entidad tiene un esquema `zod` del que se derivan el tipo TypeScript y la validación de los JSON en CI. Un fixture inválido rompe la build.
5. **La app no sabe que son mocks**: las pantallas llaman a un cliente HTTP (`fetch` a `/api/...`); en desarrollo y en las demos, **MSW** (Mock Service Worker) intercepta esas llamadas y responde con los fixtures, con latencias y errores simulables. En producción con backend, MSW no se carga.
6. **Estados desde el día uno**: cada handler puede devolver vacío, error 4xx/5xx, lento o sin conexión mediante un panel de desarrollo (`?mock=empty|error|slow`), para diseñar esos estados sin trucos.

## 2. Estructura del repo `doc-mocks`

```
doc-mocks/
  catalog/                 # escrito a mano, la "verdad" de la red de prueba
    wineries.json          # 4 bodegas (2 socias, 1 en conversación, 1 referencia)
    terroirs.json          # 10 terroirs con altitud, cepa, geolocalización, aptitud D.O.
    products.json          # 8 productos (5 vinos, 3 singanis) con notas de cata e imágenes
    pickup-points.json     # 5 puntos de recojo (2 bodegas propias, 3 licorerías)
    people.json            # usuarios de prueba por rol, con credenciales de demo
  schemas/                 # zod: una entidad por archivo, exporta schema + type
  seed/                    # generador determinista (seed fija) -> fixtures/
  fixtures/                # JSON generados: lots, vessels, logs, bottlings, collections,
                           # orders, holdings, wallets, claims, devices, shifts, deliveries,
                           # tickets, reviews, txs, events
  handlers/                # MSW por dominio: erp.ts, marketplace.ts, backoffice.ts, pos.ts, shared.ts
  scenarios/               # variantes: empty, error, slow, offline, "dia-de-vendimia", "lote-listo"
  src/index.ts             # exporta schemas, types, fixtures, handlers, scenarios
  package.json             # @doc/mocks
```

Publicación en GitHub Packages; cada app la instala como dependencia de desarrollo y ejecuta `pnpm mocks:sync` en su `postinstall` para copiar los JSON a `public/mocks/` (legibles con `fetch` sin importar nada) y registrar los handlers. Hasta que exista el paquete, la plantilla de aplicación incluye una copia de `fixtures/` y `handlers/` que se sustituye por la dependencia en cuanto se publique.

## 3. Catálogo de la red de prueba

Los nombres son ficticios pero verosímiles y coherentes con los valles reales de las landings (Valle Central de Tarija, Valle de Cinti). Ninguno corresponde a una bodega real con la que exista acuerdo.

| Bodega | Valle | Estado en la red | Productos | Puntos de recojo |
|---|---|---|---|---|
| Bodega Altos de Calamuchita | Tarija (Santa Ana) | Socia | Tannat Reserva 2024, Moscatel Blanco 2025 | Propia + Licorería La Cava (Tarija) |
| Destilería Cinti Viejo | Cinti (Camargo) | Socia | Singani Gran Reserva 2026, Singani Clásico 2025, Vino Patrimonial 2024 | Propia + Vinoteca Sur (La Paz) + Bodega & Cava Equipetrol (Santa Cruz) |
| Viñedos del Guadalquivir | Tarija (Concepción) | En conversación | Syrah 2024, Blend de Altura 2023 | — |
| Casa Uriondo (referencia) | Tarija (Uriondo) | Referencia | Singani de Altura 2025 | — |

Usuarios de prueba (contraseña de demo `demo1234`, PIN de POS `1234`): `enologa@altos.test`, `operario@altos.test`, `admin@altos.test`, `enologo@cintiviejo.test`, `gestor@drinksonchain.test`, `soporte@drinksonchain.test`, `maria@tribu.test` (miembro con 3 activos), `carlos@tribu.test` (miembro con pase activo), cajero "Juan" en Licorería La Cava.

## 4. Fixtures generados (cantidades y reglas)

| Fixture | Cantidad | Reglas de coherencia |
|---|---|---|
| `lots` | 12 | Al menos uno en cada estado de la máquina (`origen`, `vendimia`, `fermentacion`, `crianza`, `destilacion`, `reposo`, `embotellado`, `listo`, `tokenizado`); 7 vinos y 5 singanis; los singanis solo salen de terroirs aptos para D.O. |
| `vessels` | 14 | 8 tanques (vacío, lleno, fermentando con temperatura), 4 barricas con meses y fecha fin, 2 alambiques |
| `logs` | ~180 | Registros diarios de temperatura y densidad por tanque en fermentación; cortes por destilación |
| `bottlings` | 4 | Solo para lotes `embotellado`/`listo`/`tokenizado`; botellas llenadas, litros, agua añadida (singani), archivo QR |
| `collections` | 4 | Una por lote `tokenizado` (3) y una `pendiente` (lote `listo`); precio fijo en BOB, botellas totales / vendidas / retiradas, código de activo y emisor de testnet |
| `wallets` | 3 | Una por miembro (`passkey`, `gestionada`, `passkey` con firmante de recuperación); direcciones `C…` de testnet |
| `orders` | 9 | Pagadas, pendientes, fallidas; método tarjeta / QR bancario |
| `holdings` | 6 | Coherentes con `orders` pagadas menos `deliveries` |
| `claims` | 5 | Activo, usado, caducado, anulado por soporte, con punto de recojo elegido y caducidad en días |
| `devices` | 4 | Vinculados a puntos de recojo; uno pendiente de vinculación |
| `shifts` / `deliveries` | 3 / 7 | Las entregas referencian claims `usado` y aparecen en el turno del cajero |
| `tickets` | 6 | Abierto, en proceso, resuelto; uno enlazado a un claim fallido |
| `reviews` | 10 | Estrellas y comentario por producto |
| `txs` | 12 | Emisiones, transferencias y clawbacks con hash de testnet y estado |
| `events` | 20 | `lot.ready`, `mint.confirmed`, `order.paid`, `claim.confirmed`, `device.enrolled` |

Identificadores legibles y estables: `win_altos`, `ter_altos_01`, `lot_2026_sgr_01`, `col_sgr26`, `usr_maria`, `clm_0005`, `pp_lacava`, `dev_lacava_01`. Fechas relativas a una fecha de referencia fija (`2026-09-25`) para que los candados y caducidades sean reproducibles.

## 5. Ejemplo de fixture (lote)

```json
{
  "id": "lot_2026_sgr_01",
  "wineryId": "win_cintiviejo",
  "terroirId": "ter_cinti_02",
  "kind": "singani",
  "name": "Singani Gran Reserva 2026",
  "status": "reposo",
  "history": [
    { "status": "origen", "at": "2026-01-12", "by": "usr_enologo_cinti" },
    { "status": "vendimia", "at": "2026-03-04", "by": "usr_operario_cinti", "data": { "kg": 18400 } },
    { "status": "fermentacion", "at": "2026-03-06", "by": "usr_operario_cinti", "data": { "vesselId": "ves_tank_03" } },
    { "status": "destilacion", "at": "2026-04-15", "by": "usr_enologo_cinti", "data": { "headsL": 120, "heartL": 1500, "tailsL": 210, "abv": 60 } },
    { "status": "reposo", "at": "2026-04-16", "by": "usr_enologo_cinti", "data": { "minDays": 180, "unlockAt": "2026-10-13" } }
  ],
  "lock": { "kind": "reposo", "unlockAt": "2026-10-13", "reason": "Gran Reserva: mínimo 6 meses" },
  "lab": { "brix": 23.4, "ph": 3.4, "acidity": 5.9, "approved": true },
  "yield": { "kg": 18400, "baseWineL": 12100, "heartL": 1500 },
  "bottlingId": null,
  "collectionId": null
}
```

## 6. Uso en cada aplicación

| Aplicación | Fixtures que consume | Handlers | Escenarios clave |
|---|---|---|---|
| Sitio de bodegas | `wineries`, `terroirs`, `pickup-points`, `products`, `lots` (trazabilidad pública) | `shared` | red completa; bodega sin lotes |
| Landing principal | `wineries`, `products`, `collections` (precio y estado) | `shared` | — |
| ERP | `lots`, `vessels`, `logs`, `bottlings`, `terroirs`, `people`, `wallets` (cuenta de la bodega), `txs` | `erp` | día de vendimia; lote bloqueado; lote listo; sin conexión en tablet |
| Marketplace | `collections`, `products`, `wineries`, `orders`, `holdings`, `wallets`, `claims`, `pickup-points`, `reviews`, `tickets` | `marketplace` | sin cuenta; primera compra (crea cuenta y billetera); pago fallido; pase caducado; escaneo de código inválido |
| Backoffice | todo | `backoffice` | alta de bodega (crea cuenta Stellar); lote listo por emitir; emisión fallida; ticket con disputa |
| POS | `claims`, `collections`, `pickup-points`, `devices`, `shifts`, `deliveries` | `pos` | pase válido; ya canjeado; caducado; sucursal incorrecta; sin conexión (cola) |

Cada app tiene una página de desarrollo `/__mocks` (solo en desarrollo) para cambiar de escenario y usuario sin tocar código. En las demos públicas sin backend se despliega con `NEXT_PUBLIC_MOCKS=1`.

## 7. Contrato con el backend

Capa de acceso a datos por aplicación en `src/lib/api/`: un cliente HTTP tipado (`fetch` + zod en las respuestas) y un adaptador por recurso que convierte los DTO del backend a los tipos de `@doc/mocks`. Las pantallas importan solo los tipos y los hooks (`useLots()`, `useCollection(id)`), nunca URLs. Cambiar de mocks a backend real es: (1) apagar MSW, (2) apuntar `NEXT_PUBLIC_API_URL`, (3) ajustar los adaptadores donde los DTO difieran.

### 7.1 ERP (backend existente)
Por completar con la documentación de endpoints que comparta el equipo de backend. Al recibirla:
1. Listar endpoints y agruparlos por pantalla del ERP.
2. Comparar sus DTO con los esquemas de `@doc/mocks` (lote, terroir, cosecha, tanque, registro, embotellado) y ajustar los esquemas **antes** de generar fixtures del ERP, para no rehacer.
3. Anotar autenticación (tipo de token, refresco, roles), paginación, formatos de fecha y errores.
4. Escribir los handlers MSW del ERP imitando exactamente esas rutas y respuestas, de modo que la integración sea solo cambiar la URL base.

### 7.2 Resto de sistemas (backend por construir)
Los handlers de `marketplace`, `backoffice` y `pos` proponen rutas REST convencionales (`GET /collections`, `POST /orders`, `GET /me/holdings`, `POST /claims`, `POST /claims/:id/validate`, `POST /claims/:id/confirm`, `POST /wineries`, `POST /collections/:id/mint`, `GET /tx/:hash`) que se comparten con backend como borrador del contrato. Ver `04-billeteras-stellar.md` §10 para lo específico de billeteras y tokens.

### 7.3 Pasarela de pago
El checkout del Marketplace usa un adaptador `PaymentProvider` con implementación `mock` (aprueba, rechaza o demora según escenario). Cuando el banco entregue su documentación, se añade la implementación real sin tocar la pantalla; ver `06-decisiones-y-preguntas.md` §3.
