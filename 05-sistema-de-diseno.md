# 05 · Sistema de diseño Drinks on Chain

Versión 2 · 25 de septiembre de 2026 (v1 en `antiguo/`). Especifica el sistema de diseño compartido por los dos sitios públicos y las cuatro aplicaciones, y la adaptación por sistema. Las maquetas navegables están en `design-system/`:

| Archivo | Qué muestra |
|---|---|
| `design-system/tokens.css` | Los tokens reales (primitivos, semánticos, dos temas) tal como irán en `@doc/ui` |
| `design-system/00-fundamentos.html` | Paleta, tipografía, escala, espaciado, forma, movimiento, iconografía; componentes base en los dos temas |
| `design-system/01-erp.html` | ERP: shell con barra lateral, dashboard, formularios de planta, tanques, candados, tablas |
| `design-system/02-marketplace.html` | Marketplace: shell móvil con pestañas, escaparate, ficha, checkout, cava, pase de retiro, visor |
| `design-system/03-backoffice.html` | Backoffice: shell denso con barra lateral oscura, KPI, tablas, kanban, modal de emisión, helpdesk |
| `design-system/04-pos.html` | POS: PIN, escáner, semáforos, deslizador, recibo de turno |

Se abren directamente en el navegador (archivos estáticos, tipografías de Google Fonts). Son la referencia visual para construir `@doc/ui`; no son código de producción.

## 1. Idea rectora

La identidad ya existe en las dos landings: **papel, tinta y un solo oro**; tipografía serif con mucho aire; líneas finas en vez de rellenos; nada de sombras difusas ni esquinas redondeadas. El feedback sobre ella ha sido bueno y se mantiene.

Las aplicaciones no pueden ser "dibujadas": son herramientas de trabajo. Por eso el sistema tiene **dos familias** que comparten tokens y componentes pero difieren en tipografía de trabajo, densidad y decoración:

| Familia | Sistemas | Carácter | Tipografía de interfaz | Decoración |
|---|---|---|---|---|
| **Editorial** | Landing principal, sitio de bodegas, Marketplace (escaparate, ficha, visor, cava) | Revista, club, lujo silencioso | Cormorant Garamond (display) + EB Garamond (texto) | Ornamentos a tinta, fotografías con marco, separadores, letra espaciada |
| **Operativa** | ERP, Backoffice, POS; y dentro del Marketplace el checkout, los formularios y los ajustes | Instrumento claro, legible a distancia, sin adornos | Inter (interfaz y números, con cifras tabulares) + Cormorant Garamond solo en wordmark y títulos de página | Ninguna: la jerarquía la dan el espacio, el peso y el oro escaso |

Regla de convivencia: la familia operativa hereda la paleta, el radio, la hairline y el oro de la editorial; la editorial nunca importa tablas densas ni sans. Un usuario que pasa del Marketplace al POS reconoce la marca, pero cada herramienta se comporta como lo que es.

## 2. Tokens

Tres capas: **primitivos** (valores), **semánticos** (rol: fondo, texto, borde, acento, estado) y **de componente** (solo donde hace falta). Las aplicaciones consumen semánticos; los temas cambian primitivos detrás. Nombres con prefijo `--doc-` en CSS y expuestos a Tailwind 4 con `@theme`.

### 2.1 Color

Primitivos (los de las landings, más una rampa de grises cálidos y los estados operativos):

| Primitivo | Valor | Origen |
|---|---|---|
| `paper-0/1/2/3` | `#fdfcf5` · `#f9f6ee` · `#f4efe2` · `#ece5d3` | Landings (`--paper`, `--paper-2`, `--paper-3`) + un escalón más para barras y fondos de tabla |
| `ink-0/1/2/3` | `#000000` · `#464340` · `#625e54` · `#a0a095` | Landings (`--ink`, `--ink-soft`, `--muted`, `--line`) |
| `gold-500/700/300` | `#b8891f` · `#8a651a` · `#d3a642` | Landings (`--accent`, `--accent-deep`) + oro claro para fondos oscuros |
| `cava-0/1/2/3` | `#15120f` · `#1c1814` · `#241f1a` · `#2e2823` | Tema oscuro "Cava Reserva" |
| `cream-0/1/2` | `#f3eee2` · `#c9c1b3` · `#9a9184` | Texto sobre cava |
| `green-600/300` | `#3f7d4a` · `#7fc48b` | Éxito / aprobado |
| `red-600/300` | `#b3402c` · `#e0705b` | Peligro / rechazado |
| `amber-600/300` | `#b07a1a` · `#d9a441` | Aviso / candado |
| `blue-600/300` | `#3b5f8a` · `#8fb0d9` | Información / en curso (necesario en tablas y estados de transacción) |

