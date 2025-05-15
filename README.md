# 🚀 NASA APOD ETL with Apache Airflow

This project defines an Apache Airflow DAG that performs a daily ETL (Extract, Transform, Load) process using NASA's Astronomy Picture of the Day (APOD) API. The data is extracted via an HTTP request, transformed to extract key fields, and loaded into a PostgreSQL database.

---

## 📋 Table of Contents

- [Overview](#overview)
- [Technologies Used](#technologies-used)
- [Project Structure](#project-structure)
- [Airflow DAG Breakdown](#airflow-dag-breakdown)
- [How to Use](#how-to-use)
- [Connection Setup](#connection-setup)
- [Example Output](#example-output)
- [License](#license)

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

## 🗂 Project Structure

```text
.
├── dags/
│   └── ETL_pipeline.py       # The main Airflow DAG
├── README.md                 # Project documentation



