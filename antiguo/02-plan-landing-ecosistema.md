# 02 · Plan de la landing del ecosistema (v2, aprobada)

Versión 2.1 · 24 de septiembre de 2026 · Aprobada por el cliente con tres ajustes: el subdominio B2B es `bodegas.` (no `territorio.`), el Marketplace se navega sin cuenta y solo exige registro al comprar, y el POS no tiene página pública (se provisiona desde el Backoffice). Sustituye a la v1 (una portada con cuatro "puertas"), descartada por dos razones: obliga al visitante a clasificarse antes de recibir valor y junta en un mismo origen públicos con niveles de confianza distintos. Esta versión separa por **público** y por **subdominio**.

## 1. Públicos y lo que cada uno viene a hacer

| Público | Quién es | Qué busca en la web | Nivel de confianza | Sistema destino |
|---|---|---|---|---|
| **B2C · La tribu** | Consumidor, coleccionista, cliente de restaurante que escaneó una botella | Entender qué es esto, ver vinos, comprar, guardar, retirar; leer la historia de una botella | Público anónimo, luego cuenta ligera | S2 Marketplace |
| **B2B · Bodegas y viñedos** | Dueño, enólogo, agrónomo | Ver su lugar en el mapa, entender qué gana, pedir el alta, entrar al ERP | Socio con contrato | S1 ERP |
| **B2B · Puntos de recojo** | Dueño o cajero de licorería o cava | Saber qué implica ser punto autorizado, pedir el alta, entrar al POS | Socio con contrato (lo habilita una bodega o Debro) | S4 POS |
| **Interno · Debro** | Equipo gestor | Operar | Máxima; no se publicita | S3 Backoffice |

Conclusiones que fijan el diseño:

1. La **landing principal** tiene un solo público primario: **el consumidor**. Es el que llega desde el QR de una botella, desde redes o desde prensa, y el que hay que convertir. Los socios B2B llegan por relación comercial, no por SEO; para ellos basta una ruta secundaria clara y estable.
2. **Bodegas y puntos de recojo son el mismo mundo comercial** (la bodega habilita el lote; el punto lo entrega; Drinks on Chain no gestiona el lote como plataforma). Comparten sitio, tono y formulario de contacto, pero **no comparten autenticación**: cada uno entra a su propia aplicación.
3. **Ninguna landing autentica a nadie.** Las landings son sitios estáticos sin sesión, sin cookies de identidad y sin formularios de login. Autenticarse siempre significa saltar al subdominio del sistema correspondiente. Esto elimina de raíz el riesgo de mezclar públicos en un mismo acceso.
4. El **POS no aparece en la landing principal**. Su acceso vive en el sitio B2B (pie y sección "Puntos de recojo") y en la comunicación directa con los socios.
5. El **Backoffice no se enlaza desde ningún sitio público.**

## 2. Mapa de dominios

Dominio raíz por decidir (propuesta: `drinksonchain.bo` para Bolivia, con `drinksonchain.com` redirigiendo). Todo lo demás son subdominios del mismo dominio, cada uno un despliegue independiente con su propio origen, cookies y cabeceras de seguridad.

| Subdominio | Qué es | Repo | Autenticación | Enlazado desde |
|---|---|---|---|---|
| `drinksonchain.bo` | **Landing principal** (B2C primero, ruta B2B secundaria) | `drinks-on-chain-landing` (nuevo, ligero, sin WebGL) | Ninguna | Redes, prensa, boca a boca |
| `bodegas.drinksonchain.bo` | **Sitio B2B de las bodegas**: el mapa grabado actual, bodegas, parcelas, puntos de recojo, alta de socios | `drinks-on-chain-front` (el actual) | Ninguna; enlaza a `erp.` y `pos.` | Landing principal ("Para bodegas y puntos de venta") y relación comercial |
| `app.drinksonchain.bo` | **Marketplace + visor QR + cava** (S2) | `doc-marketplace-app` | **Navegación libre sin cuenta**; la cuenta ligera (y su billetera con passkey) se crea al comprar o cuando el usuario lo decide desde "Entrar" | Landing principal; el QR de cada botella apunta aquí (`app.…/b/{código}`) |
| `erp.drinksonchain.bo` | **ERP de trazabilidad** (S1) | `doc-erp-web` | Solo inicio de sesión (correo + contraseña, roles); los usuarios los crea Debro desde el Backoffice; no hay registro público | Sitio de bodegas ("Acceso bodegas") |
| `pos.drinksonchain.bo` | **Aplicación de claim** (S4) | `doc-claim-pos` | Dispositivo autorizado + PIN de sucursal; el punto lo da de alta Debro desde el Backoffice, enlazado a una bodega y a los lotes que puede entregar | Sitio de bodegas ("Acceso puntos de recojo"), correo de alta; sin página pública |
| `admin.drinksonchain.bo` | **Backoffice** (S3) | `doc-backoffice-web` | Usuario interno + 2FA, lista de IP opcional | Nadie; URL interna |

