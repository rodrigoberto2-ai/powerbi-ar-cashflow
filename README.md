# Análisis de Cuentas por Cobrar & Cash Flow — Power BI

Proyecto de portfolio: dashboard en Power BI para gestión de cuentas por cobrar
(AR), DSO y riesgo de impago, para una operación B2B ficticia con clientes en
España, Portugal, Francia, Alemania e Italia.

Este proyecto refleja el trabajo real de seguimiento financiero realizado en
UNINTER (negociación de plazos de pago, control de riesgo de impago,
seguimiento de KPIs mensuales), aquí aplicado con Power BI + DAX de forma más
avanzada que los dashboards operativos típicos.

> Este paquete está preparado para que montes tú los visuales en Power BI
> Desktop — sigue `GUIA_PASO_A_PASO.md`. El dataset y el modelo ya están listos,
> solo falta el montaje visual (~30-40 min).

## Preguntas de negocio que responde el informe

1. ¿El DSO está subiendo o bajando? ¿En qué segmento/país?
2. ¿Qué porcentaje del valor pendiente está vencido, y desde cuándo?
3. ¿Qué clientes concentran el mayor riesgo de impago?
4. ¿El cobro está siguiendo el ritmo de la facturación mes a mes?

## Archivos

```
data/
  clients.csv        → dimensión de clientes (segmento, país, plazo de pago)
  invoices.csv        → hechos: facturas emitidas, vencimiento, pago, importe
  dim_date.csv         → dimensión de fechas (para el modelo en estrella)
MODELO_DE_DATOS.md      → esquema en estrella + relaciones
MEDIDAS_DAX.md            → todas las fórmulas DAX usadas en los visuales
GUIA_PASO_A_PASO.md       → cómo montar el informe en Power BI Desktop
```

## Después de montarlo

1. Exporta 2-3 capturas de los visuales principales para este README (sección
   "Capturas").
2. Publica en Power BI Service (si tienes licencia) o exporta un PDF/vídeo
   corto navegando por el informe — el enlace va aquí y en tu portfolio.

## Capturas

_(añadir después de montarlo en Power BI Desktop)_

---

Hecho por Rodrigo Pinheiro Berto — Analista de Datos, Madrid.
