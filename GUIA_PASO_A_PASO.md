# Guía paso a paso — Montar el informe en Power BI Desktop

Tiempo estimado: 30-40 min. El modelo de datos y las fórmulas ya están listos
(ver `MODELO_DE_DATOS.md` y `MEDIDAS_DAX.md`) — aquí solo queda el montaje visual.

## 1. Importar datos (5 min)

1. Abre Power BI Desktop → `Obtener datos` → `Texto/CSV`.
2. Importa los 3 archivos de `data/`: `clients.csv`, `invoices.csv`, `dim_date.csv`.
3. `Transformar datos` → confirma los tipos: fechas como **Fecha**, `amount_eur`
   como **Número decimal**, `payment_terms_days` como **Número entero**.
4. `Cerrar y aplicar`.

## 2. Relaciones (5 min)

Sigue exactamente `MODELO_DE_DATOS.md` § Relaciones. En `Modelado →
Administrar relaciones`, crea las 3 relaciones con `DimDate`, dejando activa
solo la de `issue_date`.

## 3. Medidas (10 min)

Crea la tabla `Medidas` y pega las fórmulas de `MEDIDAS_DAX.md`, secciones 1-3.
Prueba cada una en una tabla simple antes de continuar (arrastra la medida a
un visual de tabla y comprueba que el número tiene sentido).

## 4. Página 1 — Visión General

Layout sugerido (4 tarjetas arriba + 2 gráficos):

- **Tarjeta** `DSO Actual` (medida `DSO Atual`, ver `MEDIDAS_DAX.md`)
- **Tarjeta** `% Vencido`
- **Tarjeta** `Total en Aberto` (total pendiente)
- **Tarjeta** `Total Vencido`
- **Gráfico de líneas**: eje `DimDate[year_month]`, valores `DSO`
- **Gráfico de columnas**: eje `DimDate[year_month]`, valores `Total Faturado`
  y `Total Cobrado` (dos series, misma escala, no uses eje secundario)
- **Segmentación de datos (slicer)** de `DimDate[date]` con estilo "Entre",
  para que el usuario pueda explorar el periodo — así evitas además que el
  gráfico muestre meses futuros sin datos reales.

## 5. Página 2 — Riesgo & Aging

- **Gráfico de barras horizontales**: eje `Aging Bucket` (ordenado por `Aging
  Bucket Ordem`), valores `Total em Aberto`, filtro de visual `Aging Bucket ≠
  "Paga"`. Color por categoría: verde (Al día) → amarillo → naranja → rojo
  (60+), para que se lea como escala de riesgo.
- **Gráfico de columnas**: eje `DimClients[segment]`, valores `Total
  Faturado`, con etiqueta de datos combinando el `% Vencido`.
- **Gráfico de barras horizontales**: eje `DimClients[country]`, valores
  `Total Faturado`.
- **Tabla**: `DimClients[client_name]`, `segment`, `country`, `Total
  Vencido` — ordenada de forma descendente, mostrando solo los top 10
  (`Filtro de visual` → `Top N`).

## 6. Filtros globales

Añade una segmentación de datos (`Slicer`) para `DimClients[segment]` y otra
para `DimClients[country]`, aplicadas a ambas páginas (`Sincronizar
segmentaciones`).

## 7. Antes de publicar

- Comprueba que no hay ejes dobles (ningún gráfico debe tener dos escalas).
- Todos los gráficos con 2+ series tienen leyenda visible.
- Prueba el modo oscuro de Power BI (opcional) o asegura contraste suficiente
  en el tema claro.
- Exporta 2-3 capturas para el `README.md` de este proyecto.

## 8. Publicar / compartir

- **Con licencia Power BI Pro/PPU**: `Publicar` → workspace → comparte el enlace.
- **Sin licencia**: exporta un PDF (`Archivo → Exportar → PDF`) o graba un
  vídeo corto (30-60s) navegando por el informe — úsalo en el portfolio en
  lugar del enlace en vivo.
