# 02 · Sitios públicos del ecosistema: landing principal y sitio de bodegas

Versión 2.2 · 25 de septiembre de 2026. Plan aprobado el 24-09-2026 (v2.1, en `antiguo/`), actualizado con el estado real de los dos repositorios y con la marca única Drinks on Chain. Es la referencia de los dos sitios sin autenticación; el detalle de hitos de la landing principal está en `07-roadmap-landing-principal.md` y el del sitio de bodegas en `03-roadmap-frontend.md` §Sistema 0.

## 1. Públicos

| Público | Quién es | Qué busca en la web | Confianza | Destino |
|---|---|---|---|---|
| **B2C · La tribu** | Consumidor, coleccionista, cliente de restaurante que escaneó una botella | Entender qué es esto, ver vinos, comprar, guardar, retirar; leer la historia de una botella | Anónimo, luego cuenta ligera | S2 Marketplace |
| **B2B · Bodegas y viñedos** | Dueño, enólogo, agrónomo | Ver su lugar en el mapa, entender qué ganan, pedir el alta, entrar al ERP | Socio con contrato | S1 ERP |
| **B2B · Puntos de recojo** | Dueño o cajero de licorería o cava | Saber qué implica ser punto autorizado, pedir el alta, entrar al POS | Socio habilitado por una bodega o por Drinks on Chain | S4 POS |
| **Interno** | Equipo gestor de Drinks on Chain | Operar | Máxima; no se publicita | S3 Backoffice |

Principios que fijan el diseño:

1. La landing principal tiene un solo público primario: **el consumidor**. Los socios B2B llegan por relación comercial; para ellos basta una ruta secundaria estable ("Para bodegas y puntos de venta").
2. Bodegas y puntos de recojo son el mismo mundo comercial: comparten sitio, tono y formulario de contacto, pero **no comparten autenticación**.
3. **Ninguna landing autentica a nadie**: sin sesión, sin cookies de identidad, sin formularios de login. Autenticarse siempre significa saltar al subdominio del sistema.
4. El **POS no aparece en la landing principal**; su acceso vive en el sitio B2B.
5. El **Backoffice no se enlaza desde ningún sitio público**.

## 2. Mapa de dominios

Dominio raíz por comprar (propuesta `drinksonchain.bo` con `.com` redirigiendo). Todo lo demás son subdominios, cada uno un despliegue independiente con su propio origen, cookies y cabeceras.

| Subdominio | Qué es | Repo | Autenticación | Enlazado desde |
|---|---|---|---|---|
| raíz | Landing principal (B2C primero, ruta B2B secundaria) | `drinks-on-chain-landing` | Ninguna | Redes, prensa, boca a boca |
| `bodegas.` | Sitio B2B: mapa grabado, bodegas, parcelas, puntos de recojo, alta de socios | `drinks-on-chain-front` | Ninguna; enlaza a `erp.` y `pos.` | Landing principal y relación comercial |
| `app.` | Marketplace + visor QR + cava (S2) | `doc-marketplace-app` | Navegación libre; cuenta al comprar o al pulsar "Entrar" | Landing principal; el QR de cada botella apunta a `app./b/{código}` |
| `erp.` | ERP de trazabilidad (S1) | `doc-erp-web` | Correo + contraseña; usuarios creados desde el Backoffice | Sitio de bodegas ("Acceso") |
| `pos.` | Aplicación de claim (S4) | `doc-claim-pos` | Dispositivo vinculado + PIN de sucursal | Sitio de bodegas ("Acceso"), correo de alta; sin página pública |
| `admin.` | Backoffice (S3) | `doc-backoffice-web` | Usuario interno + 2FA | Nadie |

Mientras no exista el dominio, los enlaces entre sitios usan variables de entorno (`NEXT_PUBLIC_URL_APP`, `NEXT_PUBLIC_URL_BODEGAS`, y en el sitio de bodegas `NEXT_PUBLIC_URL_LANDING`, `NEXT_PUBLIC_URL_ERP`, `NEXT_PUBLIC_URL_POS`) con los hosts de Vercel como valores de producción.

## 3. Landing principal (raíz)

Objetivo único: que un consumidor entienda en diez segundos qué es Drinks on Chain y salte al Marketplace. Objetivo secundario: que una bodega o licorería encuentre su camino en un clic.

### Estado
| Elemento | Estado |
|---|---|
| Fundaciones (Next 16, tokens, tipografías, barrera de edad, cabecera, pie, ES/EN, variables de entorno, CI, `dev`/`main`) | Hecho |
| Inicio: héroe con mapa SVG, "Cómo funciona", "Vinos en la red", "Las bodegas", "Qué garantizamos", franja B2B | Hecho (primera versión) |
| Rutas `/vinos`, `/como-funciona`, `/bodegas`, `/tecnologia`, `/historia`, `/contacto`, `/aviso-legal`, `/privacidad`, `/b/[codigo]`, 404 | Hechas |
| Despliegue en Vercel con producción desde `main` y previews desde `dev` | Hecho |
| `sitemap.xml`, `robots.txt`, imagen OG por defecto | Pendiente |
| Cabeceras de seguridad (CSP, `frame-ancestors 'none'`, `Referrer-Policy`) | Pendiente |
| Lighthouse móvil ≥ 90 verificado; fuentes autoalojadas | Pendiente |
| Corrección de marca en pie, contacto y aviso legal (solo Drinks on Chain) | Pendiente |
| Analítica con consentimiento mínimo | Pendiente (decidir herramienta) |
| Dominio real y redirecciones `www` / `.com` | Pendiente de compra |

