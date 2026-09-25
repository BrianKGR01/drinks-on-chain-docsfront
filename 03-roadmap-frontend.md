# 03 · Roadmap del frontend, paso a paso por sistema

Versión 2 · 25 de septiembre de 2026 (v1 en `antiguo/`). Alcance: solo cliente. Cada sistema se construye completo contra `@doc/mocks` con `@doc/ui`, de modo que integrar el backend sea cambiar el origen de datos. Estimaciones para **una persona de frontend a tiempo completo**; con dos, los sistemas se solapan como indica §9.

**Cómo se sigue el avance**: cada etapa tiene, debajo de su tabla, una lista **Avance** con casillas (`- [ ]` pendiente, `- [x]` hecho). Se marca una casilla cuando el paso cumple su columna "Terminado cuando" y el trabajo está en `dev`; al lado se anota la fecha y, si aplica, el PR. Cada repositorio lleva además su propio `docs/ROADMAP.md` con el detalle fino de sus tareas.

## Tablero resumido

- [ ] Etapa 0 · Fundaciones
- [ ] Sistema 0 · Sitios públicos (landing principal y sitio de bodegas)
- [ ] Etapa 1 · S1 ERP
- [ ] Etapa 2 · S2 Marketplace
- [ ] Etapa 3 · S4 POS
- [ ] Etapa 4 · S3 Backoffice
- [ ] Etapa 5 · Integración y salida

## 1. Orden y motivo

```
Etapa 0 · Fundaciones ─────────────┐
Sistema 0 · Sitios públicos (pendientes, en paralelo, pequeño)
                                   ▼
Etapa 1 · S1 ERP ──► Etapa 2 · S2 Marketplace ──► Etapa 3 · S4 POS ──► Etapa 4 · S3 Backoffice ──► Etapa 5 · Integración
```

- **ERP primero**: es el único sistema con backend existente, así que su integración puede empezar antes y validar la capa de acceso a datos y la autenticación reales.
- **Marketplace segundo**: es la cara del producto, el único con billetera y el que más decisiones de diseño concentra; el spike de Stellar de la Etapa 0 se estrena aquí.
- **POS tercero**: es pequeño y cierra el recorrido compra → pase → entrega → quema, que es la demostración que pide el Stellar Community Fund.
- **Backoffice cuarto**: es interno y su forma depende de lo que ya existe en los otros tres (colecciones, claims, puntos, dispositivos); construirlo al final evita rehacer.

Alternativa aceptable si el negocio necesita dar de alta bodegas reales antes: adelantar la sub-etapa 4B (Socios y puntos) del Backoffice justo después del ERP.

## 2. Etapa 0 · Fundaciones (2 semanas)

| Paso | Entregable | Terminado cuando |
|---|---|---|
| 0.1 Repo `doc-design-system` | Tokens (`tokens.css` + `@theme` Tailwind 4) a partir de `design-system/tokens.css`; componentes base (§3.1 de 05) sobre primitivas accesibles; AppShell, AdminShell, StoreShell, KioskShell, AuthLayout; Storybook con temas claro/oscuro; CI; publicación `@doc/ui` 0.1 en GitHub Packages | Las cinco maquetas de `design-system/` se reproducen con componentes del paquete |
| 0.2 Repo `doc-mocks` | Esquemas zod, catálogo, seed determinista, fixtures, handlers MSW por dominio, escenarios, página `/__mocks`; publicación `@doc/mocks` 0.1 | `pnpm seed` regenera los JSON idénticos; CI valida fixtures contra esquemas; **los esquemas del ERP ajustados a la documentación de endpoints recibida** |
| 0.3 Plantilla de aplicación | `create-next-app` con Next 16, TypeScript estricto, Tailwind 4, `@doc/ui`, `@doc/mocks` + MSW, `src/lib/api` con cliente tipado y adaptadores, diccionario ES, `launch.json`, CI (lint, tsc, test, build), `dev`/`main`, README, Vercel | Crear `doc-erp-web` desde la plantilla lleva menos de una hora |
| 0.4 Spike Stellar (3 días) | En testnet con `smart-account-kit`: crear smart account con passkey, desplegar vía relayer de prueba, recibir un activo clásico con clawback desde una cuenta emisora, leer saldo, ejecutar clawback desde el emisor, abrir todo en stellar.expert. Informe con tiempos, compatibilidad de dispositivos y API definitiva de `@doc/wallet` | Recorrido completo grabado; decisión cerrada con backend sobre token y relayer |
| 0.5 `@doc/wallet` 0.1 | Interfaz de 04 §7 con implementaciones `mock` y `passkey` (testnet), hooks React | El Marketplace puede construir 2A con `mock` y probar con `passkey` |
| 0.6 Corrección de marca | Pie, contacto y aviso legal de ambos sitios solo con Drinks on Chain | PR mergeado en ambos repos |