Semánticos por tema (los que usan los componentes):

| Semántico | Oro Líquido (claro) | Cava Reserva (oscuro) | Uso |
|---|---|---|---|
| `bg` / `bg-raised` / `bg-sunken` | `paper-0` / `paper-1` / `paper-2` | `cava-0` / `cava-1` / `cava-2` | Página / tarjetas y barras / fondos de tabla e inputs |
| `fg` / `fg-muted` / `fg-subtle` | `ink-0` / `ink-1` / `ink-2` | `cream-0` / `cream-1` / `cream-2` | Texto principal / secundario / ayuda |
| `border` / `border-strong` | `rgba(0,0,0,.11)` / `ink-3` | `rgba(255,255,255,.12)` / `#5a524a` | Hairline / separadores fuertes |
| `accent` / `accent-fg` / `accent-text` / `accent-soft` | `gold-500` / `paper-0` / `gold-700` / `rgba(184,137,31,.14)` | `gold-300` / `cava-0` / `#e6bf63` / `rgba(211,166,66,.18)` | Selección, CTA / texto sobre oro / oro para texto pequeño (AA) / fondos de resaltado |
| `success` / `danger` / `warning` / `info` (+ `-soft`) | `green-600` … | `green-300` … | Solo estados operativos; nunca decorativos |
| `focus` | `gold-500` | `gold-300` | Anillo de foco de 2 px con desplazamiento |
| `overlay` | `rgba(21,18,15,.55)` | `rgba(0,0,0,.65)` | Fondo de modales y sheets |

Reglas: el oro es escaso (un elemento seleccionado, un título, un CTA por pantalla; nunca fondos dorados grandes). El rojo solo significa peligro o rechazo. Todo texto cumple AA (4,5:1; 3:1 en texto grande e iconos): por eso existe `accent-text` y por eso los estados tienen variante `-soft` para fondos con texto en el color fuerte.

### 2.2 Tipografía

| Rol | Familia | Pesos | Dónde |
|---|---|---|---|
| Display | Cormorant Garamond | 400, 500, 600 (+ itálica) | Títulos editoriales, wordmark, títulos de página en apps, cifras destacadas del Marketplace |
| Texto editorial | EB Garamond | 400, 500 (+ itálica) | Párrafos de landings, Marketplace y visor |
| Interfaz | Inter | 400, 500, 600 | Toda la interfaz operativa: menús, tablas, formularios, botones, badges, cifras (`font-variant-numeric: tabular-nums`) |

Escalas (raíz fija de 16 px en aplicaciones; las landings mantienen su escala propia):

| Token | px | Uso |
|---|---|---|
| `text-2xs` | 11 | Solo etiquetas de tabla en mayúsculas espaciadas |
| `text-xs` | 12 | Ayudas, metadatos |
| `text-sm` | 14 | Cuerpo denso (Backoffice, tablas) |
| `text-md` | 16 | Cuerpo por defecto (ERP, formularios, Marketplace) |
| `text-lg` | 18 | Cuerpo cómodo, tablets |
| `text-xl` | 20 | Subtítulos |
| `text-2xl` | 24 | Títulos de sección |
| `text-3xl` | 30 | Títulos de página (display) |
| `text-4xl` | 36 | Cifras de KPI |
| `text-5xl` | 48 | Input gigante de báscula, cuenta regresiva |
| `text-6xl` | 64 | POS: cantidad a entregar |
| `text-7xl` | 96 | POS: PIN, semáforo |

Interlineado 1.2 en display, 1.5 en texto, 1.0 en cifras gigantes. Letra espaciada (`0.25em`–`0.4em`, mayúsculas) solo en la familia editorial y en las etiquetas de sección de las apps.

### 2.3 Espaciado, forma, elevación y movimiento

- **Espaciado** en múltiplos de 4: `space-1…20` = 4, 8, 12, 16, 20, 24, 32, 40, 48, 64, 80. Gutter de página 16 / 24 / 32 según ancho.
- **Radio**: `none` (editorial), `sm` 2 px (badges), `md` 4 px (inputs, botones, tarjetas de datos), `lg` 8 px (modales, sheets, tarjetas del Marketplace), `full` (pills, avatares, el punto de estado).
- **Bordes**: hairline de 1 px (`border`); nunca 2 px salvo el anillo de foco y el semáforo del POS.
- **Elevación**: ninguna en superficies; `shadow-overlay` (una sola sombra suave) para modales, sheets, menús flotantes y el "halo de papel" de etiquetas sobre imagen.
- **Movimiento**: `ease-out cubic-bezier(.22,1,.36,1)`, `ease-in-out cubic-bezier(.65,0,.35,1)`; duraciones 120 (micro), 200 (estado), 320 (transición), 600 (revelado editorial). `prefers-reduced-motion` respetado.
- **Objetivos táctiles**: ≥ 44 px en el Marketplace; ≥ 56 px en pantallas táctiles del ERP; ≥ 72 px en el POS.
- **Breakpoints**: `sm 375`, `md 768`, `lg 1024`, `xl 1280`, `2xl 1440`.
- **Capas (z-index)**: `base 0`, `sticky 10`, `dropdown 20`, `overlay 30`, `modal 40`, `toast 50`.

