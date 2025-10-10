# Data Lakehouse com PySpark, Delta Lake e Dados de Acidentes de Trânsito do Recife

### Resumo

Este projeto demonstra a construção de um pipeline de ETL (Extract, Transform, Load) de ponta-a-ponta utilizando PySpark e Delta Lake. O objetivo é ingerir múltiplos arquivos CSV anuais de acidentes de trânsito da cidade do Recife, lidar com inconsistências de schema (*schema drift*), aplicar um processo robusto de limpeza e enriquecimento e, finalmente, criar múltiplos **Data Marts** com modelo dimensional (Star Schema) prontos para consumo por ferramentas de Business Intelligence (BI).

A arquitetura implementada segue o padrão **Medallion (Bronze, Silver, Gold)**, uma abordagem moderna para a organização de um Data Lakehouse que garante qualidade, governança e performance.

-----

### Objetivo do Projeto

O principal objetivo é simular um cenário real de engenharia de dados, demonstrando competências em:

* Ingestão e processamento de dados em lote (*batch*) a partir de múltiplas fontes com schemas inconsistentes.
* Implementação de um pipeline de dados resiliente e documentado.
* Limpeza, validação de regras de negócio e enriquecimento de dados (*feature engineering*).
* Organização de um Data Lakehouse com a arquitetadura Medalhão.
* Modelagem de dados para BI, incluindo a criação de um **Star Schema** e Data Marts com granularidades diferentes.
* Demonstração de recursos avançados de governança de dados do Delta Lake (ACID, Time Travel, Auditoria).

-----

### Arquitetura Utilizada: Medallion

O pipeline é estruturado em três camadas lógicas, cada uma representada por tabelas Delta particionadas:

* **Camada Bronze (Bruta):**
    * **Função:** Ingestão dos dados crus de múltiplos arquivos CSV, servindo como uma cópia fiel e histórica da fonte. Um schema unificado é aplicado para consolidar os diferentes layouts dos arquivos.
    * **Tabela:** `bronze/acidentes`

* **Camada Silver (Limpa e Enriquecida):**
    * **Função:** Contém os dados da camada Bronze após passarem por um processo rigoroso de limpeza (padronização de texto, tratamento de nulos, correção de tipos), validação (remoção de duplicatas, filtros de regras de negócio) e enriquecimento (criação de colunas de data e hora). É a nossa "fonte única da verdade".
    * **Tabela:** `silver/acidentes_limpos`

* **Camada Gold (Agregada e Modelada):**
    * **Função:** Apresenta os dados da camada Silver em modelos otimizados para consultas analíticas. Contém múltiplos Data Marts para diferentes tipos de análise.
    * **Tabelas:**
        * **Data Mart 1 (Star Schema):** `fact_acidentes`, `dim_tempo`, `dim_localizacao`, `dim_tipo_acidente`.
        * **Data Mart 2 (Veículos):** `fact_acidente_veiculo`, `dim_vehicle`.

-----

### 🔨 Ferramentas e Tecnologias

* **Linguagem:** Python
* **Processamento de Dados:** Apache Spark 3.5.1
* **Camada de Armazenamento:** Delta Lake 3.2.0
* **Ambiente de Desenvolvimento:** Google Colab / Jupyter Notebook
* **Bibliotecas Python:** `pyspark`, `delta-spark`

-----

### 📂 Estrutura dos Artefatos

```
.
├── delta_lake/                   # Raiz do nosso Data Lakehouse, gerado pela execução
│   ├── bronze/
│   │   └── acidentes/
│   ├── silver/
│   │   └── acidentes_limpos/
│   └── gold/
│       ├── dim_localizacao/
│       ├── dim_tempo/
│       ├── dim_tipo_acidente/
│       ├── dim_vehicle/
│       ├── fact_acidentes/
│       └── fact_acidente_veiculo/
├── data/                         # Onde os arquivos CSV de origem são armazenados
│   ├── acidentes_2019.csv
│   └── ...
├── etl_acidentes_recife.ipynb    # Notebook principal com todo o código do pipeline
└── README.md                     # Este arquivo
```

