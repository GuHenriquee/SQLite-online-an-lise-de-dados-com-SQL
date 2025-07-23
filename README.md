# SQLite-online-an-lise-de-dados-com-SQL Exercícios
Curso da alura SQLite online: análise de dados com SQL na formação Conhecendo SQL 

01 - Qual é o número de Clientes que existem na base de dados ?

```sql
SELECT count(*) from clientes
```
02 - Quantos produtos foram vendidos no ano de 2022 ?
```sql
SELECT strftime('%Y', data_venda) as ano, count (venda_id) as conta 
from vendas v
join itens_venda iv
on v.id_venda = iv.venda_id
GROUP by ano
HAVING ano = '2022'
```
03 - Qual a categoria que mais vendeu em 2022 ?
```sql
SELECT strftime('%Y', data_venda) as ano, count (venda_id) as conta 
from vendas v
join itens_venda iv on v.id_venda = iv.venda_id
join produtos p on p.id_produto = iv.produto_id
join categorias c on c.id_categoria = p.categoria_id
GROUP by ano, c.nome_categoria
HAVING ano = '2022'
order by conta DESC
LIMIT 1
```
04 - Qual o primeiro ano disponível na base ?
```sql
SELECT strftime('%Y', data_venda) from vendas 
order by data_venda asc
LIMIT 1
;
```
05 - Qual o nome do fornecedor que mais vendeu no primeiro ano disponível na base ?
```sql
WITH primeiro_ano AS (
  SELECT strftime('%Y', data_venda) AS ano
  FROM vendas
  ORDER BY data_venda ASC
  LIMIT 1
),
fornecedor_mais_vendeu AS (
  SELECT f.nome
  FROM vendas v
  JOIN itens_venda iv ON v.id_venda = iv.venda_id
  JOIN produtos p ON p.id_produto = iv.produto_id
  JOIN fornecedores f ON f.id_fornecedor = p.fornecedor_id
  WHERE strftime('%Y', v.data_venda) = (SELECT ano FROM primeiro_ano)
  GROUP BY f.nome
  ORDER BY COUNT(v.id_venda) DESC
  LIMIT 1
)
SELECT * FROM fornecedor_mais_vendeu;
```
06 - Quanto ele vendeu no primeiro ano disponível na base de dados ?
```sql
WITH primeiro_ano AS (
  SELECT strftime('%Y', data_venda) AS ano
  FROM vendas
  ORDER BY data_venda ASC
  LIMIT 1
),
fornecedor_mais_vendeu AS (
  SELECT f.nome, COUNT(v.id_venda) AS total_vendas
  FROM vendas v
  JOIN itens_venda iv ON v.id_venda = iv.venda_id
  JOIN produtos p ON p.id_produto = iv.produto_id
  JOIN fornecedores f ON f.id_fornecedor = p.fornecedor_id
  WHERE strftime('%Y', v.data_venda) = (SELECT ano FROM primeiro_ano)
  GROUP BY f.nome
  ORDER BY total_vendas DESC
  LIMIT 1
)
SELECT * FROM fornecedor_mais_vendeu;
```
07 - Quais as duas categorias que mais venderam no total de todos os anos ?
```sql
SELECT count (venda_id) as conta, c.nome_categoria
from vendas v
join itens_venda iv on v.id_venda = iv.venda_id
join produtos p on p.id_produto = iv.produto_id
join categorias c on c.id_categoria = p.categoria_id
GROUP by c.id_categoria
order by conta DESC
LIMIT 2
```
08 - Crie uma tabela comparando as vendas ao longo do tempo das duas categorias que mais venderam no total de todos os anos.
```sql
SELECT data,
sum(CASE when nome_categoria == 'Eletrônicos' then conta else 0 end) as eletronicos,
sum(CASE when nome_categoria == 'Vestuário' then conta else 0 end) as vestuario
from (
  SELECT strftime('%Y/%m', v.data_venda) as data, count (venda_id) as conta, c.nome_categoria
  from vendas v
  join itens_venda iv on v.id_venda = iv.venda_id
  join produtos p on p.id_produto = iv.produto_id
  join categorias c on c.id_categoria = p.categoria_id
  where c.nome_categoria = 'Eletrônicos' or c.nome_categoria = 'Vestuário'
  GROUP by data, c.nome_categoria
  order by data ASC
)
GROUP by data;
```
09 - Calcule a porcentagem de vendas por categorias no ano de 2022.
```sql
WITH vendas_por_categoria AS ( 
  SELECT 
    c.nome_categoria, 
    COUNT(iv.venda_id) AS qtd_vendas 
  FROM itens_venda iv
  JOIN produtos p ON p.id_produto = iv.produto_id
  JOIN categorias c ON c.id_categoria = p.categoria_id
  GROUP BY c.nome_categoria
),
total AS (
  SELECT COUNT(*) AS total_vendas FROM itens_venda
)
SELECT 
  vpc.nome_categoria,
  vpc.qtd_vendas,
  ROUND((vpc.qtd_vendas * 100.0) / t.total_vendas, 2) AS porcentagem
FROM vendas_por_categoria vpc, total t
ORDER BY porcentagem DESC;
```
10 - Crie uma métrica mostrando a porcentagem de vendas a mais que a melhor categoria tem em relação a pior no ano de 2022.
```sql
WITH vendas_maior AS ( 
  SELECT 
    c.nome_categoria, 
    COUNT(iv.venda_id) AS qtd_vendas 
  FROM itens_venda iv
  JOIN produtos p ON p.id_produto = iv.produto_id
  JOIN categorias c ON c.id_categoria = p.categoria_id
  join vendas v on v.id_venda = iv.venda_id
  WHERE strftime('%Y', v.data_venda) = '2022'
  GROUP BY c.nome_categoria
  ORDER by qtd_vendas desc
  LIMIT 1
),
menor_venda AS ( 
  SELECT 
    c.nome_categoria, 
    COUNT(iv.venda_id) AS qtd_vendas 
  FROM itens_venda iv
  JOIN produtos p ON p.id_produto = iv.produto_id
  JOIN categorias c ON c.id_categoria = p.categoria_id
  join vendas v on v.id_venda = iv.venda_id
  WHERE strftime('%Y', v.data_venda) = '2022'
  GROUP BY c.nome_categoria
  ORDER by qtd_vendas asc
  LIMIT 1
),
total AS (
  SELECT COUNT(*) AS total_vendas FROM itens_venda
)
SELECT 
  vm.nome_categoria,
  ROUND((vm.qtd_vendas * 100.0) / t.total_vendas, 2) AS porcentagem,
  mv.nome_categoria,
  ROUND((mv.qtd_vendas * 100.0) / t.total_vendas, 2) AS porcentagem,
  ROUND (ROUND((vm.qtd_vendas * 100.0) / t.total_vendas, 2) - ROUND((mv.qtd_vendas * 100.0) / t.total_vendas, 2), 2) AS diferença
FROM vendas_maior vm, total t, menor_venda mv
ORDER BY porcentagem DESC;
```
