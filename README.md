# Data Lakehouse com PySpark, Delta Lake e Dados de Acidentes de Trânsito do Recife

### Resumo

Este projeto demonstra a construção de um pipeline de ETL (Extract, Transform, Load) de ponta-a-ponta utilizando PySpark e Delta Lake. O objetivo é ingerir múltiplos arquivos CSV anuais de acidentes de trânsito da cidade do Recife, lidar com inconsistências de schema (*schema drift*), aplicar um processo robusto de limpeza e enriquecimento e, finalmente, criar um **Data Mart** com modelo dimensional (Star Schema) pronto para consumo por ferramentas de Business Intelligence (BI).

A arquitetura implementada segue o padrão **Medallion (Bronze, Silver, Gold)**, uma abordagem moderna para a organização de um Data Lakehouse que garante qualidade, governança e performance.

-----

### Objetivo do Projeto

O principal objetivo é simular um cenário real de engenharia de dados, demonstrando competências em:

* Ingestão e processamento de dados em lote (*batch*) a partir de múltiplas fontes com schemas inconsistentes.
* Implementação de um pipeline de dados resiliente e documentado.
* Limpeza, validação de regras de negócio e enriquecimento de dados (*feature engineering*).
* Organização de um Data Lakehouse com a arquitetura Medalhão.
* Modelagem de dados para BI (Star Schema com tabelas Fato e Dimensão).
* Utilização dos recursos do Delta Lake (ACID, versionamento, particionamento).

-----

### Arquitetura Utilizada: Medallion

O pipeline é estruturado em três camadas lógicas, cada uma representada por tabelas Delta particionadas:

* **Camada Bronze (Bruta):**
    * **Função:** Ingestão dos dados crus de múltiplos arquivos CSV, servindo como uma cópia fiel e histórica da fonte. Um schema unificado é aplicado para consolidar os diferentes layouts dos arquivos.
    * **Tabela:** `bronze.acidentes`

* **Camada Silver (Limpa e Enriquecida):**
    * **Função:** Contém os dados da camada Bronze após passarem por um processo rigoroso de limpeza (padronização de texto, tratamento de nulos, correção de tipos), validação (remoção de duplicatas, filtros de regras de negócio) e enriquecimento (criação de colunas de data e hora). É a nossa "fonte única da verdade".
    * **Tabela:** `silver.acidentes_limpos`

* **Camada Gold (Agregada e Modelada):**
    * **Função:** Apresenta os dados da camada Silver em um modelo dimensional (Star Schema) otimizado para consultas analíticas. Contém tabelas Fato e Dimensão que formam um Data Mart para análise de acidentes.
    * **Tabelas:** `fato_acidentes`, `dim_tempo`, `dim_localizacao`, `dim_tipo_acidente`.

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
│       └── ...
├── data/                         # Onde os arquivos CSV de origem são armazenados
│   ├── acidentes_2019.csv
│   ├── acidentes_2020.csv
│   └── ...
├── etl_acidentes_recife.ipynb    # Notebook principal com todo o código do pipeline
└── README.md                     # Este arquivo
```

-----

### 🚀 Como Executar o Projeto

1.  **Abra o Notebook:** Abra o arquivo `etl_acidentes_recife.ipynb` no Google Colab.
2.  **Execute a Célula de Download:** Execute a primeira célula de código na seção `1.1` para baixar os arquivos de dados para a pasta `data/`.
3.  **Execute Todas as Células:** Execute as células restantes na ordem apresentada. As bibliotecas `pyspark` e `delta-spark` são instaladas e configuradas no início do notebook.

-----

### 📈 Etapas do Pipeline Implementadas

O notebook está dividido nas seguintes etapas:

1.  **Configuração do Ambiente:** Instalação das dependências e inicialização da `SparkSession` com o catálogo Delta Lake habilitado.

2.  **Camada Bronze - Ingestão:**
    * Análise de *schema drift* entre os diferentes arquivos anuais.
    * Definição de um `StructType` unificado para criar um superconjunto de todas as colunas.
    * Implementação de um loop de ingestão robusto com o padrão **Read, Rename, UnionByName** para consolidar todos os arquivos.
    * Enriquecimento com metadados (`source_file`, `year`).
    * Escrita da tabela `bronze.acidentes` no formato Delta, particionada por ano.

3.  **Camada Silver - Limpeza, Validação e Enriquecimento:**
    * Leitura da tabela Bronze.
    * **Limpeza:** Padronização de *case* em colunas de texto (`upper`), tratamento de valores nulos em campos categóricos (`na.fill`).
    * **Correção de Tipos:** Conversão de colunas numéricas que foram lidas como texto (devido a formatação com vírgula) para `IntegerType`, usando `regexp_replace` e `cast`.
    * **Enriquecimento:** Criação de atributos de data e tempo (`month`, `day_of_week`, `periodo_do_dia`) para facilitar análises.
    * **Validação:** Remoção de registros 100% duplicados (`dropDuplicates`) e aplicação de filtros para garantir regras de negócio (ex: contagens não negativas).
    * Escrita da tabela `silver.acidentes_limpos`.

4.  **Camada Gold - Modelagem Dimensional:**
    * Leitura da tabela Silver.
    * Criação de **Tabelas de Dimensão** (`Dim_Tempo`, `Dim_Localizacao`, `Dim_TipoAcidente`) com chaves substitutas (*surrogate keys*).
    * Criação da **Tabela Fato** (`Fato_Acidentes`) através de `JOIN`s, contendo chaves estrangeiras e métricas de negócio.

-----

### 💡 Próximos Passos

Este projeto pode ser estendido com as seguintes funcionalidades:

* **Orquestração:** Automatizar a execução do pipeline usando uma ferramenta como Airflow, Prefect ou Mage.
* **Visualização:** Conectar a camada Gold a uma ferramenta de BI (como Power BI, Tableau ou Looker Studio) para criar um dashboard interativo.
* **Testes de Qualidade:** Integrar uma biblioteca como Great Expectations para criar validações de dados mais robustas e declarativas.
* **Infraestrutura como Código:** Usar Terraform ou outra ferramenta para provisionar a infraestrutura necessária para rodar este pipeline na nuvem.

-----

### 👨‍💻 Autor

**[Vitor Dias]**

* [LinkedIn](https://www.linkedin.com/in/vggd)
* [GitHub](https://github.com/vggd18)
```
