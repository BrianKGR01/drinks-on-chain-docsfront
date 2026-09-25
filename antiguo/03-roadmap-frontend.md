# 03 · Roadmap global del frontend

Alcance: solo cliente. Cada sistema se construye completo contra datos mock (JSON + MSW) con un sistema de diseño compartido, de forma que la integración con el backend sea cambiar el origen de datos, no rehacer pantallas.

## 1. Principios

1. **Contrato primero**: los tipos y fixtures viven en `doc-mocks` y son la referencia para negociar el OpenAPI con backend. Cada pantalla se construye contra un handler MSW que devuelve esos fixtures; los estados vacío, cargando, error y sin conexión se diseñan desde el principio.
2. **Un design system, dos temas**: "Oro Líquido" (claro: landing, S1, S2) y "Cava Reserva" (oscuro: S3 sidebar, S4 completa). Componentes idénticos, tokens distintos.
3. **Móvil primero donde el usuario está en la calle** (S2), tablet primero donde está en el mostrador o la planta (S4, pantallas táctiles de S1), escritorio primero donde se trabaja con tablas (S3).
4. **Billeteras invisibles**: la complejidad de Stellar se encapsula en un paquete `doc-wallet` que los frontends consumen como hooks; ver `04-billeteras-stellar.md`.
5. **Todo desplegable desde el día uno**: cada repo con CI (lint, tsc, test, build), previews por PR en Vercel, `dev` → `main` por PR.

## 2. Etapas

### Etapa 0 · Fundaciones (semanas 1–2, en paralelo con L2)
- `doc-design-system`: tokens (color, tipografía, espaciado, radio, sombra mínima, motion), temas, fuentes, componentes base (Button, TextLink, Input, Select, Textarea, Checkbox/Radio, Badge, Card, DataTable, SlideOver, Modal, BottomSheet, Tabs, Stepper, StatCard, Toast, EmptyState, Skeleton), utilidades tipográficas heredadas de la landing, Storybook con casos claro/oscuro. Se publica en GitHub Packages como `@doc/ui`.
- `doc-mocks`: tipos TypeScript del modelo de entidades (01 §5), fixtures JSON coherentes (2 bodegas, 6 terroirs, 8 lotes en distintos estados, 4 colecciones, 3 usuarios por rol, 5 claims, 6 tickets), handlers MSW, semillas deterministas. Se publica como `@doc/mocks`.
- Plantilla de aplicación: `create-next-app` con la configuración de la landing (ESLint, Tailwind 4, `@doc/ui`, MSW en desarrollo, i18n mínima, CI, `launch.json`).
- Spike Stellar (3 días): crear billetera con passkey en testnet, recibir un token SEP-41 de prueba, firmar un desafío. Define la API de `doc-wallet`.
- Salida: dos paquetes publicados, una plantilla, un informe del spike, decisión de billetera cerrada.

### Etapa 1 · Sitios públicos (semanas 1–5)
Fases L2.1–L2.5 de `02-plan-landing-ecosistema.md` (v2): el sitio de bodegas (`drinks-on-chain-front`, mapa + bodegas + puntos de recojo + acceso) y la landing principal (`drinks-on-chain-landing`, B2C con sección y ruta B2B; roadmap propio en `07-roadmap-landing-principal.md`). Son los primeros consumidores del design system y del paquete de contenido, así que sirven para endurecerlos. Ninguno autentica usuarios.

### Etapa 2 · Sistema 1, ERP (semanas 3–7)
| Sub-etapa | Pantallas | Notas |
|---|---|---|
| 2A Acceso y panel | Login split, recuperar contraseña, Dashboard (widgets + tareas), layout con sidebar y breadcrumbs | Roles y redirección |
| 2B Origen | Directorio de terroirs (pills, grid), ficha con badge D.O., slide-over de cosecha, alta/edición | Validación D.O. en cliente |
| 2C Vendimia y tanques | Pesaje (input gigante), análisis (aprobar/rechazar), mapa de tanques, bitácora, modal de bifurcación | Táctil: objetivos ≥ 56 px |
| 2D Crianza y destilación | Barricas con cuenta regresiva, cortes del alambique, candado de reposo | Componentes `CountdownLock`, `LotTimeline` |
| 2E Embotellado y QR | Formulario final, éxito con sello animado, exportación de QR (ZIP/CSV mock) | Genera QR reales con `qrcode` apuntando al dominio de S2 |
| 2F Cuenta de la bodega | Panel de solo lectura de la cuenta Stellar de la bodega (dirección, saldos, historial) | Ver 04 §5 |
Salida: ERP navegable de punta a punta con mocks; Playwright del flujo "lote de singani" completo.

