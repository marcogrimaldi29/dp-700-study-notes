---
layout: default
title: "02 — Ingest & Transform Data"
nav_order: 4
description: "Design and implement loading patterns, batch and streaming data ingestion and transformation using PySpark, SQL, KQL, Dataflow Gen2, pipelines, shortcuts, and mirroring. DP-700 Domain 2 (30–35%)."
permalink: /02-ingest-transform-data/
mermaid: true
---

# 02 — Ingest & Transform Data
> **Official Exam Weight: 30–35%**
> 📁 [← Back to Home](/dp-700-study-notes/)

---

## 🗺 Domain Overview

```mermaid
mindmap
  root((Ingest & Transform Data))
    2.1 Loading Patterns
      Full Load
      Incremental Load
      Dimensional Model Prep
      Streaming Load
    2.2 Batch Data
      Choose Data Store
      Choose Transform Tool
      OneLake Shortcuts
      Mirroring
      Pipeline Ingestion
      PySpark Transforms
      SQL Transforms
      KQL Transforms
      Denormalization
      Aggregation
      Duplicates & Missing Data
      Late-Arriving Data
    2.3 Streaming Data
      Streaming Engine Choice
      Native Tables vs Shortcuts
      Query Acceleration
      Eventstreams
      Spark Structured Streaming
      KQL Processing
      Windowing Functions
```

---

## 📥 2.1 Design & Implement Loading Patterns

### Full Load vs Incremental Load

```mermaid
flowchart TD
    START([Loading Strategy]) --> Q1{Data volume\nand frequency?}
    Q1 -->|"Small dataset\nor first load"| FULL["📦 Full Load\n(Truncate & Reload)"]
    Q1 -->|"Large dataset\nfrequent updates"| INCR["📈 Incremental Load\n(Only changes)"]
    INCR --> Q2{Change detection\nmethod?}
    Q2 -->|"Timestamp column\n(modified_date)"| WATERMARK["🔖 Watermark / High-Water Mark\n(Track last loaded timestamp)"]
    Q2 -->|"CDC available\n(source supports it)"| CDC["🔄 Change Data Capture\n(Insert/Update/Delete tracking)"]
    Q2 -->|"No reliable\nchange column"| HASH["#️⃣ Hash Comparison\n(Compare row hashes)"]
```

### Comparison Table

| Aspect | Full Load | Incremental Load |
|--------|-----------|-----------------|
| **Data transferred** | Entire dataset every run | Only new/changed rows |
| **Complexity** | Simple | More complex (change detection) |
| **Duration** | Longer (proportional to total size) | Shorter (proportional to changes) |
| **Target operation** | Truncate + insert | Merge / upsert / append |
| **Idempotent** | Yes (always same result) | Requires careful design |
| **Best for** | Small reference tables, initial loads | Large fact tables, frequent updates |

> **Exam Caveat ⚠️:**
> - **Watermark/high-water mark** is the simplest incremental pattern — requires a reliable `modified_date` or `version` column in the source
> - **MERGE** (Delta Lake) is the preferred method for upserts in Fabric Lakehouses
> - For **CDC from Azure SQL/SQL Server**, use Fabric's **Mirroring** feature

---

### Preparing Data for a Dimensional Model

```mermaid
graph TD
    RAW["📥 Raw Data\n(Source systems)"] -->|Extract & Clean| STAGING["🧹 Staging / Bronze\n(Raw, as-is)"]
    STAGING -->|Transform & Conform| SILVER["🥈 Silver / Cleaned\n(Deduped, typed, validated)"]
    SILVER -->|Model & Aggregate| GOLD["🥇 Gold / Dimensional Model"]
    GOLD --> DIM["📊 Dimension Tables\n(Customers, Products, Dates)"]
    GOLD --> FACT["📈 Fact Tables\n(Sales, Transactions)"]
    GOLD --> SCD["🔄 Slowly Changing\nDimensions (SCD)"]
```

**Slowly Changing Dimensions (SCD):**

