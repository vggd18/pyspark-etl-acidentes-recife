# Data Lakehouse com PySpark, Delta Lake e Dados de Acidentes de Trânsito do Recife (nos ultimos 6 anos...)

### Resumo

Este projeto demonstra a construção de um pipeline de ETL (Extract, Transform, Load) de ponta-a-ponta utilizando PySpark e Delta Lake. O objetivo é ingerir e processar 6 anos de dados brutos de acidentes de trânsito da cidade do Recife, aplicar um processo de limpeza, validação e enriquecimento e, finalmente, criar tabelas agregadas prontas para consumo por ferramentas de Business Intelligence (BI) e análise de dados.

-----

### Objetivo do Projeto

O principal objetivo é simular um cenário real de engenharia de dados, onde dados brutos de uma fonte externa são processados para gerar insights valiosos. Este projeto visa demonstrar competências em:

  * Ingestão e processamento de dados em lote (batch).
  * Implementação de regras de qualidade e limpeza de dados.
  * Transformação e enriquecimento de dados para criar novas features.
  * Organização de um Data Lakehouse.
  * Utilização dos recursos ACID (atomicidade, consistência, isolamento e durabilidade) e de versionamento do Delta Lake.

-----

### Arquitetura Utilizada: 

-----

### 🔨 Ferramentas e Tecnologias

  * **Linguagem:** Python
  * **Processamento de Dados:** Apache Spark
  * **Camada de Armazenamento:** ...
  * **Ambiente de Desenvolvimento:** Google Colab / Jupyter Notebook
  * **Bibliotecas Python:** `pyspark`, `delta-spark`

-----

### 📂 Estrutura do Repositório

```
.
├── data/                   		# Onde o arquivo CSV original é armazenado
│   └── acidentes_recife_2019.csv
│   └── acidentes_recife_2020.csv
│   └── acidentes_recife_2021.csv
│   └── acidentes_recife_2022.csv
│   └── acidentes_recife_2023.csv
│   └── acidentes_recife_2024.csv
├── etl_acidentes_recife.ipynb  	# Notebook principal com todo o código do pipeline
└── README.md               		# Este arquivo
```

-----

### 🚀 Como Executar o Projeto

-----

### 💡 Próximos Passos

-----

### 👨‍💻 Autor

**[Vítor Gabriel Gomes Dias]**

  * [LinkedIn](https://www.linkedin.com/in/vggd/)
  * [GitHub](https://github.com/vggd18)