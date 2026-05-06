# Building ETL Pipelines with Python & Airflow

## Overview
ETL (Extract, Transform, Load) pipelines automate the movement and processing of data at scale. Apache Airflow is the industry-standard orchestration platform for building, scheduling, and monitoring these pipelines.

**Why Airflow:**
- Programmatic workflow definition (Python code)
- Built-in scheduling, monitoring, and alerting
- Handles retries and error recovery automatically
- Scales from GB to PB of data
- Industry standard (used at Airbnb, Netflix, Uber, etc.)

---

## Part 1: ETL Fundamentals

### Extract
Pull data from source systems:
- Databases (PostgreSQL, MySQL, MongoDB)
- APIs (REST, GraphQL)
- Files (CSV, JSON, Parquet)
- Cloud storage (S3, GCS)
- Data warehouses (Snowflake, BigQuery)

### Transform
Process and clean the data:
- Data validation and quality checks
- Deduplication and merging
- Aggregation and calculations
- Format conversion
- Enrichment with external data

### Load
Write processed data to target system:
- Data warehouse (Snowflake, BigQuery, Redshift)
- Data lake (S3, GCS)
- Database (PostgreSQL, MongoDB)
- Analytics platform (Tableau, Looker)
- Real-time systems (Kafka, pub-sub)

---

## Part 2: Apache Airflow Concepts

### DAG (Directed Acyclic Graph)
A workflow representation showing tasks and dependencies.

```
Extract from API
        ↓
Validate data
    ↙   ↘
Clean  Deduplicate
    ↘   ↙
Transform
        ↓
Load to Warehouse
        ↓
Notify team
```

### Task
Individual unit of work (extract, transform, load, etc.)

### Operator
Defines what a task does:
- `PythonOperator`: Run Python function
- `BashOperator`: Run bash command
- `PostgresOperator`: Execute SQL
- `S3Operator`: Interact with S3
- `EmailOperator`: Send email

### Scheduler
Manages DAG execution based on schedule:
- `@daily`: Every day at midnight
- `@hourly`: Every hour
- `@weekly`: Every week
- Cron syntax: `0 2 * * 1` (Monday 2am)

---

## Part 3: Building Your First Pipeline

### Simple Example

```python
from airflow import DAG
from airflow.operators.python import PythonOperator
from datetime import datetime, timedelta

# Define DAG
dag = DAG(
    'simple_etl',
    start_date=datetime(2024, 1, 1),
    schedule_interval='@daily',
    default_args={'retries': 2}
)

# Define tasks
def extract():
    """Extract data from API"""
    import requests
    data = requests.get('https://api.example.com/data').json()
    return data

def transform(ti):
    """Transform and clean data"""
    data = ti.xcom_pull(task_ids='extract')
    # Clean, validate, transform
    cleaned = [d for d in data if d['valid']]
    return cleaned

def load(ti):
    """Load to warehouse"""
    data = ti.xcom_pull(task_ids='transform')
    # Write to Snowflake/BigQuery/etc
    return f"Loaded {len(data)} records"

# Create tasks
extract_task = PythonOperator(
    task_id='extract',
    python_callable=extract,
    dag=dag
)

transform_task = PythonOperator(
    task_id='transform',
    python_callable=transform,
    dag=dag
)

load_task = PythonOperator(
    task_id='load',
    python_callable=load,
    dag=dag
)

# Define dependencies
extract_task >> transform_task >> load_task
```

---

## Part 4: Data Passing Between Tasks

### XCom (Cross-Communication)
Share data between tasks.

```python
# Push data from one task
ti.xcom_push(key='my_data', value=data)

# Pull data in another task
data = ti.xcom_pull(task_ids='previous_task', key='my_data')
```

### Example
```python
def extract(ti):
    data = fetch_from_api()
    ti.xcom_push(key='raw_data', value=data)

def transform(ti):
    data = ti.xcom_pull(task_ids='extract', key='raw_data')
    clean_data = clean(data)
    ti.xcom_push(key='clean_data', value=clean_data)

def load(ti):
    data = ti.xcom_pull(task_ids='transform', key='clean_data')
    save_to_warehouse(data)
```

---

## Part 5: Scheduling & Timing

### Schedule Intervals

```python
# Common intervals
'@once'        # Run once
'@hourly'      # Every hour
'@daily'       # Every day at midnight
'@weekly'      # Every Monday at midnight
'@monthly'     # First day of month

# Cron syntax
'0 2 * * *'    # Every day at 2 AM
'0 9 * * 1-5'  # Weekdays at 9 AM
'*/15 * * * *' # Every 15 minutes
'0 0 1 * *'    # Monthly on 1st at midnight
```

### Backfill
Run DAG for past dates.

```bash
airflow backfill simple_etl \
  --start-date 2024-01-01 \
  --end-date 2024-01-31
```

---

## Part 6: Error Handling & Retries

### Automatic Retries

```python
default_args = {
    'retries': 3,           # Retry failed tasks 3 times
    'retry_delay': timedelta(minutes=5),  # Wait 5 min between retries
    'timeout': 3600,        # Task timeout (1 hour)
}

dag = DAG('etl_pipeline', default_args=default_args)
```