| Type | Name | Behaviour | Implementation |
|------|------|-----------|----------------|
| **SCD Type 1** | Overwrite | Replace old value with new | Simple UPDATE |
| **SCD Type 2** | History | Keep full history with valid_from/valid_to | Add new row, close old row |
| **SCD Type 3** | Previous value | Keep current + previous value | Add previous_value column |

> **Exam Caveat ⚠️:** SCD Type 2 is the most common exam scenario — know how to implement it with Delta Lake MERGE operations including `valid_from`, `valid_to`, and `is_current` columns.

---

### Streaming Data Loading Patterns

```mermaid
flowchart TD
    SOURCE["📡 Streaming Source\n(Event Hubs, Kafka, custom app)"] --> ES["⚡ Eventstream\n(Route & transform)"]
    ES -->|Real-time analytics| EH["🏠 Eventhouse\n(KQL Database)"]
    ES -->|Store in lake| LH["🏠 Lakehouse\n(Delta tables)"]
    ES -->|Process with code| NB["📓 Notebook\n(Spark Structured Streaming)"]
```

---

## 🔄 2.2 Ingest & Transform Batch Data

### Choosing an Appropriate Data Store

```mermaid
flowchart TD
    START([Where to store data?]) --> Q1{Primary query\nengine?}
    Q1 -->|"T-SQL (read + write)\nStored procs needed"| WH["🏢 Warehouse"]
    Q1 -->|"Spark + SQL\nFlexible, multi-engine"| LH["🏠 Lakehouse"]
    Q1 -->|"KQL\nReal-time / time-series"| EH["⚡ Eventhouse\n(KQL Database)"]
    LH --> Q2{Structured or\nunstructured?}
    Q2 -->|"Structured\n(tables)"| TABLES["📊 Managed Tables\n(Tables/ folder)"]
    Q2 -->|"Unstructured\n(files, images, JSON)"| FILES["📁 Unmanaged Files\n(Files/ folder)"]
```

---

### Choosing the Right Transform Tool

| Need | Tool | Language |
|------|------|---------|
| No-code visual transforms, 150+ connectors | **Dataflow Gen2** | M (Power Query) |
| Complex transforms on large data | **Notebook** | PySpark / Spark SQL |
| Real-time analytics transforms | **KQL** | Kusto Query Language |
| SQL-based transforms in Warehouse | **T-SQL** | T-SQL |
| Simple SQL on Lakehouse tables | **Notebook (`%%sql`)** | Spark SQL |

---

### OneLake Shortcuts

Shortcuts provide **virtual access** to data without copying.

```mermaid
flowchart TD
    LH["🏠 Lakehouse"] -->|"Internal shortcut"| OTHER_LH["🏠 Other Lakehouse\n(same or different workspace)"]
    LH -->|"External shortcut"| ADLS["☁️ ADLS Gen2"]
    LH -->|"External shortcut"| S3["☁️ Amazon S3"]
    LH -->|"External shortcut"| GCS["☁️ Google Cloud Storage"]
    LH -->|"External shortcut"| DV["📊 Dataverse"]
```

| Shortcut Type | Data Copied? | Latency | Security |
|--------------|-------------|---------|----------|
| **Internal (OneLake → OneLake)** | No | Low | OneLake permissions |
| **External (ADLS Gen2, S3, GCS)** | No | Higher (cross-service) | Source + OneLake permissions |
| **Dataverse** | No | Medium | Dataverse + OneLake permissions |

> **Exam Caveat ⚠️:**
> - Shortcuts appear as regular folders in the Lakehouse — Spark and SQL can query them transparently
> - External shortcuts have **higher latency** than native OneLake data — consider this for performance-critical queries
> - Shortcut data is **not included** in OPTIMIZE or VACUUM operations — it's managed at the source

---

### Mirroring

Mirroring creates a **near real-time replica** of data from external sources into OneLake.

| Source | Support | Latency |
|--------|---------|---------|
| **Azure SQL Database** | GA | Near real-time |
| **Azure Cosmos DB** | GA | Near real-time |
| **Snowflake** | GA | Near real-time |
| **Azure SQL Managed Instance** | GA | Near real-time |
| **Azure Database for PostgreSQL** | GA | Near real-time |
| **Azure Database for MySQL** | GA | Near real-time |

