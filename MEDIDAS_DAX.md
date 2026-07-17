# Medidas DAX

Crea una tabla solo de medidas (`Medidas`, sin datos — `Modelado → Nueva tabla` →
`Medidas = {""}` y borra la columna) para organizarlo todo. Pega cada medida de
abajo en `Nueva medida`.

> Nota: las medidas que usan `TODAY()` son dinámicas — cambian según la fecha
> en la que abras el informe. Como el dataset es sintético y "termina" cerca
> de 2026-07-16, si lo abres mucho más tarde los números de "vencido"/"pendiente"
> pueden parecer extraños (todo en el pasado). Eso es normal en un dataset
> de portfolio; explícalo en la entrevista si te preguntan — demuestra que
> entiendes la limitación de datos estáticos frente a un sistema en producción.

## 1. Medidas base

```dax
Total Faturado = SUM(FactInvoices[amount_eur])
```

```dax
Total Cobrado =
CALCULATE(
    SUM(FactInvoices[amount_eur]),
    USERELATIONSHIP(DimDate[date], FactInvoices[payment_date]),
    NOT ISBLANK(FactInvoices[payment_date])
)
```
Usa la relación inactiva `DimDate[date] ↔ FactInvoices[payment_date]` — así, en
un gráfico por mes (eje = `DimDate[year_month]`), esta medida suma lo que se
**cobró efectivamente ese mes**, aunque la relación activa apunte a
`issue_date`.

```dax
Total em Aberto =
CALCULATE(
    SUM(FactInvoices[amount_eur]),
    ISBLANK(FactInvoices[payment_date])
)
```

```dax
Total Vencido =
CALCULATE(
    SUM(FactInvoices[amount_eur]),
    ISBLANK(FactInvoices[payment_date]),
    FactInvoices[due_date] < TODAY()
)
```

```dax
% Vencido =
DIVIDE([Total Vencido], [Total em Aberto])
```

## 2. DSO (Days Sales Outstanding)

El DSO "a día de hoy" es fácil (`Total em Aberto` ya refleja "hoy"). El reto es
un **DSO a lo largo del tiempo** (tendencia mensual) — para eso necesitas
medidas "a partir de una fecha de referencia", no solo "a partir de hoy":

```dax
AR Aberto até Data =
VAR RefDate = MAX(DimDate[date])
RETURN
CALCULATE(
    SUM(FactInvoices[amount_eur]),
    FILTER(
        ALL(FactInvoices),
        FactInvoices[issue_date] <= RefDate &&
        (ISBLANK(FactInvoices[payment_date]) || FactInvoices[payment_date] > RefDate)
    )
)
```

```dax
Faturado Últimos 90 Dias =
VAR RefDate = MAX(DimDate[date])
RETURN
CALCULATE(
    SUM(FactInvoices[amount_eur]),
    FILTER(
        ALL(FactInvoices),
        FactInvoices[issue_date] > RefDate - 90 &&
        FactInvoices[issue_date] <= RefDate
    )
)
```

```dax
DSO =
DIVIDE([AR Aberto até Data], [Faturado Últimos 90 Dias]) * 90
```

Usa `DSO` en un gráfico de líneas con `DimDate[year_month]` en el eje — el
`MAX(DimDate[date])` dentro de la variable se resuelve automáticamente al
último día de cada mes en ese contexto.

Para un valor puntual "a día de hoy" (por ejemplo en una tarjeta, sin eje de
fecha), usa esta variante:

```dax
DSO Atual = CALCULATE([DSO], DimDate[date] = TODAY())
```

## 3. Aging (tramos de retraso)

Primero crea una **columna calculada** en `FactInvoices` (no una medida, esto
es por fila):

```dax
Aging Bucket =
VAR DiasAtraso = DATEDIFF(FactInvoices[due_date], TODAY(), DAY)
RETURN
IF(
    NOT ISBLANK(FactInvoices[payment_date]), "Paga",
    IF(DiasAtraso <= 0, "Em dia",
    IF(DiasAtraso <= 30, "1-30 dias",
    IF(DiasAtraso <= 60, "31-60 dias", "60+ dias")))
)
```

Después, en el visual de aging, arrastra `Aging Bucket` al eje, `Total em
Aberto` a los valores, y aplica un **filtro de visual**: `Aging Bucket ≠
"Paga"`.

Ordena el eje manualmente (`Ordenar por columna` no sirve con texto libre) —
crea una columna auxiliar `Aging Bucket Ordem` (1=Em dia, 2=1-30, 3=31-60,
4=60+) usando una lógica independiente (sin referenciar `Aging Bucket`, para
evitar dependencia circular) y usa `Ordenar por columna` → `Aging Bucket
Ordem`.

## 4. Segmento / País

No necesitas ninguna medida nueva — `Total Faturado` y `% Vencido` ya
responden correctamente al cruzarlas con `DimClients[segment]` o
`DimClients[country]` en el eje, porque la relación `DimClients →
FactInvoices` propaga el filtro.

## 5. Formato

- Todas las medidas de importe: formato `€ #.##0` (moneda, 0 decimales).
- `% Vencido`: formato porcentaje, 1 decimal.
- `DSO`: número, 1 decimal, sufijo " días" (formato personalizado
  `0.0" días"`).
