# Apache Spark com Delta Lake e Iceberg

Projeto desenvolvido para a disciplina de Engenharia de Dados, com o objetivo de implementar e explorar os formatos de tabela abertos **Delta Lake** e **Apache Iceberg** dentro de um ambiente **PySpark** local.

---

## Cenário

O projeto utiliza uma tabela de **carros** como fonte de dados para demonstrar as operações suportadas pelos formatos Delta e Iceberg. A tabela armazena informações como placa, marca, modelo e ano dos veículos.

**Modelo da tabela `carro_delta`:**

| Campo   | Tipo   | Descrição               |
|---------|--------|-------------------------|
| id      | INT    | Identificador único     |
| placa   | STRING | Placa do veículo        |
| marca   | STRING | Fabricante              |
| modelo  | STRING | Modelo do veículo       |
| ano     | INT    | Ano de fabricação       |

---

## Configuração do Ambiente

### 1. Instalar o pyenv e configurar o Python

O **pyenv** permite gerenciar múltiplas versões do Python no mesmo sistema.

```bash
# Instalar dependências (Ubuntu/Debian/WSL)
sudo apt update && sudo apt install -y \
  build-essential libssl-dev zlib1g-dev \
  libbz2-dev libreadline-dev libsqlite3-dev \
  curl git libncursesw5-dev xz-utils tk-dev \
  libxml2-dev libxmlsec1-dev libffi-dev liblzma-dev

# Instalar o pyenv
curl https://pyenv.run | bash

# Adicionar ao shell (bash)
echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bashrc
echo 'command -v pyenv >/dev/null || export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bashrc
echo 'eval "$(pyenv init -)"' >> ~/.bashrc
source ~/.bashrc

# Instalar e definir a versão do Python
pyenv install 3.13.1
pyenv local 3.13.1
```

### 2. Instalar o Java (requisito do PySpark)

O PySpark depende da JVM para funcionar. Sem Java instalado, o erro `[JAVA_GATEWAY_EXITED]` é gerado.

```bash
sudo apt install -y openjdk-17-jdk
java -version
```

### 3. Instalar o uv e iniciar o projeto

O **uv** é um gerenciador de pacotes e projetos Python ultrarrápido (substituto moderno do pip + venv).

```bash
# Instalar o uv
curl -Ls https://astral.sh/uv/install.sh | sh

# Iniciar o projeto
uv init projeto
cd projeto

# Definir a versão do Python para o projeto
uv python pin 3.13
```

### 4. Instalar as bibliotecas necessárias

```bash
uv add pyspark==3.5.3 delta-spark==3.2.0 jupyterlab ipykernel
```

O arquivo `pyproject.toml` gerado ficará assim:

```toml
[project]
name = "projeto"
version = "0.1.0"
requires-python = ">=3.11"
dependencies = [
    "delta-spark==3.2.0",
    "ipykernel>=7.2.0",
    "jupyterlab>=4.5.6",
    "pyspark==3.5.3",
]
```

### 5. Configurar o kernel do Jupyter

Para que o VS Code ou JupyterLab reconheça o ambiente virtual do projeto:

```bash
uv run python -m ipykernel install --user --name=projeto --display-name "Python (projeto)"
```

### 6. Iniciar o JupyterLab

```bash
uv run jupyter lab
```

Ou abrir diretamente o arquivo `.ipynb` no VS Code e selecionar o kernel **Python (projeto)**.

---

## Estrutura do Projeto

```
projeto/
├── docs/               # Documentação MkDocs
│   ├── index.md
│   ├── spark.md
│   ├── delta.md
│   └── iceberg.md
├── notebooks/
│   ├── spark-delta.ipynb
│   └── spark-iceberg.ipynb
├── .python-version     # Versão do Python (pyenv)
├── mkdocs.yml
├── pyproject.toml
└── uv.lock
```