Por qué subdominios y no rutas de un mismo sitio: cada origen tiene su propia sesión (una cookie de `app.` no vale en `erp.`), su propia política de seguridad de contenido, su propio ritmo de despliegue y su propio presupuesto de rendimiento (la landing no carga el escáner de cámara; el POS no carga WebGL). Cuando un socio comparte su tablet o su portátil, nada de un sistema se filtra a otro.

Nota sobre el QR físico: el código impreso en la etiqueta apunta directamente al Marketplace (`app.`), no a la landing, para que el consumidor caiga en el scan gate con un solo salto. La landing principal existe para quien llega sin botella en la mano.

## 3. Alternativas evaluadas

| Opción | Descripción | Ventajas | Inconvenientes | Veredicto |
|---|---|---|---|---|
| A · **Una landing B2C con ruta B2B + sitio de bodegas + apps por subdominio** | Lo descrito arriba | Un mensaje claro por sitio; el mapa conserva su público natural; seguridad por origen; cada pieza crece sola | Dos sitios públicos que mantener (comparten design system) | **Recomendada** |
| B · Dos landings totalmente separadas (una marca de consumo y otra "for business") en dominios distintos | Estilo empresa grande (consumo vs. empresas) | Máxima separación | Divide la marca cuando aún no existe; duplica contenido y SEO; más caro | No ahora; posible en Fase 2 si el B2B crece |
| C · Una sola landing con selector de público ("¿Eres bodega o consumidor?") | La v1 | Un solo despliegue | Puerta de decisión antes del valor; SEO confuso; mezcla públicos; el POS queda expuesto | Descartada |
| D · El mapa actual como landing principal | Mantener lo que hay | Ya existe y es memorable | Habla a bodegas, no a consumidores; ninguna conversión B2C clara | Descartada, pero el mapa queda como sitio B2B y como imagen de marca |

## 4. Landing principal (`drinksonchain.bo`)

Objetivo único: que un consumidor entienda en diez segundos qué es Drinks on Chain y dé el salto al Marketplace. Objetivo secundario: que una bodega o licorería encuentre su camino en un clic desde el menú.

### Principios de UX aplicados
- **Un público primario por página**; el secundario tiene un enlace persistente en la navegación ("Para bodegas y puntos de venta"), como hacen las plataformas que venden a consumidores y a comercios sin partir su portada en dos.
- **El valor antes que la clasificación**: nadie elige quién es; simplemente lee y actúa.
- **Un CTA principal por pantalla**, en oro; los secundarios son enlaces de texto con subrayado.
- **Prueba antes de promesa**: se muestran vinos y bodegas reales (datos mock alineados con el Marketplace), no adjetivos.
- **Barrera de edad** una vez por sitio público de consumo (obligación legal), con recuerdo en el navegador.
- **Móvil primero**: la mayoría llega desde el teléfono.

### Estructura de la página de inicio

```
┌──────────────────────────────────────────────────────────────┐
│ [Wordmark]         Vinos · Cómo funciona · Bodegas · Historia │
│                     Para bodegas y puntos de venta   [Entrar] │
├──────────────────────────────────────────────────────────────┤
│ HÉROE                                                        │
│ Mapa grabado derivando detrás de un velo al 70 %             │
│ "Cada botella, con su lugar y su historia."                   │
│ Vinos y singanis de altura, verificados de la parcela a la    │
│ copa. Cómpralos a precio de bodega y retíralos donde quieras. │
│ [ Explorar los vinos ]   Escaneé una botella →               │
├──────────────────────────────────────────────────────────────┤
│ CÓMO FUNCIONA (cuatro viñetas a tinta, en fila)              │
│ Escanea · Descubre · Adquiere · Retira                       │
├──────────────────────────────────────────────────────────────┤
│ VINOS EN LA RED (tarjetas: botella, bodega, valle, precio)    │
│ → "Ver todo en el Marketplace"                               │
├──────────────────────────────────────────────────────────────┤
│ LAS BODEGAS (dibujo ligero del mapa + tres bodegas)           │
│ "¿Tienes una bodega o un viñedo?" → "Conocer las bodegas"    │
│   (bodegas.)                                                  │
├──────────────────────────────────────────────────────────────┤
│ CONFIANZA: qué garantizamos (trazabilidad, edición limitada, │
│ retiro seguro), con enlace a "Tecnología"                    │
├──────────────────────────────────────────────────────────────┤
│ FRANJA B2B (discreta, fondo papel-2)                          │
│ "¿Haces vino o singani? ¿Tienes una licorería?" → bodegas. │
├──────────────────────────────────────────────────────────────┤
│ PIE: Historia · Contacto · Aviso legal · Privacidad ·         │
│ ES/EN · Consumo responsable · Redes                           │
└──────────────────────────────────────────────────────────────┘
```

