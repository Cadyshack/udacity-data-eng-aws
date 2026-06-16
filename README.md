# Data Engineering with AWS — Portfolio

**Christian Cadieux** · [GitHub](https://github.com/Cadyshack)

---

## About

I'm a software engineer with roughly 10 years of experience in web development and content management systems, most recently deploying CMS infrastructure on Kubernetes on AWS. With the goal of pivoting into Data Engineering, I've been self-studying data engineering, and these four projects (completed as part of the Udacity Data Engineering with AWS Nanodegree) represent part of the work I've done to facilitate this transition in practice.

The projects follow the arc of a professional data engineering skill set: NoSQL data modeling → cloud data warehousing → cloud-native data lakehouses → orchestrated pipeline automation.

---

## Projects at a Glance

| # | Project | Core Tech | Focus |
|---|---------|-----------|-------|
| 1 | [NoSQL Data Modeling with Cassandra](#1-nosql-data-modeling-with-apache-cassandra) | Apache Cassandra, Python | Query-first schema design, ETL pipeline |
| 2 | [S3-to-Redshift Data Warehouse ETL](#2-s3-to-redshift-data-warehouse-etl) | AWS Redshift, S3, boto3 | Star schema, cloud IaC, data quality |
| 3 | [AWS Glue Data Lakehouse](#3-aws-glue-data-lakehouse) | AWS Glue, PySpark, Athena | Lakehouse zones, privacy enforcement, ML-ready data |
| 4 | [Airflow-Orchestrated Redshift Pipeline](#4-airflow-orchestrated-redshift-pipeline) | Apache Airflow, Redshift, Docker | DAG authoring, custom operators, scheduling |

---

## Project Details

### 1. NoSQL Data Modeling with Apache Cassandra

**[View Repository →](https://github.com/Cadyshack/udacity-data-eng-aws-data-modeling-cassandra)**

**Business problem:** A music streaming startup (Sparkify) accumulates daily event-log CSV files with no efficient way to query them for analytics. The analytics team has three specific questions they need answered repeatedly.

**What I built:** An ETL pipeline that consolidates ~30 daily event files into a single denormalized dataset, then loads it into three purpose-built Cassandra tables — each designed to answer exactly one analytics query efficiently, without `ALLOW FILTERING`.

**Key design decision:** Cassandra requires designing tables around the queries you intend to run rather than normalizing for storage efficiency. A critical modeling call was identifying that one query required `userId` as a clustering column (not a partition key) to deduplicate repeat listens — a conclusion reached through exploratory data analysis before the schema was finalized.

**Tech stack:** Apache Cassandra · Python (`cassandra-driver`, `pandas`) · Jupyter Notebook · Docker Compose

---

### 2. S3-to-Redshift Data Warehouse ETL

**[View Repository →](https://github.com/Cadyshack/udacity-data-eng-aws-redshift-data-warehouse-etl)**

**Business problem:** The same startup's data has grown beyond what on-premises tools can handle. Raw JSON logs and song metadata live in S3; the analytics team needs a query-optimized cloud data warehouse.

**What I built:** A cloud-native ETL pipeline with full infrastructure-as-code: Python and boto3 provision (and tear down) an IAM role, VPC security group, and multi-node Redshift cluster. Data is bulk-loaded via Redshift's `COPY` command into a star schema and validated with 15 automated data quality checks covering row counts, referential integrity, and primary key uniqueness.

**Key design decision:** Distribution keys were chosen to minimize cross-node shuffles on the most frequent join (`songplays ↔ users` on `user_id`); smaller dimension tables use `DISTSTYLE ALL` for broadcast joins. `ROW_NUMBER()` deduplication inside SQL avoids application-side loops and keeps the transformation logic transparent and auditable.

**Tech stack:** Amazon Redshift (ra3.xlplus) · Amazon S3 · AWS IAM · Python (`boto3`, `psycopg` v3) · Jupyter Notebook

---

### 3. AWS Glue Data Lakehouse

**[View Repository →](https://github.com/Cadyshack/udacity-data-eng-aws-glue-lakehouse)**

**Business problem:** STEDI manufactures an IoT balance-training device. They need to build an ML-ready dataset from device sensor readings and mobile app accelerometer data, but privacy regulations require that only data from customers who explicitly opted in to research sharing can enter the ML training set.

**What I built:** A three-zone lakehouse (Landing → Trusted → Curated) backed by S3 and catalogued in AWS Glue. Five PySpark Glue jobs progressively filter and transform the data across zones, producing a final dataset of 43,681 ML-ready training records. The pipeline enforces consent at the join boundary, not as a post-join filter, so no non-consenting customer data can enter a joined dataset in the first place.

**Key design decision:** A known data quality issue — a fulfillment bug that caused 30 serial numbers to be reused across all customers — required joining IoT sensor records to the curated (consent-verified) customer table rather than the raw landing data. This is the kind of real-world data quality problem that a three-zone architecture is specifically designed to handle cleanly.

**Tech stack:** AWS Glue (serverless Spark ETL) · AWS Glue Studio · AWS Athena · Amazon S3 · PySpark / Spark SQL · Python

---

### 4. Airflow-Orchestrated Redshift Pipeline

**[View Repository →](https://github.com/Cadyshack/udacity-data-eng-aws-airflow-redshift-warehouse)**

**Business problem:** A batch ETL pipeline that runs manually is not a production pipeline. A production system needs scheduling, automatic retries on failure, enforced task dependencies, and observable run history.

**What I built:** A production-style ELT pipeline orchestrated by Apache Airflow (running locally in Docker), loading S3 JSON data into a Redshift Serverless star schema on an hourly schedule. Four custom Airflow operators were built from scratch — staging, fact loading, dimension loading, and data quality — making the pipeline modular and reusable across datasets.

**Key design decision:** All four operators are fully parametrized; no SQL or credentials are hardcoded inside them. The `DataQualityOperator` accepts arbitrary SQL checks as configuration dicts, so it can validate any pipeline without modification. Dimension tables use truncate-insert (full rebuild each run); the fact table is append-only — the correct pattern for slowly changing dimensions and growing event logs respectively.

**Tech stack:** Apache Airflow 2.8.1 (CeleryExecutor) · Amazon Redshift Serverless · Amazon S3 · Docker / Docker Compose · Python 3

---

## Skills Demonstrated Across These Projects

| Skill Area | Specifics |
|---|---|
| Data modeling | NoSQL query-first design (Cassandra); star schema for OLAP (Redshift) |
| Cloud data warehousing | Redshift provisioning, `COPY` ingestion, distribution/sort keys, deduplication |
| Data lakehouse | Three-zone architecture (Landing / Trusted / Curated), Glue Catalog, Athena |
| Pipeline orchestration | Airflow DAGs, custom operators, hourly scheduling, retry logic |
| ETL / ELT development | Python ETL scripts, PySpark Glue jobs, SQL transformations |
| Infrastructure as code | boto3 (IAM, VPC security groups, Redshift); idempotent create/teardown |
| Data quality | Automated checks: row counts, referential integrity, duplicate detection, custom SQL |
| Containerization | Docker Compose for local Cassandra cluster and full Airflow stack |
| AWS services | S3 · Redshift · Glue · Athena · IAM |

---

*Completed as part of the [Udacity Data Engineering with AWS Nanodegree](https://www.udacity.com/course/data-engineer-nanodegree--nd027).*
