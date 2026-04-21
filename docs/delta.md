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

Onde foram aplicadas operações de inserção, atualização e exclusão.