El botón **[Entrar]** de la cabecera es el único que apunta a una aplicación: `app.`, que abre una sola pantalla de "crear cuenta o iniciar sesión". Es opcional: el Marketplace se recorre entero sin cuenta (catálogo, fichas, dinámica de preventa y retiro); la cuenta se exige en el momento de comprar. El escaneo de una botella física sigue exigiendo registro (scan gate), como fija el documento maestro. Ningún formulario de credenciales existe en este sitio.

### Rutas
| Ruta | Contenido |
|---|---|
| `/` | Lo anterior |
| `/vinos` | Vitrina de vinos y singanis (la actual, movida aquí) con "Adquirir" → `app.` |
| `/como-funciona` | El recorrido del consumidor paso a paso, con capturas del Marketplace (mock) y preguntas frecuentes |
| `/bodegas` | Página "Bodegas": dibujo ligero del mapa (SVG generado desde la geometría, sin WebGL), lista breve de bodegas de la red y enlace al sitio completo en `bodegas.` |
| `/tecnologia` | Trazabilidad, Stellar, billetera sin frase semilla, qué datos guardamos |
| `/historia`, `/contacto`, `/aviso-legal`, `/privacidad` | Editoriales (las actuales) |
| `/b/[codigo]` | Redirección a `app.drinksonchain.bo/b/[codigo]` por si alguna etiqueta antigua apunta a la raíz |

### Qué se lleva del repo actual
Tokens, tipografías, `PageShell`, `InkPhoto`, ornamentos, `AgeGate`, `Logo`, `DiscoverFooter`, páginas Historia/Vinos/Contacto/Aviso legal y el contenido de vinos. **No** se lleva la escena WebGL completa: el héroe usa una versión ligera (un vídeo corto renderizado de la escena o un SVG animado del mapa), porque la landing principal debe cargar en menos de dos segundos en móvil.

## 5. Sitio de las bodegas (`bodegas.drinksonchain.bo`)

Es el sitio actual con foco explícito en la red de socios. Público primario: bodegas y viñedos; secundario: puntos de recojo y consumidores curiosos que vienen desde la landing.

### Estructura
| Ruta | Contenido |
|---|---|
| `/` | El mapa (experiencia actual) con el navegador de parcelas y una **capa de bodegas** |
| `/valles/[valle]/[parcela]` | Ficha de parcela (la actual `/parcelas/…`, con redirección) |
| `/bodegas` y `/bodegas/[slug]` | Perfil de bodega: historia, familia, parcelas en el mapa, productos, lotes con trazabilidad pública, estado en la red, "Acceso al ERP" |
| `/puntos-de-recojo` | Qué es un punto autorizado y cómo se habilita (lo pide la tienda o lo propone la bodega; Debro lo da de alta enlazado a una bodega y a sus lotes), lista de puntos activos, "Contactar" y "Acceso al POS" |
| `/unirse` | Propuesta para bodegas: beneficios, cómo es el ERP, requisitos D.O., proceso de alta, formulario de contacto (mock hasta tener backend) |
| `/acceso` | Página pequeña con dos tarjetas: "Soy bodega → erp." y "Soy punto de recojo → pos." No es un login: son enlaces con explicación de qué credenciales necesita cada uno |
| `/historia`, `/contacto`, `/aviso-legal` | Compartidas con la landing principal (mismo contenido, mismo pie) |

### Menú
Mapa · Bodegas · Puntos de recojo · Unirse · Acceso. El botón **Acceso** es el único que sale del sitio y lleva a `/acceso`, donde el socio elige su sistema. Así el "login" tiene un solo lugar, explícito, y nunca está en la misma pantalla que la información pública.

