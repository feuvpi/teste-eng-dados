# Teste Técnico — Engenharia de Dados

Resolução dos quatro desafios de SQL de um marketplace B2B (distribuidoras de
alimentos e supermercados). O notebook está organizado em três etapas:
exploração dos dados, limpeza e resolução dos desafios.

A exploração e a limpeza precedem as análises porque parte das inconsistências
descritas no enunciado tem origem nos próprios dados. As consultas SQL são
executadas sobre SQLite, por meio da biblioteca padrão `sqlite3` — cada DataFrame
é carregado como uma tabela pela função utilitária `pysqldf`.

## Execução

Google Colab: abra o notebook, faça upload dos seis CSVs para `/content/` pelo
painel de arquivos e execute as células na ordem (`Runtime > Run all`). Não há
dependências a instalar (pandas, matplotlib, seaborn e sqlite3 já estão
disponíveis no Colab).

Execução local: instale as dependências (`pip install pandas matplotlib seaborn`),
coloque os CSVs em um diretório e ajuste a variável `BASE` na célula de leitura.

## Estrutura do notebook

| Seção | Conteúdo |
|-------|----------|
| Setup | Leitura dos seis CSVs e definição do utilitário de consulta (SQLite) |
| 1. Exploração | Tipos, valores ausentes, unicidade de chaves, integridade referencial, consistência entre tabelas e caracterização do negócio (gráficos de status, itens por pedido, desconto, GMV mensal, concentração por seller e correlação) |
| 2. Limpeza | Demonstração do efeito das chaves duplicadas e deduplicação de `orders` e `buyers` |
| 3. Colunas e premissas | Mapa das colunas utilizadas |
| Desafios 1–4 | Consultas executadas sobre os dados tratados |

## Principais observações da exploração

- A coluna `id` não é única em `orders` (2 registros) e `buyers` (1 registro),
  com valores divergentes entre as duplicatas. As duplicatas em `orders`
  multiplicam linhas na junção com `order_items`. Tratadas na etapa de limpeza.
- Integridade referencial sem registros órfãos e ausência de valores fora do
  domínio esperado.
- O único campo com valores nulos (`payments.paid_at`) só é nulo em pagamentos
  não concluídos, o que caracteriza ausência esperada.
- `orders.total_value` corresponde ao valor líquido (bruto menos desconto);
  o faturamento bruto é definido como `SUM(unit_price * qty)`.

## Premissas e decisões técnicas

- Consultas sobre SQLite (via `sqlite3`): `strftime` para datas e fator
  `1.0`/`100.0` nas razões, evitando divisão inteira.
- Janelas de tempo referenciadas na data máxima dos dados (novembro de 2024).
- Seller atribuído no nível do pedido (`orders.seller_id`).
- Normalização de status com `lower`/`trim`, `COALESCE` em descontos e proteção
  contra divisão por zero.

## Notas por desafio

1. Faturamento bruto mensal dos últimos doze meses, com quantidade de pedidos e
   ticket médio. Inclui o faturamento líquido como referência.
2. Sellers com maior crescimento de GMV entre trimestres. O trimestre corrente
   está incompleto (60 dias contra 92); uma consulta complementar normaliza o
   GMV por dia observado.
3. Pedidos com desconto acima de 40% do valor bruto, com o seller responsável.
   Uma consulta complementar agrupa os pedidos sinalizados por seller.
4. A consulta não retorna registros. A avaliação subsequente demonstra que a
   anomalia descrita não se verifica no conjunto de dados e registra as
   verificações adicionais realizadas.
