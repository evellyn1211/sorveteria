# Projeto de Banco de Dados: Sistema de Gestão para Sorveteria (Gelato & Cia)

**Disciplina:** Banco de Dados / SEDUC DEST2  
**Atividade:** Atividade III - Planejar, criar e testar um banco de dados relacional  
**Aluno:** [Seu Nome Aqui]  
**Data de Entrega:** 03/11  

---

## 1. Domínio do Banco de Dados
**Domínio:** Comercial / Varejo (Sorveteria e Gelateria).  
**Nome do Sistema:** *Gelato & Cia - Sistema de Gestão de Vendas e Estoque*.

---

## 2. Contextualização: O Problema
A *Gelato & Cia* é uma sorveteria artesanal em expansão que gerenciava suas operações diárias utilizando anotações em papel e planilhas avulsas do Excel. 

### Problemas enfrentados sem um SGBD Relacional:
* **Falta de controle de estoque:** Perda de insumos e recipientes de sorvete devido à falta de rastreamento de validade e saldo em tempo real.
* **Redundância e inconsistência de dados:** Clientes e produtos duplicados com grafias e preços diferentes.
* **Impossibilidade de rastrear vendas:** Dificuldade para saber quais sabores são os mais vendidos e quais funcionários realizaram os atendimentos.
* **Lentidão operacional:** Falta de relatórios automatizados para fechamento de caixa e cálculo de faturamento diário, mensal e por forma de pagamento.

---

## 3. A Proposta de Melhoria
Implementação de um **Sistema Gerenciador de Banco de Dados Relacional (SGBD - MySQL)** centralizado.

### Soluções entregues:
* **Integridade Referencial:** Uso de Chaves Primárias (PK) e Estrangeiras (FK) para impedir vendas sem clientes/funcionários válidos ou produtos inexistentes.
* **Controle de Vendas Detalhado:** Modelagem de itens de venda para registrar múltiplos produtos por pedido.
* **Automação de Relatórios:** Possibilidade de realizar consultas complexas (faturamento por período, produtos mais vendidos, desempenho de funcionários).

---

## 4. Modelagem Lógica do Banco de Dados

### Tabelas, Campos e Relacionamentos

1. **`Cliente`**
   * `id_cliente` (INT, PK, Auto Increment) — Identificador único
   * `nome` (VARCHAR(100), NOT NULL) — Nome do cliente
   * `cpf` (VARCHAR(14), UNIQUE) — CPF do cliente
   * `telefone` (VARCHAR(20)) — Telefone de contato
   * `email` (VARCHAR(100)) — E-mail do cliente

2. **`Funcionario`**
   * `id_funcionario` (INT, PK, Auto Increment) — Identificador único
   * `nome` (VARCHAR(100), NOT NULL) — Nome do funcionário
   * `cargo` (VARCHAR(50)) — Cargo (Atendente, Caixa, Gerente)
   * `salario` (DECIMAL(10,2)) — Salário do funcionário

3. **`Produto`**
   * `id_produto` (INT, PK, Auto Increment) — Identificador único
   * `nome` (VARCHAR(100), NOT NULL) — Sabor/Nome do item
   * `tipo` (VARCHAR(50)) — Categoria (Massa, Picolé, Bebida, Açaí)
   * `preco_unitario` (DECIMAL(10,2), NOT NULL) — Preço de venda
   * `quantidade_estoque` (INT, NOT NULL) — Saldo em estoque

4. **`Venda`**
   * `id_venda` (INT, PK, Auto Increment) — Identificador único da venda
   * `data_hora` (DATETIME, NOT NULL) — Data e hora da transação
   * `forma_pagamento` (VARCHAR(30)) — Pix, Cartão de Crédito, Dinheiro, etc.
   * `valor_total` (DECIMAL(10,2)) — Valor total da venda
   * `id_cliente` (INT, FK) — Referência a `Cliente`
   * `id_funcionario` (INT, FK) — Referência a `Funcionario`