### Estructura de la página de inicio (vigente)
Cabecera (wordmark · Vinos · Cómo funciona · Bodegas · Historia · "Para bodegas y puntos de venta" · [Entrar]) → héroe → Cómo funciona (Escanea · Descubre · Adquiere · Retira) → Vinos en la red → Las bodegas → Qué garantizamos → franja B2B → pie (Historia · Contacto · Aviso legal · Privacidad · ES/EN · consumo responsable).

"Entrar" es el único botón que apunta a una aplicación (`app./entrar`, una sola pantalla de crear cuenta o iniciar sesión). Es opcional: el Marketplace se recorre entero sin cuenta.

## 4. Sitio de las bodegas (`bodegas.`)

Es el sitio actual (la experiencia del mapa) con foco explícito en la red de socios.

### Estructura objetivo
| Ruta | Contenido | Estado |
|---|---|---|
| `/` | Mapa con navegador de parcelas y conmutador "Parcelas / Bodegas" | Mapa hecho; conmutador pendiente |
| `/valles/[valle]/[parcela]` | Ficha de parcela (hoy `/parcelas/…`, con redirección) | Ficha hecha; renombrado y redirección pendientes |
| `/bodegas`, `/bodegas/[slug]` | Perfil de bodega: historia, parcelas en el mapa, productos, lotes con trazabilidad pública, estado en la red, "Acceso al ERP" | Pendiente |
| `/puntos-de-recojo` | Qué es un punto autorizado, cómo se habilita, lista de puntos activos, "Contactar", "Acceso al POS" | Pendiente |
| `/unirse` | Propuesta para bodegas: beneficios, cómo es el ERP, requisitos D.O., proceso de alta, formulario de contacto (mock hasta backend) | Pendiente |
| `/acceso` | Dos tarjetas: "Soy bodega → erp." y "Soy punto de recojo → pos.". No es un login | Pendiente |
| `/historia`, `/contacto`, `/aviso-legal` | Compartidas con la landing principal | Hechas; corrección de marca pendiente |
| `/vinos` | Pasa a enlazar a la landing principal | Pendiente |

Menú objetivo: Mapa · Bodegas · Puntos de recojo · Unirse · Acceso. Hoy: Historia · Vinos · Parcelas · Contacto.

### Cambios en los datos
- `bodegas.json` con relación parcela ↔ bodega y estados "Socia", "En conversación", "Referencia" (aparece por contexto, sin relación comercial y sin botón de compra). Mientras no haya acuerdos, **todas las bodegas son de referencia o de prueba**.
- `puntos-de-recojo.json` con puntos de prueba enlazados a bodegas.
- Ambos alineados con las entidades de `08-datos-de-prueba.md` para que el sitio de bodegas, el Marketplace y el Backoffice muestren la misma red.

## 5. Seguridad y fronteras

```
[ raíz ]  ──enlace──►  [ app. ]  cuenta ligera + billetera (S2)
   │
   └─enlace─►  [ bodegas. ]  ──/acceso──►  [ erp. ]  bodegas (S1)
                                   └─────────►  [ pos. ]  puntos de recojo (S4)
                                                 [ admin. ]  interno, sin enlaces (S3)
```

- Las dos landings no tienen sesión, no guardan datos personales y no reciben credenciales. Los formularios de contacto y de alta envían a un endpoint del backend (o a un servicio de formularios) con protección anti-bots; nunca crean cuentas.
- Cada aplicación fija sus propias cabeceras (CSP estricta, `frame-ancestors 'none'`, cookies `HttpOnly; Secure; SameSite=Lax` limitadas a su subdominio).
- Backoffice y POS no aparecen en `sitemap.xml` ni en enlaces públicos; el POS exige dispositivo vinculado desde el Backoffice.

## 6. Alternativas evaluadas (registro)

| Opción | Veredicto |
|---|---|
| A · Landing B2C con ruta B2B + sitio de bodegas + apps por subdominio | **Aprobada** |
| B · Dos landings separadas en dominios distintos | No ahora; posible en Fase 2 |
| C · Una landing con selector de público | Descartada (puerta de decisión antes del valor, mezcla públicos) |
| D · El mapa como landing principal | Descartada como principal; queda como sitio B2B e imagen de marca |

## 7. Decisiones vigentes

1. Subdominios `bodegas.`, `app.`, `erp.`, `pos.`, `admin.` bajo un dominio único por comprar.
2. Marketplace con navegación libre; cuenta obligatoria solo al comprar; "Entrar" siempre disponible en una sola pantalla.
3. POS sin página pública; los puntos se habilitan desde el Backoffice, enlazados a una bodega y a sus lotes.
4. Landings en ES/EN; aplicaciones en ES.
5. Marca única Drinks on Chain en todos los sitios y aplicaciones.
