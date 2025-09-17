
# Airflow Assignment: GCP Dataproc PySpark Job

## 📌 Objective
Automate a workflow using **Apache Airflow** to process daily incoming CSV files from a **Google Cloud Storage (GCS) bucket** using a **Dataproc PySpark job**, and save the transformed data into a **Hive table**.

---

## 📂 Project Structure
├── airflow_job.py # Airflow DAG definition
├── employee_batch.py # PySpark job script
└── README.md # Project documentation


---

## ⚙️ Workflow Overview
The Airflow DAG `gcp_dataproc_spark_job` executes the following workflow:

1. **File Sensor Task**
   - Uses `GCSObjectExistenceSensor` to check for the daily CSV file in a GCS bucket.
   - Pokes every 5 minutes, up to 12 hours.

2. **Dataproc Cluster Creation**
   - Creates a temporary Dataproc cluster using `DataprocCreateClusterOperator`.

3. **PySpark Job Execution**
   - Submits the PySpark job (`employee_batch.py`) to the Dataproc cluster using `DataprocSubmitPySparkJobOperator`.
   - The script:
     - Reads the daily CSV (`employee.csv`) from the GCS bucket.
     - Filters employees with salary ≥ 60,000.
     - Creates a Hive database and table if they don’t exist.
     - Writes the filtered results into the Hive table (`airflow.filtered_employee`).

4. **Dataproc Cluster Deletion**
   - Deletes the Dataproc cluster after the job finishes, using `DataprocDeleteClusterOperator`.

---

## 🗂️ File Details

### `airflow_job.py`
- Defines the Airflow DAG (`gcp_dataproc_spark_job`).
- Tasks included:
  - **File Sensor** (`file_sensor_task`)
  - **Cluster Creation** (`create_cluster`)
  - **PySpark Job Submission** (`submit_pyspark_job`)
  - **Cluster Deletion** (`delete_cluster`)
- DAG is scheduled to run **once daily**.
- `catchup` is set to `False`.

### `employee_batch.py`
- PySpark script executed on Dataproc.
- Steps:
  - Creates a Spark session with Hive support enabled.
  - Reads CSV file from GCS (`gs://airflow_ass1/input_files/employee.csv`).
  - Filters employee records with salary ≥ 60,000.
  - Creates Hive database `airflow` (if not exists).
  - Creates Hive table `filtered_employee` (if not exists).
  - Appends filtered data into Hive table.

---

## 🛠️ Setup & Execution

### 1. Prerequisites
- Google Cloud Platform project with:
  - Dataproc API enabled
  - Cloud Storage bucket created (`airflow_ass1`)
- Apache Airflow environment with:
  - `apache-airflow-providers-google` package installed
  - GCP credentials configured
- PySpark script uploaded to GCS (`gs://airflow_ass1/python_file/employee_batch.py`).
- Input CSV uploaded to GCS (`gs://airflow_ass1/input_files/employee.csv`).

### 2. Deploy the DAG
- Place `airflow_job.py` into your Airflow `dags/` folder.

### 3. Start Airflow
```bash
airflow scheduler &
airflow webserver &


4. Trigger the DAG

Either wait for the daily schedule or trigger manually via Airflow UI:

Navigate to Airflow UI → DAGs → gcp_dataproc_spark_job → Trigger DAG

✅ Expected Output

A Dataproc cluster is created and destroyed dynamically.

PySpark job runs successfully.

A Hive database airflow is created.

A Hive table filtered_employee is created (if not exists).

Filtered employee records (salary ≥ 60,000) are stored in Hive.

📊 Evaluation Criteria

Correct Airflow DAG configuration.

Successful file sensing from GCS.

Successful creation & deletion of Dataproc cluster.

Correct PySpark job execution and transformation.

Data correctly written into Hive table.