```mermaid
flowchart LR
    SOURCE["🗄️ Source Database\n(Azure SQL, Cosmos DB,\nSnowflake, etc.)"] -->|"CDC / Change Feed\n(automatic)"| MIRROR["🪞 Mirrored Database\n(OneLake, Delta format)"]
    MIRROR -->|"Query via"| SQL["T-SQL\n(SQL Analytics Endpoint)"]
    MIRROR -->|"Query via"| SPARK["Spark\n(Notebooks)"]
```

> **Exam Caveat ⚠️:**
> - Mirroring vs Shortcuts: **Mirroring copies data** into OneLake (near real-time CDC), while **shortcuts point to data** without copying
> - Mirrored data is stored in **Delta format** and can be queried by Spark and SQL
> - Mirroring is **one-way** — changes in OneLake are NOT pushed back to the source

---

### Ingest Data by Using Pipelines

**Copy Activity** is the primary pipeline activity for data ingestion.

| Feature | Description |
|---------|-------------|
| **Source connectors** | 90+ connectors (databases, files, SaaS apps) |
| **Destination** | Lakehouse (Tables or Files), Warehouse, KQL Database |
| **Format support** | Parquet, CSV, JSON, ORC, Avro, Delta |
| **Parallelism** | Configurable degree of copy parallelism |
| **Staging** | Optional staging via ADLS Gen2 for large copies |

---

### Transform Data by Using PySpark

#### Common PySpark Patterns

**Read & write Delta tables:**

```python
# Read from Lakehouse table
df = spark.read.format("delta").load("Tables/raw_sales")

# Transform
from pyspark.sql.functions import col, year, month, sum as _sum

df_clean = (df
    .filter(col("amount") > 0)
    .withColumn("year", year("order_date"))
    .withColumn("month", month("order_date"))
    .dropDuplicates(["order_id"])
)

# Write to Lakehouse table
df_clean.write.format("delta").mode("overwrite").saveAsTable("cleaned_sales")
```

**Delta MERGE (upsert):**

```python
from delta.tables import DeltaTable

target = DeltaTable.forName(spark, "gold_customers")

(target.alias("t")
    .merge(df_new.alias("s"), "t.customer_id = s.customer_id")
    .whenMatchedUpdateAll()
    .whenNotMatchedInsertAll()
    .execute()
)
```

**Partitioning:**

```python
# Write partitioned by year and month
df.write.format("delta") \
    .partitionBy("year", "month") \
    .mode("overwrite") \
    .saveAsTable("partitioned_sales")
```

---

### Transform Data by Using SQL (T-SQL)

```sql
-- Create a transformed table in Warehouse
CREATE TABLE gold.monthly_sales AS
SELECT
    YEAR(order_date)  AS sale_year,
    MONTH(order_date) AS sale_month,
    region,
    SUM(amount)       AS total_amount,
    COUNT(*)          AS order_count
FROM staging.raw_sales
WHERE amount > 0
GROUP BY YEAR(order_date), MONTH(order_date), region;
```

**CTAS (CREATE TABLE AS SELECT)** is commonly used in Fabric Warehouses for large transforms.

---

### Transform Data by Using KQL

```kql
// Aggregate events by hour in an Eventhouse
SensorReadings
| where Timestamp > ago(7d)
| summarize AvgTemp = avg(Temperature),
            MaxTemp = max(Temperature),
            ReadingCount = count()
  by bin(Timestamp, 1h), DeviceId
| order by Timestamp desc
```

---

### Denormalize Data

| Pattern | Description | Use Case |
|---------|-------------|----------|
| **Flatten** | Join dimension into fact table | Star schema → flat table for BI |
| **Nest** | Store related data as struct/array | Semi-structured data in Delta |
| **Pre-aggregate** | Compute aggregates ahead of time | Dashboard performance |

---

### Handle Duplicate, Missing, and Late-Arriving Data

