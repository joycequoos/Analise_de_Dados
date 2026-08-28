# DAG — Reading a CSV File in Airflow

[← Back to Data Engineering](https://github.com/joycequoos/Data_Enginer/blob/main/README.md)

Step-by-step walkthrough of a simple Apache Airflow DAG that reads a `.csv` file with `pandas` and prints the first rows to the log — a good starting point for understanding the basic structure of a DAG before moving on to more complex tasks (loading into a database, transformations, etc.).

Original file: [`Dag_Ler_Arquivo_CSV.py`](https://github.com/joycequoos/Analise_de_Dados/blob/main/Airflow/Dag_Ler_Arquivo_CSV.py)

## Table of Contents

- [DAG Overview](#dag-overview)
- [Step 1 — Imports](#step-1--imports)
- [Step 2 — Function that reads the CSV](#step-2--function-that-reads-the-csv)
- [Step 3 — DAG default arguments](#step-3--dag-default-arguments)
- [Step 4 — Defining the DAG](#step-4--defining-the-dag)
- [Step 5 — Defining the task](#step-5--defining-the-task)
- [Step 6 — Execution order](#step-6--execution-order)
- [Next steps](#next-steps)

---

## DAG Overview

| Item             | Value                                    |
| ---------------- | ---------------------------------------- |
| **DAG name**     | `read_csv_dag_latin`                     |
| **File read**    | `/opt/airflow/data/cliente_20231019.csv` |
| **Delimiter**    | `;`                                      |
| **Encoding**     | `latin1`                                 |
| **Schedule**     | `@once` (runs a single time)             |
| **Task**         | `read_csv_task`                          |

## Step 1 — Imports

```
from airflow import DAG
from airflow.operators.python import PythonOperator
import pandas as pd
from datetime import datetime
```

- `DAG` — class used to create and configure the DAG.
- `PythonOperator` — operator that allows a Python function to be run as an Airflow task.
- `pandas` — used to read and manipulate the `.csv` file.
- `datetime` — used to define the DAG's start date.

## Step 2 — Function that reads the CSV

```
def read_csv_file():
    file_path = "/opt/airflow/data/cliente_20231019.csv"

    nomes_colunas = ['cd_cliente_legado', 'nm_cliente', 'cd_cpf_cnpj', 'cd_passaporte',
                      'tp_pessoa', 'id_pep', 'dt_cadastro']

    df = pd.read_csv(file_path, sep=';', encoding='latin1', engine='python', names=nomes_colunas)

    print(df.head())
```

This is the function that actually does the work:

1. Sets the **file path** inside the Airflow container (`/opt/airflow/data/...`).
2. Sets the **column names** manually, since the CSV has no header.
3. Reads the file with `pandas.read_csv()`, using `;` as the delimiter and `latin1` as the encoding — important for files with accented characters that aren't in UTF-8.
4. Prints the **first 5 rows** of the DataFrame (`df.head()`) to the task log, just to confirm the read worked.

## Step 3 — DAG default arguments

```
default_args = {
    'owner': 'airflow',
    'depends_on_past': False,
    'start_date': datetime(2024, 3, 7),
    'email_on_failure': False,
    'email_on_retry': False,
    'retries': 1,
    'tags': ['ETL'],
}
```

| Argument           | Value                  | What it means                                                        |
| ------------------ | ---------------------- | ---------------------------------------------------------------------|
| `owner`            | `'airflow'`            | Name of the person responsible for the DAG, shown in the Airflow UI  |
| `depends_on_past`  | `False`                | The current run does not depend on the success of the previous run  |
| `start_date`       | `datetime(2024, 3, 7)` | Date from which the DAG can start being scheduled                   |
| `email_on_failure` | `False`                | Does not send an email on failure                                   |
| `email_on_retry`   | `False`                | Does not send an email on each retry                                |
| `retries`          | `1`                    | Retries the task once if it fails                                   |
| `tags`             | `['ETL']`              | Tag used to organize and filter DAGs in the Airflow UI               |

## Step 4 — Defining the DAG

```
dag = DAG(
    'read_csv_dag_latin',
    default_args=default_args,
    description='Um exemplo de DAG para ler um arquivo CSV com delimitador ";"',
    schedule_interval='@once',
)
```

Creates the `DAG` object, combining:

- The **name** (`read_csv_dag_latin`), which identifies the DAG in the Airflow UI.
- The **default arguments** defined in the previous step.
- A **description** explaining the DAG's purpose.
- The **schedule interval** (`@once`) — meaning it runs a single time, rather than on a recurring cron schedule.

## Step 5 — Defining the task

```
read_csv_task = PythonOperator(
    task_id='read_csv_task',
    python_callable=read_csv_file,
    dag=dag,
)
```

Creates the `read_csv_task` task, of type `PythonOperator`, which:

- Receives a unique `task_id` within the DAG.
- Points, via `python_callable`, to the `read_csv_file` function defined in step 2 — this is the function that will be executed when the task runs.
- Is linked to the DAG created in the previous step (`dag=dag`).

## Step 6 — Execution order

```
read_csv_task
```

Since this DAG has only **one task**, there's no need to define dependencies between tasks (like `task_a >> task_b`) — simply referencing the task at the end of the file is enough for Airflow to recognize it as part of the DAG's execution graph.

## Next steps

- Add a second task that loads the read data into a database table, chaining it with `>>`.
- Replace `schedule_interval='@once'` with a recurring schedule (e.g., `'@daily'`) once the DAG goes to production.
- Add error handling for the CSV read (missing file, mismatched columns, etc.).
- Parameterize the file path and reference date instead of hardcoding them (e.g., using Airflow's `Variable` or `{{ ds_nodash }}`).
