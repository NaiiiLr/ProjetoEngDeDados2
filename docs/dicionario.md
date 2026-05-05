# Dicionário de Dados

Este dicionário descreve as entidades do ecossistema de e-commerce processadas no Data Lake. As tabelas seguem o esquema definido na origem (SQL Server) e são replicadas na camada **Bronze** em formato Delta Lake para garantir transações ACID.

## Estrutura das Tabelas

### 1. Tabela: `categorias`
Armazena o agrupamento lógico dos produtos disponíveis no catálogo.

| Campo | Tipo | Descrição |
| :--- | :--- | :--- |
| **id_categoria** | Integer | Chave primária e identificador único da categoria. |
| **nome_categoria** | String | Nome descritivo da categoria (ex: Games, Eletrônicos). |
| **descricao** | String | Detalhamento sobre os tipos de produtos contidos na categoria. |

### 2. Tabela: `produtos`
Contém os itens comercializados, preços e níveis de inventário. É o foco principal das demonstrações de atualização de preços e estoque via DeltaTable API.

| Campo | Tipo | Descrição |
| :--- | :--- | :--- |
| **id_produto** | Integer | Chave primária e identificador único do produto. |
| **nome** | String | Nome comercial do produto. |
| **id_categoria** | Integer | Chave estrangeira que vincula o produto a uma categoria. |
| **preco** | Decimal | Valor de venda unitário do produto. |
| **estoque** | Integer | Quantidade física disponível em armazém. |

### 3. Tabela: `clientes`
Registra as informações dos usuários cadastrados na plataforma.

| Campo | Tipo | Descrição |
| :--- | :--- | :--- |
| **id_cliente** | Integer | Chave primária e identificador único do cliente. |
| **nome** | String | Nome completo do cliente. |
| **estado** | String | Sigla da Unidade Federativa (UF) de residência. |
| **status_conta** | String | Situação atual do cadastro (ex: Ativo, Inativo). |

### 4. Tabela: `vendas`
Tabela de fatos que registra o cabeçalho de cada pedido realizado.

| Campo | Tipo | Descrição |
| :--- | :--- | :--- |
| **id_venda** | Integer | Chave primária e identificador único do pedido. |
| **id_cliente** | Integer | Chave estrangeira que identifica o comprador. |
| **data_venda** | Timestamp | Data e hora em que a transação foi realizada. |
| **valor_total** | Decimal | Soma total do pedido após descontos e frete. |

### 5. Tabela: `itens_venda`
Detalha os produtos contidos em cada venda (relacionamento N:N entre Vendas e Produtos).

| Campo | Tipo | Descrição |
| :--- | :--- | :--- |
| **id_item** | Integer | Chave primária do registro do item no pedido. |
| **id_venda** | Integer | Chave estrangeira vinculando ao cabeçalho da venda. |
| **id_produto** | Integer | Chave estrangeira identificando o produto vendido. |
| **quantidade** | Integer | Quantidade de unidades vendidas deste item. |
| **preco_unitario** | Decimal | Preço do produto no momento exato da venda. |

---

## Notas de Implementação

*   **Persistência**: Os dados são armazenados no formato Delta no bucket `bronze` do MinIO, utilizando o protocolo S3A.
*   **Versionamento**: Graças ao Delta Lake, todas as tabelas possuem histórico de versões, permitindo consultas de *Time Travel* para auditar mudanças nos dados.
*   **Integridade**: A relação entre as tabelas é validada durante a fase de transformação do pipeline, garantindo a consistência referencial entre as camadas.

---

## Próximos Passos

Para compreender como estas tabelas são extraídas da origem transacional e processadas através das camadas do Data Lake, consulte a página detalhada da **[Arquitetura do Projeto](arquitetura.md)**.