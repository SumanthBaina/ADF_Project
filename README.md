# End-to-End Azure Data Engineering Project: Airline Analytics Lakehouse

## Project Overview
This project demonstrates a full-scale **Hybrid Data Lakehouse** solution built on Azure. It ingests flight data from diverse sources (On-Premises SQL, Local CSVs, and REST APIs), processes it using a **Medallion Architecture** (Bronze, Silver, Gold), and visualizes insights via **Power BI**.

The solution is designed to handle **Schema Drift**, **Incremental Loading**, and **Network Isolation** using Self-Hosted Integration Runtimes.

---

## Architecture
**Flow:** `On-Prem/API Sources` ➡️ `ADF (Ingest)` ➡️ `ADLS Gen2 (Bronze)` ➡️ `Data Flows (Silver/Delta)` ➡️ `Data Flows (Gold/Star Schema)` ➡️ `Power BI`

### 🔹 Key Components:
* **Hybrid Ingestion:** Securely moving data from a local Private Network to Cloud using **Self-Hosted Integration Runtime (SHIR)**.
* **Orchestration:** A Master Pipeline triggers child pipelines for modular execution.
* **Transformation:** **Delta Lake** implementation for ACID transactions and Upsert (SCD Type 1) logic.
* **Monitoring:** Automated Email Alerts via **Azure Logic Apps** on pipeline failure.

---

##  Key Features (Technical Deep Dive)

### 1. Dynamic Hybrid Ingestion 🌍
* **Self-Hosted IR:** Configured to bridge the local private network with Azure, including PowerShell configuration to disable local folder path validation for flexibility.
* **Metadata-Driven Pipelines:** Used `Get Metadata` and `ForEach` loops to dynamically iterate through files, allowing the pipeline to handle 10 files or 1000 files without code changes.
* **API Integration:** Ingested real-time data using **Web Activities** to call REST APIs and store responses as JSON.

### 2. Intelligent Incremental Loading 🔄
* Implemented a **Watermark Pattern** to fetch only "New" or "Modified" records.
* **Logic:** 1. Lookup `OldWatermark` (from a control table).
    2. Lookup `NewWatermark` (MAX(Date) from Source).
    3. Copy Data where `SourceDate > OldWatermark` AND `SourceDate <= NewWatermark`.
    4. Update Control Table with new high-water mark.

### 3. Silver Layer: Data Quality & Delta Lake 🛡️
* **Schema Drift Handling:** Enabled "Allow Schema Drift" in Data Flows to adapt to changing source columns.
* **SCD Type 1 (Upsert):** Used `Alter Row` transformation to update existing records and insert new ones based on primary keys (e.g., `Flight_ID`), ensuring the Silver layer is always current.
* **Format:** Stored as **Delta Parquet** for performance and version history.

### 4. Gold Layer: Star Schema & Aggregations 🌟
* **Joins:** Complex join logic merging `FactBookings` with `DimAirport` (Origin/Dest), `DimAirline`, and `DimPassenger`.
* **Business Logic:** Calculated custom metrics:
    * **Yield Per Minute:** `(Ticket Revenue / Flight Duration)` to measure route efficiency.
    * **Route Profitability:** Aggregated revenue by `Origin` -> `Destination`.

### 5. Automated Alerting 🚨
* Integrated **Azure Logic Apps** with ADF.
* If any activity fails, ADF triggers a Web Activity that sends a POST request to Logic Apps, which formats and sends an **Email Notification** to the support team with Error Details and Pipeline Name.

---

## 🛠️ Tech Stack
* **Cloud Platform:** Microsoft Azure
* **ETL Orchestration:** Azure Data Factory (ADF)
* **Storage:** Azure Data Lake Storage Gen2 (ADLS)
* **Compute:** Azure Integration Runtimes & Data Flow Clusters
* **Database:** Azure SQL Database & On-Premises SQL Server
* **Format:** Delta Lake (Parquet)
* **Visualization:** Power BI Desktop
* **DevOps:** Git Integration (Feature Branch Workflow)

---

## 📊 Analytics & Dashboards
The Power BI dashboard connects directly to the **Gold Delta Tables** via the ADLS Gen2 DFS endpoint.

### Key Insights Generated:
1.  **Route Efficiency Matrix:** A Scatter Plot comparing *Flight Volume* vs. *Yield Per Minute* to identify high-potential routes.
2.  **Revenue Leaders:** Bar charts identifying top-performing airlines.
3.  **Operational KPIs:** Tracked Total Revenue, Total Flights, and Average Ticket Price across time.

---

## 📝 Setup & Configuration
1.  **Prerequisites:** Azure Subscription, SQL Server (Local), Power BI Desktop.
2.  **Deployment:**
    * Clone this repository.
    * Import ARM Templates into your ADF instance.
    * Run the SQL script `init_db.sql` to create control tables.
    * Configure the Self-Hosted IR on your local machine.

## 🤝 Acknowledgements
Special thanks to the data engineering community for resources on Lakehouse patterns and ADF performance tuning.