### Etapa 3 · Sistema 2, Marketplace + visor (semanas 5–10)
| Sub-etapa | Pantallas | Notas |
|---|---|---|
| 3A Onboarding y billetera | Registro ligero (teléfono + correo, Google/Apple), OTP, splash de bóveda que crea la billetera con passkey | Primer uso real de `doc-wallet` |
| 3B Catálogo y compra | Escaparate, PDP con CTA persistente, checkout en slide-over (tarjeta / QR bancario mock), confirmación | Pasarela pendiente de decisión |
| 3C Cava y claim | Mi Cava (tarjetas), detalle de activo, generador de pase con QR temporal y cuenta atrás | El pase incluye firma de la billetera |
| 3D Visor QR | Scan gate, escáner de cámara, viaje del producto (scroll narrativo), cata y brand story, reviews | Reutiliza componentes de la demo de la landing |
| 3E PWA y perfil | Instalable, bottom tabs, perfil, ayuda (crea tickets) | Lighthouse móvil ≥ 90 |
Salida: app instalable con mocks; Playwright del flujo "María" (escaneo → registro → compra → claim).

### Etapa 4 · Sistema 3, Backoffice (semanas 9–12)
| Sub-etapa | Pantallas |
|---|---|
| 4A Acceso y dashboard | Login con 2FA, KPI, panel de alertas del ERP |
| 4B Socios y puntos de recojo | Directorio, perfil de bodega, generar credenciales, alta de puntos de recojo y cajeros (no documentado, necesario) |
| 4C Tokenización | Pipeline kanban, modal de minting (revisión + precio + "Aprobar, mintear y publicar"), estado de la transacción Stellar con enlace al explorador |
| 4D Soporte | Helpdesk, resolución de disputas con verificación de cava por correo |
Salida: backoffice con tablas densas, filtros y paginación sobre mocks; Playwright del flujo "Luz verde a la colección".

### Etapa 5 · Sistema 4, POS (semanas 11–13)
| Sub-etapa | Pantallas |
|---|---|
| 5A Acceso | PIN de sucursal, bloqueo por inactividad |
| 5B Escaneo | Escáner continuo (`@zxing/browser`), linterna, semáforo verde con swipe, semáforo rojo con motivo |
| 5C Cierre | Éxito y vuelta automática, historial del turno, cierre y bloqueo, cola sin conexión |
Salida: PWA en modo kiosco probada en tablet real; Playwright del flujo "canje perfecto".

### Etapa 6 · Integración y salida (semanas 13–16)
Sustituir handlers MSW por el cliente HTTP real, reconciliar tipos con el OpenAPI, autenticación real, `doc-wallet` en testnet con relayer real, pruebas de extremo a extremo entre apps, accesibilidad AA, rendimiento, observabilidad (Sentry), despliegue a staging.

### Etapa 7 · Fase 2 (post-MVP)
Preventas con fases de precio, mercado secundario P2P (firma de órdenes con la billetera), KYC con proveedor externo, tesorería y regalías en el Backoffice, soporte NFC (Web NFC en Android; iOS requiere app nativa), API de POS para cadenas.

## 3. Dependencias entre etapas

```
Etapa 0 ─┬─► Etapa 1 (sitios públicos)  ──► Etapa 6
         ├─► Etapa 2 (ERP)  ────────────┐
         ├─► Etapa 3 (Marketplace) ◄────┼── modelo de Lote/Colección/Claim de 0
         ├─► Etapa 4 (Backoffice) ◄─────┘
         └─► Etapa 5 (POS) ◄── formato del pase de retiro definido en 3C
Spike Stellar (en 0) ──► 3A, 2F, 4C
```

Con dos personas: A hace Etapa 1 y luego Etapa 3; B hace Etapa 0 (paquetes) y luego Etapa 2, después Etapa 4; Etapa 5 la toma quien termine antes. Con tres personas, la tercera toma Etapa 0 y Etapa 5 y hace la integración.

## 4. Datos de prueba

- Un único juego de fixtures coherente en `doc-mocks/fixtures/*.json`: las bodegas de la landing son las bodegas del ERP y del Backoffice; los lotes del ERP son las colecciones del Marketplace; los claims del Marketplace aparecen en el historial del POS.
- Cada app corre MSW en el navegador en desarrollo y en las pruebas; en producción de demostración se despliega con MSW activo detrás de un flag `NEXT_PUBLIC_MOCKS=1` para enseñar el producto sin backend.
- Los QR generados por el ERP mock son escaneables por el Marketplace mock (mismo dominio de demostración).

## 5. Definición de terminado por pantalla

Diseñada en los dos temas si aplica · móvil y escritorio · estados vacío, cargando, error · textos en ES (EN solo landing y S2) · accesible por teclado y con lector de pantalla · sin errores de consola · cubierta por la prueba de flujo de su etapa · documentada en Storybook si introduce un componente nuevo.

## 6. Riesgos principales

| Riesgo | Mitigación |
|---|---|
| El backend define un modelo distinto al de los mocks | Compartir `doc-mocks` con backend desde la Etapa 0 y negociar el OpenAPI sobre él |
| Passkeys no disponibles en dispositivos antiguos de la tribu | Plan B con login social (Privy/Web3Auth) detrás de la misma API de `doc-wallet`; ver 04 |
| Pasarela de pago sin definir | Checkout abstraído con un adaptador; mock hasta la decisión |
| Cámara en tablets baratas | Probar hardware real en la Etapa 5; fallback de entrada manual del código |
| Cuatro apps y un equipo pequeño | Plantilla común, design system, una sola configuración de CI |
