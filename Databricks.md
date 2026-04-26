# Databricks in Azure

<img width="1647" height="855" alt="image" src="https://github.com/user-attachments/assets/963694f6-a69f-4221-9d70-a5f54a21c52d" />

[(refer from microsoft)](https://learn.microsoft.com/en-us/azure/databricks/_extras/documents/reference-architecture-databricks-on-azure.pdf)

## How it works in Databricks

At a high level:

User / SQL / Notebook  
↓  
Databricks SQL or Spark  
↓  
Unity Catalog  
↓  
Lakehouse Federation Connection  
↓  
External Source  
Snowflake / PostgreSQL / SQL Server / BigQuery / Redshift / etc.

Unity Catalog  -> Central governance layer for data and AI assets

### Lakehouse Federation

Lakehouse Federation lets Databricks connect to external data systems without moving or copying the data into Databricks.

Lakehouse Federation lets Databricks connect to external systems like Snowflake, BigQuery, PostgreSQL, MySQL, Redshift, Azure SQL, Synapse, Salesforce, SAP, AWS Glue, Hive Metastore, and other catalogs, so users can discover, query, and control that data from Databricks through Unity Catalog

After connecting these systems, users can discover, query, and manage that external data directly from Databricks using Unity Catalog.

## What Lakehouse Federation Does

Lakehouse Federation allows Databricks users to:

- Access external data sources from Databricks
- Query data without copying it into Databricks
- Discover tables, schemas, and catalogs from external systems
- Apply governance and access control using Unity Catalog
- Work with data across multiple platforms from one place
- Reduce the need for data duplication and migration

## Why It Is Useful

Without Lakehouse Federation, companies often need to move data from external systems into Databricks before users can analyze it.

With Lakehouse Federation, Databricks can connect to the external system and query the data where it already exists.

This helps teams save time, reduce storage costs, and avoid unnecessary data movement.

## Example

Assume your company has customer data in Snowflake and sales data in PostgreSQL.

Using Lakehouse Federation, Databricks can connect to both systems and allow users to query that data directly from Databricks.

For example:
``` Sql
SELECT 
  c.customer_id,
  c.name,
  t.total_amount,
  b.invoice_status
FROM snowflake_catalog.sales.transactions t
JOIN postgres_catalog.app.customers c
  ON t.customer_id = c.customer_id
JOIN azure_sql_catalog.billing.invoices b
  ON c.customer_id = b.customer_id;
```

## Big picture

In Databricks Lakehouse Federation, there are **two federation patterns**:

1.  **Query Federation** — Databricks queries an external database/warehouse through a connector.
2.  **Catalog Federation** — Databricks/Unity Catalog federates external catalog metadata and reads the underlying table data directly from storage.

# 1\. Query Federation

**Query Federation** means Databricks connects to an external system like **Snowflake, PostgreSQL, MySQL, SQL Server, Redshift, BigQuery, Azure Synapse, or another database**, and sends SQL queries to that system.

Think of it like:

Databricks SQL / Notebook  
        ↓  
Unity Catalog connection  
        ↓  
External database / warehouse  
        ↓  
Result comes back to Databricks

## Simple meaning

Databricks does **not own the data** and usually does **not read the files directly**. Instead, Databricks talks to the external database using a connector/JDBC-style connection.

# 2\. Catalog Federation

**Catalog Federation** means Databricks connects to an external catalog/metastore, brings that metadata into Unity Catalog as a **foreign catalog**, and then Databricks reads the underlying data files directly from object storage.

Think of it like:

Databricks SQL / Spark  
        ↓  
Unity Catalog  
        ↓  
Foreign catalog metadata  
        ↓  
Cloud object storage files  
        ↓  
Databricks compute reads the data

## Simple meaning

Catalog federation is more about **federating metadata** than just querying an external database.


# Very simple analogy

## Query Federation

Like calling another restaurant and asking them to cook the food, then they send you the dish.

Databricks asks Snowflake/PostgreSQL to process data.  
External system does part of the work.  
Databricks receives the result.

## Catalog Federation

Like getting the recipe/menu from another restaurant, but cooking the food in your own kitchen.

Databricks uses external catalog metadata.  
Databricks reads the files directly.  
Databricks compute does the processing.

# When should you use Query Federation?

Use **Query Federation** when:

| Situation | Example |
| --- | --- |
| Data is in an external database | PostgreSQL, MySQL, SQL Server |
| Data is in a warehouse | Snowflake, BigQuery, Redshift |
| You want quick access without ETL | Explore customer data before ingestion |
| You want to join Databricks data with external data | Join Delta features with Snowflake transactions |
| You are building a migration plan | Validate external source before moving to Delta |


# When should you use Catalog Federation?

Use **Catalog Federation** when:

| Situation | Example |
| --- | --- |
| You already have tables registered in Hive Metastore | Legacy Databricks Hive tables |
| Your metadata is in AWS Glue | Glue catalog tables over S3 data |
| You are migrating to Unity Catalog | Incrementally move schemas/tables |
| You want central governance without immediately moving all tables | Hybrid UC + external catalog model |
| Your data is already in cloud object storage | S3, ADLS, GCS |


# Architecture examples

## Query Federation architecture

User  
 ↓  
Databricks SQL Warehouse / Notebook  
 ↓  
Unity Catalog  
 ↓  
Foreign Connection  
 ↓  
External Database  
PostgreSQL / Snowflake / BigQuery / SQL Server  
 ↓  
Result returned to Databricks

Best for:

Operational DB + Warehouse + Databricks ML

Example:

PostgreSQL customers  
\+ Snowflake transactions  
\+ Databricks churn features  
\= ML training dataset


## Catalog Federation architecture

User  
 ↓  
Databricks SQL Warehouse / Notebook  
 ↓  
Unity Catalog  
 ↓  
Foreign Catalog  
 ↓  
External Metastore / Catalog  
Hive Metastore / AWS Glue / Snowflake catalog  
 ↓  
Cloud object storage  
S3 / ADLS / GCS  
 ↓  
Databricks compute processes files

Best for:

Existing data lake migration to Unity Catalog

Example:

AWS Glue tables on S3  
→ exposed in Unity Catalog  
→ governed and queried from Databricks

* * *

# Which one is better?

It depends on where your data and metadata live.

| Requirement | Best choice |
| --- | --- |
| Query PostgreSQL, MySQL, SQL Server | Query Federation |
| Query Snowflake or BigQuery tables | Query Federation |
| Govern old Hive Metastore tables | Catalog Federation |
| Federate AWS Glue metadata | Catalog Federation |
| Migrate gradually to Unity Catalog | Catalog Federation |
| Join operational DB data with Delta tables | Query Federation |
| Build long-term lakehouse governance over object storage | Catalog Federation |
| Avoid external warehouse compute cost | Catalog Federation may be better |
| Need real-time operational data access | Query Federation may be better |


# Interview-ready answer

**Query Federation** allows Databricks to query external databases and warehouses by pushing SQL to the remote source and returning results to Databricks. It is useful for accessing systems like PostgreSQL, Snowflake, BigQuery, Redshift, and SQL Server without copying the data.

**Catalog Federation** allows Unity Catalog to federate metadata from external catalogs or metastores such as Hive Metastore or AWS Glue, so Databricks can govern and query those tables while reading the underlying data directly from cloud object storage using Databricks compute.

In short:

> Query Federation connects to external query engines. Catalog Federation connects to external catalogs and lets Databricks process the underlying data.