5. **`Item_Venda`** (Tabela Intermediária / Associativa)
   * `id_venda` (INT, FK, PK composta) — Referência a `Venda`
   * `id_produto` (INT, FK, PK composta) — Referência a `Produto`
   * `quantidade` (INT, NOT NULL) — Quantidade vendida do produto
   * `preco_subtotal` (DECIMAL(10,2), NOT NULL) — Subtotal do item

### Cardinalidades e Tipos de Relacionamento:
* **Cliente -> Venda:** `1 : N` (Um cliente pode realizar zero ou várias vendas; uma venda pertence a um único cliente).
* **Funcionario -> Venda:** `1 : N` (Um funcionário pode atender zero ou várias vendas; uma venda é realizada por um único funcionário).
* **Venda <-> Produto:** `N : M` (Uma venda pode conter vários produtos; um produto pode estar em várias vendas). Resolvido através da tabela **`Item_Venda`** (`1:N` de Venda para Item_Venda e `1:N` de Produto para Item_Venda).

---

## 5. Aplicação das Formas Normais (1FN, 2FN, 3FN e 4FN)

* **1FN (Primeira Forma Normal):**
  * *Aplicação:* Todos os atributos possuem valores atômicos e indivisíveis. Não há colunas com múltiplos telefones ou listas de produtos numa mesma célula.
  * *Por quê:* Evita inconsistências na busca e garante que cada registro seja atômico.
* **2FN (Segunda Forma Normal):**
  * *Aplicação:* O banco já atende à 1FN e todos os atributos não-chave dependem totalmente da chave primária. Na tabela `Item_Venda` (chave composta por `id_venda` e `id_produto`), a `quantidade` e o `preco_subtotal` dependem da combinação completa de ambas as chaves.
  * *Por quê:* Elimina a dependência parcial de atributos em chaves primárias compostas.
* **3FN (Terceira Forma Normal):**
  * *Aplicação:* O banco atende à 2FN e não existem dependências transitivas. Atributos como o `cargo` ou `salario` pertencem exclusivamente à tabela `Funcionario` e não são duplicados na tabela `Venda`.
  * *Por quê:* Garante que mudanças em dados secundários (ex: alteração no preço do produto) não exijam alteração no histórico das vendas passadas.
* **4FN (Quarta Forma Normal):**
  * *Aplicação:* Elimina dependências multivaloradas independentes na mesma relação. Caso o produto tivesse múltiplos fornecedores e múltiplos ingredientes, esses seriam isolados em tabelas próprias.
  * *Por quê:* Impede redundâncias desnecessárias e multiplicação cartesiana de registros.

---

## 6. Instruções SQL

