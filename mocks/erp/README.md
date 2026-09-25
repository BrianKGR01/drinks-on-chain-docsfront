# Datos de prueba del ERP

JSON con las formas **exactas** de los DTO del backend (OpenAPI del 25-09-2026). Se generan con:

```bash
python generate.py
```

El script es determinista (UUID v5 y semilla fija; fecha de referencia 2026-09-25): volver a ejecutarlo produce los mismos archivos. El contrato, el mapa endpoint ↔ pantalla y las reglas de coherencia están en [`../../09-contrato-erp-backend.md`](../../09-contrato-erp-backend.md).

| Archivo | Qué es |
|---|---|
| `wineries.json` | Bodegas con sus miembros (`WineryResponseDto`) |
| `users.json` | Perfiles (`UserProfileResponseDto`) más `_mock.password` (`demo1234`) |
| `wallets.json` | Billeteras custodiales (`WalletResponseDto`) |
| `auth-login.json` | Respuesta de `POST /v1/auth/login` por usuario (clave = `_mock.key`) |
| `terroirs.json` | Parcelas |
| `harvest-batches.json` | Lotes de vendimia (pesaje + análisis + estado fitosanitario) |
| `fermentation-tanks.json`, `fermentation-logs.json`, `enological-treatments.json` | Tanques, lecturas y tratamientos |
| `wine-aging.json` | Crianzas con `lockUntilDate` |
| `production-batches.json`, `production-rest-status.json` | Destilaciones y su reposo de 180 días |
| `bottling.json` | Embotellados con código de lote, hash y anclaje |
| `lab-analyses.json` | Certificados de laboratorio |
| `traceability-public.json` | Pasaporte público por código de lote |
| `lots-view.json` | Vista derivada "Lote" para las pantallas del ERP (no existe en el backend) |

Estos archivos se mudan al repo `doc-mocks` en la Etapa 0.2, donde el generador se reescribe en TypeScript y se añaden los handlers MSW.
