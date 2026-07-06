# 🎬 End-to-End Azure & Databricks Data Engineering Pipeline (Netflix Catalog)

## 📌 Project Overview
This project demonstrates a robust, enterprise-grade Data Engineering pipeline built on **Azure** and **Databricks**, utilizing the **Medallion Architecture** (Bronze, Silver, Gold). The pipeline processes Netflix catalog data, transforming raw files into a highly optimized **Star Schema** ready for Business Intelligence reporting. 

The project strictly follows Data Engineering best practices, including dynamic orchestration, automated schema inference, data deduplication, and **Slowly Changing Dimensions (SCD Type 2)** for historical data tracking.

## 🏗️ High-Level Architecture
<img width="2429" height="1504" alt="netflix" src="https://github.com/user-attachments/assets/2b6c0c79-bbdb-4f39-815d-6568a7938177" />



## 🛠️ Technology Stack
* **Cloud Platform:** Microsoft Azure (ADLS Gen2)
* **Data Integration:** Azure Data Factory (ADF)
* **Data Processing:** Azure Databricks, PySpark, Python
* **Data Storage:** Delta Lake (Delta Format)
* **Data Governance & Cataloging:** Databricks Unity Catalog & Metastore
* **Orchestration:** Databricks Workflows / Jobs

---

## 🚀 Pipeline Execution Phases

### 🥉 1. Ingestion & Bronze Layer (Raw Data)
The initial extraction phase ensures that raw data from GitHub is securely and dynamically ingested into the Azure Data Lake.
* **Dynamic ADF Pipeline:** Built an Azure Data Factory pipeline utilizing `Validation` and `ForEach` loop activities to dynamically extract multiple dataset files from GitHub and sink them into the Raw container.
* **Databricks Autoloader:** Implemented Databricks Autoloader (`cloudFiles`) to incrementally and efficiently process files landing in the Raw container, dumping them into the **Bronze Layer** as Delta tables while handling schema evolution.

### 🥈 2. Transformation & Silver Layer (Cleansed Data)
The Silver layer focuses on data quality, filtering, and structuring.
* **Dynamic Notebooks & Widgets:** Designed reusable, dynamic PySpark notebooks utilizing Databricks `dbutils.widgets`. This allowed the passing of parameters to dynamically process auxiliary datasets (e.g., directors, cast, categories) without hardcoding values.
* **Data Cleaning:** Implemented robust PySpark transformations to handle missing values, trim whitespace, remove unwanted line breaks, and drop corrupted rows (e.g., dropping null business keys like `show_id`).
* **Deduplication:** Ensured data integrity by removing exact duplicates before writing to the Silver layer as Delta tables.

### 🥇 3. Serving & Gold Layer (Business Ready)
The Gold layer models the cleansed data into a highly optimized **Star Schema** inside the Unity Catalog for fast querying and BI reporting.
* **Controlled Execution:** Due to the complexity of dynamic schema creation and historical data tracking, the Silver-to-Gold processes were executed via controlled, sequential runs rather than fully automated triggers, ensuring absolute data integrity.
* **Star Schema Design:** Constructed a centralized Fact Table (`fact_netflix_content`) and multiple surrounding Dimension Tables (`dim_date`, `dim_titles`, `dim_directors`, etc.).
* **Surrogate Keys (SK):** Generated unique Surrogate Keys for all dimensions using PySpark Window functions (`row_number()`).
* **Bridge Tables (Many-to-Many):** Engineered specialized Bridge Tables (e.g., `bridge_title_cast`, `bridge_title_category`) to resolve many-to-many relationships.
* **SCD Type 2 Implementation (The Highlight):** Engineered a complex **Slowly Changing Dimension Type 2** architecture for the `dim_titles` table. Utilized Delta Lake's `MERGE INTO` (Upsert) capabilities alongside `MD5` Row Hashing to accurately detect changes, expire old records (`is_active = False`), and append newly updated active records.

---

## ⚙️ Orchestration & Scheduling Strategy
Instead of a single monolithic pipeline, the orchestration is strategically decoupled to optimize compute resources and handle varying data arrival rates:
* **Workflow 1 (Dynamic Reference Data Load):** An automated Databricks Workflow that dynamically iterates through auxiliary files (directors, cast, category, etc.). It triggers the transformation notebooks using dynamic widgets and loads the data into the Silver layer.
* **Workflow 2 (Conditional Main Table Load):** A dedicated, scheduled Databricks Workflow exclusively for the heavy `netflix_titles` dataset. This workflow utilizes conditional time-based triggers (configured to execute the `4_silver` transformation notebook specifically on **Day 7**), loading the core dataset into the Silver layer at scheduled intervals.
* **Data Governance:** All tables across the Medallion architecture are securely registered and governed under the Databricks Unity Catalog (`netflix_catalog.silver...`, `netflix_catalog.gold...`).

Passionate about building scalable data pipelines, writing optimized PySpark code, and solving complex data modeling challenges.
