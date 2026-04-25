# Apache Iceberg

## O que é o Apache Iceberg?

O **Apache Iceberg** é um formato de tabela open source de alto desempenho para grandes conjuntos de dados analíticos. Foi criado pela Netflix e hoje é um projeto top-level da Apache Software Foundation.

Assim como o Delta Lake, o Iceberg adiciona suporte a transações ACID, versionamento e operações DML sobre arquivos armazenados em data lakes: mas com foco especial em escalabilidade extrema e compatibilidade com múltiplos engines (Spark, Flink, Trino, Hive, etc.).

---

## Comparativo: Delta Lake vs Apache Iceberg

| Característica | Delta Lake | Apache Iceberg |
|---------------|------------|----------------|
| Criado por | Databricks | Netflix |
| Licença | Apache 2.0 | Apache 2.0 |
| Transações ACID | Sim | Sim |
| Time Travel | Sim | Sim |
| Schema Evolution | Sim | Sim (mais avançado) |
| Partition Evolution | Não | Sim |
| Compatibilidade de engines | Spark-first | Multi-engine nativo |
| Log de transações | JSON (`_delta_log`) | Avro + JSON + Parquet |
| Hidden Partitioning | Não | Sim |

---

## Principais Recursos

### Hidden Partitioning

O Iceberg gerencia partições de forma transparente: você não precisa incluir a coluna de partição nas queries, pois o engine resolve automaticamente:

```sql
-- Sem hidden partitioning (Hive-style), você precisaria filtrar assim:
SELECT * FROM eventos WHERE dt = '2024-04-25'

-- Com Iceberg, o filtro em qualquer campo particionado funciona:
SELECT * FROM eventos WHERE timestamp > '2024-04-25 00:00:00'
```

### Partition Evolution

Permite alterar a estratégia de particionamento sem reescrever os dados existentes:

```python
spark.sql("""
    ALTER TABLE carro_iceberg
    ADD PARTITION FIELD bucket(4, id)
""")
```

### Schema Evolution

Suporta adição, renomeação e reordenação de colunas sem quebrar leituras anteriores:

```python
spark.sql("""
    ALTER TABLE carro_iceberg ADD COLUMN quilometragem DOUBLE
""")
```

---

## Configuração do Ambiente

Para usar Iceberg com PySpark, configure a `SparkSession` com o catálogo Iceberg:

```python
from pyspark.sql import SparkSession

spark = (
    SparkSession
    .builder
    .master("local[*]")
    .config("spark.jars.packages",
        "org.apache.iceberg:iceberg-spark-runtime-3.5_2.12:1.5.0")
    .config("spark.sql.extensions",
        "org.apache.iceberg.spark.extensions.IcebergSparkSessionExtensions")
    .config("spark.sql.catalog.spark_catalog",
        "org.apache.iceberg.spark.SparkSessionCatalog")
    .config("spark.sql.catalog.spark_catalog.type", "hive")
    .config("spark.sql.catalog.local", "org.apache.iceberg.spark.SparkCatalog")
    .config("spark.sql.catalog.local.type", "hadoop")
    .config("spark.sql.catalog.local.warehouse", "./iceberg-warehouse")
    .getOrCreate()
)
```

---

## Exemplos Práticos

### Criar a tabela

```python
spark.sql("""
    CREATE TABLE local.db.carro_iceberg (
        id     INT,
        placa  STRING,
        marca  STRING,
        modelo STRING,
        ano    INT
    )
    USING iceberg
""")
```

---

### INSERT

```python
spark.sql("""
    INSERT INTO local.db.carro_iceberg VALUES
        (1, 'XYZ1J34', 'Renault', 'Sandero',  2021),
        (2, 'RLC5B93', 'GM',      'Tracker',  2020),
        (3, 'ABV1V23', 'Ford',    'EcoSport', 2022)
""")

spark.sql("SELECT * FROM local.db.carro_iceberg").show()
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

### UPDATE

```python
spark.sql("""
    UPDATE local.db.carro_iceberg
    SET ano = 2023
    WHERE id = 1
""")
```

---

### DELETE

```python
spark.sql("""
    DELETE FROM local.db.carro_iceberg
    WHERE id = 3
""")
```

---

### Histórico de Snapshots (Time Travel)

O Iceberg versiona os dados através de **snapshots**. Cada operação DML gera um novo snapshot:

```python
spark.sql("""
    SELECT * FROM local.db.carro_iceberg.snapshots
""").show(truncate=False)
```

Consultando dados de um snapshot anterior:

```python
# Por ID do snapshot
spark.read \
    .option("snapshot-id", 123456789) \
    .format("iceberg") \
    .load("local.db.carro_iceberg") \
    .show()

# Por timestamp
spark.read \
    .option("as-of-timestamp", "2024-04-25 12:00:00") \
    .format("iceberg") \
    .load("local.db.carro_iceberg") \
    .show()
```

---

### Estrutura de arquivos no disco

```
iceberg-warehouse/
└── db/
    └── carro_iceberg/
        ├── data/
        │   └── part-00000-....parquet
        └── metadata/
            ├── v1.metadata.json      ← snapshot 1 (CREATE)
            ├── v2.metadata.json      ← snapshot 2 (INSERT)
            ├── v3.metadata.json      ← snapshot 3 (UPDATE)
            ├── snap-....avro         ← arquivos de snapshot
            └── version-hint.text
```

Diferente do Delta (que usa JSON no `_delta_log`), o Iceberg usa uma combinação de **JSON** (metadados), **Avro** (manifests) e **Parquet** (dados).
