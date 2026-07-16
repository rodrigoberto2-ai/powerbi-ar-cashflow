# Análise de Contas a Receber & Cash Flow — Power BI

Projeto de portfólio: dashboard em Power BI para gestão de contas a receber (AR),
DSO e risco de incumprimento, para uma operação B2B fictícia com clientes em
Espanha, Portugal, França, Alemanha e Itália.

Este projeto espelha o trabalho real de acompanhamento financeiro feito na
UNINTER (negociação de prazos de pagamento, controlo de risco de impago,
seguimento de KPIs mensais) — aqui aplicado com Power BI + DAX de forma mais
avançada do que dashboards operacionais típicos.

> Este pacote foi preparado para seres tu a montar os visuais no Power BI
> Desktop — segue `GUIA_PASSO_A_PASSO.md`. O dataset e o modelo já estão prontos,
> só falta a montagem visual (~30-40 min).

## Perguntas de negócio que o relatório responde

1. O DSO está a subir ou a descer? Em que segmento/país?
2. Que percentagem do valor em aberto está vencida, e há quanto tempo?
3. Que clientes concentram o maior risco de incumprimento?
4. A cobrança está a acompanhar a faturação mês a mês?

## Ficheiros

```
data/
  clients.csv      → dimensão de clientes (segmento, país, prazo de pagamento)
  invoices.csv      → factos: faturas emitidas, vencimento, pagamento, valor
  dim_date.csv       → dimensão de datas (para o modelo em estrela)
MODELO_DE_DADOS.md   → esquema em estrela + relações
MEDIDAS_DAX.md        → todas as fórmulas DAX usadas nos visuais
GUIA_PASSO_A_PASSO.md → como montar o relatório no Power BI Desktop
```

## Depois de montado

1. Exporta 2-3 screenshots dos visuais principais para este README (secção
   "Screenshots").
2. Publica no Power BI Service (se tiveres licença) ou exporta um PDF/vídeo
   curto a navegar pelo relatório — o link vai aqui e no teu portfólio.

## Screenshots

_(adicionar depois de montado no Power BI Desktop)_

---

Feito por Rodrigo Pinheiro Berto — Analista de Datos, Madrid.
