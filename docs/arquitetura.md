# Arquitetura do Sistema

A arquitetura deste projeto foi desenhada para simular um ambiente corporativo moderno de Engenharia de Dados. O projeto utiliza o padrão estrutural conhecido como **Medallion Architecture** (Arquitetura Medalhão) e implementa o conceito de um Data Lake Cloud-Native operando localmente através de containers.

## 1. Conceitos Fundamentais

### Desacoplamento de Computação e Armazenamento
Ao contrário de bancos de dados relacionais tradicionais, onde o armazenamento físico em disco e o motor de consultas compartilham o mesmo servidor, este pipeline isola essas responsabilidades:

* **Armazenamento (Storage)**: Gerenciado exclusivamente pelo MinIO, que atua como um repositório centralizado de objetos.
* **Computação (Compute)**: Executada pelos clusters do Apache Spark através do Jupyter.

Este paradigma garante maior escalabilidade técnica e financeira, permitindo que o poder de processamento e o espaço de armazenamento sejam dimensionados de forma independente.

## 2. Componentes da Infraestrutura

Todo o ambiente de desenvolvimento é orquestrado utilizando Docker e Docker Compose, garantindo isolamento de dependências e reprodutibilidade da infraestrutura. Os serviços centrais são:

* **Jupyter Notebook (PySpark)**: Interface de desenvolvimento onde os pipelines de ingestão e transformação de dados são codificados.
* **Microsoft SQL Server**: Atua como o banco de dados transacional primário (OLTP). Ele simula o sistema backend da aplicação de E-commerce.
* **MinIO**: Atua como o Data Lake propriamente dito. Ele responde a requisições via API S3 e armazena os arquivos de dados nos respectivos estágios de processamento.

## 3. Fluxo de Dados e Padrão Medalhão

O pipeline obedece a um fluxo rigoroso de ingestão, garantindo auditoria e qualidade dos dados em cada estágio.

### A. Camada de Origem (Source / OLTP)
Os dados gerados pelo processo de negócio residem no SQL Server. Esta camada possui modelagem relacional rigorosa e foco em operações de alta concorrência (leitura e escrita rápidas).

### B. Landing Zone (Área de Pouso / Raw Data)
* **Destino**: MinIO (Bucket `landing-zone`).
* **Formato**: CSV.
* **Processo**: Os dados são extraídos do banco transacional e salvos no formato bruto. Não há aplicação de regras de negócio ou transformações nesta etapa. 
* **Objetivo**: Manter um registro histórico inalterado dos dados da origem. Isso permite que o pipeline seja reprocessado em caso de falhas sem a necessidade de sobrecarregar o banco de dados original com novas consultas.

### C. Camada Bronze (Ingestion / Delta)
* **Destino**: MinIO (Bucket `bronze`).
* **Formato**: Parquet gerenciado pelo protocolo Delta Lake.
* **Processo**: O Apache Spark realiza a leitura dos arquivos da Landing Zone, estrutura os dados em DataFrames e realiza a gravação no bucket Bronze no formato Delta.
* **Objetivo**: Trazer confiabilidade aos arquivos do Data Lake. A partir deste ponto, o Delta Lake passa a gerenciar os metadados, garantindo transações ACID (Atomicidade, Consistência, Isolamento, Durabilidade) e habilitando recursos avançados como *Time Travel* (consulta de histórico) e evolução de esquema.

## 4. Integração Spark e MinIO (Protocolo S3A)
Para permitir que o Apache Spark interaja com o MinIO de forma nativa, o projeto utiliza o protocolo `s3a://` fornecido pelas bibliotecas do Hadoop-AWS. Isso garante que o código desenvolvido seja "Cloud Ready". Caso a infraestrutura seja migrada para a nuvem da AWS, os caminhos de leitura e escrita do Spark permanecerão praticamente idênticos.