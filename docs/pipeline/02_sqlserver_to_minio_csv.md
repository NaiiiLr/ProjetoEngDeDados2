# 02. Ingestão Landing Zone: SQL Server para MinIO (CSV)

Este módulo, baseado no notebook `02_sqlserver_to_minio_csv.ipynb`, detalha o processo de extração de dados do sistema de origem e sua carga inicial no Data Lake. Esta etapa representa a transição dos dados de um ambiente relacional (OLTP) para um ambiente de armazenamento de objetos.

## 1. Objetivo do Módulo

A finalidade desta etapa é realizar a extração completa das tabelas do banco de dados `Ecommerce` no SQL Server e transportá-las para a **Landing Zone** no MinIO. Os dados são mantidos em seu formato bruto (CSV) para garantir que exista uma cópia fiel da origem no Data Lake antes de qualquer processamento transformador.

## 2. Processo de Extração e Carga

O pipeline de extração segue um fluxo técnico estruturado para garantir a integridade do transporte:

* **Identificação de Tabelas**: O script realiza uma consulta ao `INFORMATION_SCHEMA.TABLES` do SQL Server para listar dinamicamente todas as tabelas de base presentes no esquema `dbo`.
* **Gestão do Bucket**: Utilizando a biblioteca `boto3`, o sistema verifica a existência do bucket `landing-zone`. Caso o bucket não esteja presente no MinIO, ele é criado automaticamente para receber os arquivos.
* **Extração em Memória**: Para cada tabela identificada, o sistema executa uma leitura via SQL e carrega os registros em um DataFrame Pandas.
* **Conversão Direta**: Os dados são convertidos para o formato CSV e transformados em um fluxo de bytes (bytes stream) em memória. Isso evita a necessidade de gravar arquivos temporários no disco local do servidor de processamento, otimizando a performance e a segurança.
* **Upload para Object Storage**: O fluxo de bytes é enviado diretamente para o MinIO via protocolo S3A, sendo armazenado com o nome da tabela original (ex: `clientes.csv`).

## 3. Benefícios da Landing Zone

A implementação desta camada intermediária traz vantagens estratégicas para a arquitetura de dados:

* **Minimização de Impacto**: Ao extrair todos os dados de uma única vez e armazená-los na Landing Zone, reduzimos a carga de leitura no banco de dados de produção (SQL Server) durante as fases posteriores de transformação.
* **Auditabilidade**: Como os arquivos CSV são cópias exatas da origem, eles servem como um ponto de auditoria para validar se os dados processados nas camadas subsequentes (Bronze, Prata, Ouro) permanecem consistentes.
* **Recuperação**: Em caso de erros nas etapas de processamento do Spark, é possível reiniciar o pipeline a partir da Landing Zone sem consultar novamente o banco de dados transacional.

## 4. Validação do Processo

Ao concluir o transporte, o módulo executa uma rotina de conferência no MinIO para listar os objetos criados e validar o tamanho dos arquivos. Esta validação garante que todas as tabelas mapeadas na origem foram devidamente replicadas para o armazenamento de objetos.

---

## Próximo Passo

Com os dados brutos devidamente armazenados na Landing Zone, o próximo estágio consiste em processar esses arquivos utilizando o Apache Spark e convertê-los para o formato Delta Lake. Prossiga para a documentação do **[Processamento Bronze: CSV para Delta (03)](03_csv_to_delta.md)**.