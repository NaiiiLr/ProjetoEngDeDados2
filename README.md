![Apache Spark](https://img.shields.io/badge/Apache%20Spark-F68A1E?style=for-the-badge&logo=apachespark&logoColor=white)
![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white)
![MkDocs](https://img.shields.io/badge/MkDocs-Material-526EE5?style=for-the-badge)
![Delta Lake](https://img.shields.io/badge/Delta%20Lake-00ADD8?style=for-the-badge&logo=delta&logoColor=white)
![MinIO](https://img.shields.io/badge/MinIO-C72E49?style=for-the-badge&logo=minio&logoColor=white)
![SQL Server](https://img.shields.io/badge/SQL%20Server-CC2927?style=for-the-badge&logo=microsoftsqlserver&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)

# Data Platform: E-commerce Architecture

Este projeto implementa um pipeline de Engenharia de Dados completo para uma operação de e-commerce, utilizando a Arquitetura Medalhão. O objetivo é demonstrar o fluxo de dados desde a origem transacional até um ambiente analítico com suporte a transações ACID e governança.

##  Documentação Completa

Toda a arquitetura, modelagem de dados, diagramas ER e a explicação técnica detalhada das operações transacionais foram documentadas e publicadas utilizando o MkDocs.

 **[Acesse a Documentação Pública do Projeto Aqui](https://naiiilr.github.io/ProjetoEngDeDados2/)**

---

## Etapas do Pipeline

O processamento é dividido em seis estágios principais para garantir a integridade e a rastreabilidade dos dados:

1.  **Limpeza de Ambiente (Reset)**: Utiliza a API do MinIO para remover buckets e arquivos residuais, garantindo um estado inicial limpo para os testes.
2.  **Geração de Dados Sintéticos**: Criação programática via Pandas de uma base de e-commerce contendo clientes, produtos, categorias e transações.
3.  **Configuração da Origem (OLTP)**: Carga dos dados gerados em uma instância do SQL Server 2025 para simular o banco de dados de produção da loja.
4.  **Ingestão Landing Zone**: Extração das tabelas do SQL Server e armazenamento no MinIO em formato bruto (CSV).
5.  **Processamento Camada Bronze**: Conversão dos arquivos CSV para o formato Delta Lake utilizando Apache Spark, aplicando tipagem e compressão Parquet.
6.  **Operações DML e Governança**: Demonstração de comandos INSERT, UPDATE e DELETE, além do uso de History e Time Travel nativos do Delta Lake.

## Arquitetura de Dados

A infraestrutura é baseada no desacoplamento total entre o motor de processamento distribuído e o armazenamento de objetos compatível com S3.

## Arquitetura

```
┌─────────────────┐     ┌─────────────────┐     ┌──────────────────┐     ┌───────────────────┐
│ Python (Pandas) │ ──> │   SQL Server    │ ──> │    MinIO (S3)    │ ──> │    MinIO (S3)     │
│                 │     │                 │     │                  │     │                   │
│  Geração Local  │     │   2025 (OLTP)   │     │  landing-zone/   │     │     bronze/       │
│  (Mock Data)    │     │  DB: Ecommerce  │     │  (CSVs brutos)   │     │  (Delta Tables)   │
│  5 arquivos CSV │     │  5 tabelas      │     │  1 CSV / tabela  │     │  ACID & History   │
└─────────────────┘     └─────────────────┘     └──────────────────┘     └───────────────────┘
    Notebook 00             Notebook 01             Notebook 02           Notebooks 03 e 04
   (Geração Dados)         (Setup e Carga)       (Extração Landing)      (Transformação e DML)
```

* Nota: O notebook utilitário `_reset_minio.ipynb` atua de forma paralela como o "passo zero" para garantir a limpeza e idempotência do ambiente no MinIO antes de iniciar o fluxo principal.

## Pré-requisitos

- **Linux** (Ubuntu 24.04 ou WSL do Windows 11)
- **Docker** e **Docker Compose** v2+
- **Python 3.11** (PySpark 3.5.3 requer Python ≤ 3.12)
- **Java 11** (OpenJDK)
- **UV** (gerenciador de pacotes Python) — [instalação](https://github.com/astral-sh/uv)
- **ODBC Driver 18 for SQL Server** — [instalação](https://learn.microsoft.com/sql/connect/odbc/linux-mac/installing-the-microsoft-odbc-driver-for-sql-server)

## Setup do Ambiente

### 1. Subir os Containers (SQL Server + MinIO)

```bash
docker compose up -d
```

**Containers criados:**

| Container       | Imagem                                      | Portas          |
|----------------|---------------------------------------------|-----------------|
| sqlserver-2025 | `mcr.microsoft.com/mssql/server:2025-latest` | `1433`          |
| minio          | `minio/minio:RELEASE.2025-02-03T21-03-04Z`  | `9020`, `9021`  |

**Credenciais:**

| Serviço     | Usuário       | Senha              |
|-------------|---------------|---------------------|
| SQL Server  | `sa`          | `SqlServer@2025!`  |
| MinIO       | `minioadmin`  | `minioadmin`       |

**Console MinIO:** http://localhost:9021

### 2. Configurar Variáveis de Ambiente

Crie o arquivo de configuração a partir do template fornecido:

```bash
cp .env.example .env
```

### 3. Configurar o Ambiente Python

```bash
uv venv
source .venv/bin/activate
uv sync
```

### 4. Instalar ODBC Driver - Client SQL Server (Ubuntu)

```bash
# Ubuntu 24.04
sudo apt install -y unixodbc-dev
curl https://packages.microsoft.com/keys/microsoft.asc | sudo tee /etc/apt/trusted.gpg.d/microsoft.asc
sudo curl https://packages.microsoft.com/config/ubuntu/24.04/prod.list | sudo tee /etc/apt/sources.list.d/mssql-release.list
sudo apt update
sudo ACCEPT_EULA=Y apt install -y msodbcsql18
```
### 4.1. Validar instalação do ODBC Driver - Client SQL Server (Ubuntu)

```bash
# Ubuntu 24.04
odbcinst -q -d
```

Deve retornar `[ODBC Driver 18 for SQL Server]`.

### 5. Executar os Laboratórios (Jupyter Lab)

Com o ambiente ativado e as dependências instaladas, inicie a instância do Jupyter Lab para acessar e executar os testes interativos de comparação:

```bash
uv run jupyter lab
```

Após rodar o comando, o terminal exibirá uma URL (`ex: http://localhost:8888/lab?token=...`). Lá, você poderá abrir e executar as células dos arquivos que estão em `/notebook/...`.

### 6. Executar a Documentação Localmente (MkDocs)

Toda a documentação técnica foi construída utilizando MkDocs. Se desejar rodar o site de documentação localmente na sua máquina, utilize o comando:

```bash
uv run mkdocs serve
```

## Executando o Projeto

Execute os notebooks **em ordem**:

| # | Notebook | Descrição |
| :--- | :--- | :--- |
| 0 | `_reset_minio.ipynb` | Executa o reset completo do ambiente de armazenamento, removendo buckets e arquivos residuais no MinIO. |
| 1 | `00_generate_custom_data.ipynb` | Realiza a limpeza do diretório local e gera os arquivos CSV com a massa de dados sintéticos para o e-commerce. |
| 2 | `01_setup_sqlserver.ipynb` | Configura o database `Ecommerce` e realiza a carga inicial das tabelas no SQL Server. |
| 3 | `02_sqlserver_to_minio_csv.ipynb` | Extrai os dados das tabelas do SQL Server e realiza o upload para a Landing Zone no MinIO em formato CSV. |
| 4 | `03_csv_to_delta.ipynb` | Lê os arquivos da Landing Zone e realiza a conversão para o formato Delta Lake na camada Bronze. |
| 5 | `04_dml_delta.ipynb` | Demonstra a execução de transações ACID (INSERT, UPDATE, DELETE) e recursos de History e Time Travel. |

> **Importante:** Selecione o ambiente virtual (`.venv`) como Kernel do Jupyter antes de executar.

## Estrutura do Projeto

```text
spark-delta-minio-sqlserver/
├── data/                                # Dados gerados via script (Mock Data)
│   ├── categorias.csv
│   ├── clientes.csv
│   ├── itens_venda.csv
│   ├── produtos.csv
│   └── vendas.csv
├── docs/                                # Documentação técnica via MkDocs
│   ├── assets/
│   │   └── modeloER.png
│   ├── pipeline/
│   │   ├── _reset_minio.md
│   │   ├── 00_generate_custom_data.md
│   │   ├── 01_setup_sqlserver.md
│   │   ├── 02_sqlserver_to_minio_csv.md
│   │   ├── 03_csv_to_delta.md
│   │   └── 04_dml_delta.md
│   ├── arquitetura.md
│   ├── dicionario.md
│   └── index.md
├── notebook/                            # Notebooks Jupyter do pipeline
│   ├── _reset_minio.ipynb               # Reset e idempotência do MinIO
│   ├── 00_generate_custom_data.ipynb    # Setup e geração de massa de dados
│   ├── 01_setup_sqlserver.ipynb         # Carga inicial no SQL Server
│   ├── 02_sqlserver_to_minio_csv.ipynb  # Extração SQL → Landing Zone (MinIO)
│   ├── 03_csv_to_delta.ipynb            # Transformação CSV → Delta Lake (Bronze)
│   └── 04_dml_delta.ipynb               # Transações ACID, DML e Time Travel
├── .env.example                         # Template de variáveis de ambiente
├── .gitignore                           # Arquivos e pastas ignorados no Git
├── .python-version                      # Versão do Python (3.11)
├── docker-compose.yml                   # Orquestração: SQL Server 2025 + MinIO
├── mkdocs.yml                           # Configuração do site da documentação
├── pyproject.toml                       # Gerenciamento de dependências (UV)
├── README.md                            # Apresentação do projeto
└── uv.lock                              # Lockfile das dependências
```

## Tecnologias Utilizadas

- **Apache Spark 3.5.3** (PySpark) — Motor de processamento distribuído
- **Delta Lake 3.2.0** — Formato de armazenamento com suporte ACID
- **MinIO** — Object Storage compatível com S3
- **SQL Server 2025** — Banco de dados relacional (Developer Edition)
- **Docker Compose** — Orquestração de containers
- **Python 3.11** com UV

## Conceitos Demonstrados

- **Extração de dados** de banco relacional (SQL Server)
- **Object Storage** como repositório de dados (MinIO/S3)
- **Delta Lake** como formato de armazenamento lakehouse
- **Transações ACID** em data lakes
- **DML** (INSERT, UPDATE, DELETE) em tabelas Delta
- **Versionamento** de dados (History e Time Travel)
- **Arquitetura Medalhão** (Landing Zone → Bronze)

## Links e Referências

- [Apache Spark (PySpark) - Documentação](https://spark.apache.org/docs/latest/api/python/)
- [Delta Lake - Guia Oficial](https://docs.delta.io/latest/index.html)
- [MinIO - Object Storage](https://min.io/docs/minio/linux/index.html)
- [SQL Server 2025 no Docker](https://learn.microsoft.com/sql/linux/quickstart-install-connect-docker)
- [MkDocs Material - Documentação do Tema](https://squidfunk.github.io/mkdocs-material/)
- [Pandas - Geração e Manipulação de Dados](https://pandas.pydata.org/docs/)
- [UV - Gerenciador de Pacotes Python](https://docs.astral.sh/uv/)