### 2.4 Iconografía

Un solo set de trazo fino (1,5 px a 24 px), sin rellenos: Lucide como base por su cobertura y licencia, con iconos propios a tinta para el dominio (botella, copa, barrica, alambique, tanque, racimo, sello). Tamaños 16 / 20 / 24 / 32; en el POS 48 / 96.

## 3. Componentes

Inventario en tres niveles. Cada componente se documenta en Storybook en los dos temas y en móvil / escritorio.

### 3.1 Base (todos los sistemas)
Button (primario en oro, secundario con borde, terciario de texto, destructivo; tamaños sm / md / lg / xl táctil; con icono; cargando), IconButton, TextLink, Input (con prefijo/sufijo; variante `numeric` y `giant`), Textarea, Select, Combobox, Checkbox, Radio, Switch, Field (etiqueta + ayuda + error + obligatorio), FormSection, Badge (estado con punto de color; variantes soft y strong), Tag, Pill (filtro), Avatar, Tooltip, Toast, Alert (inline), Skeleton, Spinner, Progress, EmptyState, ErrorState, Divider, Card, KeyValueList, Tabs, Breadcrumbs, Pagination, DataTable (ordenación, filtros, densidad cómoda / compacta, selección, acciones por fila, estados vacío y cargando), Modal, SlideOver, BottomSheet, Popover, Menu, CommandPalette (buscador global), Stepper, Timeline, StatCard, Countdown, QRCode, CameraScanner (compartido por Marketplace y POS), Wordmark, ThemeProvider.

### 3.2 Editoriales (landings, Marketplace)
PageShell, SmallHeading + HeadingSeparator, TextHeading, Prose, Highlight, Specifications, InkPhoto, BottleIllustration, SoilProfile, VineOrnament, GlassBottleOrnament, DiscoverFooter, LetterSplit, AgeGate, MapPreview.

### 3.3 Específicos por sistema
- **ERP**: AppShell (sidebar + topbar), LotStatusBadge, LotTimeline, BigNumberInput (báscula), LabReadingCard, TankGrid + TankCard, DecisionModal (bifurcación), CountdownLock, BarrelRow, StillCutsForm, BottlingSummary, QrExportCard, DoBadge (aptitud D.O.), WineryAccountPanel.
- **Marketplace**: StoreHeader, BottomTabs, HeroBanner, BottleCard, PriceTag, StickyBuyBar, CheckoutSheet (pasos: cantidad → pago → confirmación), OrderStatus, VaultSplash, CavaGrid + HoldingCard, ClaimTicket (QR + caducidad + punto), PickupPointPicker, ScanGate, JourneyTimeline, TastingCards, StarRating, ReviewForm, WalletSettings.
- **Backoffice**: AdminShell (sidebar oscura + topbar clara), KpiCard, AlertsFeed, WineryTable, WineryDrawer, CredentialsDialog, PickupPointForm, DeviceEnrollCard, MintPipeline (kanban), MintReviewModal, TxStatusBadge (pendiente / enviada / confirmada / fallida, con hash), CollectionCard, TicketTable, TicketSplitView, HoldingsLookup, RoleMatrix.
- **POS**: KioskShell, PinPad, ScannerViewport (retícula, linterna), TrafficLightOverlay (verde / rojo), SwipeToConfirm, DeliverySuccess, ShiftReceipt, OfflineBanner, ManualCodeEntry.

## 4. Layouts estándar

| Shell | Sistemas | Estructura | Anchos |
|---|---|---|---|
| **AppShell** | ERP | Sidebar fija 264 px (colapsable a 72 px en tablet) con wordmark serif, navegación por módulos y usuario abajo; topbar 56 px con migas de pan y acción principal a la derecha; contenido con gutter 24–32 px y ancho máximo 1280 px | 1024–1440 |
| **AdminShell** | Backoffice | Igual que AppShell pero sidebar en tema Cava Reserva, topbar con buscador global (⌘K) y notificaciones; contenido en tema claro, ancho máximo 1440 px, tablas a sangre | 1280–1440+ |
| **StoreShell** | Marketplace | Móvil: cabecera compacta + pestañas inferiores 64 px (Inicio, Escáner, Cava, Perfil) + área con `safe-area`; escritorio: cabecera transparente con Catálogo, Mi Cava, Entrar/Perfil; contenido máximo 1200 px | 375 / 768 / 1200 |
| **KioskShell** | POS | Pantalla completa apaisada, sin navegación; una acción por pantalla; barra de estado fina arriba (sucursal, conexión, hora); botón secundario abajo | 1024×768, 1194×834 |
| **AuthLayout** | ERP (dividido: imagen + formulario), Backoffice (centrado, mínimo), Marketplace (pantalla única "Entrar" centrada sobre imagen velada), POS (PIN centrado sobre cava) | — | — |
| **PageShell** | Landings y páginas editoriales del Marketplace (visor, historia de la bodega) | Columna editorial centrada, separadores, pie de descubrimiento | — |

