# Delta Lake

## O que é o Delta Lake?

O **Delta Lake** é um formato de tabela open source que adiciona uma camada de confiabilidade ao data lake. Ele traz suporte a transações **ACID**, versionamento de dados e operações DML (INSERT, UPDATE, DELETE) sobre arquivos Parquet armazenados em disco: recursos que o Parquet puro não oferece.

Foi criado pela Databricks e hoje é mantido pela Linux Foundation.

---

## Principais Recursos

| Recurso | Descrição |
|---------|-----------|
| Transações ACID | Garante atomicidade, consistência, isolamento e durabilidade |
| Time Travel | Consulta versões anteriores da tabela |
| Schema Evolution | Adiciona/altera colunas sem recriar a tabela |
| DML completo | Suporte a UPDATE, DELETE e MERGE |
| `_delta_log` | Log de transações em JSON que registra cada operação |

---

## Como funciona internamente

Cada tabela Delta é composta por:

- **Arquivos Parquet**: os dados propriamente ditos
- **`_delta_log/`**: diretório com arquivos JSON registrando cada transação (commit log)

```
spark-warehouse/carro_delta/
├── _delta_log/
│   ├── 00000000000000000000.json   ← CREATE TABLE
│   ├── 00000000000000000001.json   ← INSERT
│   ├── 00000000000000000002.json   ← ALTER TABLE
│   └── ...
├── part-00000-....parquet
└── part-00001-....parquet
```

---

## Exemplos Práticos

### Criar a tabela

```python
spark.sql("""
    CREATE TABLE carro_delta (id INT, placa STRING) USING delta
""")
```

!!! tip "Recriando a tabela"
    Se a pasta já existir de uma execução anterior, limpe antes de recriar:
    ```python
    import shutil
    spark.sql("DROP TABLE IF EXISTS carro_delta")
    shutil.rmtree("spark-warehouse/carro_delta", ignore_errors=True)
    ```

---

### INSERT

Inserindo múltiplos registros de uma vez:

```python
spark.sql("""
    INSERT INTO carro_delta VALUES
        (1, 'XYZ1J34'),
        (2, 'RLC5B93'),
        (3, 'ABV1V23')
""")

spark.sql("SELECT * FROM carro_delta").show()
```

```
+---+-------+
| id|  placa|
+---+-------+
|  1|XYZ1J34|
|  2|RLC5B93|
|  3|ABV1V23|
+---+-------+
```

---

### ALTER TABLE (Schema Evolution)

Adicionando novas colunas sem recriar a tabela:

```python
spark.sql("""
    ALTER TABLE carro_delta ADD COLUMNS (marca STRING, modelo STRING, ano INT)
""")

spark.sql("SELECT * FROM carro_delta").show()
```

```
+---+-------+-----+------+----+
| id|  placa|marca|modelo| ano|
+---+-------+-----+------+----+
|  1|XYZ1J34| null|  null|null|
|  2|RLC5B93| null|  null|null|
|  3|ABV1V23| null|  null|null|
+---+-------+-----+------+----+
```

---

### UPDATE

Atualizando registros existentes por condição:

```python
spark.sql("""
    UPDATE carro_delta
    SET marca = 'Renault', modelo = 'Sandero', ano = 2021
    WHERE id = 1
""")

spark.sql("""
    UPDATE carro_delta
    SET marca = 'GM', modelo = 'Tracker', ano = 2020
    WHERE id = 2
""")

spark.sql("""
    UPDATE carro_delta
    SET marca = 'Ford', modelo = 'EcoSport', ano = 2022
    WHERE id = 3
""")

spark.sql("SELECT * FROM carro_delta").show()
```

```
+---+-------+-------+--------+----+
| id|  placa|  marca|  modelo| ano|
+---+-------+-------+--------+----+
|  1|XYZ1J34|Renault| Sandero|2021|
|  2|RLC5B93|     GM| Tracker|2020|
|  3|ABV1V23|   Ford|EcoSport|2022|
+---+-------+-------+--------+----+
```

---

### DELETE

Removendo registros por condição:

```python
spark.sql("""
    DELETE FROM carro_delta WHERE id = 3
""")

spark.sql("SELECT * FROM carro_delta").show()
```

```
+---+-------+-------+-------+----+
| id|  placa|  marca| modelo| ano|
+---+-------+-------+-------+----+
|  1|XYZ1J34|Renault|Sandero|2021|
|  2|RLC5B93|     GM|Tracker|2020|
+---+-------+-------+-------+----+
```

---

### Histórico de Versões (Time Travel)

O Delta registra cada operação. Para visualizar o histórico completo:

```python
from delta.tables import DeltaTable

carro = DeltaTable.forPath(spark, "./spark-warehouse/carro_delta")
carro.history().show(truncate=False)
```

Ou via SQL:

```python
spark.sql("DESCRIBE HISTORY carro_delta").show(truncate=False)
```

Exemplo de saída:

```
+-------+---------------------------+---------+----------+-----------+
|version|timestamp                  |operation|userName   |operationParameters|
+-------+---------------------------+---------+----------+-----------+
|      5|2024-04-25 14:00:00.000    |DELETE   |NULL      |...        |
|      4|2024-04-25 13:59:00.000    |UPDATE   |NULL      |...        |
|      3|2024-04-25 13:58:00.000    |UPDATE   |NULL      |...        |
|      2|2024-04-25 13:57:00.000    |UPDATE   |NULL      |...        |
|      1|2024-04-25 13:56:00.000    |INSERT   |NULL      |...        |
|      0|2024-04-25 13:55:00.000    |CREATE TABLE|NULL   |...        |
+-------+---------------------------+---------+----------+-----------+
```

Consultando uma versão anterior:

```python
# Versão 1 (logo após o INSERT)
spark.read.format("delta") \
    .option("versionAsOf", 1) \
    .load("./spark-warehouse/carro_delta") \
    .show()
```

---

### Verificar se é uma Delta Table

```python
from delta.tables import DeltaTable

DeltaTable.isDeltaTable(spark, "spark-warehouse/carro_delta")
# True
```
