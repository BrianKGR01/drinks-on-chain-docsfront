# 05 · Sistema de diseño compartido

Se extrae de la landing (`drinks-on-chain-front/src/app/globals.css` y los módulos CSS) y se publica como `@doc/ui`. Aquí se fija el alcance; la implementación es la Etapa 0 del roadmap. El detalle por sistema (ajustes de tema y componentes específicos) se hará en un documento posterior, como pidió el cliente.

## 1. Tokens

### Color
| Token | Oro Líquido (claro) | Cava Reserva (oscuro) | Uso |
|---|---|---|---|
| `--paper` | `#fdfcf5` | `#15120f` | Fondo de página |
| `--paper-2` / `--paper-3` | `#f9f6ee` / `#f4efe2` | `#1c1814` / `#241f1a` | Superficies, barras |
| `--ink` | `#000000` | `#f3eee2` | Texto principal |
| `--ink-soft` / `--muted` | `#464340` / `#625e54` | `#c9c1b3` / `#9a9184` | Texto secundario |
| `--line` / `--hairline` | `#a0a095` / `rgba(0,0,0,.11)` | `#5a524a` / `rgba(255,255,255,.12)` | Reglas y bordes |
| `--accent` | `#b8891f` | `#d3a642` | Oro: selección, títulos |
| `--accent-deep` | `#8a651a` | `#e6bf63` | Oro para texto pequeño (contraste) |
| `--accent-soft` | `rgba(184,137,31,.14)` | `rgba(211,166,66,.18)` | Fondos de resaltado |
| `--success` / `--danger` / `--warning` | `#3f7d4a` / `#b3402c` / `#b07a1a` | `#7fc48b` / `#e0705b` / `#d9a441` | Solo estados operativos (ERP, POS); nunca decorativos |

El rojo desaparece como acento de marca (decisión del cliente); vuelve únicamente como color de peligro en botones destructivos y en el semáforo rojo del POS, donde el significado es literal.

### Tipografía
- Display: Cormorant Garamond 400/500/600 (títulos, etiquetas espaciadas, navegación).
- Texto: EB Garamond 400/500 (párrafos, tablas de lectura).
- Utilidad (nuevo): Inter o IBM Plex Sans 400/500/600 para tablas densas, inputs numéricos, PIN y semáforos del POS, donde la legibilidad a distancia manda. Uso restringido a S1, S3 y S4.
- Escala: la landing usa raíz fluida (`.8333vw` entre 768 y 1920 px). Las aplicaciones usan raíz fija de 16 px y una escala modular 1.2 (12, 14, 16, 20, 24, 29, 35, 42 px), porque las herramientas de trabajo no deben cambiar de tamaño con la ventana.

### Espaciado, forma y movimiento
- Espaciado en múltiplos de 4 px (4–64) más `--gutter` de 16/24/32 según breakpoint.
- Radio 0 por defecto (herencia editorial); 4 px en inputs y tarjetas de datos; 999 px en pills y el indicador del sonido.
- Sombras: ninguna en superficies; un solo "halo de papel" (`box-shadow` con el color del papel) para etiquetas sobre imagen o mapa.
- Motion: `--ease-out: cubic-bezier(.22,1,.36,1)`, `--ease-in-out: cubic-bezier(.65,0,.35,1)`; duraciones 150 (micro), 300 (transición), 600 (revelado), 1600 (velo). `prefers-reduced-motion` respetado en todos los componentes.

### Breakpoints
`sm 375`, `md 768`, `lg 1024`, `xl 1440`. S2 se diseña a 375 y 768; S4 a 1024×768 y 1194×834 (tablets); S1 y S3 a 1280 y 1440.

## 2. Componentes

### Base (todos los sistemas)
Button (primario de texto con subrayado, secundario con borde, destructivo, tamaños m/l/xl táctil), IconButton, TextLink, Input (línea inferior; variante numérica gigante), Select, Textarea, Checkbox, Radio, Switch, Field (etiqueta + ayuda + error), Badge/Pill, Tag, Card, Divider, Avatar, Tooltip, Toast, Skeleton, EmptyState, ErrorState, Spinner, Progress, Tabs, Breadcrumbs, Pagination, SlideOver, Modal, BottomSheet, Popover/Menu, DataTable (ordenación, filtros, densidad), StatCard, Timeline, Stepper, KeyValueList, QRCode, Countdown.

### Editoriales (landing, S2 visor, perfiles de bodega)
PageShell, SmallHeading + HeadingSeparator, TextHeading, Prose, Highlight, Specifications, InkPhoto, BottleIllustration, SoilProfile, VineOrnament, GlassBottleOrnament, DiscoverFooter, LetterSplit, AgeGate.

### Específicos por sistema (se definen en sus etapas)
S1: BigNumberInput, TankGrid, CountdownLock, DecisionModal, LotTimeline, LabReadingCard. S2: BottleCard, StickyBuyBar, CheckoutSheet, VaultSplash, ClaimTicket, JourneyTimeline, StarRating, CameraScanner. S3: KanbanBoard, MintReviewModal, TxStatusBadge, TicketSplitView, GlobalSearch. S4: PinPad, CameraScanner (compartido con S2), SwipeToConfirm, TrafficLightOverlay, ShiftReceipt.

## 3. Reglas de uso

1. El oro es escaso: un elemento seleccionado, un título, un CTA por pantalla. Nunca fondos dorados grandes.
2. Todo lo que se puede dibujar a línea se dibuja a línea (iconos de trazo fino, ilustraciones SVG), no con relleno.
3. Las fotos van con marco de hairline, mezcla multiplicar y crédito visible.
4. Contraste mínimo AA (4,5:1 texto normal; 3:1 texto grande y UI). `--accent-deep` existe por eso.
5. Objetivos táctiles ≥ 44 px (≥ 56 px en S1 táctil y S4).
6. Sin `border-radius` grande ni sombras difusas: la identidad es papel e imprenta.

## 4. Entrega

- Repo `doc-design-system` con Storybook (casos claro/oscuro, móvil/escritorio), pruebas visuales básicas, `CHANGELOG` y versionado semántico; publicación en GitHub Packages.
- Tokens exportados como CSS custom properties y como preset de Tailwind 4 (`@theme`).
- Cada app fija la versión y actualiza con PR.
