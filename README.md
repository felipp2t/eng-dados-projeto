# Projeto Spark com Delta Lake e Apache Iceberg

## 📌 Descrição  
Este projeto demonstra o uso do Apache Spark com Delta Lake e Apache Iceberg para manipulação de dados com operações de INSERT, UPDATE e DELETE, simulando um ambiente de engenharia de dados com suporte a operações ACID.

## ⚙️ Tecnologias utilizadas  
- Apache Spark  
- Delta Lake  
- Apache Iceberg  
- Docker  
- Jupyter Notebook  

## 🚀 Como executar  

### 1. Clonar o repositório  
git clone https://github.com/felipp2t/eng-dados-projeto.git  
cd eng-dados-projeto  

### 2. Subir o ambiente  
docker compose up  

### 3. Acessar o Jupyter  
Abrir o link com token exibido no terminal no navegador  

## 📊 Funcionalidades  

### Delta Lake  
- Criação de tabela  
- Inserção de dados (INSERT)  
- Atualização de dados (UPDATE)  
- Exclusão de dados (DELETE)  

### Apache Iceberg  
- Criação de tabela  
- Inserção de dados (INSERT)  
- Atualização de dados (UPDATE)  
- Exclusão de dados (DELETE)  

## 💾 Persistência de dados  
Os dados são persistidos utilizando volumes do Docker, garantindo que não sejam perdidos ao reiniciar o container.

## 📁 Estrutura  
- notebooks/delta.ipynb  
- notebooks/iceberg.ipynb  
- data/  
- docker-compose.yml  
- README.md  

## 👨‍💻 Integrantes  
- Lucas  
- Felipe  
