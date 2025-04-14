# Banco de Dados - SQL

Este documento apresenta exemplos de comandos SQL para operações em banco de dados, baseado no material do SENAI.

## Estrutura Básica de Banco de Dados

### Criação de Banco de Dados

```sql
CREATE DATABASE biblioteca CHARACTER SET utf8mb4;
```

### Visualizar Bancos de Dados

```sql
SHOW DATABASES;
```

## Modelo Físico (SQL)

### Criação de Tabelas

```sql
CREATE TABLE livros (
    id INT NOT NULL PRIMARY KEY AUTO_INCREMENT,
    titulo VARCHAR(100) NOT NULL,
    isbn VARCHAR(20) NOT NULL,
    ano INT NOT NULL,
    categoria_id INT NOT NULL
);

CREATE TABLE categorias (
    id INT NOT NULL PRIMARY KEY AUTO_INCREMENT,
    nome VARCHAR(45) NOT NULL,
    descricao VARCHAR(100)
);

-- Criando relação entre tabelas com chave estrangeira
ALTER TABLE livros
ADD CONSTRAINT fk_livros_categorias
FOREIGN KEY (categoria_id) REFERENCES categorias(id);
```

## Operações CRUD

### CREATE (C) - INSERT

Inserção de dados nas tabelas:

```sql
-- Inserindo categorias
INSERT INTO categorias (nome, descricao)
VALUES ('Ficção Científica', 'Livros que exploram conceitos científicos avançados');

INSERT INTO categorias (nome, descricao)
VALUES ('Romance', 'Narrativas centradas em relações amorosas');

-- Inserindo livros
INSERT INTO livros (titulo, isbn, ano, categoria_id)
VALUES ('Fundação', '9788576572664', 1951, 1);

INSERT INTO livros (titulo, isbn, ano, categoria_id)
VALUES ('Orgulho e Preconceito', '9788544001820', 1813, 2);
```

### READ (R) - SELECT

Consultas ao banco de dados:

```sql
-- Selecionar todos os livros
SELECT * FROM livros;

-- Selecionar livros com informações de categoria
SELECT 
    l.id, l.titulo, l.ano, c.nome AS categoria
FROM
    livros l
JOIN
    categorias c ON l.categoria_id = c.id
WHERE
    l.ano > 1900;
```

### UPDATE (U)

Atualização de registros:

```sql
-- Atualizando o ano de um livro
UPDATE livros
SET ano = 1952
WHERE id = 1;
```

### DELETE (D)

Exclusão de registros:

```sql
-- Removendo um livro específico
DELETE FROM livros
WHERE id = 1;
```

## Recursos Avançados em SQL

### Views

Criação e uso de views:

```sql
-- Criando uma view que mostra livros com suas categorias
CREATE VIEW vw_livros_categorias AS
SELECT
    l.id, l.titulo, l.isbn, l.ano, c.nome AS categoria, c.descricao AS categoria_descricao
FROM
    livros l
JOIN
    categorias c ON l.categoria_id = c.id;

-- Usando a view
SELECT * FROM vw_livros_categorias WHERE ano > 1900;
```
Visualizar dados da view criada:

```sql
-- Visualizando a view 
SELECT * FROM vw_livros_categorias;

-- Visualizando a view com filtro por ano
SELECT * FROM vw_livros_categorias WHERE ano > 1900;

```
Como listar todas as VIEWS do banco?

```sql
-- Listando as views do banco 
SHOW FULL TABLES WHERE Table_Type = 'VIEW';

```

### Stored Procedures

Procedimentos armazenados:

```sql
-- Procedure para adicionar um novo livro
DELIMITER //
CREATE PROCEDURE sp_adicionar_livro(
    IN p_titulo VARCHAR(100),
    IN p_isbn VARCHAR(20),
    IN p_ano INT,
    IN p_categoria_id INT
)
BEGIN
    INSERT INTO livros (titulo, isbn, ano, categoria_id)
    VALUES (p_titulo, p_isbn, p_ano, p_categoria_id);
    
    SELECT 'Livro adicionado com sucesso!' AS mensagem;
END //
DELIMITER ;

-- Executando a procedure para adicionar livros
CALL sp_adicionar_livro('Neuromancer', '9788576570493', 1984, 1);
CALL sp_adicionar_livro('Dom Casmurro', '9783161484100', 1899, 1);

```
Visualizar dados da procedure criada:

