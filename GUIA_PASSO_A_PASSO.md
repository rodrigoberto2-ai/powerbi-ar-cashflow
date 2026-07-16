# Guia passo-a-passo — Montar o relatório no Power BI Desktop

Tempo estimado: 30-40 min. O modelo de dados e as fórmulas já estão prontos
(ver `MODELO_DE_DADOS.md` e `MEDIDAS_DAX.md`) — aqui é só a montagem visual.

## 1. Importar dados (5 min)

1. Abre o Power BI Desktop → `Obter dados` → `Texto/CSV`.
2. Importa os 3 ficheiros de `data/`: `clients.csv`, `invoices.csv`, `dim_date.csv`.
3. `Transformar Dados` → confirma tipos: datas como **Data**, `amount_eur` como
   **Número decimal**, `payment_terms_days` como **Número inteiro**.
4. `Fechar e Aplicar`.

## 2. Relações (5 min)

Segue exatamente `MODELO_DE_DADOS.md` § Relações. No `Modelação → Gerir
relações`, cria as 3 relações com `DimDate`, deixando só a de `issue_date`
ativa.

## 3. Medidas (10 min)

Cria a tabela `Medidas` e cola as fórmulas de `MEDIDAS_DAX.md`, secções 1-3.
Testa cada uma numa tabela simples antes de avançar (arrasta a medida para um
visual de tabela e confere que o número faz sentido).

## 4. Página 1 — Visão Geral

Layout sugerido (4 cartões no topo + 2 gráficos):

- **Cartão** `DSO` (medida `DSO`, filtrado ao mês mais recente)
- **Cartão** `% Vencido`
- **Cartão** `Total em Aberto`
- **Cartão** `Total Vencido`
- **Gráfico de linhas**: eixo `DimDate[year_month]`, valores `DSO`
- **Gráfico de colunas**: eixo `DimDate[year_month]`, valores `Total Faturado`
  e `Total Cobrado` (duas séries, mesma escala — não uses eixo secundário)

## 5. Página 2 — Risco & Aging

- **Gráfico de barras horizontais**: eixo `Aging Bucket` (ordenado por `Aging
  Bucket Ordem`), valores `Total em Aberto`, filtro visual `Aging Bucket ≠
  "Paga"`. Cor por categoria: verde (Em dia) → amarelo → laranja → vermelho
  (60+), para ler como escala de risco.
- **Gráfico de colunas**: eixo `DimClients[segment]`, valores `Total
  Faturado`, rótulo de dados com `% Vencido`.
- **Gráfico de barras horizontais**: eixo `DimClients[country]`, valores
  `Total Faturado`.
- **Tabela**: `DimClients[client_name]`, `segment`, `country`, `Total
  Vencido` — ordenada decrescente, mostra só os top 10 (`Filtro de visual` →
  `Top N`).

## 6. Filtros globais

Adiciona um painel de segmentação (`Slicer`) para `DimClients[segment]` e
outro para `DimClients[country]`, aplicados a ambas as páginas (`Sincronizar
segmentações`).

## 7. Antes de publicar

- Confere que não há eixos duplos (nenhum gráfico deve ter duas escalas).
- Todos os gráficos com 2+ séries têm legenda visível.
- Testa o modo escuro do Power BI (opcional) ou garante contraste suficiente
  no tema claro.
- Exporta 2-3 screenshots para o `README.md` deste projeto.

## 8. Publicar / partilhar

- **Com licença Power BI Pro/PPU**: `Publicar` → workspace → partilha o link.
- **Sem licença**: exporta um PDF (`Ficheiro → Exportar → PDF`) ou grava um
  vídeo curto (30-60s) a navegar pelo relatório — usa isso no portfólio em vez
  do link ao vivo.