Estados obligatorios en cada pantalla: cargando (skeleton, no spinner a pantalla completa), vacío (EmptyState con acción), error (ErrorState con reintento), sin conexión donde aplique (Marketplace y POS).

## 5. Adaptación por sistema

### ERP · tema Oro Líquido, familia operativa, densidad cómoda
Sensación: cuaderno de bitácora limpio. Inter 16 px, tablas cómodas, formularios con etiquetas encima y ayuda debajo. Pantallas táctiles (pesaje, análisis, bitácora) con objetivos ≥ 56 px y el input gigante en display 48–64 px. El oro marca el lote seleccionado, el badge D.O. y el botón principal. Verde/rojo solo en aprobar/rechazar y en el semáforo de tanques; ámbar en candados. Sin fotografías salvo en el login.

### Marketplace · tema Oro Líquido, mezcla editorial + operativa, mobile-first
Escaparate, ficha, cava y visor en familia editorial (Cormorant + EB Garamond, fotografías con marco, mucho aire, tarjetas con radio `lg`). Checkout, pase de retiro, perfil y ajustes en familia operativa (Inter, campos claros, pasos numerados). Bottom sheets para no perder contexto. El oro en el precio, el CTA persistente y el estado "listo para retirar". PWA instalable, `safe-area` respetada, escáner con permiso explicado antes de pedirlo.

### Backoffice · barra lateral Cava Reserva, contenido Oro Líquido, densidad compacta
Herramienta intensiva de escritorio: Inter 14 px, tablas compactas con cabeceras pegajosas, filtros como pills, paginación, columnas ordenables, acciones por fila en menú. KPI con cifra display 36 px. Kanban con tarjetas mínimas (lote, bodega, botellas, fecha). Modal de emisión al 80 % con bloque de solo lectura y bloque comercial separados por una hairline; el botón "Aprobar, emitir y publicar" es el único oro de la pantalla y pide confirmación. Estados de transacción con TxStatusBadge (info = pendiente/enviada, success = confirmada, danger = fallida).

### POS · tema Cava Reserva completo, familia operativa, densidad gigante
Un fondo oscuro que no deslumbra, texto crema, Inter 24 px de base, cantidad a entregar en 64–96 px. Semáforo verde y rojo a pantalla completa con borde de 16 px y icono de 96 px. Deslizador de 72 px de alto. PIN con teclas de 96 px. Un botón secundario por pantalla, abajo. Sin menús, sin tablas salvo el recibo de turno (lista monoespaciada a alto contraste). Todo legible a un metro.

## 6. Accesibilidad y calidad
- Contraste AA en todos los pares; AAA en el POS (7:1) por la lectura a distancia.
- Foco visible siempre (anillo `focus` de 2 px con desplazamiento de 2 px).
- Navegación por teclado completa en ERP y Backoffice; atajos: `/` buscar, `⌘K` paleta, `Esc` cerrar.
- `aria-live` en toasts, estados de transacción y semáforos.
- Tamaños de fuente en `rem`; las apps no reducen el texto con la ventana.
- Pruebas visuales por componente en los dos temas; Lighthouse accesibilidad ≥ 95.

## 7. Entrega e implementación
- Repo `doc-design-system` publicado como `@doc/ui` (GitHub Packages): `tokens.css` (custom properties + `@theme` de Tailwind 4), componentes React con `class-variance-authority`, Storybook con casos claro/oscuro y móvil/escritorio, pruebas visuales, `CHANGELOG`, versionado semántico.
- Primitivas accesibles de base para overlays, menús, tabs y combobox (Radix UI o Base UI) envueltas con nuestros tokens; nada se expone sin envolver.
- Cada aplicación fija la versión y actualiza con PR. Las landings migran a `@doc/ui` cuando el paquete exista (hoy usan copias).
- Orden de construcción en la Etapa 0: tokens y temas → Button, Input, Field, Badge, Card, DataTable, Modal, SlideOver, BottomSheet, Toast, EmptyState → AppShell y StoreShell → específicos de cada sistema en su etapa.
