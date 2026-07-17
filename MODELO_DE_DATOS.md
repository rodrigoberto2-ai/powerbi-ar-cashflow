# Modelo de Datos — Esquema en Estrella

## Tablas

### `FactInvoices` (de `invoices.csv`)
| Columna | Tipo | Descripción |
|---|---|---|
| invoice_id | Texto | Clave de la factura |
| client_id | Texto | FK → DimClients |
| issue_date | Fecha | Fecha de emisión |
| due_date | Fecha | Fecha de vencimiento (issue_date + plazo del cliente) |
| payment_date | Fecha (puede estar vacío) | Fecha en que se pagó. Vacío = todavía pendiente |
| amount_eur | Número decimal | Importe de la factura |
| status | Texto | Paga / Vencida / Pendiente (calculado en la generación) |

### `DimClients` (de `clients.csv`)
| Columna | Tipo | Descripción |
|---|---|---|
| client_id | Texto | Clave primaria |
| client_name | Texto | Nombre del cliente |
| segment | Texto | Enterprise / PYME / Startup |
| country | Texto | País del cliente |
| payment_terms_days | Número entero | Plazo de pago negociado |

### `DimDate` (de `dim_date.csv`)
| Columna | Tipo | Descripción |
|---|---|---|
| date | Fecha | Clave primaria (una fila por día) |
| year | Entero | |
| month_number | Entero | 1-12 |
| month_name | Texto | Nombre del mes (ES) |
| year_month | Texto | "2026-07", útil para ordenar/agrupar |
| quarter | Texto | Q1-Q4 |
| day_of_week | Texto | |

## Relaciones

```
DimClients (1) ──── (*) FactInvoices [client_id]
DimDate    (1) ──── (*) FactInvoices [date] ←→ [issue_date]   ← relación ACTIVA
DimDate    (1) ──── (*) FactInvoices [date] ←→ [due_date]     ← relación inactiva (usar USERELATIONSHIP)
DimDate    (1) ──── (*) FactInvoices [date] ←→ [payment_date] ← relación inactiva (usar USERELATIONSHIP)
```

**¿Por qué 3 relaciones con DimDate?** Una factura tiene 3 fechas relevantes
(emisión, vencimiento, pago), pero solo puede existir **una relación activa**
por tabla de fechas en Power BI. La relación activa queda en `issue_date`
(para analizar la facturación por mes); las otras dos quedan **inactivas** y
se activan dentro de medidas DAX específicas con `USERELATIONSHIP()` — así se
calcula, por ejemplo, "importe cobrado por mes de pago" aunque la relación
activa apunte a la fecha de emisión.

## Pasos para montarlo en Power BI Desktop

1. `Obtener datos` → `Texto/CSV` → importar los 3 archivos de `data/`.
2. En `Herramientas de tabla → Tipo de datos`, confirma que `issue_date`,
   `due_date`, `payment_date` y `date` (en DimDate) están como **Fecha**, y
   `amount_eur` como **Número decimal**.
3. En `Modelado → Administrar relaciones`, crea las 3 relaciones de arriba (la
   de `issue_date` queda activa por defecto; las otras créalas y desmarca
   "Activa").
4. Oculta `client_id` e `invoice_id` de `FactInvoices` en la vista de informe
   (no se necesitan en los visuales, solo en las relaciones).
