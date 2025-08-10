
# 📊 Desafio de BI – Análise de E-commerce

## 📌 Contexto
Você foi contratado por um e-commerce para desenvolver um **painel analítico** que auxilie a área comercial e de logística a tomar decisões baseadas em dados.  
O painel deve mostrar a evolução das vendas, o comportamento dos clientes e a gestão do estoque nos últimos 3 anos.

**Período de análise:** 01/01/2023 a 09/08/2025  
**Moeda:** R$  
**Fuso horário:** America/São_Paulo

---

## 📂 Bases de Dados
Os arquivos disponibilizados contêm dados sintéticos, já modelados para um cenário de **Data Warehouse**.

### `dim_produtos.csv`
| Coluna       | Descrição |
|--------------|-----------|
| ProdutoID    | Chave primária |
| SKU          | Código único do produto |
| NomeProduto  | Nome descritivo |
| Categoria    | Categoria de produto |
| Marca        | Marca do produto |
| Fornecedor   | Nome do fornecedor |
| PrecoBase    | Preço de referência |
| Ativo        | 1 = ativo, 0 = inativo |

### `dim_clientes.csv`
| Coluna       | Descrição |
|--------------|-----------|
| ClienteID    | Chave primária |
| CPF_CNPJ     | Documento sintético |
| NomeCliente  | Nome do cliente |
| UF           | Unidade federativa |
| DataCadastro | Data de registro do cliente no sistema |

### `fato_vendas.csv` *(nível: linha de pedido)*
| Coluna         | Descrição |
|----------------|-----------|
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

### `fato_estoque.csv` *(nível: snapshot mensal por produto)*
| Coluna       | Descrição |
|--------------|-----------|
| ProdutoID    | FK para `dim_produtos` |
| DataRef      | Último dia do mês |
| QtdEstoque   | Quantidade em estoque |

---

## 🎯 Objetivos do Painel
O painel deve conter as seguintes visões e KPIs:

1. **Comparativo de vendas (últimos 3 anos)**  
   - `ValorLiquido` por mês, com comparação ano a ano.

2. **Evolução do estoque no período**  
   - `QtdEstoque` por produto, categoria e total.

3. **Novos clientes por mês**  
   - Definição: primeira compra no mês de referência.

4. **Clientes inativos**  
   - Definição: última compra há mais de 6 meses.

5. **Vendas acumuladas (YTD)**  
   - Soma de `ValorLiquido` no ano corrente até a data selecionada.

6. **Ticket médio**  
   - `ValorLiquido / DISTINCTCOUNT(PedidoID)`  
   - Exibir por canal e categoria.

7. **Média de itens vendidos (últimos 30 dias)**  
   - Média diária de `Qtde` considerando os últimos 30 dias.

---

## 📏 Regras de Negócio Esperadas
- **Primeira compra** = `MIN(DataHoraVenda)` por `ClienteID`  
- **Última compra** = `MAX(DataHoraVenda)` por `ClienteID`  
- **Novos clientes** = primeira compra no mês  
- **Inativos** = última compra anterior a (data de referência – 180 dias)  
- **Ticket médio** = `SUM(ValorLiquido) / DISTINCTCOUNT(PedidoID)`  
- **YTD** = soma de `ValorLiquido` no ano até a data selecionada  

---

## 🛠 Requisitos Técnicos
- Modelagem **em estrela** (fatos e dimensões separadas)  
- Criar **tabela calendário** (`dCalendario`) com colunas: Ano, Mês, Trimestre, YTD, Últimos30Dias etc.  
- Relacionamentos: filtros devem fluir das dimensões para as fatos  
- Medidas DAX bem nomeadas e comentadas

---

## 📊 Avaliação
O candidato será avaliado de acordo com:
- **Qualidade da modelagem**  
- **Correção das medidas**  
- **Clareza do dashboard**  
- **Documentação das decisões**  
- **Boas práticas de performance**

---

## 🚀 Diferenciais
- Segmentação por **canal** e **categoria** no comparativo 3 anos  
- **Cohort de clientes** por mês de primeira compra  
- Métrica de **cobertura de estoque** (dias de cobertura)  
- Tratamento de **dados faltantes** em estoque (ex.: carry-forward)

---

## 📦 Entrega
- Arquivo `.pbix` com o painel construído  
- Documentação em `.md` ou no próprio Power BI  
