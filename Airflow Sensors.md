## Airflow Sensors

- Derived from `airflow.sensors.base_sensor_operator`

Sensor Arguments:
- `mode`: How to check for the connection
    - `mode='poke'` => the default, runs repeatedly
    - `mode='reschedule'` => give up the task slot and try again later.
- `poke_interval`: used in the poke mode and tells Airflow how often to check for the condition. This should be atleast 1 min to keep from overloading the Airflow scheduler. Value in seconds.
- `timeout`: How long to wait in seconds before failing the task, make sure the the timeout is significantly smaller than your timeout interval.
- Sensors are also operators, they also include normal operatir attributes such as task_id and dag.

### 1. File Sensor
It's a part of the `airflow.contrib.sensors` library. Checks for the existence of a file at a certain location. Can also check if any files exist within a directory.

Example:
```py
from airflow.contrib.sensors.file_sensor import FileSensor

file_sensor_task = FileSensor(
    task_id = 'file_sense',
    filepath = 'salesdata.csv',
    poke_interval = 300,    #In seconds; sensor mode is 'poke' -default
    dag = sales_report_dag
)

init_sales_cleanup >> file_sensor_task >> generate_report
```
### Other type of sensors

- `ExternalTaskSensor`: Wait for a task in another DAG to complete.
- `HttpSensor`: Request a web URL and check for content.
- `SqlSensor`: Runs a SQL query to check for content.

Other sensors are available in `airflow.sensors` and `airflow.contrib.sensors` libraries

```py
from airflow.models import DAG
from airflow.operators.bash_operator import BashOperator
from airflow.operators.python_operator import PythonOperator
from airflow.operators.http_operator import SimpleHttpOperator
from airflow.contrib.sensors.file_sensor import FileSensor

dag = DAG(
   dag_id = 'update_state',
   default_args={"start_date": "2019-10-01"}
)

precheck = FileSensor(
   task_id='check_for_datafile',
   filepath='salesdata_ready.csv',
   dag=dag)

part1 = BashOperator(
   task_id='generate_random_number',
   bash_command='echo $RANDOM',
   dag=dag
)

import sys
def python_version():
   return sys.version

part2 = PythonOperator(
   task_id='get_python_version',
   python_callable=python_version,
   dag=dag)
   
part3 = SimpleHttpOperator(
   task_id='query_server_for_external_ip',
   endpoint='https://api.ipify.org',
   method='GET',
   dag=dag)
   
precheck >> part3 >> part2
```

### Airflow Executors
In Airflow, an executor is the component that actually runs the tasks defined within your workflows. Each executor has different capabilities and behaviors for running the set of tasks. Some may run a single task at a time on a local system, while other might split individual tasks among all the systems in a cluster.

Some different type of executors:
- `SequentialExecutor`
- `LocalExecutor`
- `CeleryExecutor`

The **SequentialExecutor** is the default execution engine in Airflow. It runs one task at a time. This means having multiple workflows scheduled around the same timeframe may cause things to take longer than expected. It's useful for debugging as it's fairly simple to follow the flow of tasks. While it's recommended for learning & testing, it's not recommended for production due to the limitations of task resources.

The **LocalExecutor** runs entirely on a single system. It basically treats each task as a process on the local system, and is able to start as many concurrent tasks as permitted by the system resources. Parallelism is defined by the user. It can utilize all resources of a given host system.

The **CeleryExecutor**: Celery is a general queuing system written in Python that allows multiple systems to communicate as a basic cluster. Using CeleryExecutor, multiple Airflow systems can be configured as workers for a given set of workflows and tasks. You can add extra systems to better balance workloads.

The type of executor being used can be checked from `airflow.cfg` file. Look for the `executor=` line to look-out for the type of executor.

You can also determine the type of executor by running the `airflow list_dags` from the CLI.

### Airflow program with Sensors and tasks.
```py
from airflow.models import DAG
from airflow.operators.bash_operator import BashOperator
from airflow.contrib.sensors.file_sensor import FileSensor
from datetime import datetime

report_dag = DAG(
    dag_id = 'execute_report',
    schedule_interval = "0 0 * * *"
)
#FileSensor with `reschedule` mode
precheck = FileSensor(
    task_id='check_for_datafile',
    filepath='salesdata_ready.csv',
    start_date=datetime(2020,2,20),
    mode='reschedule',
    dag=report_dag
)

generate_report_task = BashOperator(
    task_id='generate_report',
    bash_command='generate_report.sh',
    start_date=datetime(2020,2,20),
    dag=report_dag
)

precheck >> generate_report_task
```