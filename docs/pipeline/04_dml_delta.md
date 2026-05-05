# 04. Transações ACID e DML no Delta Lake

Este módulo final, baseado no notebook `04_dml_delta.ipynb`, demonstra o poder do Delta Lake em transformar um Data Lake em uma base de dados funcional, permitindo operações de manipulação de dados (DML) com as mesmas garantias de consistência de um banco de dados relacional tradicional (ACID).

## 1. Registro e Integração Spark SQL

Para facilitar a manipulação, as tabelas Delta armazenadas no bucket `bronze` do MinIO são registradas no catálogo do Spark como tabelas SQL. Isso permite que o engenheiro de dados utilize tanto a sintaxe SQL padrão quanto a API de Tabelas Delta do Python para interagir com os dados.

*   **Persistência de Localização**: Ao registrar a tabela com a cláusula `LOCATION`, o Spark vincula os metadados do catálogo diretamente aos arquivos físicos no MinIO.
*   **Interoperabilidade**: Uma vez registradas, as tabelas podem ser consultadas através de comandos `SELECT` convencionais, facilitando a integração com ferramentas de BI e análise.

## 2. Operações de Manipulação (DML)

O módulo exemplifica o ciclo completo de modificação de dados em um ambiente de e-commerce:

*   **INSERT**: Demonstra a adição de novos registros de categorias (ex: Games), produtos (ex: PlayStation 5) e clientes, provando que o Data Lake pode crescer de forma incremental.
*   **UPDATE**: Ilustra a atualização de informações existentes, como a correção de descrições de categorias ou o ajuste de preços de produtos. O Delta Lake gerencia a substituição dos arquivos Parquet de forma atômica para refletir essas mudanças.
*   **DELETE**: Apresenta a remoção de registros específicos. Diferente de um sistema de arquivos comum onde a deleção é manual e arriscada, no Delta Lake a remoção é transacional e reversível.

## 3. Governança e Rastreabilidade

O grande diferencial técnico explorado neste módulo é a capacidade de auditoria nativa do Delta Lake:

*   **History**: Através do comando `DESCRIBE HISTORY`, é possível visualizar todas as operações realizadas na tabela, identificando quem realizou a alteração, quando e qual foi o comando executado.
*   **Time Travel (Viagem no Tempo)**: Permite que o Spark carregue uma versão anterior da tabela (Snapshot Isolation). Isso é fundamental para auditorias, rollback de erros ou para comparar o estado atual dos dados com o estado original logo após a ingestão inicial.
*   **Isolamento de Snapshot**: Durante a execução de um UPDATE ou DELETE, os usuários que estão apenas lendo os dados não são afetados e não encontram estados inconsistentes, graças ao controle de concorrência otimista.

## 4. Conclusão do Pipeline

Com a execução deste módulo, o pipeline completa o fluxo total de dados:

1.  **Origem**: Dados gerados e armazenados no SQL Server.
2.  **Ingestão**: Transferência para a Landing Zone (CSV) no MinIO.
3.  **Qualificação**: Conversão para a camada Bronze em formato Delta Lake.
4.  **Manipulação**: Aplicação de regras de negócio e manutenção via transações ACID.

Esta arquitetura estabelece uma base sólida e escalável para a evolução do projeto rumo às camadas analíticas superiores (Prata e Ouro).