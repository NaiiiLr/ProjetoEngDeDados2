# Utilitário de Reset: Preparação do Ambiente (MinIO)

Este módulo, baseado no notebook `_reset_minio.ipynb`, atua como o estágio inicial (passo zero) para garantir a integridade do ciclo de vida dos dados no projeto. Ele fornece um mecanismo para retornar o ambiente de armazenamento de objetos ao seu estado inicial absoluto.

## Funcionalidades do Script

O utilitário executa uma rotina de limpeza automatizada que abrange as seguintes etapas técnicas:

1. **Conexão e Autenticação**: O script utiliza as bibliotecas `boto3` e `python-dotenv` para estabelecer uma conexão segura com o endpoint do MinIO, carregando as credenciais e configurações de buckets diretamente de um arquivo `.env`.
2. **Mapeamento de Escopo**: A varredura é concentrada nos buckets `landing-zone` (armazenamento bruto) e `bronze` (armazenamento Delta Lake).
3. **Expurgo de Dados**: O sistema identifica e remove todos os arquivos residuais, incluindo formatos CSV, arquivos Parquet e logs de metadados do Delta Lake.
4. **Reinstalação de Buckets**: Para garantir a remoção de qualquer configuração de metadados corrompida, o script destrói os buckets existentes e os recria totalmente vazios.

## Importância Técnica e Idempotência

Em arquiteturas de Data Lake, a execução repetida de rotinas de ingestão sobre o mesmo diretório pode causar problemas críticos:

* **Duplicação de Registros**: Sem a limpeza prévia, novos arquivos podem ser somados aos antigos, gerando duplicidade nas consultas analíticas.
* **Corrupção de Testes**: Arquivos de log residuais de execuções falhas podem interferir na consistência das transações ACID do Delta Lake.
* **Isolamento**: Rodar este utilitário assegura que os resultados observados no pipeline sejam fruto exclusivo da execução atual, facilitando a depuração de erros.

## Pré-requisitos para Execução

Para que o reset funcione conforme o esperado, a infraestrutura deve atender aos seguintes critérios:

* **Orquestração**: O ambiente Docker Compose deve estar ativo (`docker compose up -d`) para disponibilizar o serviço do MinIO.
* **Configuração**: O arquivo `.env` na raiz do projeto deve conter as chaves `MINIO_ENDPOINT`, `MINIO_ACCESS_KEY`, `MINIO_SECRET_KEY` e os nomes dos buckets definidos.

---

## Próximo Passo

Com o ambiente de armazenamento devidamente limpo e os buckets reiniciados, o pipeline pode seguir para a criação da base de dados sintética. Prossiga para a documentação da **[Geração de Dados Customizados (00)](00_generate_custom_data.md)**.