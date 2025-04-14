# Resolução

## Implementação SQL

### 1. Criação do Banco de Dados

```sql
-- Criação do banco de dados
-- 1ª Digitação (Início das respostas)

-- Seleciona o banco de dados

```

### 2. Criação das Tabelas

```sql
-- Tabela de palestrantes






-- Tabela de eventos







-- Tabela de inscrições







```

### 3. Inserção de Dados Iniciais

```sql
-- Inserir palestrantes



-- Inserir eventos



-- Inserir algumas inscrições



```

### 4. View para Consulta

```sql
-- View para listar eventos com detalhes
CREATE VIEW vw_eventos_detalhados AS
SELECT 
    e.id,
    e.titulo,
    e.data_evento,
    e.local,
    e.capacidade,
    p.nome AS palestrante,
    p.especialidade,
    COUNT(i.id) AS total_inscritos,
    e.capacidade - COUNT(i.id) AS vagas_disponiveis
FROM 
    eventos e
LEFT JOIN 
    palestrantes p ON e.palestrante_id = p.id
LEFT JOIN 
    inscricoes i ON e.id = i.evento_id
GROUP BY 
    e.id, e.titulo, e.data_evento, e.local, e.capacidade, p.nome, p.especialidade;

-- Consultar a view
SELECT * FROM vw_eventos_detalhados;
```

### 5. Consultas Úteis

```sql
-- 1. Listar todos os eventos com suas respectivas informações



-- 2. Mostrar apenas eventos com vagas disponíveis (usando a view criada)
SELECT id, titulo, data_evento, local, vagas_disponiveis 
FROM vw_eventos_detalhados
WHERE vagas_disponiveis > 0;

-- 3. Listar participantes inscritos em um evento específico



```