```sql

-- Visualizando se os livros foram inclusos
SELECT * FROM livros;


-- Visualizando a procedure
SHOW PROCEDURE STATUS WHERE Db = 'biblioteca';


```

### Triggers

Exemplo de trigger para registro de alterações:

```sql
DELIMITER //
CREATE TRIGGER tr_log_alteracoes_livros
AFTER UPDATE ON livros
FOR EACH ROW
BEGIN
    INSERT INTO log_alteracoes (tabela, id_registro, campo_alterado, valor_antigo, valor_novo, data_alteracao)
    VALUES (
        'livros',
        NEW.id,
        CASE
            WHEN OLD.titulo != NEW.titulo THEN 'titulo'
            WHEN OLD.ano != NEW.ano THEN 'ano'
            ELSE 'outro'
        END,
        CASE
            WHEN OLD.titulo != NEW.titulo THEN OLD.titulo
            WHEN OLD.ano != NEW.ano THEN CAST(OLD.ano AS CHAR)
            ELSE ''
        END,
        CASE
            WHEN OLD.titulo != NEW.titulo THEN NEW.titulo
            WHEN OLD.ano != NEW.ano THEN CAST(NEW.ano AS CHAR)
            ELSE ''
        END,
        NOW()
    );
END //
DELIMITER ;
```

```markdown
# 📌 Como Visualizar os Triggers Criados?

Para listar todos os triggers no MySQL, use:

```sql
SHOW TRIGGERS FROM biblioteca;
```

Se quiser ver o código do trigger específico, use:

```sql
SHOW CREATE TRIGGER tr_log_alteracoes_livros;
```

---

# 📌 Como Testar o Trigger?

## 🔹 1️⃣ Criar a Tabela `log_alteracoes` (se ainda não existir)

O trigger insere dados em `log_alteracoes`, então certifique-se de que essa tabela existe:

```sql
CREATE TABLE log_alteracoes (
    id INT AUTO_INCREMENT PRIMARY KEY,
    tabela VARCHAR(50),
    id_registro INT,
    campo_alterado VARCHAR(50),
    valor_antigo TEXT,
    valor_novo TEXT,
    data_alteracao DATETIME
);
```

---

## 🔹 2️⃣ Inserir um Registro na Tabela `livros` para Teste

```sql
INSERT INTO livros (id, titulo, isbn, ano, categoria_id) 
VALUES (1, 'O Senhor dos Anéis', '9780345339706', 1954, 2);
```

---

## 🔹 3️⃣ Atualizar um Registro na Tabela `livros` para Acionar o Trigger

```sql
UPDATE livros 
SET titulo = 'O Hobbit' 
WHERE id = 1;
```

Isso aciona o trigger e insere um novo registro na tabela `log_alteracoes`.

---

## 🔹 4️⃣ Verificar se o Trigger Funcionou

Após a atualização, consulte a tabela `log_alteracoes` para ver o que foi registrado:

```sql
SELECT * FROM log_alteracoes;
```

Se tudo estiver certo, a saída mostrará que o **campo `titulo` foi alterado**, incluindo o **valor antigo** (`O Senhor dos Anéis`) e o **novo** (`O Hobbit`).

---

# 📌 Como Excluir um Trigger?

Se precisar excluir o trigger, use:

```sql
DROP TRIGGER tr_log_alteracoes_livros;
```

---

# 🔥 Conclusão

✅ O trigger monitora alterações na tabela `livros` e registra mudanças na tabela `log_alteracoes`.  
✅ Para testar, **atualize um registro** e depois consulte `log_alteracoes`.  
✅ Para ver os triggers existentes, use `SHOW TRIGGERS FROM biblioteca;`.


