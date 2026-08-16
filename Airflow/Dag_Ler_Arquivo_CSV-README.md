# DAG — Leitura de Arquivo CSV no Airflow

Passo a passo de uma DAG simples do Apache Airflow que lê um arquivo `.csv` com `pandas` e imprime as primeiras linhas no log — um bom ponto de partida para entender a estrutura básica de uma DAG antes de evoluir para tarefas mais complexas (carga em banco, transformações, etc.).

Arquivo original: [`Dag_Ler_Arquivo_CSV.py`](https://github.com/joycequoos/Analise_de_Dados/blob/main/Airflow/Dag_Ler_Arquivo_CSV.py)

## Índice

- [Visão geral da DAG](#visão-geral-da-dag)
- [Passo 1 — Importações](#passo-1--importações)
- [Passo 2 — Função que lê o CSV](#passo-2--função-que-lê-o-csv)
- [Passo 3 — Argumentos padrão da DAG](#passo-3--argumentos-padrão-da-dag)
- [Passo 4 — Definindo a DAG](#passo-4--definindo-a-dag)
- [Passo 5 — Definindo a tarefa (task)](#passo-5--definindo-a-tarefa-task)
- [Passo 6 — Ordem de execução](#passo-6--ordem-de-execução)
- [Próximos passos](#próximos-passos)

---

## Visão geral da DAG

| Item | Valor |
| --- | --- |
| **Nome da DAG** | `read_csv_dag_latin` |
| **Arquivo lido** | `/opt/airflow/data/cliente_20231019.csv` |
| **Delimitador** | `;` |
| **Codificação** | `latin1` |
| **Agendamento** | `@once` (executa uma única vez) |
| **Tarefa** | `read_csv_task` |

## Passo 1 — Importações

```python
from airflow import DAG
from airflow.operators.python import PythonOperator
import pandas as pd
from datetime import datetime
```

- `DAG` — classe usada para criar e configurar a DAG.
- `PythonOperator` — operador que permite executar uma função Python como uma tarefa do Airflow.
- `pandas` — usado para ler e manipular o arquivo `.csv`.
- `datetime` — usado para definir a data de início da DAG.

## Passo 2 — Função que lê o CSV

```python
def read_csv_file():
    file_path = "/opt/airflow/data/cliente_20231019.csv"

    nomes_colunas = ['cd_cliente_legado', 'nm_cliente', 'cd_cpf_cnpj', 'cd_passaporte',
                      'tp_pessoa', 'id_pep', 'dt_cadastro']

    df = pd.read_csv(file_path, sep=';', encoding='latin1', engine='python', names=nomes_colunas)

    print(df.head())
```

Essa é a função que efetivamente faz o trabalho:

1. Define o **caminho do arquivo** dentro do contêiner do Airflow (`/opt/airflow/data/...`).
2. Define os **nomes das colunas** manualmente, já que o CSV não tem cabeçalho.
3. Lê o arquivo com `pandas.read_csv()`, usando `;` como delimitador e `latin1` como codificação — importante para arquivos com acentuação que não estejam em UTF-8.
4. Imprime as **5 primeiras linhas** do DataFrame (`df.head()`) no log da tarefa, apenas para validar que a leitura funcionou.

## Passo 3 — Argumentos padrão da DAG

```python
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

| Argumento | Valor | O que significa |
| --- | --- | --- |
| `owner` | `'airflow'` | Nome do responsável pela DAG, exibido na interface do Airflow |
| `depends_on_past` | `False` | A execução atual não depende do sucesso da execução anterior |
| `start_date` | `datetime(2024, 3, 7)` | Data a partir da qual a DAG pode começar a ser agendada |
| `email_on_failure` | `False` | Não envia e-mail em caso de falha |
| `email_on_retry` | `False` | Não envia e-mail a cada nova tentativa |
| `retries` | `1` | Tenta executar a tarefa novamente 1 vez em caso de falha |
| `tags` | `['ETL']` | Etiqueta usada para organizar e filtrar DAGs na interface do Airflow |

## Passo 4 — Definindo a DAG

```python
dag = DAG(
    'read_csv_dag_latin',
    default_args=default_args,
    description='Um exemplo de DAG para ler um arquivo CSV com delimitador ";"',
    schedule_interval='@once',
)
```

Cria o objeto `DAG`, associando:

- O **nome** (`read_csv_dag_latin`), que identifica a DAG na interface do Airflow.
- Os **argumentos padrão** definidos no passo anterior.
- Uma **descrição** explicando o propósito da DAG.
- O **intervalo de agendamento** (`@once`) — ou seja, ela roda uma única vez, e não em um cron recorrente.

## Passo 5 — Definindo a tarefa (task)

```python
read_csv_task = PythonOperator(
    task_id='read_csv_task',
    python_callable=read_csv_file,
    dag=dag,
)
```

Cria a tarefa `read_csv_task`, do tipo `PythonOperator`, que:

- Recebe um `task_id` único dentro da DAG.
- Aponta, via `python_callable`, para a função `read_csv_file` definida no passo 2 — é essa função que será executada quando a tarefa rodar.
- É associada à DAG criada no passo anterior (`dag=dag`).

## Passo 6 — Ordem de execução

```python
read_csv_task
```

Como essa DAG tem apenas **uma tarefa**, não é necessário definir dependências entre tarefas (como `tarefa_a >> tarefa_b`) — a própria referência à tarefa no final do arquivo já é suficiente para o Airflow reconhecê-la como parte do grafo de execução da DAG.

## Próximos passos

- Adicionar uma segunda tarefa que carregue os dados lidos em uma tabela do banco de dados, encadeando-a com `>>`.
- Trocar `schedule_interval='@once'` por um agendamento recorrente (ex.: `'@daily'`) quando a DAG for para produção.
- Adicionar tratamento de erros na leitura do CSV (arquivo ausente, colunas divergentes, etc.).
- Parametrizar o caminho do arquivo e a data de referência em vez de deixá-los fixos no código (ex.: usando `Variable` ou `{{ ds_nodash }}` do Airflow).
