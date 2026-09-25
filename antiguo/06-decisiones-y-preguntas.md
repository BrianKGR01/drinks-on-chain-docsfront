# 06 · Decisiones tomadas y preguntas abiertas

## Decisiones (con fecha)

| Fecha | Decisión | Motivo |
|---|---|---|
| 2026-09-24 | El equipo construye **solo el frontend** de los cuatro sistemas y la landing; el backend lo hace otro equipo | Alcance acordado con el cliente |
| 2026-09-24 | Todo se construye primero contra **datos mock en JSON** (MSW) con tipos compartidos | Tener el frontend listo antes del backend |
| 2026-09-24 | Cadena: **Stellar** (smart wallets con passkeys, tokens SEP-41 en Soroban) | Acceso a grants del Stellar Community Fund; UX sin frase semilla |
| 2026-09-24 | Billetera del consumidor: **passkey-kit** por defecto, login social (Privy/Web3Auth) como plan B, billeteras externas opcionales; todo detrás de `doc-wallet` | Ver 04 |
| 2026-09-24 | Las bodegas tienen cuenta institucional gestionada por Debro con firmante propio opcional; no operan cripto en el MVP | Fricción cero para la bodega |
| 2026-09-24 | Acento de marca **oro líquido**; el rojo solo como color de peligro y semáforo | Pedido del cliente (el rojo leía como error) |
| 2026-09-24 | **Aprobado**: landing principal B2C (`drinks-on-chain-landing`, dominio raíz) con sección y ruta B2B; el mapa actual pasa a ser el **sitio de bodegas** en `bodegas.`; cada sistema en su subdominio con su propia autenticación (`app.`, `erp.`, `pos.`, `admin.`); las landings no autentican a nadie; el POS no tiene página pública ni enlace desde la landing principal; el Backoffice no se enlaza desde ningún sitio público | Seguridad por origen y un mensaje claro por público; ver 02 |
| 2026-09-24 | Marketplace: **navegación libre sin cuenta**; la cuenta (y la billetera) se crea al comprar o cuando el usuario pulsa "Entrar"; el escaneo de botella sigue exigiendo registro (scan gate del documento maestro) | Menos fricción antes de la intención de compra |
| 2026-09-24 | ERP y POS: **solo inicio de sesión**; los usuarios de bodega y los puntos de recojo (enlazados a una bodega y a sus lotes) los provisiona Debro desde el Backoffice | Control de acceso centralizado |
| 2026-09-24 | Un repositorio por sistema bajo `BrianKGR01`; paquetes compartidos `@doc/ui`, `@doc/mocks`, `@doc/wallet` | Ciclos de vida distintos, reutilización explícita |
| 2026-09-24 | Conventional Commits; trabajo en `dev`; PR `dev → main` por hito | Historial limpio |
| 2026-09-24 | Tipografías libres (Cormorant Garamond, EB Garamond) en lugar de Shipley/Sabon | Licencias |
| 2026-09-24 | Fotografías solo con licencia libre y crédito visible hasta recibir material de las bodegas | Derechos de autor |

## Preguntas para el cliente (Debro Solutions)

1. **Bodegas socias**: ¿con cuáles hay acuerdo o conversación hoy? Define qué perfiles pueden mostrar "Adquirir" y cuáles quedan como referencia.
2. **Pasarela de pago** en Bolivia: tarjeta (¿Libélula, Todotix, PagosNet?) o QR bancario (¿qué banco?). Condiciona el checkout de S2.
3. **Dominio raíz**: pendiente de compra (los subdominios `bodegas.`, `app.`, `erp.`, `pos.`, `admin.` ya están decididos).
4. **Idiomas** de las aplicaciones operativas: ¿solo español en el MVP?
5. **Puntos de recojo** iniciales: ¿cuántos, dónde, con qué hardware (tablet Android o iPad)?
6. **Pase de retiro**: ¿válido en cualquier sucursal o en una elegida al generarlo? ¿Caducidad (minutos u horas)?
7. **Marca**: ¿existe logotipo o el wordmark tipográfico actual es el definitivo? ¿Colores corporativos de Debro para el Backoffice?
8. **Aviso legal y edad**: textos legales definitivos (Ley 259) y política de privacidad.

## Preguntas para el equipo de backend

1. **Contrato del token**: ¿un contrato multi-lote o uno por lote? ¿Quién despliega y custodia la clave emisora (HSM, Turnkey, KMS)?
2. **Relayer**: ¿Launchtube con cuenta de la SDF o relayer propio (fee bump)? ¿Cómo entrega el cliente el token de sesión?
3. **Indexación** de balances y eventos: ¿Mercury o servicio propio? ¿Websocket o polling para `lot.ready`, `mint.confirmed`, `claim.confirmed`?
4. **Autenticación**: ¿un proveedor de identidad único (OIDC) para los cuatro sistemas o sesiones separadas? ¿2FA del Backoffice con TOTP?
5. **Modelo de datos**: ¿aceptan `doc-mocks` como borrador del OpenAPI para converger?
6. **Firma del pase de retiro**: ¿desafío tipo SEP-10 firmado por la smart wallet, o autorización Soroban pre-firmada para la quema?
7. **Entornos**: testnet ahora; ¿fecha objetivo para mainnet?
8. **Archivos de QR** del ERP: ¿los genera el frontend (ZIP con PNG/SVG) o el backend (PDF para imprenta)?

## Supuestos vigentes mientras no haya respuesta

- Token fungible por lote, 1 unidad = 1 botella, `decimals = 0`.
- Pase de retiro válido en cualquier sucursal autorizada, caducidad 30 minutos.
- Pago en fiat; ningún usuario paga XLM.
- Español por defecto; inglés solo en landing y Marketplace.
- Subdominios por sistema bajo un dominio único; hasta la compra, los enlaces entre sitios usan variables de entorno con valores de desarrollo.