```mermaid
flowchart TD
    DATA["📥 Incoming Data"] --> DUP{Duplicates?}
    DUP -->|Yes| DEDUP["dropDuplicates()\nor MERGE with\nDELETE + INSERT"]
    DUP -->|No| MISS{Missing values?}
    MISS -->|Yes| FILL["fillna() / COALESCE()\nDefault values\nDrop if critical"]
    MISS -->|No| LATE{Late-arriving?}
    LATE -->|Yes| MERGE_LATE["MERGE into existing\npartitions\nUpdate aggregates"]
    LATE -->|No| DONE["✅ Clean data"]
```

> **Exam Caveat ⚠️:**
> - **Late-arriving data** in streaming scenarios is handled via **watermarks** in Spark Structured Streaming
> - In batch scenarios, use **MERGE** to upsert late-arriving records into the correct partition
> - Always design incremental loads to be **idempotent** — rerunning should produce the same result

---

## ⚡ 2.3 Ingest & Transform Streaming Data

### Choosing an Appropriate Streaming Engine

```mermaid
flowchart TD
    START([Streaming Need]) --> Q1{Analytics style?}
    Q1 -->|"Real-time dashboards\ntime-series, logs"| KQL_ENGINE["⚡ Eventhouse + KQL\n(Best for exploration & alerts)"]
    Q1 -->|"Complex transforms\njoin with batch data"| SSS["📓 Spark Structured Streaming\n(Notebooks)"]
    Q1 -->|"Route events\nno-code transforms"| ES["🔄 Eventstream\n(Event routing & processing)"]
```

### Comparison

| Feature | Eventstream | Spark Structured Streaming | KQL |
|---------|-------------|---------------------------|-----|
| **Latency** | Seconds | Seconds–minutes | Seconds |
| **Coding** | No-code / low-code | PySpark | KQL queries |
| **Best for** | Event routing, simple transforms | Complex joins, ML on streams | Real-time analytics, alerts |
| **Destination** | Eventhouse, Lakehouse, custom | Lakehouse (Delta) | KQL Database |
| **Windowing** | Basic | Full (tumbling, sliding, session) | Full (tumbling, sliding, hopping) |

---

### Native Tables vs OneLake Shortcuts in Real-Time Intelligence

| Feature | Native KQL Table | OneLake Shortcut |
|---------|-----------------|-----------------|
| **Data location** | Eventhouse (native storage) | OneLake (external reference) |
| **Ingestion** | Direct ingestion, lowest latency | No ingestion — query in place |
| **Performance** | Fastest KQL queries | Slower (cross-service reads) |
| **Use case** | Hot path — active real-time analytics | Warm/cold path — historical lookups |

**Query acceleration for OneLake shortcuts:**

| Feature | Standard Shortcut | Query-Accelerated Shortcut |
|---------|------------------|---------------------------|
| **Caching** | None | Automatic caching in Eventhouse |
| **Performance** | Slower (reads from OneLake) | Faster (reads from cache) |
| **Freshness** | Real-time | Near real-time (cache refresh interval) |
| **Use case** | Infrequent queries on cold data | Frequent queries on shortcut data |

> **Exam Caveat ⚠️:** **Query acceleration** creates a cached copy of shortcut data in the Eventhouse — it improves performance but adds CU consumption and slight data freshness delay.

---

### Process Data by Using Eventstreams

```mermaid
flowchart LR
    SRC["📡 Sources\n• Azure Event Hubs\n• Custom App\n• Azure IoT Hub\n• Kafka"] --> ES["⚡ Eventstream\n(Route + Transform)"]
    ES -->|"Destination"| EH["🏠 Eventhouse"]
    ES -->|"Destination"| LH["🏠 Lakehouse"]
    ES -->|"Destination"| WH["🏢 Warehouse"]
    ES -->|"Destination"| CUSTOM["🔧 Custom Endpoint"]
```

**Eventstream capabilities:**

| Feature | Description |
|---------|-------------|
| **No-code transforms** | Filter, manage fields, group by, aggregate |
| **Multiple destinations** | Route same stream to multiple targets |
| **Schema registry** | Avro, JSON schema support |
| **Error handling** | Dead-letter queue for failed events |