**Avance**
- [ ] 0.1 Repo `doc-design-system` y `@doc/ui` 0.1
- [ ] 0.2 Repo `doc-mocks` y `@doc/mocks` 0.1 (esquemas del ERP ajustados a su documentación)
- [ ] 0.3 Plantilla de aplicación
- [ ] 0.4 Spike Stellar en testnet
- [ ] 0.5 `@doc/wallet` 0.1
- [ ] 0.6 Corrección de marca en ambos sitios (PR mergeado) — hecha en `dev` de ambos repos · 25-09-2026; falta el merge a `main`

## 3. Sistema 0 · Sitios públicos (pendientes; 2,5 semanas, en paralelo con Etapas 0–1)

### Landing principal (`drinks-on-chain-landing`), 1 semana
Detalle en `07-roadmap-landing-principal.md`: marca, sitemap/robots/OG y metadatos por ruta, cabeceras de seguridad, fuentes autoalojadas y Lighthouse ≥ 90, textos y accesibilidad, analítica, PR.

### Sitio de bodegas (`drinks-on-chain-front`), 1,5 semanas
| Paso | Entregable |
|---|---|
| B.1 Datos | `bodegas.json` y `puntos-de-recojo.json` alineados con el catálogo de `@doc/mocks`; relación parcela ↔ bodega; estados Socia / En conversación / Referencia |
| B.2 Navegación | Menú Mapa · Bodegas · Puntos de recojo · Unirse · Acceso; pie común; `/vinos` enlaza a la landing principal; variables `NEXT_PUBLIC_URL_LANDING`, `_ERP`, `_POS` |
| B.3 Páginas | `/acceso` (dos tarjetas), `/unirse` (propuesta + formulario mock), `/puntos-de-recojo`, `/bodegas` y `/bodegas/[slug]` con mini-mapa SVG de sus parcelas |
| B.4 Mapa | Conmutador Parcelas / Bodegas; marcadores de sede; al seleccionar una bodega se encuadran sus parcelas |
| B.5 Rutas | `/parcelas/[v]/[p]` → `/valles/[v]/[p]` con redirección permanente; sitemap/robots; cabeceras; marca |

Terminado cuando: navegar entre los dos sitios y volver funciona en móvil y escritorio con URLs reales; Lighthouse accesibilidad ≥ 95 en ambos.

