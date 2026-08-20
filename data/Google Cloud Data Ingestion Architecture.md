```yaml
---
title: "Google Cloud Data Ingestion Architecture"
date: "2026-07-29"
source_notebook: "Google Cloud Data Ingestion Architecture"
tags: [gcp, data-engineering, ETL, ELT, bigquery, dataform, dataproc]
---
````
## Introduction to the Data Life Cycle

In Google Cloud, data engineering tasks revolve around four core stages to move raw data from its source to actionable insights: **Replicate and migrate**, **Ingest**, **Transform**, and **Store**. A data engineer's primary role is to construct the pipelines that move data from **data sources** (where raw data originates) to **data sinks** (where processed data is stored for consumption).





```mermaid
graph LR
    A[Replicate & Migrate] --> B[Ingest]
    B --> C[Transform]
    C --> D[Store]

    style A fill:#e1f5fe,stroke:#4285f4,stroke-width:2px
    style B fill:#e1f5fe,stroke:#4285f4,stroke-width:2px
    style C fill:#fff3e0,stroke:#fbbc05,stroke-width:2px
    style D fill:#e8f5e9,stroke:#34a853,stroke-width:2px
````

---

## Data Types and Formats

Before migrating data, it is critical to identify the format of the data being ingested:

- **Unstructured Data:** Information stored in a non-tabular form, such as documents, images, audio files, or video. Unstructured data is usually stored in **Cloud Storage**, though BigQuery can read it via object tables.
- **Structured Data:** Information organized in tables, rows, and columns.

---

## Stage 1 & 2: Replicate, Migrate, and Ingest

Bringing data from external (on-premises or multi-cloud) systems into Google Cloud relies on choosing the right ingestion tool based on your data volume, network bandwidth, and synchronization needs.

### Migration Tools Decision Matrix

> [!WARNING] The Network Bandwidth Bottleneck The ease of migrating data depends heavily on data size and network bandwidth. For example, migrating 1 TB of data on a 100 Gbps network takes about 2 minutes, but on a 100 Mbps network, the exact same transfer takes 30 hours.

- **gcloud storage command:** Best for ad-hoc, small to medium-sized transfers from file systems, object stores, or HDFS.
- **Storage Transfer Service:** Recommended for online transfers of datasets larger than 1 TB.
- **Transfer Appliance:** A Google-owned hardware appliance shipped to your data center. Ideal for offline migrations of massive datasets (e.g., 7 TB, 40 TB, 300 TB) where network bandwidth is highly restricted.
- **Datastream:** A Change Data Capture (CDC) and replication service that continuously replicates data from RDBMS (Oracle, MySQL, PostgreSQL, SQL Server) to BigQuery or Cloud Storage with millisecond latency.
- **Database Migration Service (DMS):** Facilitates seamless, one-off or continuous transitions from on-premises RDBMS workloads directly to managed databases like Cloud SQL or AlloyDB.

---

## Stage 3: Data Transformation

Transformation adds business value to raw data by adjusting, cleaning, joining, or formatting it for downstream consumption.

![Transformation Services Add Value](Screenshot 2026-07-05 031402.png) _Google Cloud Transformation Services_

### ETL vs. ELT Architecture

> [!TIP] ETL vs. ELT: Where does the compute happen? Deciding between ETL and ELT impacts technology choice, cost, and scalability.
> 
> - **ETL (Extract, Transform, Load):** Transformation happens in an _intermediary system_ (like Dataflow or Dataproc) before reaching the target. Use this if the downstream system is not powerful enough to process massive transformations.
> - **ELT (Extract, Load, Transform):** Data is loaded into the target system first, and transformation utilizes the target system's compute power. This pattern is highly preferred for modern Data Warehouses like BigQuery.

```mermaid
flowchart TD
    subgraph ETL [ETL Pattern]
        direction LR
        E1[Extract] --> T1[Transform in Intermediary] --> L1[Load]
    end

    subgraph ELT [ELT Pattern]
        direction LR
        E2[Extract] --> L2[Load to Data Warehouse] --> T2[Transform in DW]
    end
```

### Applying ELT within BigQuery

