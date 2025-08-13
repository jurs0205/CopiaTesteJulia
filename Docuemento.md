# TesteJulia
# Documentação Painel Analise E-Commerce 

## 1. Objetivo do Painel
Este painel foi desenvolvido para acompanhar indicadores de evolução de vendas, comportamento dos clientes e gestão de estoque, com foco em métricas de desempenho e retenção.

---

## 2. Fontes de Dados
- **Compilado_Vendas**:
Esta base é um compilado das bases de vendas, cada linha representa um pedido, o qual pode ter mais de um item. Está dividida nas seguintes colunas
### `fato_vendas.csv` *(nível: linha de pedido)*
| Coluna         | Descrição |
|----------------|-----------|
| Nome da Origem       | Nome do arquivo de origem |
| PedidoID       | Identificador do pedido |
| DataHoraVenda  | Data e hora da venda |
| ClienteID      | FK para `dim_clientes` |
| ProdutoID      | FK para `dim_produtos` |
| Canal          | Ecommerce, Marketplace, Loja Física |
| Qtde           | Quantidade de itens |
| PrecoUnitario  | Preço unitário aplicado |
| DescontoPct    | Percentual de desconto |
| ValorBruto     | Qtde × PrecoUnitario |
| ValorDesconto  | ValorBruto × DescontoPct |
| ValorLiquido   | ValorBruto - ValorDesconto |
| Data  | Data da venda |

- **Calendário**: 
Tabela de datas gerada em dax para suportar cálculos de tempo. Ela foi construída juntando as datas de estoque e vendas, fazendo um compilado.

| Coluna         | Descrição |
|----------------|-----------|
| Data      | Data do calendário|
| Ano  | Ano referente a data |
| Mês      | Mês referente a data |
| Trimestre      | Trimestre referente a data |
| Últimos 30 dias         | Análise se a data está nos últimos 30 dias |
| Últimos 6 meses           | Análise se a data está nos últimos 6 meses |

- **Estoque**: 
Base de estoque atual e histórico. Essa base mostra a última atualização do estoque de cada produto
### `fato_estoque.csv` *(nível: snapshot mensal por produto)*
| Coluna       | Descrição |
|--------------|-----------|
| ProdutoID    | FK para `dim_produtos` |
| DataRef      | Último dia do mês |
| QtdEstoque   | Quantidade em estoque |

---

## 3. Estrutura do Modelo
### 3.1 Relacionamentos
- Calendário[Data] → Compilado_Vendas[Data] 

- Calendário[Data] → Estoque[Data] 

- Dim_produtos[ProdutoID] → Compilado_Vendas[ProdutoID]

- Dim_produtos[ProdutoID] → Estoque[ProdutoID]

- Dim_cliente[ClienteID] → Compilado_Vendas[ClienteID] 

## 4. Medidas DAX
### 4.1 Métricas de Vendas
- **Valor Líquido** 
Valor líquido total
```DAX
SUM(Compilado_Vendas[ValorLiquido])
```

- **Vendas Acumuladas (YTD)** 
Vendas acumuladas desde o dia 01/01/2025 até a data selecionada
```DAX
TOTALYTD([Valor Líquido],'Calendário'[Data])
```

- **Qtde itens vendidos** 
Quantidade de itens vendidos
```DAX
COUNTROWS(Compilado_Vendas)*MAX(Compilado_Vendas[Qtde])
```

- **Média de itens vendidos (últimos 30 dias)** 
Média diária de itens vendidos nos últimos 30 dias 
```DAX
VAR Qtde_itens_vendidos =
CALCULATE( [Qtde itens vendidos], 'Calendário'[últimos 30 dias] = "sim")
VAR Qtde_dias =
CALCULATE(COUNTROWS('Calendário'), 'Calendário'[últimos 30 dias] = "sim")
RETURN
DIVIDE(Qtde_itens_vendidos, Qtde_dias)
```

- **Ticket Médio Pedidos** 
Ticket Médio dos pedidos
```DAX
VAR Qtde_distinta_pedidos = 
DISTINCTCOUNT(Compilado_Vendas[PedidoID])
RETURN
DIVIDE([Valor Líquido],Qtde_distinta_pedidos)
```

### 4.2 Métricas de Estoque
- **Estoque total** 
Estoque total dos produtos
```DAX
VAR Tabela = 
SUMMARIZE(Estoque,Estoque[ProdutoID],"Max estoque",MAX(Estoque[QtdEstoque]))
VAR soma = 
SUMX(Tabela,[Max estoque])
RETURN
soma
```

- **Dias de cobertura** 
Quantos dias de estoque a empresa ainda tem considerando o consumo médio diário
```DAX
VAR Qtde_dias =
COUNTROWS('Calendário')
VAR Consumo_Diario = 
DIVIDE([Qtde itens vendidos], Qtde_dias)
RETURN
DIVIDE([Estoque total],Consumo_Diario)
```

### 4.3 Métricas de Clientes
- **Qtde de clientes** 
Quantidade de clientes
```DAX
DISTINCTCOUNT(Compilado_Vendas[ClienteID])
```

- **Qtde de clientes inativos** 
Quantidade de clientes inativos
```DAX
CALCULATE([Qtde de clientes],Compilado_Vendas[Cliente ativo] = "não")
```

- **Qtde novos clientes por mês** 
Quantidade de clientes novos por mês
```DAX
CALCULATE([Qtde de clientes],Compilado_Vendas[Primeira compra] = "Sim")
```

### 4.4 Métricas de Cohort
- **Retenção** 
Retenção dos clientes, análise por cohort
```DAX
VAR TotalCohort =
CALCULATE([Qtde de clientes],FILTER(ALL(Compilado_Vendas),[Meses Desde Cohort] = 0 && Compilado_Vendas[Cohort Mês e Ano] = MAX(Compilado_Vendas[Cohort Mês e Ano])))
RETURN
DIVIDE([Qtde de clientes], TotalCohort)
```




## 5. Transformações no Power Query
- União de arquivos de vendas mensais em uma tabela única
- Criação de uma coluna apenas de data sem hora no compilado de vendas
- Criação de uma tabela em branco apenas para compilar as medidas em dax

## 6. Criação de colunas calculadas
- Foram criadas colunas calculadas na tabela de compilado vendas para auxiliar na construção das medidas em dax, são elas: Primeira compra, Cliente Ativo, Cohort mês e ano e Meses desde cohort

## 7. Observações
- O painel utiliza segmentadores para filtrar os dados
- Na visão de valor líquido por tempo é possível segmentar por canal e categoria, foi criado um indicador que alterna o gráfico para esses formatos de canal e categoria.

---
**Autor:** Júlia Rosa  
**Data:** 13/08/2025