**Avance · landing principal** (detalle en `drinks-on-chain-landing/docs/ROADMAP.md`)
- [x] Barrera de edad: sin desplazamiento debajo y entrada siempre al héroe · 25-09-2026
- [x] Barrera de edad sobre el mapa del héroe, con transición al héroe (como el sitio de bodegas) · 25-09-2026
- [x] Botón "Conocer las bodegas" visible en la sección de bodegas · 25-09-2026
- [x] Corrección de marca (pie, contacto, aviso legal) · 25-09-2026
- [x] `sitemap.xml`, `robots.txt`, imagen OG por defecto y metadatos por ruta · 25-09-2026
- [x] Cabeceras de seguridad · 25-09-2026
- [ ] Fuentes autoalojadas y Lighthouse móvil ≥ 90 — fuentes autoalojadas y accesibilidad 100 hechas; rendimiento móvil 69–77, pendiente
- [ ] Revisión de textos ES/EN y accesibilidad por teclado — teclado hecho (barrera modal, foco visible); falta revisar los textos existentes
- [x] Analítica (Vercel Web Analytics, sin cookies) · 25-09-2026 — falta activarla en el panel de Vercel
- [ ] PR `dev → main` — abierto, pendiente de revisión

**Avance · sitio de bodegas** (detalle en `drinks-on-chain-front/docs/ROADMAP.md`)
- [x] Barrera de edad: sin desplazamiento ni interacción debajo · 25-09-2026
- [x] B.1 Datos: bodegas y puntos de recojo alineados con el catálogo de `08` · 25-09-2026
- [x] B.2 Navegación: menú nuevo, pie común, `/vinos` → landing, variables de entorno · 25-09-2026
- [x] B.3 Páginas: `/acceso`, `/unirse`, `/puntos-de-recojo`, `/bodegas`, `/bodegas/[slug]` · 25-09-2026
- [x] B.4 Mapa: conmutador Parcelas / Bodegas y encuadre de las parcelas de una bodega · 25-09-2026
- [x] B.5 Rutas: `/parcelas → /valles`, sitemap/robots, cabeceras, marca · 25-09-2026
- [ ] PR `dev → main` — abierto, pendiente de revisión

## 4. Etapa 1 · Sistema 1, ERP (`doc-erp-web`, 4 semanas)

Shell: AppShell claro, Inter 16 px, sidebar por módulos (Panel · Origen · Vendimia · Vinificación · Crianza · Destilación · Envasado · Cuenta de la bodega · Ajustes). Datos: handlers `erp` de `@doc/mocks` imitando los endpoints reales.

| Sub-etapa | Pantallas | Componentes nuevos | Reglas en cliente | Semana |
|---|---|---|---|---|
| 1A Acceso y panel | Login dividido (imagen + formulario), recuperar contraseña, selector de bodega, Dashboard (StatCards: lotes activos, kilos hoy, alertas; tabla de tareas pendientes), perfil, ajustes | AppShell, LotStatusBadge | Redirección por rol | 1 |
| 1B Origen | Directorio de terroirs (búsqueda, pills por cepa, grid de tarjetas), ficha con DoBadge, alta/edición, slide-over "Nueva cosecha" | DoBadge, TerroirCard | Aptitud D.O.: Moscatel de Alejandría y altitud > 1.600 m | 1 |
| 1C Vendimia y vinificación | Pesaje con BigNumberInput, análisis con LabReadingCard (Brix, pH, acidez) y botones aprobar/rechazar, TankGrid, bitácora con tabla y "Añadir registro", DecisionModal de bifurcación | BigNumberInput, LabReadingCard, TankGrid, DecisionModal | Táctil ≥ 56 px; la bifurcación bloquea la ruta contraria | 2 |
| 1D Crianza y destilación | Tabla de barricas con madera, meses y CountdownLock; formulario de cortes (cabeza, corazón, cola, grado); candado de reposo con días restantes | CountdownLock, BarrelRow, StillCutsForm | El embotellado permanece deshabilitado hasta que el candado llega a cero | 3 |
| 1E Envasado y QR | Formulario de embotellado (agua añadida, botellas), éxito con sello, BottlingSummary, exportación de QR (ZIP/CSV mock con códigos reales que apuntan a `app./b/{código}`) | BottlingSummary, QrExportCard, LotTimeline | Conciliación kilos → litros → botellas con avisos de merma | 3 |
| 1F Cuenta de la bodega | Panel de solo lectura: dirección, activos por lote, historial con enlaces al explorador; oculto por bandera si el backend no lo expone | WineryAccountPanel | — | 4 |
| 1G Calidad | Estados vacío/cargando/error en todas las pantallas, teclado, lector de pantalla, Playwright del flujo "Singani Gran Reserva 2026" de origen a QR | — | — | 4 |

