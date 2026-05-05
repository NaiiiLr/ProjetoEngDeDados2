# 03. Processamento Bronze: CSV para Delta

Este módulo, baseado no notebook `03_csv_to_delta.ipynb`, detalha a etapa de transformação e qualificação dos dados dentro do Data Lake. Aqui, os arquivos brutos (CSV) da Landing Zone são convertidos para o formato Delta Lake, elevando o nível de maturidade dos dados para a camada **Bronze**.

## 1. Configuração da SparkSession

Para possibilitar o processamento escalável e a integração com o armazenamento de objetos, a SparkSession é configurada com dependências específicas:

*   **Delta Lake**: Inclusão dos pacotes `io.delta:delta-spark` e configuração das extensões de catálogo do Spark para habilitar o suporte nativo ao formato Delta.
*   **Hadoop AWS**: Utilização do pacote `hadoop-aws` para permitir a comunicação via protocolo **S3A**.
*   **Integração MinIO**: Configuração de endpoints, chaves de acesso e desabilitação de SSL para permitir que o cluster Spark trate o MinIO como um sistema de arquivos distribuído compatível com S3.

## 2. Lógica de Processamento e Conversão

O pipeline de transformação executa um fluxo automatizado para cada arquivo identificado na Landing Zone:

1.  **Leitura com Inferência**: O Spark realiza a leitura dos arquivos CSV aplicando a inferência de esquema (`inferSchema: true`). Isso garante que tipos de dados como inteiros, decimais e datas sejam identificados corretamente desde a primeira camada.
2.  **Escrita em Formato Delta**: Os dados são persistidos no bucket `bronze` utilizando o formato Delta. O modo de escrita configurado é o `overwrite`, garantindo que a carga possa ser reiniciada sem duplicar metadados corrompidos.
3.  **Estrutura de Pastas**: Cada tabela (categorias, produtos, vendas, etc.) ganha seu próprio diretório no bucket Bronze, contendo os arquivos Parquet de dados e a pasta `_delta_log` para controle de transações.

## 3. Benefícios da Camada Bronze (Delta Lake)

A conversão para Delta Lake transforma arquivos estáticos em tabelas inteligentes, trazendo recursos fundamentais para a engenharia de dados:

*   **Transações ACID**: Garante que as operações de escrita sejam atômicas. Se uma carga falhar no meio do processo, o estado da tabela não é corrompido.
*   **Time Travel (Viagem no Tempo)**: O Delta Lake mantém um histórico de versões, permitindo consultar como os dados estavam em um ponto específico do passado.
*   **Desempenho**: O uso do formato Parquet por baixo do Delta oferece alta compressão e leitura otimizada por colunas, ideal para análise de grandes volumes.

## 4. Validação e Consistência

Ao final da conversão, o módulo utiliza a API `DeltaTable` para validar se os destinos são, de fato, tabelas Delta válidas. O script também realiza a leitura de amostras dos dados diretamente da camada Bronze para confirmar que a estrutura e o conteúdo foram preservados com sucesso.

---

## Próximo Passo

Com as tabelas Delta devidamente criadas na camada Bronze, o pipeline está pronto para demonstrar manipulações de dados avançadas. Prossiga para a documentação de **[Transações ACID e DML (04)](04_dml_delta.md)**.