# Delta Lake

## O que é
Delta Lake é uma camada de armazenamento que adiciona confiabilidade e suporte a operações ACID ao Apache Spark.

## Principais características
- Suporte a operações ACID
- Permite UPDATE e DELETE
- Controle de versão dos dados
- Maior confiabilidade

## Uso no projeto
No projeto, o Delta Lake foi utilizado para manipulação de dados com operações completas de CRUD.

### Operações realizadas

#### INSERT
Adição de novos registros na tabela.

#### UPDATE
Atualização de dados existentes com base em condições.

#### DELETE
Remoção de registros da tabela.

## Exemplo
Foi criada uma tabela de clientes contendo:
- id
- nome
- estado

## Exemplo prático com PySpark

```python
from delta.tables import DeltaTable

# Criar dados
data = [(1, "Lucas", "SC"), (2, "Felipe", "SC")]
df = spark.createDataFrame(data, ["id", "nome", "estado"])

# Salvar como Delta
df.write.format("delta").save("data/clientes_delta")

# Atualizar dados
delta_table = DeltaTable.forPath(spark, "data/clientes_delta")
delta_table.update(condition="id = 1", set={"estado": "'PR'"})

# Ler dados
df = spark.read.format("delta").load("data/clientes_delta")
df.show()