Terminado cuando: el recorrido completo del caso de ejemplo del documento maestro se hace con mocks en escritorio y tablet, con Playwright verde y sin errores de consola. Integración temprana: en la semana 4 se conecta el login y el módulo de origen al backend real del ERP para validar la capa de acceso a datos.

**Avance**
- [ ] 1A Acceso y panel
- [ ] 1B Origen
- [ ] 1C Vendimia y vinificación
- [ ] 1D Crianza y destilación
- [ ] 1E Envasado y QR
- [ ] 1F Cuenta de la bodega
- [ ] 1G Calidad (Playwright del flujo de ejemplo)

## 5. Etapa 2 · Sistema 2, Marketplace (`doc-marketplace-app`, 5 semanas)

Shell: StoreShell (pestañas inferiores en móvil, cabecera en escritorio), PWA instalable, familia editorial en escaparate/ficha/visor/cava y operativa en checkout/perfil. Datos: handlers `marketplace`; billetera: `@doc/wallet` en modo `mock` (y `passkey` en testnet para pruebas).

| Sub-etapa | Pantallas | Componentes nuevos | Semana |
|---|---|---|---|
| 2A Catálogo sin cuenta | Escaparate (HeroBanner + grid de BottleCard), ficha de producto con imagen pegajosa, notas, historia de la bodega, terroir; StickyBuyBar; página de bodega; búsqueda y filtros | StoreShell, BottleCard, PriceTag, StickyBuyBar | 1 |
| 2B Cuenta y billetera | Pantalla única "Entrar" (teléfono + correo, Google/Apple), OTP, términos; VaultSplash "Preparando tu cava" que llama a `createWallet()` (detección de passkeys; cuenta gestionada si no hay soporte); sugerencia de recuperación; perfil y ajustes con WalletSettings | VaultSplash, WalletSettings | 2 |
| 2C Compra | CheckoutSheet en tres pasos (cantidad y punto de recojo preferido → pago con `PaymentProvider` mock: tarjeta / QR → confirmación); si no hay cuenta, 2B se abre dentro del flujo; OrderStatus; historial de pedidos; pago fallido | CheckoutSheet, OrderStatus | 3 |
| 2D Cava y pase | Mi Cava (CavaGrid de HoldingCard), detalle del activo ("Ver trazabilidad", "Retirar"), PickupPointPicker (puntos habilitados para el lote), ClaimTicket con QR, caducidad en días y estado; historial de pases; pase caducado o anulado | HoldingCard, PickupPointPicker, ClaimTicket | 4 |
| 2E Visor QR | `/b/{código}`: ScanGate con registro ligero, JourneyTimeline con datos del lote, TastingCards, brand story, ReviewForm; escáner con cámara y entrada manual; código inválido | ScanGate, JourneyTimeline, TastingCards, StarRating, CameraScanner | 5 |
| 2F Transversal y calidad | Notificaciones, ayuda que crea tickets, PWA (manifest, iconos, `safe-area`, sin conexión en cava), Lighthouse móvil ≥ 90, Playwright del flujo "María" (escaneo → registro → compra → pase) | — | 5 |

Terminado cuando: el flujo "María" funciona en un móvil real con mocks; la creación de billetera con passkey funciona en testnet en al menos un Android y un iPhone; con `NEXT_PUBLIC_MOCKS=1` la app se puede enseñar sin backend.

**Avance**
- [ ] 2A Catálogo sin cuenta
- [ ] 2B Cuenta y billetera
- [ ] 2C Compra
- [ ] 2D Cava y pase
- [ ] 2E Visor QR
- [ ] 2F Transversal y calidad

