# Apache Spark (PySpark)

## O que é o Apache Spark?

O **Apache Spark** é um framework de processamento de dados distribuído, criado para processar grandes volumes de dados de forma rápida e eficiente. Ele opera in-memory (na RAM), o que o torna muito mais veloz que abordagens tradicionais baseadas em disco como o Hadoop MapReduce.

O **PySpark** é a API Python do Spark, que permite utilizar todo o poder do Spark com a sintaxe do Python.

---

## Conceitos Fundamentais

### SparkSession

A `SparkSession` é o ponto de entrada de qualquer aplicação PySpark. Ela encapsula o `SparkContext` e provê acesso às APIs de SQL, DataFrames e Datasets.

```python
from pyspark.sql import SparkSession

spark = (
    SparkSession
    .builder
    .master("local[*]")  # (1)
    .getOrCreate()
)
```

1. `local[*]` executa o Spark localmente usando todos os núcleos disponíveis da CPU.

### DataFrame

O **DataFrame** é a estrutura de dados principal do Spark. Similar a uma tabela de banco de dados ou a um DataFrame do Pandas, mas distribuído e otimizado para processamento paralelo.

```python
df = spark.read.csv("dados.csv", header=True, inferSchema=True)
df.show()
df.printSchema()
```

### Spark SQL

O Spark SQL permite executar queries SQL padrão diretamente sobre DataFrames e tabelas registradas:

```python
spark.sql("SELECT * FROM minha_tabela WHERE ano > 2020").show()
```

---

## Configuração com Delta Lake

Para habilitar o Delta Lake no Spark, é necessário configurar as extensões durante a criação da `SparkSession`:

```python
from pyspark.sql import SparkSession
from delta import *

spark = (
    SparkSession
    .builder
    .master("local[*]")
    .config("spark.jars.packages", "io.delta:delta-spark_2.12:3.2.0")  # (1)
    .config("spark.sql.extensions", "io.delta.sql.DeltaSparkSessionExtension")  # (2)
    .config("spark.sql.catalog.spark_catalog", "org.apache.spark.sql.delta.catalog.DeltaCatalog")  # (3)
    .getOrCreate()
)
```

1. Baixa automaticamente o pacote Delta Lake do Maven durante a inicialização.
2. Habilita a extensão SQL do Delta no Spark.
3. Substitui o catálogo padrão do Spark pelo catálogo Delta.

---

## Requisitos do Ambiente

| Componente | Versão usada | Descrição |
|------------|-------------|-----------|
| Python     | 3.13        | Gerenciado pelo pyenv |
| Java (JDK) | 17          | Obrigatório para rodar a JVM do Spark |
| PySpark    | 3.5.3       | API Python do Apache Spark |
| Delta Spark| 3.2.0       | Integração Delta Lake + Spark |
| Scala      | 2.12        | Runtime interno do Spark (não instalado separadamente) |

!!! warning "Java é obrigatório"
    O PySpark inicia um processo Java (JVM) internamente. Se o Java não estiver instalado, o erro `[JAVA_GATEWAY_EXITED]` será gerado ao criar a SparkSession.

---

## Modos de Execução

| Modo | Configuração | Uso |
|------|-------------|-----|
| Local (1 core) | `local` | Testes simples |
| Local (N cores) | `local[4]` | Paralelismo controlado |
| Local (todos os cores) | `local[*]` | Desenvolvimento local: usado neste projeto |
| Cluster | `spark://host:7077` | Produção distribuída |

---

## Fluxo do Projeto

```
Dados → SparkSession → DataFrame → Delta Table / Iceberg Table → Consultas SQL
```

O Spark atua como o motor de processamento central, e tanto o Delta Lake quanto o Iceberg são **formatos de tabela** que se integram a ele para adicionar capacidades como versionamento, ACID e time travel.
