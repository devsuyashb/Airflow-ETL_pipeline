# 🚀 NASA APOD ETL with Apache Airflow

This project defines an Apache Airflow DAG that performs a daily ETL (Extract, Transform, Load) process using NASA's Astronomy Picture of the Day (APOD) API. The data is extracted via an HTTP request, transformed to extract key fields, and loaded into a PostgreSQL database.

---


## 📖 Overview

This DAG does the following:

1. **Creates a table** in PostgreSQL if it doesn't exist.
2. **Sends a GET request** to NASA's APOD API to retrieve daily astronomical image data.
3. **Parses the JSON response** and extracts key metadata.
4. **Inserts the data** into a PostgreSQL table.

---

## 🛠 Technologies Used

- Python
- Apache Airflow
- PostgreSQL
- NASA APOD API
- Docker (optional for Airflow environment)

---


## 🔧 1. Create PostgreSQL Table

```python
@task
def create_table():
    postgres_hook = PostgresHook(postgres_conn_id='my_postgres_connection')
    create_table_query = """
    CREATE TABLE IF NOT EXISTS apod_data (
        id SERIAL PRIMARY KEY,
        title VARCHAR(255),
        explanation TEXT,
        url TEXT,
        date DATE,
        media_type VARCHAR(50)
    );
    """
    postgres_hook.run(create_table_query)
```
Creates the table **apod_data** if it doesn't already exist.

## 📘 DAG Configuration Parameters

Here are the key parameters used in the Airflow DAG:

- **`dag_id`**: Name of the DAG.
- **`start_date`**: Starts one day before today.
- **`schedule='@daily'`**: Runs the DAG daily.
- **`catchup=False`**: Skips historical backfilling and runs only for the current date onward.

---
## 🌐 2. Extract Data from NASA API

  ```python
  
  extract_apod = HttpOperator(
    task_id='extract_apod',
    http_conn_id='nasa_api',
    endpoint='planetary/apod?api_key={{ conn.nasa_api.extra_dejson.api_key }}',
    method='GET',
    response_filter=lambda response: response.text,
    log_response=True,
    do_xcom_push=True)
  
   ```
## 🚀 API Extraction with `HttpOperator`

- Uses **Airflow’s `HttpOperator`** to call NASA’s APOD API.
- The `{{ conn.nasa_api.extra_dejson.api_key }}` dynamically reads the API key stored in your **Airflow connection**.
- The response is **logged** and **pushed to XCom** for downstream tasks.

✅ **Tip**: We can also use `response.json()` if you prefer handling the result as a Python dictionary.

##  🔁 3. Transform API Data

```python

@task
def transform_apod_data(response_text):
    response = json.loads(response_text)
    return {
        'title': response.get('title', ''),
        'explanation': response.get('explanation', ''),
        'url': response.get('url', ''),
        'date': response.get('date', ''),
        'media_type': response.get('media_type', '')
    }

```
- Parses raw response text into a **Python dictionary**.
- Extracts and structures only the **required fields**.

## 🗃 4. Load Data into PostgreSQL

```Python

@task
def load_data_to_postgres(apod_data):
    postgres_hook = PostgresHook(postgres_conn_id='my_postgres_connection')
    insert_query = """
    INSERT INTO apod_data (title, explanation, url, date, media_type)
    VALUES (%s, %s, %s, %s, %s);
    """
    postgres_hook.run(insert_query, parameters=(
        apod_data['title'],
        apod_data['explanation'],
        apod_data['url'],
        apod_data['date'],
        apod_data['media_type']
    ))

```
- Inserts transformed data into your **PostgreSQL** table using **PostgresHook**.

## 🔗 Task Dependencies

```python
create_table() >> extract_apod
transformed = transform_apod_data(extract_apod.output)
load_data_to_postgres(transformed)
```
# ✅ Summary (ETL Flow)

| Step      | Task                   | Description                          |
|-----------|------------------------|--------------------------------------|
| Extract   | `extract_apod`         | Pull NASA's APOD data via API        |
| Transform | `transform_apod_data`  | Parse and clean JSON response        |
| Load      | `load_data_to_postgres`| Save parsed data into PostgreSQL     |