-----

### 🚀 Como Executar o Projeto

1.  **Abra o Notebook:** Abra o arquivo `etl_acidentes_recife.ipynb` no Google Colab.
2.  **Execute a Célula de Download:** Execute a primeira célula de código na seção `1.1` para baixar os arquivos de dados para a pasta `data/`.
3.  **Execute Todas as Células:** Execute as células restantes na ordem apresentada. As bibliotecas e configurações necessárias são instaladas no início do notebook.

-----

### 📈 Etapas do Pipeline Implementadas

1.  **Configuração do Ambiente:** Instalação das dependências e inicialização da `SparkSession` com o catálogo Delta Lake habilitado.

2.  **Camada Bronze - Ingestão:**
    * Análise de *schema drift* entre os arquivos anuais.
    * Definição de um `StructType` unificado.
    * Implementação de um loop de ingestão robusto com o padrão **Read, Rename, UnionByName**.
    * Enriquecimento com metadados (`source_file`, `year`).
    * Escrita da tabela `bronze.acidentes` no formato Delta, particionada por ano.

3.  **Camada Silver - Limpeza, Validação e Enriquecimento:**
    * Leitura da tabela Bronze.
    * **Limpeza:** Padronização de *case* (`upper`), tratamento de valores nulos (`na.fill`).
    * **Correção de Tipos:** Conversão de colunas numéricas lidas como texto (devido a formatação com vírgula) para `IntegerType`, usando `regexp_replace` e `cast`.
    * **Enriquecimento:** Criação de atributos de data/tempo (`month`, `day_of_week`, `periodo_do_dia`).
    * **Validação:** Remoção de duplicatas (`dropDuplicates`) e aplicação de filtros para garantir regras de negócio.
    * Escrita da tabela `silver.acidentes_limpos`.

4.  **Camada Gold - Modelagem Dimensional:**
    * Leitura da tabela Silver.
    * **Data Mart 1 (Star Schema):**
        * Criação de Tabelas de Dimensão (`Dim_Tempo`, `Dim_Localizacao`, `Dim_TipoAcidente`) com chaves substitutas.
        * Criação da Tabela Fato (`Fato_Acidentes`) através de `JOIN`s, com métricas de negócio.
    * **Data Mart 2 (Análise de Veículos):**
        * Criação da `Dim_Vehicle`.
        * Aplicação de uma transformação **Unpivot** (`stack`) para mudar a granularidade dos dados.
        * Criação de uma segunda Tabela Fato (`Fato_Acidentes_Veiculos`) de granularidade fina.
    * Salvamento de todas as tabelas Fato e Dimensão na camada Gold.

5.  **Demonstração dos Recursos do Delta Lake:**
    * **ACID Transactions:** Execução de uma operação de `UPDATE` na tabela Silver para padronizar dados de forma segura.
    * **Auditoria e Histórico:** Uso do comando `DESCRIBE HISTORY` para auditar as transações na tabela.
    * **Time Travel:** Demonstração da capacidade de consultar uma versão anterior da tabela, antes da correção ser aplicada.

-----

### 💡 Próximos Passos

Este projeto pode ser estendido com as seguintes funcionalidades:

* **Orquestração:** Automatizar a execução do pipeline usando uma ferramenta como Airflow, Prefect ou Mage.
* **Visualização:** Conectar a camada Gold a uma ferramenta de BI (como Power BI, Tableau ou Looker Studio) para criar um dashboard interativo.
* **Testes de Qualidade:** Integrar uma biblioteca como Great Expectations para criar validações de dados mais robustas.
* **Infraestrutura como Código:** Usar Terraform para provisionar a infraestrutura necessária para rodar este pipeline na nuvem (ex: Databricks, AWS Glue, Google Dataproc).

-----

### 👨‍💻 Autor

**Vitor Gabriel Gomes Dias**

* [LinkedIn](https://www.linkedin.com/in/vggd/)
* [GitHub](https://github.com/vggd18)
