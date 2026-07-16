# Modelo de Dados — Esquema em Estrela

## Tabelas

### `FactInvoices` (de `invoices.csv`)
| Coluna | Tipo | Descrição |
|---|---|---|
| invoice_id | Texto | Chave da fatura |
| client_id | Texto | FK → DimClients |
| issue_date | Data | Data de emissão |
| due_date | Data | Data de vencimento (issue_date + prazo do cliente) |
| payment_date | Data (pode ser vazio) | Data em que foi paga. Vazio = ainda em aberto |
| amount_eur | Número decimal | Valor da fatura |
| status | Texto | Paga / Vencida / Em Aberto (calculado na geração) |

### `DimClients` (de `clients.csv`)
| Coluna | Tipo | Descrição |
|---|---|---|
| client_id | Texto | Chave primária |
| client_name | Texto | Nome do cliente |
| segment | Texto | Enterprise / PYME / Startup |
| country | Texto | País do cliente |
| payment_terms_days | Número inteiro | Prazo de pagamento negociado |

### `DimDate` (de `dim_date.csv`)
| Coluna | Tipo | Descrição |
|---|---|---|
| date | Data | Chave primária (uma linha por dia) |
| year | Inteiro | |
| month_number | Inteiro | 1-12 |
| month_name | Texto | Nome do mês (ES) |
| year_month | Texto | "2026-07", útil para ordenar/agrupar |
| quarter | Texto | Q1-Q4 |
| day_of_week | Texto | |

## Relações

```
DimClients (1) ──── (*) FactInvoices [client_id]
DimDate    (1) ──── (*) FactInvoices [date] ←→ [issue_date]   ← relação ATIVA
DimDate    (1) ──── (*) FactInvoices [date] ←→ [due_date]     ← relação inativa (usar USERELATIONSHIP)
DimDate    (1) ──── (*) FactInvoices [date] ←→ [payment_date] ← relação inativa (usar USERELATIONSHIP)
```

**Porquê 3 relações com DimDate?** Uma fatura tem 3 datas relevantes (emissão,
vencimento, pagamento) mas só pode existir **uma relação ativa** por tabela de
datas no Power BI. A relação ativa fica em `issue_date` (para analisar
faturação por mês); as outras duas ficam **inativas** e são ativadas dentro de
medidas DAX específicas com `USERELATIONSHIP()` — é assim que se calcula, por
exemplo, "valor cobrado por mês de pagamento" mesmo com a relação ativa a
apontar para a data de emissão.

## Passos para montar no Power BI Desktop

1. `Obter dados` → `Texto/CSV` → importar os 3 ficheiros de `data/`.
2. Em `Ferramentas de tabela → Tipo de dados`, confirma que `issue_date`,
   `due_date`, `payment_date` e `date` (em DimDate) estão como **Data**, e
   `amount_eur` como **Número decimal**.
3. Em `Modelação → Gerir relações`, cria as 3 relações acima (a de `issue_date`
   fica ativa por defeito; as outras cria-as e marca "Ativa" desligado).
4. Esconde `client_id` e `invoice_id` de `FactInvoices` da vista de relatório
   (não são precisos nos visuais, só nas relações).
