# BEEGOL_PERCENTIS_TRAFEGO Notebook Documentation

## Overview
This notebook processes Beegol weekly traffic data from Snowflake, calculates various consumption statistics including percentiles, and updates a destination table with new data. The notebook is designed to avoid duplicate entries by checking for existing data before insertion.

## Data Source
- **Source Table**: `dwdev.DWADM.IND_BEEGOL_WEEKLY`
- **Destination Table**: `DWDEV.DATASCIENCE.BEEGOL_PERCENTIS_TRAFEGO`

## Process Flow

### 1. Data Extraction
- Connects to Snowflake using Snowpark
- Extracts relevant fields from the source table:
  - ID_CONTRACT
  - SEGMENT
  - DOWN_NOMINAL_BROADBAND_MBPS (renamed as SPEED)
  - VOL_DL_MONTH_GB (renamed as CONSUMO)
  - SOURCE_FILE
  - DATA (derived from SOURCE_FILE)

### 2. Data Transformation
- Extracts reference week (`SEMA_REF`) from the source file name
- Converts `SEMA_REF` to datetime format
- Removes rows with null consumption values
- Calculates the following statistics grouped by week, segment, and speed:
  - Count
  - Mean
  - Median
  - Minimum
  - Maximum
  - 70th percentile
  - 75th percentile
  - 80th percentile

### 3. Data Loading
- Adds a timestamp column (`DATA_INGESTAO`) with the current time
- Identifies unique weeks in the processed data
- Checks for existing weeks in the destination table
- Inserts only new data (weeks that don't already exist in the destination table)

## Key Functions

### Data Extraction
```python
sql_query = """
SELECT
    ID_CONTRACT,
    SEGMENT,
    DOWN_NOMINAL_BROADBAND_MBPS AS SPEED,
    VOL_DL_MONTH_GB AS CONSUMO,
    SOURCE_FILE,
    COALESCE(
        TO_DATE(REGEXP_SUBSTR(SOURCE_FILE, '[0-9]{8}'), 'YYYYMMDD'),
        TO_DATE(REGEXP_SUBSTR(SOURCE_FILE, '[0-9]{4}-[0-9]{2}-[0-9]{2}'), 'YYYY-MM-DD')
    ) AS DATA
FROM dwdev.DWADM.IND_BEEGOL_WEEKLY
"""
```

### Data Transformation
```python
agg_dict = {
    "CONSUMO": [
        'count',
        'mean',
        'median',
        'min',
        'max',
        lambda x: np.percentile(x.dropna(), 70),
        lambda x: np.percentile(x.dropna(), 75),
        lambda x: np.percentile(x.dropna(), 80)
    ]
}

df_stats_res = df_raw_emp.groupby(['SEMA_REF', 'SEGMENT', 'SPEED'], as_index=False).agg(agg_dict)
```

### Duplicate Prevention Logic
```python
# Pegar semanas únicas que queremos inserir
unique_list = (
    df_snowpark
    .select(col("SEMA_REF").cast("string").alias("SEMA_REF"))
    .distinct()
    .to_pandas()["SEMA_REF"]
    .tolist()
)

# Pegar semanas que JÁ EXISTEM na tabela destino
semanas_existentes = (
    session.table(tabela_destino)
    .select(col("SEMA_REF").cast("string").alias("SEMA_REF"))
    .filter(col("SEMA_REF").isin(unique_list))
    .distinct()
    .to_pandas()["SEMA_REF"]
    .tolist()
)

# Filtrar apenas semanas novas
semanas_para_inserir = [s for s in unique_list if s not in semanas_existentes]
```

## Output Table Structure

The destination table `DWDEV.DATASCIENCE.BEEGOL_PERCENTIS_TRAFEGO` contains the following columns:

| Column Name | Description |
|-------------|-------------|
| SEMA_REF | Reference week |
| SEGMENT | Customer segment |
| SPEED | Nominal broadband speed in Mbps |
| CONSUMO_count | Count of consumption records |
| CONSUMO_mean | Mean consumption in GB |
| CONSUMO_median | Median consumption in GB |
| CONSUMO_min | Minimum consumption in GB |
| CONSUMO_max | Maximum consumption in GB |
| CONSUMO_PERCENTIL_70 | 70th percentile of consumption in GB |
| CONSUMO_PERCENTIL_75 | 75th percentile of consumption in GB |
| CONSUMO_PERCENTIL_80 | 80th percentile of consumption in GB |
| DATA_INGESTAO | Timestamp of data ingestion |

## Dependencies
- pandas
- numpy
- snowflake.snowpark

## Notes
- The notebook assumes the destination table already exists
- Data is appended to the destination table, not overwritten
- Only new weeks of data are inserted to avoid duplicates



```
📊 Task: TK_DS_BEEGOL_PERCENTIS_TRAFEGO_ETL
📁 Caminho
DWDEV / DATASCIENCE / TK_DS_BEEGOL_PERCENTIS_TRAFEGO_ETL
📌 Visão Geral

Task responsável por executar o notebook BEEGOL_PERCENTIS_TRAFEGO, realizando o processamento ETL relacionado a percentis de tráfego.

A execução é orquestrada via Snowflake Task, utilizando warehouse dedicado.

⚙️ Configuração da Task
🔹 Nome
DWDEV.DATASCIENCE.TK_DS_BEEGOL_PERCENTIS_TRAFEGO_ETL
🔹 Warehouse
DEV_DATASCIENCE_WH
🔹 Estado
Started
🔹 Timeout
60 minutos
🔹 Auto-Suspend
10 failures
🔹 Auto-Retry
Não configurado
⏰ Agendamento
Configuração atual exibida:

06:00 AM

Apenas às terças-feiras

UTC -03:00

Definição SQL:
schedule='USING CRON 0 6 * * 2 America/Sao_Paulo'
Interpretação do CRON:
Campo	Valor	Significado
Minuto	0	No minuto 0
Hora	6	Às 06:00
Dia do mês	*	Todo dia
Mês	*	Todo mês
Dia da semana	2	Terça-feira
Timezone	America/Sao_Paulo	Fuso horário brasileiro
🧠 Lógica Executada

A task executa um notebook armazenado no Snowflake:

create or replace task DWDEV.DATASCIENCE.TK_DS_BEEGOL_PERCENTIS_TRAFEGO_ETL
warehouse=DEV_DATASCIENCE_WH
schedule='USING CRON 0 6 * * 2 America/Sao_Paulo'
as EXECUTE NOTEBOOK "DWDEV"."DATASCIENCE"."BEEGOL_PERCENTIS_TRAFEGO"();
📘 Notebook executado:
DWDEV.DATASCIENCE.BEEGOL_PERCENTIS_TRAFEGO
🔐 Permissões
Role	Permissão
SNOWFLAKE_DEV_CIENTISTA_DADOS	OWNERSHIP
🆔 Identificador da Task
01c26461-3185-19c9-0000-0000000073ce
📎 Resumo Técnico

Task agendada semanalmente (terça-feira às 06:00)

Executa notebook ETL de percentis de tráfego

Warehouse dedicado de Data Science

Timeout de 60 minutos

Sem política de retry configurada

Se você quiser, posso também:

🔹 Gerar versão para Confluence

🔹 Gerar versão técnica mais detalhada (incluindo arquitetura)

🔹 Criar template padrão para todas suas tasks Snowflake

🔹 Gerar diagrama da arquitetura da execução

Me fala qual padrão você usa no seu time (runbook? documentação técnica? governança?) que eu deixo no formato ideal 🚀
```