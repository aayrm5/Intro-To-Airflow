# DataCamp Notes

### Simple DAG Definition:
 
```py
etl_dag = DAG(
   dag_id='etl_pipeline',
   default_args= {"start_date":"2020-01-08"}
)
```

### Running a simple Airflow task:
`airflow run <dag_id> <task_id> <start_date>` 

Using a DAG named example-etl, a task named download-file and a start date of 2020-01-10: 

```
airflow run example-etl download-file 2020-01-10
```

You can always use the `<airflow -h>` command to obtain further information about the airflow command.

### What is a DAG:
- Directed, there is an inherent flow representing dependencies between components.
- Acyclic, does not loop / cycle / repeat.
- Graph, the actual set of components.
- Seen in Airflow, Spark, and Luigi.

### DAG in Airflow:

**With in Airflow, DAG’s:**
- Are written in Python, but can use components written in other languages.
- Are made up of components (typically tasks) to be executed, such as operators, sensors, etc.
- Contain dependencies defined explicitly or implicitly. (Copy the file to the server before trying to import it to the database service)

**DAG Example:**

```py
from airflow.models import DAG
from datetime import datetime

default_arguments = {
	‘owner’ : ‘jdoe’,
	‘email’ : ‘jdoe@datacamp.com’
	‘Start_date’ : datetime(2020,1,20)
}

etl_dag = DAG( 
dag_id = ‘etl_workflow’, 
default_args = default_arguments 
)
```

### List the dags present in the Airflow:

`airflow list_dags`


**Another DAG Example:**
```py
from airflow.models import DAG
from airflow.operators.bash_operator import BashOperator

dag = DAG(
    dag_id = "example_dag",
    default_args = {"start_date":"2019-01-01"}
)

part1 = BashOperator(
    task_id = 'generate_random_number',
    bash_command = 'echo $RANDOM',
    dag = dag
)
```

### Airflow command to start the webserver on port 9090:
`airflow webserver -p 9090`

**DAG Example with multiple operators :**
```py
from airflow.models import DAG
from airflow.operators.bash_operator import BashOperator
from airflow.operators.python_operator import PythonOperator
from airflow.operators.http_operator import SimpleHttpOperator

dag = DAG(
    dag_id = 'update_status,
    default_args = {"start_date":"2019-01-01"}
)

part1 = BashOperator(
    task_id = 'generate_random_number',
    bash_command = 'echo $RANDOM',
    dag = dag
)

import sys
def python_version():
    return sys.version

part2 = PythonOperator(
    task_id = 'get_python_version',
    python_callable = python_version,
    dag = dag
)

part3 = SimpleHttpOperator(
    task_id = 'query_server_for_external_ip',
    endpoint = 'https://api.ipify.org',
    method = 'GET',
    dag = dag
)

# Part3 runs first before Part2
part3 >> part2
```

