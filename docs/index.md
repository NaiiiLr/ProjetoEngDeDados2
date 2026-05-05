# Data Platform: E-commerce Architecture

Bem-vindo à documentação oficial do projeto de arquitetura e pipeline de dados voltado para o setor de e-commerce.

Este projeto demonstra a construção de um ambiente de dados robusto, simulando o fluxo de dados desde a origem transacional até o processamento analítico com transações ACID no Data Lake.

## Objetivo do Projeto
O objetivo central é implementar um pipeline de Engenharia de Dados completo que garanta a integridade, escalabilidade e rastreabilidade das informações. O projeto foca em:

* **Ingestão Idempotente**: Processos que podem ser repetidos sem duplicar dados.
* **Consistência**: Uso de Delta Lake para garantir transações ACID.
* **Evolução de Esquema**: Capacidade de lidar com mudanças nos dados ao longo do tempo.

## Arquitetura de Dados
A solução utiliza a abordagem de **Arquitetura Medalhão**, movendo os dados através de diferentes estágios de maturidade:

1. **Origem (OLTP)**: Banco de dados relacional SQL Server.
2. **Landing Zone**: Armazenamento de arquivos brutos (CSV) no MinIO.
3. **Camada Bronze (Delta)**: Dados convertidos para formato Delta Lake, mantendo o histórico de transações.

## O Papel do MinIO no Data Lake
O **MinIO** é um servidor de armazenamento de objetos de alto desempenho, focado em nuvem privada. No contexto deste projeto, ele desempenha o papel de **Object Storage**, substituindo serviços como o Amazon S3 em ambiente local.

### Por que MinIO?
* **Compatibilidade com S3**: Ele implementa a API do Amazon S3, permitindo que o Spark utilize o protocolo `s3a://` para ler e escrever dados como se estivesse na nuvem.
* **Persistência Desacoplada**: Permite separar o processamento (Spark) do armazenamento, garantindo que os dados persistam mesmo que os containers de processamento sejam encerrados.
* **Escalabilidade**: Projetado para lidar com petabytes de dados e alta taxa de transferência, ideal para cargas de trabalho de Big Data.
* **Interface Visual**: Oferece um console administrativo para visualização e gerenciamento dos buckets (`landing-zone` e `bronze`), facilitando a auditoria manual dos arquivos.

## Stack Tecnológico
Para a construção desta infraestrutura, foram utilizadas as seguintes ferramentas:

* **Armazenamento de Objetos**: [MinIO](https://min.io/) (Alvo principal do Data Lake).
* **Engine de Processamento**: [Apache Spark](https://spark.apache.org/) (PySpark).
* **Formato de Tabela**: [Delta Lake](https://delta.io/) (Camada de confiabilidade).
* **Banco de Dados de Origem**: [Microsoft SQL Server](https://www.microsoft.com/sql-server).
* **Orquestração de Ambiente**: [Docker](https://www.docker.com/) e Docker Compose.
* **Linguagem Principal**: [Python](https://www.python.org/).

## Fluxo de Execução
Para garantir o funcionamento correto do pipeline, os cadernos (notebooks) devem ser executados na ordem definida no menu lateral:

1. **_reset_minio**: Limpeza dos buckets no MinIO para garantir um ambiente limpo.
2. **00_generate_custom_data**: Criação dos dados sintéticos para o teste.
3. **01_setup_sqlserver**: Configuração e carga inicial da origem.
4. **02_sqlserver_to_minio_csv**: Extração e ingestão na Landing Zone do MinIO.
5. **03_csv_to_delta**: Processamento e conversão para tabelas Delta na camada Bronze.
6. **04_dml_delta**: Demonstração de operações ACID (Insert, Update, Delete).

---
Navegue pelo menu lateral para explorar os detalhes técnicos de cada etapa do processo ou consulte o **[Dicionário do Projeto](dicionario.md)** para entender a modelagem das tabelas.