## 6. Etapa 3 · Sistema 4, POS (`doc-claim-pos`, 2 semanas)

Shell: KioskShell oscuro, pantalla completa, apaisado, PWA en modo kiosco. Datos: handlers `pos`.

| Sub-etapa | Pantallas | Componentes nuevos | Semana |
|---|---|---|---|
| 3A Acceso | Vinculación del dispositivo (código de alta del Backoffice), PIN de sucursal con PinPad, bloqueo por inactividad, barra de estado (sucursal, conexión, hora) | KioskShell, PinPad | 1 |
| 3B Escaneo y semáforo | ScannerViewport continuo con retícula y linterna; TrafficLightOverlay verde (foto, "ENTREGAR: N botellas", producto, cliente) con SwipeToConfirm; rojo con motivo (ya canjeado, caducado, punto no habilitado) y "Volver a escanear"; entrada manual; permiso de cámara denegado | ScannerViewport, TrafficLightOverlay, SwipeToConfirm, ManualCodeEntry | 1 |
| 3C Confirmación y turno | DeliverySuccess de 2 s con vuelta automática; "Confirmado en la red" cuando llega el hash; ShiftReceipt (hora, producto, cantidad), "Cerrar turno y bloquear"; cola sin conexión con OfflineBanner | DeliverySuccess, ShiftReceipt, OfflineBanner | 2 |
| 3D Calidad | Prueba en iPad y tablet Android reales (cámara, brillo, kiosco), Playwright del flujo "canje perfecto" | — | 2 |

Terminado cuando: un pase generado en el Marketplace mock se escanea desde la pantalla de otro dispositivo y la entrega aparece en el turno; AAA de contraste verificado.

**Avance**
- [ ] 3A Acceso
- [ ] 3B Escaneo y semáforo
- [ ] 3C Confirmación y turno
- [ ] 3D Calidad en tablets reales

## 7. Etapa 4 · Sistema 3, Backoffice (`doc-backoffice-web`, 4 semanas)

Shell: AdminShell (sidebar Cava Reserva, contenido claro, buscador ⌘K), Inter 14 px, tablas compactas. Datos: handlers `backoffice`.

| Sub-etapa | Pantallas | Componentes nuevos | Semana |
|---|---|---|---|
| 4A Acceso y dashboard | Login centrado con 2FA, Dashboard con KpiCard (usuarios, botellas tokenizadas, tickets abiertos) y AlertsFeed del ERP, usuarios internos y RoleMatrix | AdminShell, KpiCard, AlertsFeed, RoleMatrix | 1 |
| 4B Socios y puntos | WineryTable (nombre, región, volumen, estado), WineryDrawer con datos institucionales, alta de bodega (el backend crea la cuenta Stellar: se muestra dirección y estado), CredentialsDialog "Generar credenciales ERP", PickupPointForm (punto ↔ bodegas ↔ lotes, PIN, cajeros), DeviceEnrollCard (código de alta para el POS) | WineryTable, WineryDrawer, CredentialsDialog, PickupPointForm, DeviceEnrollCard | 2 |
| 4C Tokenización | MintPipeline kanban (listo · en revisión · emitiendo · publicado · fallido), MintReviewModal al 80 % (bloque de lectura del ERP + precio fijo + "Aprobar, emitir y publicar" con confirmación), CollectionCard con TxStatusBadge y enlace al explorador, despublicar | MintPipeline, MintReviewModal, TxStatusBadge, CollectionCard | 3 |
| 4D Soporte | TicketTable con filtros y urgencia, TicketSplitView (historial + herramientas), HoldingsLookup por correo, anular pase, autorizar entrega manual, cerrar ticket, auditoría de claims | TicketTable, TicketSplitView, HoldingsLookup | 4 |
| 4E Calidad | Teclado completo, paginación y ordenación en todas las tablas, Playwright del flujo "Luz verde a la colección" | — | 4 |

