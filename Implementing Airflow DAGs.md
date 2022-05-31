# Operators
- Represent a single task in a workflow
- Run independently (usually)
- Generally do not share information b/w operators
- Various operators to perform different tasks.

e.g. `DummyOperator(task_id='example', dag=dag)`

## 1. BashOperator
Requires 3 arguements:
- The task_id
- The Bash Command
- The name of the dag it belongs to

**Example script:**
```py
# Import the BashOperator
from airflow.operators.bash_operator import BashOperator

# Define the BashOperator 
cleanup = BashOperator(
    task_id='cleanup_task',
    # Define the bash_command
    bash_command='cleanup.sh',
    # Add the task to the dag
    dag=analytics_dag
)

# Define a second operator to run the `consolidate_data.sh` script
consolidate = BashOperator(
    task_id='consolidate_task',
    bash_command='consolidate_data.sh',
    dag=analytics_dag)

# Define a final operator to execute the `push_data.sh` script
push_data = BashOperator(
    task_id='pushdata_task',
    bash_command='push_data.sh',
    dag=analytics_dag)
```
### Implementing the order in which the tasks should run using **Upstream (>>)/ Downstream (<<)** operator: 
```py
# Define a new pull_sales task
pull_sales = BashOperator(
    task_id='pullsales_task',
    bash_command = 'wget https://salestracking/latestinfo?json',
    dag=analytics_dag
)

# Set pull_sales to run prior to cleanup
pull_sales >> cleanup

# Configure consolidate to run after cleanup
consolidate << cleanup

# Set push_data to run last
consolidate >> push_data
```

### Someways of defining the Upstream/Downstream operators:
```
pull_data << initialize_process
pull_data >> clean >> run_ml_pipeline
generate_reports << run_ml_pipeline
```

## 2. Python Operator
Required Arguments:
- task_id
- dag entry
- **python_callable** argument set to the name of the function. You can pass arguments or keyword style arguments into the Python callable as needed.
- **op_kwargs** is defined as dict and can be used to pass arguments to the python function.

```py
from airflow.operators.python_operator import PythonOperator

def sleep(length_of_time):
    time.sleep(length_of_time)

sleep_task = PythonOperator(
    task_id = 'sleep',
    python_callable = sleep,
    op_kwargs = {'length_of_time':5},
    dag=example_dag
)
```

Another Example
```py
def pull_file(URL, savepath):
    r = requests.get(URL)
    with open(savepath, 'wb') as f:
        f.write(r.content)   
    # Use the print method for logging
    print(f"File pulled from {URL} and saved to {savepath}")

from airflow.operators.python_operator import PythonOperator

# Create the task
pull_file_task = PythonOperator(
    task_id='pull_file',
    # Add the callable
    python_callable=pull_file,
    # Define the arguments
    op_kwargs={'URL':'http://dataserver/sales.json', 'savepath':'latestsales.json'},
    dag=process_sales_dag
)
```

## 3. Email Operator
```py
from airflow.operators.email_operator import EmailOperator

email_task = EmailOperator(
    task_id = 'email_sales_report',
    to = 'sales_manager@example.com',
    subject = 'Automated Sales Report',
    html_content = 'Attached is the latest sales report',
    files = 'latest_sales.xlsx',
    dag = example_dag
)
```

### Set the order of tasks:
`pull_file_task >> email_task`

## Airflow Scheduling
When scheduling a DAG, there are several attributes of note:
- `start_date`: The date/time to initially schedule the DAG run.
- `end_date`: Optional attribute for when to stop running new DAG instances.
- `max_tries`: Optional attribute for how many attempts to make.
- `schedule_interval`: How often to schedule the DAG; Between the `start_date` and `end_date`; Can be defined via `cron` syntax or via built-in presets.

## Defining Scheduling Arguments:
```py
# Update the scheduling arguments as defined
default_args = {
  'owner': 'Engineering',
  'start_date': datetime(2019, 11, 1),
  'email': ['airflowresults@datacamp.com'],
  'email_on_failure': False,
  'email_on_retry': False,
  'retries': 3,
  'retry_delay': timedelta(minutes=20)
}

dag = DAG('update_dataflows', default_args=default_args, schedule_interval='30 12 * * 3') # 12:30 pm on wednesday
```