### Cambios en el mapa
- Conmutador "Parcelas / Bodegas" en el navegador inferior; en modo bodegas los marcadores son las sedes y al seleccionar una se encuadran sus parcelas.
- Relación parcela ↔ bodega en los datos (`bodegas.json`), con estados "Socia", "En conversación", "Referencia" (aparece por contexto, sin relación comercial y sin botón de compra).
- Los puntos de recojo activos pueden dibujarse como marcadores de otro tipo (una botella pequeña) cuando existan.

## 6. Seguridad y fronteras

```
[ drinksonchain.bo ]  ──enlace──►  [ app. ]  cuenta ligera + passkey (S2)
        │
        └─enlace─►  [ bodegas. ]  ──/acceso──►  [ erp. ]  bodegas (S1)
                                        └─────────►  [ pos. ]  puntos de recojo (S4)
                                                      [ admin. ]  interno, sin enlaces (S3)
```

- Las dos landings no tienen sesión, no guardan datos personales y no reciben credenciales. Los formularios de contacto y de alta envían a un endpoint del backend (o a un servicio de formularios) con protección anti-bots, nunca crean cuentas.
- Cada aplicación fija sus propias cabeceras (CSP estricta, `frame-ancestors 'none'`, cookies `HttpOnly; Secure; SameSite=Lax` limitadas a su subdominio).
- El Backoffice y el POS no aparecen en `sitemap.xml` ni en enlaces públicos; el POS además exige dispositivo autorizado por el Backoffice.
- Una identidad compartida entre sistemas (si backend la implementa con OIDC) no cambia nada de lo anterior: la sesión sigue siendo por subdominio.

## 7. Qué pasa con las páginas actuales

| Hoy en `drinks-on-chain-front` | Mañana |
|---|---|
| `/` mapa | `bodegas.` `/` |
| `/parcelas/[valle]/[parcela]` | `bodegas.` `/valles/[valle]/[parcela]` (redirección) |
| `/vinos` | Landing principal `/vinos`; el territorio conserva la ficha de vino dentro de cada parcela |
| `/historia`, `/contacto`, `/aviso-legal` | Ambos sitios (contenido compartido desde el paquete de contenido) |
| Menú actual (Historia · Vinos · Parcelas · Contacto) | Sitio de bodegas: Mapa · Bodegas · Puntos de recojo · Unirse · Acceso |

## 8. Fases

| Fase | Entregable | Repo | Estimación |
|---|---|---|---|
| L2.1 Datos | `bodegas.json`, `puntos-de-recojo.json`, relación parcela↔bodega, capa de datos mock alineada con `doc-mocks` | `drinks-on-chain-front` | 1 semana |
| L2.2 Sitio de bodegas | Menú nuevo, `/acceso`, `/unirse`, `/puntos-de-recojo`, redirecciones, capa de bodegas en el mapa, perfiles `/bodegas/[slug]` con mini-mapa | `drinks-on-chain-front` | 2,5 semanas |
| L2.3 Landing principal | Nuevo repo desde la plantilla común: inicio, `/como-funciona`, `/vinos`, `/bodegas`, `/tecnologia`, editoriales, héroe ligero, barrera de edad, SEO/OG, analítica | `drinks-on-chain-landing` | 2 semanas |
| L2.4 Dominios y despliegue | Compra del dominio, DNS, un proyecto de Vercel por subdominio, cabeceras de seguridad, redirecciones `.com → .bo`, `sitemap` y `robots` por sitio | ambos | 0,5 semana |
| L2.5 Calidad | Lighthouse ≥ 90 móvil en la landing principal, accesibilidad AA, textos ES/EN revisados | ambos | 0,5 semana |

Total: ~6,5 semanas de una persona; ~4 con dos (territorio y landing principal en paralelo desde la segunda semana). Comparten el paquete `@doc/ui` de la Etapa 0 y un paquete de contenido (`@doc/content`: bodegas, vinos, textos editoriales) para no duplicar datos.

## 9. Decisiones del cliente (24 de septiembre de 2026)

1. Opción A aprobada; subdominios `bodegas.`, `app.`, `erp.`, `pos.`, `admin.`.
2. El dominio raíz se comprará; los nombres de subdominio se mantienen sea cual sea la extensión.
3. Marketplace: navegación libre; cuenta obligatoria solo al comprar; "Entrar" disponible siempre en una sola pantalla de registro/inicio de sesión.
4. POS sin página pública: los puntos se habilitan a petición de la tienda o de la bodega y Debro los da de alta desde el Backoffice, enlazados a una bodega y a sus lotes.

El roadmap secuencial de la landing principal está en `07-roadmap-landing-principal.md`.