Terminado cuando: alta de bodega → credenciales → lote listo → emisión → colección publicada se recorre con mocks y la colección aparece en el Marketplace mock (mismos fixtures).

**Avance**
- [ ] 4A Acceso y dashboard
- [ ] 4B Socios y puntos
- [ ] 4C Tokenización
- [ ] 4D Soporte
- [ ] 4E Calidad

## 8. Etapa 5 · Integración y salida (3 semanas)

1. Capa de acceso a datos real por sistema: apagar MSW, `NEXT_PUBLIC_API_URL`, ajustar adaptadores donde los DTO difieran (el ERP ya validado en 1G).
2. Autenticación real por sistema (sesiones por subdominio; OIDC si backend lo implementa).
3. `@doc/wallet` en testnet con el relayer real de backend; pruebas en dispositivos.
4. `PaymentProvider` real con la pasarela del banco (según su modalidad; 06 §3).
5. Pruebas de extremo a extremo entre aplicaciones (ERP → Backoffice → Marketplace → POS).
6. Accesibilidad AA/AAA, rendimiento, observabilidad (Sentry), cabeceras, staging por subdominio, dominio real.
7. Demostración para el Stellar Community Fund con código abierto.

**Avance**
- [ ] 1. Capa de acceso a datos real
- [ ] 2. Autenticación real
- [ ] 3. `@doc/wallet` con el relayer real
- [ ] 4. `PaymentProvider` real
- [ ] 5. Pruebas de extremo a extremo entre aplicaciones
- [ ] 6. Accesibilidad, rendimiento, observabilidad, cabeceras, staging, dominio
- [ ] 7. Demostración para el Stellar Community Fund

## 9. Calendario con dos personas (≈ 14 semanas)

| Semanas | Persona A | Persona B |
|---|---|---|
| 1–2 | Etapa 0: design system, plantilla | Etapa 0: mocks (con la documentación del ERP), spike Stellar, `@doc/wallet` |
| 3–4 | Sistema 0: sitios públicos pendientes | ERP 1A–1C |
| 5–6 | Marketplace 2A–2B | ERP 1D–1G + integración temprana del ERP |
| 7–9 | Marketplace 2C–2F | POS 3A–3D, luego Backoffice 4A–4B |
| 10–11 | Backoffice 4C–4D (con B) | Backoffice 4C–4E |
| 12–14 | Integración: Marketplace, POS, pagos, billetera | Integración: ERP, Backoffice, pruebas cruzadas |

## 10. Definición de terminado por pantalla

Diseñada con `@doc/ui` en el tema del sistema · móvil y escritorio donde aplique · estados vacío, cargando, error y sin conexión donde aplique · textos en español (ES/EN solo en landings) · accesible por teclado y lector de pantalla · sin errores de consola · cubierta por la prueba de flujo de su etapa · componentes nuevos documentados en Storybook · Conventional Commits en `dev` y PR a `main` al cerrar la sub-etapa.

## 11. Riesgos

| Riesgo | Mitigación |
|---|---|
| El backend define modelos distintos a los mocks | Ajustar los esquemas del ERP a su documentación en la Etapa 0; compartir `@doc/mocks` como borrador de contrato para el resto |
| `smart-account-kit` cambia o falla en dispositivos | Todo detrás de `@doc/wallet`; cuenta gestionada como respaldo; versiones fijadas |
| Pasarela sin documentación durante meses | Adaptador `PaymentProvider` soporta redirección, widget y QR; se integra en la Etapa 5 |
| Cámara en tablets económicas | Pruebas con hardware real en 3D; entrada manual del código |
| Cuatro aplicaciones y un equipo pequeño | Plantilla común, design system, una configuración de CI, mocks compartidos |
| Alcance del Backoffice crece (puntos, dispositivos, auditoría) | Se construye al final con lo aprendido; 4B puede adelantarse si el negocio lo pide |