---

### Process Data by Using Spark Structured Streaming

```python
# Read streaming data from Event Hubs via Eventstream
df_stream = (spark.readStream
    .format("delta")
    .option("readChangeFeed", "true")
    .table("bronze_events")
)

# Transform
from pyspark.sql.functions import window, avg, count

df_agg = (df_stream
    .withWatermark("event_time", "10 minutes")
    .groupBy(
        window("event_time", "5 minutes"),
        "device_id"
    )
    .agg(
        avg("temperature").alias("avg_temp"),
        count("*").alias("event_count")
    )
)

# Write to Delta table
(df_agg.writeStream
    .format("delta")
    .outputMode("append")
    .option("checkpointLocation", "Files/checkpoints/sensor_agg")
    .toTable("silver_sensor_aggregates")
)
```

> **Exam Caveat ⚠️:**
> - **Watermark** is required for stateful aggregations on streaming data — it tells Spark how long to wait for late data
> - **Checkpoint location** is mandatory for fault-tolerant streaming — stores offset/state between runs
> - Use **`readChangeFeed`** on Delta tables to read changes as a stream (CDC on Delta)

---

### Windowing Functions

| Window Type | Description | Use Case |
|-------------|-------------|----------|
| **Tumbling** | Fixed-size, non-overlapping, contiguous | "Count events per 5-minute block" |
| **Sliding / Hopping** | Fixed-size, overlapping by slide interval | "Average over last 10 min, updated every 2 min" |
| **Session** | Dynamic size, based on activity gaps | "Group user clicks until 30-min inactivity" |

```mermaid
graph LR
    subgraph "Tumbling Window (5 min)"
        T1["[0:00–0:05)"] --> T2["[0:05–0:10)"] --> T3["[0:10–0:15)"]
    end
```

```mermaid
graph LR
    subgraph "Sliding Window (10 min, slide 5 min)"
        S1["[0:00–0:10)"]
        S2["[0:05–0:15)"]
        S3["[0:10–0:20)"]
    end
```

**KQL windowing example:**

```kql
// Tumbling window: events per 5-minute block
SensorReadings
| summarize EventCount = count(), AvgTemp = avg(Temperature)
  by bin(Timestamp, 5m), DeviceId
```

**PySpark windowing example:**

```python
from pyspark.sql.functions import window

df_windowed = (df_stream
    .groupBy(window("event_time", "5 minutes"), "device_id")
    .count()
)
```

---

## 📊 Quick-Reference Scenario Table

| Scenario | Requirement | Answer |
|----------|-------------|--------|
| Copy data from Azure SQL to Lakehouse | Batch ingestion | **Pipeline Copy Activity** |
| No-code transform CSV → Lakehouse table | Visual ETL | **Dataflow Gen2** |
| Complex PySpark join on 100M+ rows | Code-first large-scale | **Notebook** |
| Access ADLS Gen2 data without copying | Virtual access | **OneLake Shortcut** |
| Near real-time replica of Azure SQL | Automatic CDC | **Mirroring** |
| Process IoT events in real-time | Stream routing | **Eventstream → Eventhouse** |
| 5-minute tumbling window aggregation | Streaming analytics | **KQL `bin()` or Spark `window()`** |
| Upsert changed rows into Delta table | Incremental load | **Delta MERGE** |
| Track late-arriving streaming events | Late data handling | **Watermark in Spark Structured Streaming** |
| Query external S3 data from Lakehouse | Cross-cloud access | **OneLake Shortcut (S3)** |
| SCD Type 2 on customer dimension | History tracking | **Delta MERGE with valid_from/valid_to** |
| Aggregate real-time sensor data | Time-series analytics | **KQL in Eventhouse** |
| Speed up queries on shortcut data in RTI | Performance | **Query acceleration** |

---

[← 01 — Implement & Manage an Analytics Solution](/dp-700-study-notes/01-implement-manage-analytics-solution/) | [03 — Monitor & Optimize an Analytics Solution →](/dp-700-study-notes/03-monitor-optimize-analytics-solution/)