```sql
-- 1. CRIAR O BANCO DE DADOS
CREATE DATABASE sorveteria_db;
USE sorveteria_db;

-- 2. CRIAR AS TABELAS
CREATE TABLE Cliente (
    id_cliente INT AUTO_INCREMENT PRIMARY KEY,
    nome VARCHAR(100) NOT NULL,
    cpf VARCHAR(14) UNIQUE,
    telefone VARCHAR(20),
    email VARCHAR(100)
);

CREATE TABLE Funcionario (
    id_funcionario INT AUTO_INCREMENT PRIMARY KEY,
    nome VARCHAR(100) NOT NULL,
    cargo VARCHAR(50),
    salario DECIMAL(10,2)
);

CREATE TABLE Produto (
    id_produto INT AUTO_INCREMENT PRIMARY KEY,
    nome VARCHAR(100) NOT NULL,
    tipo VARCHAR(50),
    preco_unitario DECIMAL(10,2) NOT NULL,
    quantidade_estoque INT NOT NULL
);

CREATE TABLE Venda (
    id_venda INT AUTO_INCREMENT PRIMARY KEY,
    data_hora DATETIME NOT NULL,
    forma_pagamento VARCHAR(30),
    valor_total DECIMAL(10,2),
    id_cliente INT,
    id_funcionario INT,
    FOREIGN KEY (id_cliente) REFERENCES Cliente(id_cliente),
    FOREIGN KEY (id_funcionario) REFERENCES Funcionario(id_funcionario)
);

CREATE TABLE Item_Venda (
    id_venda INT,
    id_produto INT,
    quantidade INT NOT NULL,
    preco_subtotal DECIMAL(10,2) NOT NULL,
    PRIMARY KEY (id_venda, id_produto),
    FOREIGN KEY (id_venda) REFERENCES Venda(id_venda),
    FOREIGN KEY (id_produto) REFERENCES Produto(id_produto)
);

-- 3. POPULAR TABELAS (MINÍMO 5 REGISTROS CADA)

INSERT INTO Cliente (nome, cpf, telefone, email) VALUES
('Ana Silva', '111.222.333-44', '11999990001', 'ana@email.com'),
('Bruno Costa', '222.333.444-55', '11999990002', 'bruno@email.com'),
('Carla Souza', '333.444.555-66', '11999990003', 'carla@email.com'),
('Diego Lima', '444.555.666-77', '11999990004', 'diego@email.com'),
('Elena Rostova', '555.666.777-88', '11999990005', 'elena@email.com');

INSERT INTO Funcionario (nome, cargo, salario) VALUES
('Carlos Atendente', 'Atendente', 1800.00),
('Mariana Gerente', 'Gerente', 3500.00),
('João Sorveteiro', 'Atendente', 1800.00),
('Fernanda Caixa', 'Caixa', 2000.00),
('Lucas Auxiliar', 'Auxiliar de Cozinha', 1600.00);

INSERT INTO Produto (nome, tipo, preco_unitario, quantidade_estoque) VALUES
('Casquinha Chocolate', 'Massa', 8.50, 100),
('Sundae Morango', 'Taça', 15.00, 50),
('Picolé de Limão', 'Picolé', 4.00, 200),
('Milkshake Ovaltine', 'Bebida', 22.00, 40),
('Açaí 500ml', 'Açaí', 18.00, 80);

INSERT INTO Venda (data_hora, forma_pagamento, valor_total, id_cliente, id_funcionario) VALUES
('2024-10-10 14:30:00', 'Pix', 30.50, 1, 1),
('2024-10-10 15:00:00', 'Cartão de Crédito', 22.00, 2, 4),
('2024-10-11 16:15:00', 'Dinheiro', 8.50, 3, 3),
('2024-10-12 18:00:00', 'Pix', 33.00, 4, 1),
('2024-10-13 19:45:00', 'Cartão de Débito', 18.00, 5, 4);

INSERT INTO Item_Venda (id_venda, id_produto, quantidade, preco_subtotal) VALUES
(1, 1, 1, 8.50),  -- 1 Casquinha Chocolate
(1, 4, 1, 22.00), -- 1 Milkshake Ovaltine
(2, 4, 1, 22.00), -- 1 Milkshake Ovaltine
(3, 1, 1, 8.50),  -- 1 Casquinha Chocolate
(4, 2, 1, 15.00), -- 1 Sundae Morango
(4, 5, 1, 18.00), -- 1 Açaí 500ml
(5, 5, 1, 18.00); -- 1 Açaí 500ml

-- 4. 15 CONSULTAS CONTEXTUALIZADAS COM RESPOSTAS

-- C1: Listar todas as vendas com nome do cliente e do funcionário
SELECT V.id_venda, V.data_hora, C.nome AS Cliente, F.nome AS Funcionario, V.valor_total
FROM Venda V
INNER JOIN Cliente C ON V.id_cliente = C.id_cliente
INNER JOIN Funcionario F ON V.id_funcionario = F.id_funcionario;
-- Resposta: Exibe as 5 vendas detalhando os nomes envolvidos.

-- C2: Faturamento total agrupado por forma de pagamento
SELECT forma_pagamento, SUM(valor_total) AS Faturamento
FROM Venda
GROUP BY forma_pagamento;
-- Resposta: Pix (63.50), Cartão de Crédito (22.00), Dinheiro (8.50), Cartão de Débito (18.00).

-- C3: Buscar clientes cujo nome contém 'Silva' ou 'Costa'
SELECT * FROM Cliente WHERE nome LIKE '%Silva%' OR nome LIKE '%Costa%';
-- Resposta: Ana Silva e Bruno Costa.

-- C4: Buscar vendas com valor entre R$ 20,00 e R$ 35,00 ordenadas por valor
SELECT * FROM Venda WHERE valor_total BETWEEN 20.00 AND 35.00 ORDER BY valor_total DESC;
-- Resposta: Venda 4 (33.00), Venda 1 (30.50), Venda 2 (22.00).

-- C5: Produtos do tipo 'Massa' ou 'Taça' com estoque abaixo de 60 unidades
SELECT * FROM Produto WHERE tipo IN ('Massa', 'Taça') AND quantidade_estoque < 60;
-- Resposta: Sundae Morango (estoque: 50).

-- C6: Quantidade total de itens vendidos por produto
SELECT P.nome, SUM(IV.quantidade) AS Total_Vendido
FROM Item_Venda IV
INNER JOIN Produto P ON IV.id_produto = P.id_produto
GROUP BY P.nome;
-- Resposta: Casquinha Chocolate (2), Milkshake Ovaltine (2), Açaí 500ml (2), Sundae Morango (1).

-- C7: Média salarial dos funcionários por cargo
SELECT cargo, AVG(salario) AS Média_Salario FROM Funcionario GROUP BY cargo;
-- Resposta: Atendente (1800.00), Gerente (3500.00), Caixa (2000.00), Auxiliar de Cozinha (1600.00).

-- C8: Listar os 3 produtos mais caros da sorveteria
SELECT * FROM Produto ORDER BY preco_unitario DESC LIMIT 3;
-- Resposta: Milkshake Ovaltine (22.00), Açaí 500ml (18.00), Sundae Morango (15.00).

-- C9: Vendas realizadas a partir de 12/10/2024
SELECT * FROM Venda WHERE data_hora >= '2024-10-12 00:00:00';
-- Resposta: Vendas de id 4 e 5.

-- C10: Funcionários que realizaram atendimentos e o total atendido por cada um
SELECT F.nome, COUNT(V.id_venda) AS Total_Atendimentos
FROM Funcionario F
INNER JOIN Venda V ON F.id_funcionario = V.id_funcionario
GROUP BY F.nome;
-- Resposta: Carlos Atendente (2), Fernanda Caixa (2), João Sorveteiro (1).

-- C11: Listar produtos que ainda NÃO foram vendidos
SELECT P.nome FROM Produto P
LEFT JOIN Item_Venda IV ON P.id_produto = IV.id_produto
WHERE IV.id_produto IS NULL;
-- Resposta: Picolé de Limão.

-- C12: Valor da venda mais cara e da venda mais barata
SELECT MAX(valor_total) AS Venda_Mais_Cara, MIN(valor_total) AS Venda_Mais_Barata FROM Venda;
-- Resposta: Venda Mais Cara (33.00), Venda Mais Barata (8.50).

-- C13: Detalhes dos itens de uma venda específica (Ex: Venda ID 1)
SELECT V.id_venda, P.nome AS Produto, IV.quantidade, IV.preco_subtotal
FROM Item_Venda IV
INNER JOIN Venda V ON IV.id_venda = V.id_venda
INNER JOIN Produto P ON IV.id_produto = P.id_produto
WHERE V.id_venda = 1;
-- Resposta: Casquinha Chocolate (1x - 8.50), Milkshake Ovaltine (1x - 22.00).

-- C14: Total gasto por cliente na sorveteria
SELECT C.nome, SUM(V.valor_total) AS Total_Gasto
FROM Cliente C
INNER JOIN Venda V ON C.id_cliente = V.id_cliente
GROUP BY C.nome
ORDER BY Total_Gasto DESC;
-- Resposta: Diego Lima (33.00), Ana Silva (30.50), Bruno Costa (22.00), Elena Rostova (18.00), Carla Souza (8.50).

-- C15: Contagem de produtos cadastrados por tipo
SELECT tipo, COUNT(*) AS Qtd_Produtos FROM Produto GROUP BY tipo;
-- Resposta: Massa (1), Taça (1), Picolé (1), Bebida (1), Açaí (1).
