# Apache Iceberg

## O que é
Apache Iceberg é um formato de tabela para grandes volumes de dados, focado em performance, confiabilidade e suporte a operações ACID.

## Principais características
- Suporte a operações ACID
- Alta performance para consultas analíticas
- Gerenciamento eficiente de grandes volumes de dados
- Integração com Apache Spark

## Uso no projeto
No projeto, o Apache Iceberg foi utilizado para manipulação de dados através de comandos SQL diretamente no Spark.

### Operações realizadas

#### INSERT
Inserção de novos registros na tabela.

#### UPDATE
Atualização de registros existentes com base em condições.

#### DELETE
Remoção de registros da tabela.

## Exemplo
Foi criada uma tabela de clientes contendo:
- id
- nome
- estado

Onde foram aplicadas operações de inserção, atualização e exclusão.
