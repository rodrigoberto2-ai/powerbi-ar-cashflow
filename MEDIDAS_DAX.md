# Medidas DAX

Cria uma tabela só de medidas (`Medidas`, sem dados — `Modelação → Nova Tabela` →
`Medidas = {""}` e apaga a coluna) para organizar tudo. Cola cada medida abaixo em
`Nova Medida`.

> Nota: as medidas que usam `TODAY()` são dinâmicas — mudam consoante a data em
> que abres o relatório. Como o dataset é sintético e "termina" perto de
> 2026-07-16, se abrires isto muito mais tarde os números de "vencido"/"em
> aberto" vão parecer estranhos (tudo no passado). Isso é esperado num dataset
> de portfólio; explica isso na entrevista se perguntarem — mostra que percebes
> a limitação de dados estáticos vs. um sistema em produção.

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
Usa a relação inativa `DimDate[date] ↔ FactInvoices[payment_date]` — assim, num
gráfico por mês (eixo = `DimDate[year_month]`), esta medida soma o que foi
**efetivamente cobrado naquele mês**, mesmo com a relação ativa a apontar para
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

DSO "como se fosse hoje" é fácil (`Total em Aberto` já reflete "hoje"). O
desafio é um **DSO ao longo do tempo** (tendência mensal) — para isso precisas
de medidas "a partir de uma data de referência", não só "a partir de hoje":

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

Usa `DSO` num gráfico de linhas com `DimDate[year_month]` no eixo — o
`MAX(DimDate[date])` dentro da variável resolve automaticamente para o último
dia de cada mês nesse contexto.

## 3. Aging (faixas de atraso)

Primeiro cria uma **coluna calculada** em `FactInvoices` (não uma medida — isto
é por linha):

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

Depois, no visual de aging, arrasta `Aging Bucket` para o eixo, `Total em
Aberto` para os valores, e aplica um **filtro de visual**: `Aging Bucket ≠
"Paga"`.

Ordena o eixo manualmente (`Ordenar por coluna` não serve para texto livre) —
cria uma coluna auxiliar `Aging Bucket Ordem` (1=Em dia, 2=1-30, 3=31-60,
4=60+) e usa `Ordenar por coluna` → `Aging Bucket Ordem`.

## 4. Segmento / País

```dax
Faturado por Segmento = [Total Faturado]   -- usa com DimClients[segment] no eixo
```
Não precisas de medida nova — `Total Faturado` e `% Vencido` já respondem
corretamente quando cruzadas com `DimClients[segment]` ou `DimClients[country]`
no eixo, porque a relação `DimClients → FactInvoices` propaga o filtro.

## 5. Formatação

- Todas as medidas de valor: formato `€ #.##0` (moeda, 0 casas decimais).
- `% Vencido`: formato percentagem, 1 casa decimal.
- `DSO`: número, 1 casa decimal, sufixo " dias" (formato personalizado
  `0.0" dias"`).