### Conditional Execution

```python
from airflow.operators.bash import BashOperator

# Only run if previous task succeeded
task1 >> BashOperator(
    task_id='task2',
    bash_command='echo success',
    trigger_rule='all_success'
)

# Run even if previous failed
task1 >> BashOperator(
    task_id='cleanup',
    bash_command='rm /tmp/data',
    trigger_rule='all_done'
)
```

---

## Part 7: Monitoring & Alerts

### Built-in Monitoring
- Web UI showing DAG runs and status
- Execution history and logs
- Task duration and performance
- Email notifications on failure

### Custom Alerts

```python
def alert_on_failure(context):
    """Called when task fails"""
    task = context['task']
    exception = context['exception']
    print(f"Task {task} failed: {exception}")
    # Send email, Slack message, etc.

default_args = {
    'on_failure_callback': alert_on_failure
}
```

### Health Checks

```python
def check_data_quality(ti):
    """Validate data before loading"""
    data = ti.xcom_pull(task_ids='transform')
    assert len(data) > 0, "No data to load"
    assert all(d['value'] for d in data), "Missing values"
    return "Data quality check passed"

quality_task = PythonOperator(
    task_id='quality_check',
    python_callable=check_data_quality,
    dag=dag
)

transform_task >> quality_task >> load_task
```

---

## Part 8: Real-World Example: Data Warehouse Pipeline

```python
from airflow import DAG
from airflow.operators.python import PythonOperator
from airflow.operators.postgres_operator import PostgresOperator
from datetime import datetime, timedelta

dag = DAG(
    'data_warehouse_etl',
    start_date=datetime(2024, 1, 1),
    schedule_interval='@daily',
    default_args={
        'retries': 2,
        'retry_delay': timedelta(minutes=5)
    }
)

def extract_from_api():
    """Extract customer data from API"""
    import requests
    response = requests.get('https://api.crm.com/customers')
    return response.json()

def transform_data(ti):
    """Clean and enrich data"""
    raw_data = ti.xcom_pull(task_ids='extract')
    
    cleaned = []
    for record in raw_data:
        cleaned.append({
            'customer_id': record['id'],
            'email': record['email'].lower(),
            'country': record['address']['country'],
            'created_at': record['created_date']
        })
    
    ti.xcom_push(key='cleaned_data', value=cleaned)

def load_to_warehouse(ti):
    """Load to Snowflake"""
    from snowflake.sqlalchemy import create_engine
    
    data = ti.xcom_pull(task_ids='transform', key='cleaned_data')
    
    engine = create_engine('snowflake://...')
    df = pd.DataFrame(data)
    df.to_sql('customers', engine, if_exists='replace')

# Tasks
extract = PythonOperator(
    task_id='extract',
    python_callable=extract_from_api,
    dag=dag
)

transform = PythonOperator(
    task_id='transform',
    python_callable=transform_data,
    dag=dag
)

load = PythonOperator(
    task_id='load',
    python_callable=load_to_warehouse,
    dag=dag
)

quality_check = PostgresOperator(
    task_id='quality_check',
    sql='SELECT COUNT(*) FROM customers WHERE email IS NULL',
    postgres_conn_id='snowflake_conn',
    dag=dag
)

# Dependencies
extract >> transform >> load >> quality_check
```

---

## Part 9: Production Deployment

### Installation
```bash
pip install apache-airflow
airflow db init
airflow webserver
airflow scheduler
```

### Deployment Considerations
- **Distributed setup**: Multiple workers for parallel execution
- **Logging**: Centralized logging (ELK, CloudWatch)
- **Monitoring**: Track DAG performance and SLAs
- **Version control**: Store DAGs in Git
- **Secrets management**: Secure credentials (AWS Secrets, HashiCorp Vault)

### Performance Optimization
- Use `max_active_runs` to limit concurrent DAG runs
- Parallelize independent tasks
- Use connections instead of hardcoding credentials
- Implement caching where possible
- Monitor resource usage

---

## Summary

**ETL Pipelines:**
- Automate data movement and processing at scale
- Apache Airflow provides scheduling, monitoring, retry logic
- Python makes it flexible and powerful
- Suitable for everything from GB to PB of data

**Key Concepts:**
- DAG: Workflow structure
- Task: Individual unit of work
- Operator: What the task does
- Scheduler: When to run

**Building Pipelines:**
1. Define DAG with schedule
2. Create tasks (extract, transform, load)
3. Set dependencies
4. Add error handling
5. Deploy and monitor

---

## Resources

- Apache Airflow Docs: https://airflow.apache.org/docs/
- Airflow Best Practices: https://airflow.apache.org/docs/apache-airflow/stable/best-practices.html
- Python Data Science: https://pandas.pydata.org/
- Snowflake Integration: https://docs.snowflake.com/

---

*This guide was created by **Rework Digital** - Resources Department for automation professionals.*

Questions? Reach out: resource@reworkdigital.io | Follow on GitHub: https://github.com/Reworkdigital-io