In an ELT pattern, structured data lands in BigQuery **staging tables**. Transformations are then executed directly inside BigQuery using SQL scripts or SQL workflows before the refined data is moved to **production tables**.

![ELT Pipeline in BigQuery](Screenshot 2026-07-13 175853.png) _Visualizing an ELT Pipeline after data is staged_

### Transformation Services

- **Dataform:** A serverless framework for developing, testing, version-controlling, and scheduling ELT pipelines using **SQLX** (an extension of SQL with JavaScript). Dataform development uses workspaces that contain `definitions` (for `.sqlx` files), `includes` (for JavaScript files), and `workflow_settings.yaml` to unify transformation, testing (assertions), and automation.
    - _Reference:_ ![Dataform Development Workspace](Screenshot 2026-07-13 180601.png)
    - _Reference:_ ![Dataform Pipeline Unification](Screenshot 2026-07-13 180358.png)
- **Dataproc:** A managed service for running open-source Apache Hadoop and Spark workloads. It uses permanent or ephemeral clusters, and Serverless Spark, to execute distributed batch processing.
    - _Reference:_ ![Dataproc Workflow Templates](Screenshot 2026-07-15 083113.png)
- **Dataflow:** A serverless, fully managed service built on Apache Beam. It is uniquely powerful because it handles _both_ streaming and batch ETL workloads seamlessly using the exact same code.
- **Cloud Data Fusion & Dataprep:** GUI-based and UI-friendly tools. Dataprep is excellent for serverless data wrangling, while Data Fusion builds robust visual integrations (via the CDAP framework).

---

## Stage 4: Storing Processed Data (Sinks)

A **data sink** is the final repository where refined data rests, waiting to be consumed by data scientists, BI dashboards, or machine learning models.

### Storing Unstructured Data

Unstructured bytes are stored in **Cloud Storage (GCS)**. Objects are accessed via HTTP requests and can scale up to 5 TB per object.

- **Storage Classes:** To optimize costs, data can be assigned to different storage classes based on access frequency: **Standard** (active data), **Nearline** (accessed once a month), **Coldline** (accessed once every 90 days), and **Archive** (accessed once a year).

### Options for Storing Structured Data

Google Cloud provides purpose-built databases depending on whether the workload is transactional (OLTP) or analytical (OLAP), and whether the data model is SQL or NoSQL.

![Options for Storing Structured Data](Screenshot 2026-07-05 031808.png) _Structured Data Sink Options_

- **BigQuery:** A fully managed, serverless enterprise **Data Warehouse** built specifically for heavy analytical workloads.
- **Cloud SQL:** A managed relational database (SQL) for local/regional transactional workloads.
- **AlloyDB:** High-performance, fully managed PostgreSQL-compatible database.
- **Cloud Spanner:** A globally scalable, strongly consistent relational SQL database.
- **Bigtable:** A wide-column NoSQL database with sub-10 millisecond latency, ideal for high-throughput streaming analytical and transactional workloads.
- **Firestore:** A fast, serverless NoSQL document database built for web and mobile application development.

> [!NOTE] Data Lake vs. Data Warehouse A **Data Lake** (like Cloud Storage) stores raw, unprocessed data in any format. A **Data Warehouse** (like BigQuery) stores highly structured, pre-processed, and aggregated data built explicitly for analytical queries and reporting.

---

## Workflow Orchestration & Automation

To tie all these pipeline stages together autonomously, data engineers rely on orchestration tools:

- **Cloud Composer:** A fully managed service built on **Apache Airflow**. It uses Python-based Directed Acyclic Graphs (DAGs) to orchestrate complex ETL/ELT pipelines across multiple GCP and third-party services.
- **Cloud Scheduler:** Used for triggering batch or time-based automated workflows (e.g., executing a Dataform ELT pipeline every night).
- **Event-based execution:** Workflows that trigger instantly upon an event, such as a file landing in a Cloud Storage bucket, which then automatically kicks off a Dataproc transformation job.

## Governance

- **Dataplex:** Once data is distributed across data lakes and warehouses, Dataplex acts as a unified layer to centrally discover, manage, monitor, secure, and govern the metadata.