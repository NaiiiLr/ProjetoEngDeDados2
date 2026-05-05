# 00. Geração de Dados Customizados

Este módulo, baseado no notebook `00_generate_custom_data.ipynb`, é responsável pela preparação do diretório local e pela criação da massa de dados inicial (Mock Data) que alimentará o banco de dados transacional do projeto.

## 1. Limpeza e Setup do Diretório

Antes da geração dos dados, o script realiza a gestão do sistema de arquivos local para garantir que não existam resquícios de execuções anteriores.

*   **Identificação da Raiz**: O script identifica automaticamente o diretório raiz do projeto, independentemente de onde o ambiente Jupyter foi iniciado.
*   **Gestão da Pasta de Dados**: O diretório `data/` é mapeado de forma absoluta; caso ele já exista, o conteúdo é removido recursivamente e a pasta é recriada do zero.
*   **Objetivo**: Esta etapa garante que a pasta esteja limpa e preparada para receber apenas os novos arquivos CSV que serão gerados na etapa seguinte.

## 2. Geração de Dados Transacionais (E-commerce)

A criação do conjunto de dados utiliza a biblioteca **Pandas** para simular o backend de uma loja online. O foco é construir um modelo de dados relacional que respeite as dependências de negócio.

### Conjuntos de Dados Gerados

O modelo é composto por cinco entidades fundamentais:

*   **Categorias**: Classificação lógica dos produtos (ex: Eletrônicos, Vestuário, Livros).
*   **Produtos**: Itens do catálogo com seus respectivos preços, estoques e vínculos com categorias.
*   **Clientes**: Cadastro de usuários contendo estado de residência (UF) e status da conta.
*   **Vendas**: Registro do cabeçalho das transações, incluindo data da compra e valor total.
*   **Itens de Venda**: Detalhamento granular de cada transação, relacionando produtos e quantidades vendidas.

## 3. Exportação e Formatação

Após a definição dos DataFrames em memória, os dados são persistidos no disco seguindo padrões específicos para facilitar a ingestão posterior:

*   **Formato**: Arquivos `.csv` com codificação `utf-8-sig`.
*   **Limpeza de Índices**: A exportação utiliza o parâmetro `index=False`, garantindo que a coluna de controle numérico do Pandas não seja incluída no arquivo final, mantendo a estrutura idêntica à de uma tabela de banco de dados.
*   **Localização**: Todos os arquivos são salvos dentro da pasta `data/` criada na etapa de setup.

---

## Próximo Passo

Com os arquivos CSV gerados localmente, o próximo estágio consiste em configurar a instância do banco de dados e realizar a carga inicial dessas informações. Prossiga para a documentação do **[Setup Origem: SQL Server (01)](01_setup_sqlserver.md)**.