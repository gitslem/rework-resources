# Build an ETL Pipeline with Apache Airflow
## Professional Data Engineering Implementation Guide

---

## Table of Contents

1. [Introduction](#introduction)
2. [ETL Fundamentals](#etl-fundamentals)
3. [Apache Airflow Architecture](#apache-airflow-architecture)
4. [Getting Started](#getting-started)
5. [Building Your First DAG](#building-your-first-dag)
6. [Advanced DAG Patterns](#advanced-dag-patterns)
7. [Data Integration Scenarios](#data-integration-scenarios)
8. [Monitoring & Observability](#monitoring--observability)
9. [Production Deployment](#production-deployment)
10. [Performance Optimization](#performance-optimization)

---

## Introduction

Apache Airflow has become the industry standard for orchestrating complex data workflows. With Airflow, you can:

- **Orchestrate complex pipelines**: Coordinate tasks across multiple systems
- **Build reproducible workflows**: Code-based DAGs ensure consistency
- **Monitor in real-time**: Web UI, logs, and metrics for full visibility
- **Scale to enterprise**: Handle thousands of concurrent tasks
- **Enable collaboration**: Version control, code review, documentation
- **Integrate anything**: 400+ pre-built operators for popular tools

This guide walks you through building production-grade ETL pipelines using Apache Airflow, from simple data loads to complex multi-system orchestrations.

### Who Should Read This Guide

- **Data Engineers** building data pipelines
- **Analytics Engineers** orchestrating transformations
- **DevOps Engineers** managing data infrastructure
- **ML Engineers** automating feature engineering
- **Analytics Managers** implementing data workflows
- **Automation Architects** designing enterprise systems

---

## ETL Fundamentals

### The ETL Process

```
Extract → Transform → Load
   ↓          ↓         ↓
Source    Business    Target
Data      Logic       Data
          
Extract:  Pull data from sources
Transform: Clean, validate, enrich
Load:     Store in target system
```

### ETL vs ELT

**ETL (Extract-Transform-Load)**
```
Source → Transform (external) → Target
Cost: Higher (transform outside)
Speed: Slower (serialized processing)
Best for: Small-to-medium data
```

**ELT (Extract-Load-Transform)**
```
Source → Target → Transform (in-place)
Cost: Lower (parallel processing)
Speed: Faster (load first)
Best for: Big data, cloud data warehouses
```

### Key ETL Concepts

| Concept | Definition | Example |
|---------|-----------|---------|
| **Extraction** | Pulling data from sources | SQL query, API call |
| **Validation** | Checking data quality | Row counts, nulls |
| **Transformation** | Business logic application | Join, aggregate, dedupe |
| **Loading** | Writing to destination | Insert into DB |
| **Orchestration** | Managing workflow execution | Airflow DAG |
| **Scheduling** | Triggering at set times | Daily at 2 AM |
| **Monitoring** | Tracking pipeline health | Success/failure alerts |
| **Error Handling** | Managing failures gracefully | Retries, fallbacks |

### ETL Data Flow

```
Data Sources           Processing            Data Warehouse
├─ Databases      →   ├─ Extract        →   ├─ Fact tables
├─ APIs           →   ├─ Validate       →   ├─ Dimension tables
├─ Files          →   ├─ Transform      →   ├─ Staging tables
├─ Streams        →   ├─ Deduplicate    →   ├─ Archives
└─ Events         →   └─ Load           →   └─ Data Lake
```

---

## Apache Airflow Architecture

### Core Components

```
┌─────────────────────────────────────────────────────┐
│              Airflow Architecture                    │
├─────────────────────────────────────────────────────┤
│                                                      │
│  ┌──────────────┐    ┌──────────────┐              │
│  │  Webserver   │◄──►│ Scheduler    │              │
│  │  (UI)        │    │              │              │
│  └──────────────┘    └──────────────┘              │
│       ▲                     │                       │
│       │                     ▼                       │
│       │            ┌────────────────┐              │
│       │            │  Task Queue    │              │
│       │            │  (Redis/RabbitMQ)           │
│       │            └────────────────┘              │
│       │                     │                       │
│       └─────────────────────┴─────────────────────┘
│                              │                      │
│                    ┌─────────▼─────────┐            │
│                    │     Workers       │            │
│                    │ (Execute Tasks)   │            │
│                    └───────────────────┘            │
│                              │                      │
│                    ┌─────────▼─────────┐            │
│                    │    Metadata DB    │            │
│                    │ (PostgreSQL)      │            │
│                    └───────────────────┘            │
│                                                      │
└─────────────────────────────────────────────────────┘
```

### Key Components Explained

#### 1. **Scheduler**
- Monitors all DAGs and tasks
- Triggers task execution based on dependencies
- Manages task state transitions
- Single point of orchestration

#### 2. **Executor**
- Runs tasks on available resources
- Types: Sequential, Local, Celery, Kubernetes
- Manages parallelization

#### 3. **Metadata Database**
- Stores DAG definitions
- Tracks task execution history
- Records task state and logs
- Manages connections and variables

#### 4. **Web Server**
- Visual interface for DAG management
- Task monitoring and logs
- Trigger manual runs
- Monitor execution metrics

#### 5. **Workers**
- Execute individual tasks
- Can be local or remote
- Scale horizontally

### DAG Structure

```
DAG = Collection of Tasks + Dependencies

Task 1 (Extract)
    ↓
Task 2 (Validate)
    ├─ Task 3 (Transform A)
    └─ Task 4 (Transform B)
    ↓
Task 5 (Load)
```

---

## Getting Started

### Prerequisites

```bash
# System requirements
- Python 3.8+
- 2GB+ RAM
- PostgreSQL or MySQL (for metadata)
- Basic understanding of Python

# Installation
pip install apache-airflow
pip install apache-airflow[celery,postgres,aws,gcp]

# Initialize Airflow
airflow db init
airflow webserver --port 8080
airflow scheduler
```

### Installation for Development

```bash
# Create isolated environment
python -m venv airflow-env
source airflow-env/bin/activate

# Install Airflow with common extras
pip install "apache-airflow==2.8.0" \
    apache-airflow-providers-amazon \
    apache-airflow-providers-google \
    apache-airflow-providers-postgres \
    apache-airflow-providers-http

# Initialize database
export AIRFLOW_HOME=~/airflow
airflow db init

# Create admin user
airflow users create \
    --username admin \
    --password admin \
    --firstname Admin \
    --lastname User \
    --role Admin \
    --email admin@example.com
```

### Project Structure

```
project/
├── dags/
│   ├── __init__.py
│   ├── etl_database.py
│   ├── etl_api.py
│   └── data_validation.py
├── plugins/
│   ├── operators/
│   │   └── custom_operator.py
│   └── hooks/
│       └── custom_hook.py
├── logs/
├── tests/
│   ├── test_dag_validation.py
│   └── test_data_quality.py
├── config/
│   ├── connections.yaml
│   └── variables.yaml
└── requirements.txt
```

---

## Building Your First DAG

### Simple Data Pipeline: CSV to Database

```python
from datetime import datetime, timedelta
from airflow import DAG
from airflow.operators.python import PythonOperator
from airflow.providers.postgres.operators.postgres import PostgresOperator
from airflow.providers.http.sensors.http import HttpSensor
import pandas as pd
import logging

# Define default arguments
default_args = {
    'owner': 'data-team',
    'retries': 2,
    'retry_delay': timedelta(minutes=5),
    'start_date': datetime(2024, 1, 1),
    'email': ['alerts@company.com'],
    'email_on_failure': True,
}

# Create DAG
dag = DAG(
    'etl_csv_to_db',
    default_args=default_args,
    description='Load CSV data to PostgreSQL',
    schedule_interval='@daily',  # Run daily
    catchup=False,
)

# Python function for extraction
def extract_data():
    """Extract data from CSV file"""
    logging.info("Extracting data from CSV...")
    df = pd.read_csv('/data/input.csv')
    logging.info(f"Extracted {len(df)} rows")
    
    # Store in XCom for next task
    return df.to_json()

# Python function for transformation
def transform_data(**context):
    """Transform extracted data"""
    logging.info("Transforming data...")
    
    # Get data from previous task
    json_data = context['task_instance'].xcom_pull(task_ids='extract_task')
    df = pd.read_json(json_data)
    
    # Apply transformations
    df['created_at'] = pd.to_datetime(df['created_at'])
    df['amount'] = df['amount'].astype(float)
    df = df.dropna(subset=['customer_id'])
    
    logging.info(f"Transformed to {len(df)} rows")
    return df.to_json()

# Define tasks
extract_task = PythonOperator(
    task_id='extract_task',
    python_callable=extract_data,
    dag=dag,
)

transform_task = PythonOperator(
    task_id='transform_task',
    python_callable=transform_data,
    dag=dag,
)

load_task = PostgresOperator(
    task_id='load_task',
    postgres_conn_id='postgres_default',
    sql='''
        INSERT INTO sales (customer_id, amount, created_at)
        SELECT customer_id, amount, created_at
        FROM staging_sales
        WHERE created_at > NOW() - INTERVAL '1 day'
    ''',
    dag=dag,
)

# Define dependencies
extract_task >> transform_task >> load_task
```

### Key DAG Concepts

#### **Operators**
Pre-built components for specific tasks:

```python
# Python execution
PythonOperator(
    task_id='task1',
    python_callable=my_function,
)

# SQL execution
PostgresOperator(
    task_id='task2',
    sql='SELECT * FROM table',
    postgres_conn_id='postgres_default',
)

# Bash execution
BashOperator(
    task_id='task3',
    bash_command='python /scripts/process.py',
)

# HTTP API calls
SimpleHttpOperator(
    task_id='task4',
    http_conn_id='api',
    endpoint='data/export',
    method='GET',
)
```

#### **Sensors**
Wait for external conditions:

```python
# Wait for file
FileSensor(
    task_id='wait_for_file',
    filepath='/data/input.csv',
    timeout=3600,  # 1 hour
    poke_interval=60,  # Check every 60 seconds
)

# Wait for database data
SqlSensor(
    task_id='wait_for_data',
    conn_id='postgres_default',
    sql="SELECT COUNT(*) FROM raw_data WHERE DATE(created_at) = '{{ ds }}'",
    success_condition='success_condition=>"{{value}}" > 0',
)

# Wait for API response
HttpSensor(
    task_id='wait_for_api',
    http_conn_id='api',
    endpoint='status',
    allowed_statuses=[200],
    timeout=60,
)
```

#### **Variables & Connections**
Manage secrets and configuration:

```python
from airflow.models import Variable
from airflow.hooks.base import BaseHook

# Access variables
api_key = Variable.get('api_key')
batch_size = Variable.get('batch_size', default_var=1000)

# Access connections
conn = BaseHook.get_connection('postgres_default')
username = conn.login
password = conn.password
host = conn.host
port = conn.port
```

---

## Advanced DAG Patterns

### Pattern 1: Dynamic Task Generation

Generate tasks based on configuration:

```python
from airflow import DAG
from airflow.operators.python import PythonOperator
from datetime import datetime, timedelta

default_args = {
    'owner': 'data-team',
    'start_date': datetime(2024, 1, 1),
}

dag = DAG(
    'dynamic_tasks',
    default_args=default_args,
    schedule_interval='@daily',
)

# Configuration for sources
sources = {
    'postgres_db': {
        'table': 'users',
        'conn_id': 'postgres_default',
    },
    'mysql_db': {
        'table': 'orders',
        'conn_id': 'mysql_default',
    },
    'mongodb': {
        'collection': 'events',
        'conn_id': 'mongo_default',
    },
}

# Create tasks dynamically
for source_name, config in sources.items():
    task = PythonOperator(
        task_id=f'extract_{source_name}',
        python_callable=extract_from_source,
        op_kwargs=config,
        dag=dag,
    )
```

### Pattern 2: Conditional Execution with Branching

Execute different paths based on conditions:

```python
from airflow.operators.python import PythonOperator, BranchPythonOperator
from airflow.operators.dummy import DummyOperator

def choose_path(**context):
    """Determine which branch to follow"""
    df = context['task_instance'].xcom_pull(task_ids='check_data_quality')
    
    if df['quality_score'] > 0.95:
        return 'load_production'
    else:
        return 'load_staging'

# Branching task
branch_task = BranchPythonOperator(
    task_id='branch_by_quality',
    python_callable=choose_path,
    dag=dag,
)

# Production load
load_prod = PythonOperator(
    task_id='load_production',
    python_callable=load_to_prod,
)

# Staging load
load_staging = PythonOperator(
    task_id='load_staging',
    python_callable=load_to_staging,
)

# Set dependencies
branch_task >> [load_prod, load_staging]
```

### Pattern 3: Error Handling and Retries

Implement robust error handling:

```python
from airflow.exceptions import AirflowException
from airflow.utils.decorators import apply_defaults

def extract_with_retry():
    """Extract with retry logic"""
    max_retries = 3
    retry_count = 0
    
    while retry_count < max_retries:
        try:
            # Extraction logic
            data = fetch_data()
            return data
        except ConnectionError as e:
            retry_count += 1
            if retry_count >= max_retries:
                raise AirflowException(f"Failed after {max_retries} retries: {str(e)}")
            # Exponential backoff
            sleep(2 ** retry_count)

# Task configuration with retries
extract_task = PythonOperator(
    task_id='extract_with_retry',
    python_callable=extract_with_retry,
    retries=3,
    retry_delay=timedelta(minutes=5),
    retry_exponential_backoff=True,
    max_retry_delay=timedelta(hours=1),
    dag=dag,
)
```

### Pattern 4: Parallel Processing

Run multiple tasks concurrently:

```python
from datetime import datetime, timedelta
from airflow import DAG
from airflow.operators.python import PythonOperator

default_args = {
    'owner': 'data-team',
    'start_date': datetime(2024, 1, 1),
}

dag = DAG(
    'parallel_processing',
    default_args=default_args,
    schedule_interval='@daily',
    max_active_runs=2,  # Max 2 DAG runs in parallel
)

# Create multiple extract tasks
extract_tasks = []
sources = ['source_a', 'source_b', 'source_c', 'source_d']

for source in sources:
    task = PythonOperator(
        task_id=f'extract_{source}',
        python_callable=extract_data,
        op_kwargs={'source': source},
        dag=dag,
        pool='extract_pool',  # Limit concurrent execution
        pool_slots=1,
    )
    extract_tasks.append(task)

# Single merge task
merge_task = PythonOperator(
    task_id='merge_data',
    python_callable=merge_datasets,
    dag=dag,
)

# All extracts → merge
for task in extract_tasks:
    task >> merge_task
```

### Pattern 5: Data Quality Validation

Validate data at each stage:

```python
from airflow.operators.python import PythonOperator
from airflow.exceptions import AirflowException

def validate_data(**context):
    """Validate data quality"""
    json_data = context['task_instance'].xcom_pull(task_ids='load_data')
    df = pd.read_json(json_data)
    
    # Validate row count
    if len(df) == 0:
        raise AirflowException("No data loaded!")
    
    # Validate columns
    required_cols = ['id', 'name', 'created_at']
    missing_cols = set(required_cols) - set(df.columns)
    if missing_cols:
        raise AirflowException(f"Missing columns: {missing_cols}")
    
    # Validate data types
    if not pd.api.types.is_datetime64_any_dtype(df['created_at']):
        raise AirflowException("created_at should be datetime")
    
    # Validate constraints
    if df['id'].isnull().any():
        raise AirflowException("id column contains nulls")
    
    # Calculate quality metrics
    quality_score = (1 - df.isnull().sum().sum() / (len(df) * len(df.columns))) * 100
    
    logging.info(f"Data quality score: {quality_score:.2f}%")
    
    if quality_score < 90:
        raise AirflowException(f"Quality score {quality_score:.2f}% below threshold")
    
    return {'quality_score': quality_score, 'row_count': len(df)}

validate_task = PythonOperator(
    task_id='validate_data',
    python_callable=validate_data,
    dag=dag,
)
```

---

## Data Integration Scenarios

### Scenario 1: Database to Data Warehouse

```python
# Extract from OLTP, load to DW

extract_task = PostgresOperator(
    task_id='extract_from_oltp',
    postgres_conn_id='oltp_db',
    sql='''
        SELECT id, customer_id, order_date, total_amount
        FROM orders
        WHERE created_at > '{{ ds }}'
    ''',
)

transform_task = PythonOperator(
    task_id='transform',
    python_callable=apply_business_logic,
)

load_task = PostgresOperator(
    task_id='load_to_dw',
    postgres_conn_id='dw_db',
    sql='''
        INSERT INTO fact_orders (id, customer_id, order_date, total_amount)
        SELECT * FROM staging_orders
    ''',
)

extract_task >> transform_task >> load_task
```

### Scenario 2: API to Data Lake

```python
def fetch_api_data(**context):
    """Fetch from API and store in S3"""
    import requests
    import json
    import boto3
    
    url = 'https://api.example.com/data'
    response = requests.get(url, headers={'Authorization': f'Bearer {api_token}'})
    data = response.json()
    
    # Store in S3
    s3_client = boto3.client('s3')
    s3_key = f"raw-data/{context['ds']}/api_data.json"
    s3_client.put_object(
        Bucket='data-lake',
        Key=s3_key,
        Body=json.dumps(data)
    )
    
    return s3_key

extract_task = PythonOperator(
    task_id='fetch_api',
    python_callable=fetch_api_data,
)
```

### Scenario 3: Multi-Source Consolidation

```python
# Consolidate data from multiple sources

# Extract from each source
extract_postgres = PostgresOperator(task_id='extract_postgres', ...)
extract_mongodb = PythonOperator(task_id='extract_mongodb', ...)
extract_api = PythonOperator(task_id='extract_api', ...)

# Transform each source
transform_postgres = PythonOperator(task_id='transform_postgres', ...)
transform_mongodb = PythonOperator(task_id='transform_mongodb', ...)
transform_api = PythonOperator(task_id='transform_api', ...)

# Merge all sources
merge_task = PythonOperator(task_id='merge_data', ...)

# Load to warehouse
load_task = PostgresOperator(task_id='load_warehouse', ...)

# Dependencies
extract_postgres >> transform_postgres >> merge_task
extract_mongodb >> transform_mongodb >> merge_task
extract_api >> transform_api >> merge_task
merge_task >> load_task
```

---

## Monitoring & Observability

### Health Checks and Alerts

```python
from airflow.providers.slack.operators.slack_webhook import SlackWebhookOperator

def check_data_freshness(**context):
    """Check if data is recent enough"""
    conn = BaseHook.get_connection('postgres_default')
    # Query logic
    if data_age_hours > 48:
        raise AirflowException("Data is stale (>48 hours old)")

# Slack alerts on failure
slack_alert = SlackWebhookOperator(
    task_id='slack_alert',
    http_conn_id='slack_webhook',
    message='''
        DAG {{ dag.dag_id }} failed
        Task: {{ task.task_id }}
        Error: {{ ti.log_url }}
    ''',
    trigger_rule='one_failed',
)
```

### Logging Best Practices

```python
import logging
from airflow.models import Variable

logger = logging.getLogger(__name__)

def process_data():
    """Process with detailed logging"""
    logger.info("Starting data processing")
    logger.debug(f"Batch size: {Variable.get('batch_size')}")
    
    try:
        df = load_data()
        logger.info(f"Loaded {len(df)} rows")
        
        df = transform_data(df)
        logger.info(f"Transformed to {len(df)} rows")
        
    except Exception as e:
        logger.error(f"Error processing data: {str(e)}", exc_info=True)
        raise
    
    logger.info("Data processing completed successfully")
    return df
```

### Metrics and Monitoring

```python
from airflow.models import TaskInstance
from datetime import datetime, timedelta

def get_pipeline_metrics():
    """Get metrics about pipeline health"""
    from airflow import settings
    from airflow.models import DagRun
    from sqlalchemy import func
    
    session = settings.Session()
    
    # Failed tasks in last 7 days
    failed_tasks = session.query(TaskInstance).filter(
        TaskInstance.state == 'failed',
        TaskInstance.end_date > datetime.utcnow() - timedelta(days=7)
    ).count()
    
    # Average task duration
    avg_duration = session.query(
        func.avg(TaskInstance.duration)
    ).filter(
        TaskInstance.state == 'success'
    ).scalar()
    
    return {
        'failed_tasks_7d': failed_tasks,
        'avg_task_duration': avg_duration,
    }
```

---

## Production Deployment

### Deployment Architecture

```
Development → Staging → Production

Dev Environment:
  - Single server setup
  - SequentialExecutor
  - SQLite metadata DB

Staging Environment:
  - Multi-server setup
  - LocalExecutor
  - PostgreSQL metadata DB
  - Same configs as prod

Production Environment:
  - Distributed setup
  - CeleryExecutor or Kubernetes
  - PostgreSQL + Replication
  - High availability
  - Monitoring & alerting
```

### Docker Deployment

```dockerfile
FROM python:3.9-slim

WORKDIR /opt/airflow

# Install dependencies
RUN apt-get update && apt-get install -y \
    build-essential \
    postgresql-client \
    && rm -rf /var/lib/apt/lists/*

# Install Python packages
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy DAGs
COPY dags /opt/airflow/dags
COPY plugins /opt/airflow/plugins

# Create user
RUN useradd -m -u 50000 airflow

USER airflow

EXPOSE 8080

CMD ["airflow", "webserver"]
```

### Kubernetes Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: airflow-webserver
spec:
  replicas: 2
  selector:
    matchLabels:
      app: airflow-webserver
  template:
    metadata:
      labels:
        app: airflow-webserver
    spec:
      containers:
      - name: airflow
        image: airflow:latest
        ports:
        - containerPort: 8080
        env:
        - name: AIRFLOW__CORE__EXECUTOR
          value: KubernetesExecutor
        - name: AIRFLOW__CORE__DAGS_FOLDER
          value: /opt/airflow/dags
        volumeMounts:
        - name: airflow-logs
          mountPath: /opt/airflow/logs
      volumes:
      - name: airflow-logs
        persistentVolumeClaim:
          claimName: airflow-logs-pvc
```

### Configuration Management

```python
# config.py - Environment-specific settings
import os

class Config:
    """Base configuration"""
    DEBUG = False
    TESTING = False
    
    # Database
    SQLALCHEMY_DATABASE_URI = os.getenv(
        'AIRFLOW__DATABASE__SQL_ALCHEMY_CONN',
        'postgresql://user:password@localhost/airflow'
    )
    
    # Email
    SMTP_HOST = os.getenv('SMTP_HOST', 'smtp.gmail.com')
    SMTP_PORT = int(os.getenv('SMTP_PORT', 587))
    
    # Slack
    SLACK_WEBHOOK_URL = os.getenv('SLACK_WEBHOOK_URL')

class DevelopmentConfig(Config):
    DEBUG = True
    AIRFLOW__CORE__DAGS_FOLDER = '/opt/airflow/dags'

class ProductionConfig(Config):
    DEBUG = False
    # Production-specific settings
```

---

## Performance Optimization

### Optimization Strategies

```python
# 1. Task Pool Management
# Limit concurrent tasks by resource
task1 = PythonOperator(
    task_id='heavy_task_1',
    pool='heavy_compute',
    pool_slots=4,  # Takes 4 slots
)

task2 = PythonOperator(
    task_id='heavy_task_2',
    pool='heavy_compute',
)

# 2. Parallelization
from airflow.operators.bash import BashOperator

tasks = []
for i in range(10):
    task = BashOperator(
        task_id=f'parallel_task_{i}',
        bash_command=f'process_batch.py {i}',
    )
    tasks.append(task)

# 3. Incremental Loading
def load_incremental(**context):
    """Load only new/changed data"""
    last_run = context['dag_run'].external_trigger_id or '2024-01-01'
    
    sql = f"""
        SELECT * FROM raw_data
        WHERE updated_at > '{last_run}'
    """
    # Execute query
```

### Best Practices Summary

**Do's ✅**
- ✅ Use XCom for task communication
- ✅ Implement data validation
- ✅ Monitor pipeline health
- ✅ Set up proper error handling
- ✅ Use template variables for flexibility
- ✅ Version control DAGs
- ✅ Test DAGs before production
- ✅ Document dependencies

**Don'ts ❌**
- ❌ Hard-code values in DAGs
- ❌ Create circular dependencies
- ❌ Ignore task failures
- ❌ Use unbounded task creation
- ❌ Store secrets in DAG files
- ❌ Block tasks with long operations
- ❌ Overlapping scheduled runs
- ❌ Ignore logging and monitoring

---

## Real-World Implementation Examples

### Example 1: Customer Data Warehouse

```python
# Complex multi-source ETL

dag = DAG(
    'customer_data_warehouse',
    schedule_interval='@daily',
    default_args=default_args,
)

# Parallel extracts from multiple sources
extract_crm = PostgresOperator(
    task_id='extract_crm',
    sql='SELECT * FROM crm_customers WHERE updated_at > {{ ds }}',
)

extract_ecommerce = PostgresOperator(
    task_id='extract_ecommerce',
    sql='SELECT * FROM ecom_orders WHERE date > {{ ds }}',
)

extract_support = PythonOperator(
    task_id='extract_support',
    python_callable=fetch_support_tickets,
)

# Transformation
transform_crm = PythonOperator(
    task_id='transform_crm',
    python_callable=transform_customer_data,
)

# Merge
merge_data = PythonOperator(
    task_id='merge_customer_data',
    python_callable=merge_all_sources,
)

# Quality check
quality_check = PythonOperator(
    task_id='quality_check',
    python_callable=validate_customer_data,
)

# Load
load_warehouse = PostgresOperator(
    task_id='load_warehouse',
    sql='INSERT INTO fact_customers SELECT * FROM staging',
)

# Update dimensions
load_dimensions = PostgresOperator(
    task_id='load_dimensions',
    sql='REFRESH MATERIALIZED VIEW dim_customers',
)

# Reporting
generate_report = PythonOperator(
    task_id='generate_report',
    python_callable=create_daily_report,
)

# Dependencies
[extract_crm, extract_ecommerce, extract_support] >> transform_crm >> merge_data
merge_data >> quality_check >> load_warehouse >> load_dimensions >> generate_report
```

---

## Troubleshooting & Common Issues

| Issue | Symptom | Solution |
|-------|---------|----------|
| **DAG not running** | No execution | Check DAG syntax, verify schedule, check logs |
| **Task timeout** | Task hangs | Increase timeout, optimize query, check resources |
| **Memory issues** | Worker crashes | Process in batches, reduce parallelization |
| **Data loss** | Records missing | Implement idempotency, add reconciliation |
| **Slow pipeline** | Long execution time | Profile tasks, parallelize, optimize queries |
| **Connection failures** | API/DB errors | Implement retries, check credentials, add sensors |

---

## Conclusion

Apache Airflow enables you to:
✅ Build scalable data pipelines
✅ Monitor and debug easily
✅ Implement best practices
✅ Scale from small to enterprise
✅ Integrate with any data source

**Next Steps:**
1. Set up Airflow locally
2. Build your first DAG
3. Test with sample data
4. Deploy to staging
5. Monitor and optimize
6. Deploy to production

---

## Resources

- **Apache Airflow Docs**: https://airflow.apache.org/docs/
- **Community**: https://airflow.apache.org/community/
- **GitHub**: https://github.com/apache/airflow
- **Astronomer Registry**: https://registry.astronomer.io/

---

## About Rework Digital

This guide was created by **Rework Digital** - Resources Department for automation professionals building innovative solutions.

**Resources Department Contact:** resource@reworkdigital.io  
**Follow us on GitHub:** https://github.com/Reworkdigital-io

---

*Last Updated: 2026-04-10*  
*Version: 1.0*
