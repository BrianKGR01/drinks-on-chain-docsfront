# 03 · Roadmap del frontend por olas

Versión 3 · 27 de septiembre de 2026 (v2 en [`antiguo/03-roadmap-frontend-v2.md`](antiguo/03-roadmap-frontend-v2.md), v1 en [`antiguo/03-roadmap-frontend.md`](antiguo/03-roadmap-frontend.md)). Alcance: las cuatro aplicaciones (ERP, Marketplace, POS, Backoffice), los dos sitios públicos y los paquetes compartidos (`@drinks-on-chain/ui`, `@drinks-on-chain/mocks`, plantilla). El backend entró en el alcance del ecosistema el 26-09-2026 y su roadmap vive en `docs-back/08-roadmap.md` ([`drinks-on-chain-docsback`](https://github.com/drinks-on-chain/drinks-on-chain-docsback/blob/main/08-roadmap.md)).

**Calendario común**: el orden en el tiempo lo fija el **plan maestro** (`PLAN-MAESTRO.md`), documento local de coordinación que vive en la carpeta paraguas del ecosistema, fuera de este repositorio. Allí están las olas, las pistas (OPS, BE, SC, PK, ERP, BO, MK, POS, WEB, E2E, DOC), los hitos H0–H6 y el tablero con los IDs `O<ola>-<pista>-<n>` que se citan aquí. Las 16 contradicciones entre esta documentación y las decisiones acordadas del backend (A-01…A-34, 27 y 28-09-2026) se listan en `plan/02-reconciliacion-documentos.md` (también local) y quedan registradas en `docs-back/04` §2.3 (C30–C45).

## Qué cambia respecto a la v2

| # | Cambio | Motivo |
|---|---|---|
| 1 | El roadmap se ordena por **olas** del plan maestro, no por sistemas consecutivos. Backoffice 4A–4B se adelanta a la Ola 1 | A-34, R9: el ciclo del MVP manda (back office y bodegas → ERP → tokenización → Marketplace → POS) |
| 2 | **0.4 (spike de passkeys) y 0.5 (`@doc/wallet`) salen del MVP** | R2, A-04, A-05, A-28: el backend crea direcciones custodiales derivadas; smart accounts en Fase 2 |
| 3 | Marketplace **2B**: cuenta por correo, sin passkey ni `VaultSplash`; dirección informativa de solo lectura | R2, R6, A-13 |
| 4 | Marketplace **2D**: cava con **un NFT por botella** y **pase de canje** con caducidad en horas (24 h por defecto, D-15) que se regenera | R1, R3, A-01, A-07 |
| 5 | POS **3A**: tableta vinculada + **PIN personal del cajero**; **3B/3C**: captura del **código de botella** | R4, R5, A-25, A-26 |
| 6 | Backoffice **4B**: solicitudes e **invitaciones** en lugar de credenciales; **4C**: **bandeja de solicitudes de tokenización** en lugar del kanban "lote listo" | R7, R8, A-08, A-10, A-03 |
| 7 | ERP: sub-etapas nuevas **1H–1L** (sesiones y organización activa, cuenta y equipo, lote real y códigos por botella, autorizar tokenización, puntos de canje) | R7, R8, R10–R12 |
| 8 | Nombres reales: repos `drinks-on-chain-<sistema>` en la organización `drinks-on-chain`, paquetes `@drinks-on-chain/*` (antes `@doc/*`, `doc-*`) | Decisión del 25-09 (06 §1) |
| 9 | Los handlers MSW de los sistemas nuevos se generan desde el **OpenAPI borrador** de cada ola, no desde las rutas propuestas en 08 §7.2 | R16, OPS-12 |

**Cómo se sigue el avance**: cada ola tiene una lista **Avance** con casillas (`- [ ]` pendiente, `- [x]` hecho). Se marca una casilla cuando el paso cumple su "terminado cuando" (§11 y `plan/04` §7) y el trabajo está en `dev`; al lado se anota la fecha y el PR o commit, y se marca a la vez la tarea del tablero del plan maestro. Cada repositorio lleva además su `docs/ROADMAP.md` con el detalle fino. Las casillas marcadas en la v2 se conservan con su fecha original.

## Tablero resumido

- [ ] Ola 0 · Cimientos y saneamiento (en curso desde el 27-09-2026)
- [ ] Ola 1 · Back office y alta de bodegas
- [ ] Ola 2 · ERP completo y trazabilidad confiable
- [ ] Ola 3 · Tokenización y cadena
- [ ] Ola 4 · Marketplace
- [ ] Ola 5 · Canje y POS
- [ ] Ola 6 · Salida a producción
- [ ] Ola F · Precios y pagos (cuando el negocio y el banco lo definan)

## 1. Orden y motivo

```
Ola 0 · Cimientos ─► Ola 1 · Back office y bodegas ─► Ola 2 · ERP confiable ─► Ola 3 · Tokenización
                                                                                     │
Ola F · Precios y pagos ◄┄┄ Ola 6 · Producción ◄─ Ola 5 · Canje y POS ◄─ Ola 4 · Marketplace
```

- **El ciclo del MVP manda** (A-34): sin bodegas activas no hay ERP con equipo; sin lote no hay tokenización; sin NFT no hay Marketplace; sin NFT vendidos no hay canje. El orden de la v2 (ERP → Marketplace → POS → Backoffice) queda sustituido (R9); el propio 03 v2 ya admitía adelantar 4B.
- **Contrato primero**: al abrir cada ola el backend publica el OpenAPI borrador; `@drinks-on-chain/mocks` se regenera desde él; las apps construyen contra mocks mientras el backend implementa y se conectan al entorno de desarrollo al cierre de la ola.
- **Una ola por delante como máximo**: una app puede adelantarse a su ola con los mocks del OpenAPI borrador de la siguiente (por ejemplo, Marketplace 2A en la Ola 2 o 2B–2C en la Ola 3), nunca dos.
- **Cada ola termina en un hito** (H0–H6) verificado con pruebas entre aplicaciones (`drinks-on-chain-e2e`), no solo con la suma de las partes.

## 2. Olas y tareas de frontend

| Ola | Backend (`docs-back/08`) | Tareas de frontend (IDs del plan maestro) | Sub-etapas de este roadmap | Hito |
|---|---|---|---|---|
| **0** · Cimientos y saneamiento | Etapa 0 | O0-PK-1 `ui` 0.2 publicada, plantilla a `main`, `parseDecimal` · O0-PK-2 `mocks` 0.2 y prueba de contrato · O0-ERP-1 cliente de API con sesiones nuevas · O0-ERP-2 integración temprana del ERP · O0-WEB-1 cierre del Sistema 0 · O0-DOC-1 reconciliación | 0.1–0.3, 0.6, 0.7–0.9, Sistema 0, ERP 1A–1G (hechas), **1H** | H0: el ERP inicia sesión, cambia de organización y lista parcelas contra desarrollo |
| **1** · Back office y alta de bodegas | Etapa 1 | O1-PK-1 mocks `backoffice`/`identity`, Combobox, CommandPalette, tablas densas · O1-BO-1 Backoffice creado y 4A · O1-BO-2 4B · O1-ERP-1 cuenta, organización y equipo · O1-WEB-1 `/unirse` real · O1-E2E-1 repo E2E y recorrido H1 | **4A**, **4B**, **1I**, `/unirse` | H1: de cero a bodega con equipo |
| **2** · ERP completo y trazabilidad confiable | Etapa 2 · B.3 | O2-PK-1 mocks ERP v2 y `public` · O2-ERP-1 lote real y códigos de botella · O2-MK-1 Marketplace creado, visor 2E real y catálogo 2A (mocks) · O2-WEB-1 enlaces al Marketplace y perfiles públicos · O2-E2E-1 recorrido H2 y pruebas de elusión | **1J**, **2E**, **2A** (mocks) | H2: "Singani Gran Reserva 2026" contra desarrollo; elusiones rechazadas con 422 |
| **3** · Tokenización y cadena | Etapa 3 | O3-PK-1 mocks `tokenization`/`chain`, OpenAPI borrador de la Etapa 4, `TxStatusBadge` · O3-ERP-1 autorizar tokenización y cuenta de la bodega · O3-BO-1 4C · O3-MK-1 2B y 2C (mocks) · O3-E2E-1 recorrido H3 en testnet | **1K**, **1F** real, **4C**, **2B**–**2C** (mocks) | H3: 100 NFT emitidos en testnet y hash anclado al certificar |
| **4** · Marketplace | Etapa 4 | O4-PK-1 mocks `marketplace` y `pos`, `CameraScanner` · O4-MK-1 Marketplace real, cava, reseñas, PWA · O4-BO-1 pedidos y moderación · O4-POS-1 POS creado, 3A–3B (mocks) · O4-E2E-1 recorrido H4 | **2A**–**2C** reales, **2D** (cava), reseñas de **2E**, **2F**, **4F**, **3A**–**3B** (mocks) | H4: compra en preventa, "pago recibido", dos NFT en la cava |
| **5** · Canje y POS | Etapa 5 | O5-POS-1 POS real y 3D en tabletas · O5-MK-1 pase de canje y post-canje · O5-BO-1 puntos, soporte 4D y campañas · O5-ERP-1 puntos de la bodega · O5-WEB-1 puntos y postulación en el sitio de bodegas · O5-E2E-1 ciclo completo | **3A**–**3D** reales, pase de **2D**, post-canje de **2E**, **4D**, **1L** | H5: ciclo del MVP de la etapa 1 a la 12 en testnet |
| **6** · Salida a producción | Etapa 6 · B.4 | O6-FE-1 calidad de salida del frontend · O6-E2E-1 recorrido en staging y demo SCF | §9 (integración y salida) | H6: aprobación del cliente |
| **F** · Precios y pagos | Etapa F | `PaymentProvider` real con la pasarela del banco, política de precio, reembolsos, textos legales | §9.4 | — |

## 3. Ola 0 · Cimientos y saneamiento

### 3.1 Fundaciones (antes Etapa 0)

| Paso | Entregable | Terminado cuando |
|---|---|---|
| 0.1 `drinks-on-chain-design-system` | Tokens, componentes base, shells (AppShell, AdminShell, StoreShell, KioskShell, AuthLayout), Storybook, CI, publicación de `@drinks-on-chain/ui` por GitHub Release | Las cinco maquetas de `design-system/` se reproducen con componentes del paquete |
| 0.2 `drinks-on-chain-mocks` | Esquemas zod, catálogo, seed determinista, fixtures, handlers MSW, escenarios; `@drinks-on-chain/mocks` 0.1 | `pnpm seed` regenera los JSON idénticos; CI valida fixtures contra esquemas |
| 0.3 `drinks-on-chain-app-template` | Next 16, TypeScript estricto, Tailwind 4, `ui` + `mocks` + MSW, `src/lib/api`, diccionario ES, CI | Crear una app desde la plantilla lleva menos de una hora |
| ~~0.4 Spike Stellar con passkeys~~ | **Fuera del MVP** (R2, A-28): la billetera del consumidor la crea el backend; las smart accounts con `smart-account-kit` quedan para la Fase 2 | — |
| ~~0.5 `@doc/wallet`~~ | **Fuera del MVP** (R2, A-28): el cliente no crea ni firma con billeteras; solo muestra la dirección que devuelve la API | — |
| 0.6 Corrección de marca | Pie, contacto y aviso legal de ambos sitios solo con Drinks on Chain | Fusionado en `main` de ambos repos |
| 0.7 `ui` 0.2 publicada (O0-PK-1) | Etiqueta `v0.2.0` en `main` y su Release; ERP y plantilla consumen `ui` 0.2.x; plantilla a `main`; `lib/format.ts` (`parseDecimal` único) en la plantilla | El ERP compila con `ui` 0.2.x y la plantilla tiene `main` con CI verde |
| 0.8 `mocks` 0.2 (O0-PK-2) | Listas `{ items, total, limit, offset }` (máximo 100) y errores con `details: [{ field, message }]` (R11); sesión con cookie y organización activa; prueba de contrato de los fixtures contra el OpenAPI; `pnpm openapi:pull` | Los fixtures pasan la prueba de contrato contra el OpenAPI de la Etapa 0 |
| 0.9 Integración temprana (O0-ERP-2) | Login, perfil y Origen del ERP contra el backend de desarrollo (prueba `backend-real`) | La prueba `backend-real` pasa contra desarrollo |

**Avance**
- [x] 0.1 Repo `drinks-on-chain-design-system` y `@drinks-on-chain/ui` 0.1 · 2026-09-25 (v0.1.0)
- [x] 0.2 Repo `drinks-on-chain-mocks` y `@drinks-on-chain/mocks` 0.1 (esquemas del ERP ajustados a su documentación) · 2026-09-25 (v0.1.0)
- [x] 0.3 Plantilla de aplicación · 2026-09-25 (`drinks-on-chain-app-template`; el ERP nació de ella)
- ~~0.4 Spike Stellar en testnet~~ · **fuera del MVP** (R2, A-28: smart accounts en Fase 2)
- ~~0.5 `@doc/wallet` 0.1~~ · **fuera del MVP** (R2, A-28)
- [x] 0.6 Corrección de marca en ambos sitios · 2026-09-25 (hecha en `dev`; fusionada en `main` con el PR #4 de cada repo el mismo día)
- [ ] 0.7 `ui` 0.2 publicada, plantilla a `main`, `parseDecimal` compartido (O0-PK-1)
- [ ] 0.8 `mocks` 0.2 y prueba de contrato (O0-PK-2)
- [ ] 0.9 Integración temprana del ERP con desarrollo (O0-ERP-2)

### 3.2 Sistema 0 · Sitios públicos (O0-WEB-1)

Landing principal (`drinks-on-chain-landing`, detalle en `07-roadmap-landing-principal.md`) y sitio de bodegas (`drinks-on-chain-front`). Pasos B.1–B.5 del sitio de bodegas como en la v2. Para cerrar el Sistema 0 en la Ola 0 falta: Lighthouse móvil ≥ 90 en la landing, revisión de textos ES/EN, variables `NEXT_PUBLIC_URL_*` en el proyecto de Vercel de bodegas con puertos coherentes (ERP en 3002), CI y pruebas de humo en ambos sitios. "Puntos de recojo" pasa a llamarse **puntos de canje** en los textos (R4); la página y el formulario de postulación de un punto llegan en la Ola 5 (O5-WEB-1).

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
- [x] PR `dev → main` · 25-09-2026 (PR #4 fusionado)
- [ ] CI y pruebas de humo (O0-WEB-1)

**Avance · sitio de bodegas** (detalle en `drinks-on-chain-front/docs/ROADMAP.md`)
- [x] Barrera de edad: sin desplazamiento ni interacción debajo · 25-09-2026
- [x] B.1 Datos: bodegas y puntos de recojo alineados con el catálogo de `08` · 25-09-2026
- [x] B.2 Navegación: menú nuevo, pie común, `/vinos` → landing, variables de entorno · 25-09-2026
- [x] B.3 Páginas: `/acceso`, `/unirse`, `/puntos-de-recojo`, `/bodegas`, `/bodegas/[slug]` · 25-09-2026
- [x] B.4 Mapa: conmutador Parcelas / Bodegas y encuadre de las parcelas de una bodega · 25-09-2026
- [x] B.5 Rutas: `/parcelas → /valles`, sitemap/robots, cabeceras, marca · 25-09-2026
- [x] PR `dev → main` · 25-09-2026 (PR #4 fusionado)
- [ ] Variables `NEXT_PUBLIC_URL_*` en Vercel, puertos coherentes, CI y pruebas de humo (O0-WEB-1)

### 3.3 ERP: lo hecho y la adaptación a sesiones (O0-ERP-1)

Las sub-etapas 1A–1G del ERP (`drinks-on-chain-erp`) están hechas contra mocks y se describen en §5. En la Ola 0 se añade **1H** (§5) y se conecta el ERP al backend de desarrollo (0.9).

**Avance**
- [x] 1A Acceso y panel · 2026-09-25 (sin selector de bodega: el backend no permitía cambiar la bodega activa, 09 §8 punto 13; lo resuelve 1H)
- [x] 1B Origen · 2026-09-25
- [x] 1C Vendimia y vinificación · 2026-09-25
- [x] 1D Crianza y destilación · 2026-09-25
- [x] 1E Envasado y QR · 2026-09-25 (códigos por botella provisionales, 09 §8 punto 20; los reales llegan en 1J)
- [x] 1F Cuenta de la bodega · 2026-09-25 (activos por lote pendientes del backend; con datos reales en la Ola 3)
- [x] 1G Calidad (Playwright del flujo de ejemplo) · 2026-09-25 (auditoría axe, teclado y estados; falta probar con datos reales del backend, 10 §2.1)
- [ ] 1H Sesiones y organización activa (O0-ERP-1)

## 4. Olas 1 a 6: avance

**Ola 1 · Back office y alta de bodegas**
- [ ] 4A Backoffice creado, acceso con 2FA, tablero, usuarios internos (O1-BO-1)
- [ ] 4B Solicitudes, alta directa, bodegas, equipo, configuración y bitácora (O1-BO-2)
- [ ] 1I Cuenta, organización y equipo en el ERP (O1-ERP-1)
- [ ] `/unirse` del sitio de bodegas con el formulario real y captcha (O1-WEB-1)
- [ ] Mocks `backoffice`/`identity` y componentes de tabla (O1-PK-1)

**Ola 2 · ERP completo y trazabilidad confiable**
- [ ] 1J Lote real y códigos por botella (O2-ERP-1)
- [ ] Marketplace creado (`drinks-on-chain-marketplace`), 2E visor real y 2A catálogo con mocks (O2-MK-1)
- [ ] Enlaces al Marketplace y perfiles públicos de bodega (O2-WEB-1)
- [ ] Mocks ERP v2 y dominio `public` (O2-PK-1)

**Ola 3 · Tokenización y cadena**
- [ ] 1K Autorizar tokenización y 1F con la cuenta real de la bodega (O3-ERP-1)
- [ ] 4C Bandeja de solicitudes de tokenización y colecciones (O3-BO-1)
- [ ] 2B Cuenta por correo y 2C compra, contra mocks de la Etapa 4 (O3-MK-1)
- [ ] Mocks `tokenization`/`chain`, OpenAPI borrador de la Etapa 4, `TxStatusBadge` (O3-PK-1)

**Ola 4 · Marketplace**
- [ ] 2A–2C contra el backend real (O4-MK-1)
- [ ] 2D Cava con NFT por botella y línea de tiempo del lote (O4-MK-1)
- [ ] Reseñas en 2E (O4-MK-1)
- [ ] 2F Transversal, PWA y calidad (O4-MK-1)
- [ ] 4F Pedidos, colecciones con ventas y moderación de reseñas (O4-BO-1)
- [ ] POS creado (`drinks-on-chain-pos`), 3A y 3B contra mocks (O4-POS-1)
- [ ] Mocks `marketplace` y `pos`, `CameraScanner` (O4-PK-1)

**Ola 5 · Canje y POS**
- [ ] 3A–3C contra el backend real, código de botella, turnos, cola sin conexión (O5-POS-1)
- [ ] 3D Calidad en tabletas reales (O5-POS-1, con el usuario)
- [ ] Pase de canje en 2D, puntos habilitados, post-canje en 2E, ayuda que crea tickets (O5-MK-1)
- [ ] 4D Puntos de canje, cajeros, soporte y campañas (O5-BO-1)
- [ ] 1L Puntos de canje de la bodega en el ERP (O5-ERP-1)
- [ ] Puntos de canje desde la API y postulación de un punto en el sitio de bodegas (O5-WEB-1)

**Ola 6 · Salida a producción**: ver §9.

## 5. Sistema 1 · ERP (`drinks-on-chain-erp`, `erp.`)

Shell: AppShell claro, Inter 16 px, sidebar por módulos (Panel · Origen · Vendimia · Vinificación · Crianza · Destilación · Envasado · Cuenta de la bodega · Equipo · Ajustes). Datos: handlers `erp` de `@drinks-on-chain/mocks`, que siguen el OpenAPI del backend (09).

| Sub-etapa | Ola | Pantallas | Componentes nuevos | Reglas en cliente |
|---|---|---|---|---|
| 1A Acceso y panel | hecha | Login dividido, Dashboard (StatCards y tareas pendientes), perfil, ajustes | AppShell, LotStatusBadge | Redirección por rol |
| 1B Origen | hecha | Directorio de terroirs, ficha con DoBadge, alta/edición | DoBadge, TerroirCard | Aptitud D.O. (el servidor manda desde la Ola 2) |
| 1C Vendimia y vinificación | hecha | Pesaje, análisis, TankGrid, bitácora, DecisionModal | BigNumberInput, LabReadingCard, TankGrid, DecisionModal | Táctil ≥ 56 px |
| 1D Crianza y destilación | hecha | Barricas con CountdownLock, cortes, candado de reposo | CountdownLock, BarrelRow, StillCutsForm | El embotellado sigue bloqueado hasta que el candado llega a cero |
| 1E Envasado y QR | hecha | Embotellado, BottlingSummary, exportación de QR | BottlingSummary, QrExportCard, LotTimeline | Conciliación kilos → litros → botellas |
| 1F Cuenta de la bodega | hecha (mocks); real en Ola 3 | Panel de solo lectura: dirección de la bodega, contrato NFT, historial con enlaces al explorador | WineryAccountPanel | — |
| 1G Calidad | hecha | Estados, teclado, lector de pantalla, Playwright "Singani Gran Reserva 2026" | — | — |
| **1H Sesiones y organización activa** | 0 | Cliente de API con acceso de 15 min y **renovación silenciosa** por cookie `HttpOnly` a través del proxy de mismo origen (`/api/v1/*`), una sola renovación en vuelo, cierre con aviso ante `AUTH_REFRESH_REUSED`/`AUTH_SESSION_REVOKED`; **selector de organización** en la cabecera cuando hay más de una membresía; formularios que marcan el campo exacto con `details[].field` | OrgSwitcher | Al cambiar de organización se vacía la caché de consultas (contrato de sesiones, `plan/contratos/o0-sesiones-y-estandares.md`) |
| **1I Cuenta, organización y equipo** | 1 | **Aceptar invitación** (cuenta nueva o membresía añadida), recuperar contraseña, verificar correo, **equipo de la bodega** (invitar, reenviar, anular, cambiar rol, bloquear), bitácora propia del dueño, pantalla de bodega no activa | InviteAcceptForm, TeamTable, AuditLogTable | Solo el dueño gestiona el equipo; nunca se muestran ni envían contraseñas (R7) |
| **1J Lote real y códigos por botella** | 2 | Paso de `LotView` derivada a la entidad `Lot`; dictamen separado del pesaje; conciliación con mermas; **código único por botella** (8 caracteres Crockford) y exportación por botella para la imprenta; reportes y archivos privados; errores 422 de reglas con su motivo | BottleCodeExport, RuleViolationNotice | El QR impreso apunta a la URL configurable `/b/{código}` (R12) |
| **1K Autorizar tokenización** | 3 | "**Autorizar tokenización**" de un lote con su cuota, en cualquier momento del proceso (preventa); estado de la solicitud (pendiente, cambios pedidos, aprobada, rechazada, emitida); 1F con la cuenta y el contrato reales | TokenizationRequestForm, RequestStatusBadge | Solo el dueño autoriza; la aprobación del back office es configurable (D-20) |
| **1L Puntos de canje** | 5 | Puntos de canje de la bodega (si está habilitada) y lotes que entrega cada punto | PickupPointTable | — |

Terminado cuando (por ola): H0 el ERP inicia sesión, cambia de organización y lista parcelas contra desarrollo; H1 el dueño acepta la invitación e invita a su equipo; H2 el caso de ejemplo se recorre contra desarrollo de la parcela a los códigos de botella y las elusiones fallan con 422 explicado; H3 la bodega autoriza la tokenización de un lote.

## 6. Sistema 2 · Marketplace (`drinks-on-chain-marketplace`, `app.`)

Shell: StoreShell (pestañas inferiores en móvil, cabecera en escritorio), PWA instalable, familia editorial en escaparate, ficha, visor y cava; operativa en checkout y perfil. Datos: handlers `marketplace` y `public` generados desde el OpenAPI borrador de cada ola (R16). **Sin billetera en el cliente**: el backend crea una dirección custodial derivada por consumidor (SEP-0005, sin fondear) y firma todo; la app solo la muestra (R2, A-04, A-28).

| Sub-etapa | Ola | Pantallas | Componentes nuevos |
|---|---|---|---|
| 2A Catálogo sin cuenta | 2 (mocks) · 4 (real) | Escaparate, ficha de producto con imagen pegajosa, notas, historia de la bodega, terroir; StickyBuyBar; página de bodega; búsqueda y filtros. El precio llega de la colección y puede faltar (A-32) | StoreShell, BottleCard, PriceTag, StickyBuyBar |
| 2B Cuenta por correo | 3 (mocks) · 4 (real) | **Registro y entrada solo con correo** (A-13) con captcha y verificación de correo; recuperar contraseña; términos y mayoría de edad por declaración; perfil con la **dirección informativa de solo lectura** y enlace al explorador. Sin passkeys, sin `VaultSplash`, sin SMS ni proveedores sociales (R2, R6) | AuthSheet, AddressReadOnly |
| 2C Compra | 3 (mocks) · 4 (real) | CheckoutSheet (cantidad con el máximo por compra configurable → pago con `PaymentProvider` de prueba → confirmación); si no hay cuenta, 2B se abre dentro del flujo; **aviso explícito de "pago recibido"** antes de mostrar los NFT (A-23); historial de pedidos; pago fallido | CheckoutSheet, OrderStatus |
| 2D Cava y pase de canje | 4 (cava) · 5 (pase) | Mi Cava con **un NFT por botella** (número de botella, lote, bodega) y la **línea de tiempo del lote** en preventa; detalle del NFT ("Ver trazabilidad", "Canjear"); puntos de canje habilitados; **pase de canje** con QR, caducidad **en horas** (24 h por defecto, D-15) y botón **"Generar otro"** al caducar, uno activo por NFT (A-07); ventana de canje en días (A-19) con aviso; historial de canjes | TokenCard, PickupPointPicker, ClaimTicket |
| 2E Visor público | 2 (visor) · 4 (reseñas) · 5 (post-canje) | `/b/{código}` resuelve **botella o lote** (R12) sin cuenta (A-21): JourneyTimeline con datos reales del pasaporte, verificación del hash anclado, TastingCards, historia de la bodega; **reseña para cualquier persona con sesión**, con marca de "verificada" si canjeó (R13, D-19); vista post-canje; escáner con cámara y entrada manual; código inválido | JourneyTimeline, TastingCards, StarRating, ReviewForm, CameraScanner |
| 2F Transversal y calidad | 4 · 5 (ayuda) | Notificaciones por correo, ayuda que crea tickets (Ola 5), PWA (manifest, iconos, `safe-area`, sin conexión en la cava), Lighthouse móvil ≥ 90, Playwright del flujo "María" (escaneo → registro → compra → cava → pase) | — |

Terminado cuando: H4 un consumidor se registra por correo, compra dos botellas en preventa, ve "pago recibido" y sus dos NFT en la cava; H5 genera un pase de canje, lo canjea en el POS y ve el post-canje. Con `NEXT_PUBLIC_MOCKS=1` la app se puede enseñar sin backend.

## 7. Sistema 4 · POS (`drinks-on-chain-pos`, `pos.`)

Shell: KioskShell oscuro, pantalla completa, apaisado, PWA en modo kiosco (iPad o tablet Android). Datos: handlers `pos` desde el OpenAPI borrador de la Etapa 5. Contraste AAA.

| Sub-etapa | Ola | Pantallas | Componentes nuevos |
|---|---|---|---|
| 3A Acceso | 4 (mocks) · 5 (real) | **Vinculación de la tableta** al punto de canje (código de un solo uso emitido desde el back office o por la bodega), **PIN personal del cajero** (no de sucursal), apertura de turno, bloqueo por inactividad, barra de estado (punto, cajero, conexión, hora) (R4, IAM-11, IAM-12) | KioskShell, PinPad |
| 3B Escaneo y semáforo | 4 (mocks) · 5 (real) | ScannerViewport continuo con retícula y linterna; semáforo verde (foto, producto, número de botella, cliente) → **captura del código de botella** (escáner o teclado) según `canje.codigoBotella.modo` (desactivado, opcional, obligatorio) → SwipeToConfirm; rojo con motivo (ya canjeado, pase caducado, punto no habilitado, código de botella no coincide) y "Volver a escanear"; entrada manual; permiso de cámara denegado (R5, A-26) | ScannerViewport, TrafficLightOverlay, BottleCodeCapture, SwipeToConfirm, ManualCodeEntry |
| 3C Confirmación y turno | 5 | DeliverySuccess con vuelta automática; "Confirmado en la red" cuando llega la quema (`redeem_burn`); ShiftReceipt y "Cerrar turno y bloquear"; cola sin conexión con OfflineBanner | DeliverySuccess, ShiftReceipt, OfflineBanner |
| 3D Calidad | 5 | Prueba en iPad y tablet Android reales (cámara, brillo, kiosco) con el usuario; Playwright del flujo "canje perfecto" | — |

Terminado cuando: H5, un pase generado en el Marketplace se escanea desde otra pantalla, se registra el código de botella, se confirma y la entrega aparece en el turno y la cava del cliente.

## 8. Sistema 3 · Backoffice (`drinks-on-chain-backoffice`, `admin.`)

Shell: AdminShell (sidebar oscura, contenido claro, buscador ⌘K), Inter 14 px, tablas compactas. Datos: handlers `backoffice` e `identity` desde el OpenAPI de la Etapa 1. Se construye **primero** entre las apps nuevas (R9).

| Sub-etapa | Ola | Pantallas | Componentes nuevos |
|---|---|---|---|
| 4A Acceso y tablero | 1 | Login con **2FA TOTP**, tablero (bodegas, solicitudes pendientes, tickets abiertos), usuarios internos por **invitación** con rol (superusuario, administración, operaciones, soporte) y RoleMatrix | AdminShell, KpiCard, AlertsFeed, RoleMatrix |
| 4B Solicitudes, bodegas y equipo | 1 | **Bandeja de solicitudes** de alta (aprobar, rechazar con motivo, agendar reunión opcional) y **alta directa**; ambas envían una **invitación** al dueño, nunca contraseñas (R7, A-08, A-10); directorio y ficha de bodega (suspender, reactivar; la cuenta en la red nace al activarse); **equipo de una bodega** (añadir, cambiar rol, bloquear); configuración general y por bodega con historial; bitácora con filtros y exportación | RequestInbox, WineryTable, WineryDrawer, InviteDialog, ConfigEditor, AuditLogTable |
| 4C Tokenización | 3 | **Bandeja de solicitudes de tokenización** que envían las bodegas desde el ERP (aprobar, pedir cambios, rechazar) en lugar del kanban "lote listo" (R8, A-03, D-20); datos comerciales de la colección (el precio puede quedar vacío, A-32); colecciones con estado de emisión y enlace al explorador; publicar, pausar, ampliar cuota | TokenizationInbox, CollectionCard, TxStatusBadge |
| 4D Puntos de canje, soporte y campañas | 5 | Puntos de canje por los tres caminos (bodega, soporte, postulación) y cajeros con máximos configurables (A-25); códigos de vinculación de tabletas; TicketTable y TicketSplitView; consulta de cuenta por correo, **entrega asistida**, extensión de ventana, corrección de canje; campañas post-canje (A-27) | PickupPointForm, DeviceEnrollCard, TicketTable, TicketSplitView, AccountLookup |
| 4E Calidad | cada ola | Teclado completo, paginación (`limit` ≤ 100) y ordenación en todas las tablas, Playwright del recorrido de cada hito | — |
| 4F Pedidos y reseñas | 4 | Pedidos, colecciones con ventas, moderación de reseñas | OrderTable, ReviewModeration |

Terminado cuando: H1 superusuario → operaciones → solicitud aprobada → invitación aceptada → equipo; H3 solicitud de tokenización aprobada y colección emitida en testnet; H5 punto de canje con cajero y un ticket resuelto con entrega asistida.

## 9. Ola 6 · Integración y salida (antes Etapa 5)

1. Cada app contra el backend real de staging, sin MSW (`NEXT_PUBLIC_MOCKS` desactivado), con el proxy de mismo origen hacia `API_ORIGIN`.
2. Sesiones reales por subdominio con renovación en cookie de primera parte y organización activa (el diseño queda fijado en la Ola 0).
3. ~~`@doc/wallet` con el relayer real~~ · fuera del MVP (A-28): no hay billetera en el cliente.
4. `PaymentProvider` real con la pasarela del banco: **Ola F** (A-14, A-32).
5. Pruebas de extremo a extremo entre aplicaciones en `drinks-on-chain-e2e` (ya existentes desde la Ola 1) contra staging.
6. Accesibilidad AA/AAA, rendimiento, observabilidad (Sentry), cabeceras, dominio real (O6-FE-1).
7. Demostración para el Stellar Community Fund con código abierto (O6-E2E-1).

**Avance**
- [ ] 1. Apps contra staging sin mocks
- [ ] 2. Sesiones reales por subdominio
- ~~3. `@doc/wallet` con el relayer real~~ · fuera del MVP
- [ ] 4. `PaymentProvider` real (Ola F)
- [ ] 5. Pruebas de extremo a extremo contra staging
- [ ] 6. Accesibilidad, rendimiento, observabilidad, cabeceras, dominio
- [ ] 7. Demostración para el Stellar Community Fund

## 10. Calendario

El calendario vale el del plan maestro (§5), que suma las semanas del backend y del frontend. Referencia para dos personas de frontend:

| Ola | Semanas | Frontend |
|---|---|---|
| 0 | 1–2 | Sistema 0, paquetes (`ui` 0.2, `mocks` 0.2), ERP 1H, integración temprana, reconciliación |
| 1 | 3–5 | Backoffice 4A–4B, ERP 1I, `/unirse` |
| 2 | 6–8 | ERP 1J, Marketplace 2E y 2A (mocks) |
| 3 | 9–11 | ERP 1K, Backoffice 4C, Marketplace 2B–2C (mocks) |
| 4 | 12–14 | Marketplace 2A–2F real, Backoffice 4F, POS 3A–3B (mocks) |
| 5 | 15–17 | POS real y 3D, pase de canje, Backoffice 4D, ERP 1L, puntos en el sitio de bodegas |
| 6 | 18–19 | Integración y salida |

## 11. Definición de terminado por pantalla

Diseñada con `@drinks-on-chain/ui` en el tema del sistema · móvil y escritorio donde aplique · estados vacío, cargando, error y sin conexión donde aplique · datos solo a través de hooks sobre `src/lib/api` · números con el `parseDecimal` único · errores de validación marcados en el campo exacto (`details[].field`) · textos en español (ES/EN solo en landings) · accesible por teclado y lector de pantalla · sin errores de consola · cubierta por la prueba de flujo de su sub-etapa y, al cierre de la ola, por el recorrido E2E del hito · componentes nuevos documentados en Storybook · Conventional Commits en `dev`, puertas locales verdes (`lint`, `typecheck`, `test`, `build`, `e2e`) y PR `dev → main` al cerrar la ola (`plan/04` §7).

## 12. Riesgos

| Riesgo | Mitigación |
|---|---|
| El OpenAPI de una ola llega tarde o cambia | Las apps van como mucho una ola por delante; mocks regenerados desde el borrador; prueba de contrato en `mocks` (O0-PK-2); cambios incompatibles anunciados en el PR del backend y en `docs-back/04` |
| Sesiones con cookie entre `vercel.app` y el dominio de la API (Safari bloquea cookies de terceros) | Proxy de mismo origen en cada app (`/api/v1/*` → `API_ORIGIN`) hasta tener dominio propio |
| La migración del ERP de `LotView` a la entidad `Lot` rompe pantallas hechas | Cambios aditivos del backend, bandera para convivir, pruebas `backend-real` en CI |
| Preguntas abiertas que afectan pantallas (D-15 caducidad del pase, D-19 reseñas, D-20 aprobación de la tokenización, D-21 postulación de puntos) | Supuestos de `plan/02` §3 y de 06 §6; parámetros leídos de la configuración, no fijos en el cliente |
| Cámara en tabletas económicas | Pruebas con hardware real en 3D; entrada manual del código de pase y de botella |
| Pasarela sin documentación durante meses | `PaymentProvider` con adaptador de prueba; la integración real es la Ola F |
| Cuatro aplicaciones, un repo E2E y un equipo pequeño | Plantilla común, design system, una configuración de CI, mocks compartidos, un agente por pista y repo (plan maestro §2) |
