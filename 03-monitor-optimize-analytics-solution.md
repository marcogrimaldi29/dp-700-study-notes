---
layout: default
title: "03 — Monitor & Optimize an Analytics Solution"
nav_order: 5
description: "Monitor Fabric items, identify and resolve errors across pipelines, dataflows, notebooks, eventhouses, and T-SQL, and optimize performance for Spark, queries, and streaming. DP-700 Domain 3 (30–35%)."
permalink: /03-monitor-optimize-analytics-solution/
mermaid: true
---

# 03 — Monitor & Optimize an Analytics Solution
> **Official Exam Weight: 30–35%**
> 📁 [← Back to Home](/dp-700-study-notes/)

---

## 🗺 Domain Overview

```mermaid
mindmap
  root((Monitor & Optimize))
    3.1 Monitor Fabric Items
      Data Ingestion Monitoring
      Data Transformation Monitoring
      Semantic Model Refresh
      Workspace Monitoring
      Alerts
    3.2 Identify & Resolve Errors
      Pipeline Errors
      Dataflow Gen2 Errors
      Notebook Errors
      Eventhouse Errors
      Eventstream Errors
      T-SQL Errors
      OneLake Shortcut Errors
    3.3 Optimize Performance
      Lakehouse Table Optimization
      Pipeline Optimization
      Data Warehouse Optimization
      Eventstream & Eventhouse Optimization
      Spark Performance
      Query Performance
```

---

## 📡 3.1 Monitor Fabric Items

### Monitoring Hub

The **Monitoring Hub** is the central place in Fabric to track all running and recent activities.

```mermaid
graph TD
    HUB["📡 Monitoring Hub\n(Central dashboard)"] --> PIPE["🔧 Pipeline Runs"]
    HUB --> NB["📓 Notebook Runs"]
    HUB --> DF["🔄 Dataflow Gen2 Runs"]
    HUB --> SPARK["⚡ Spark Sessions"]
    HUB --> REFRESH["🔄 Semantic Model Refreshes"]
    HUB --> COPY["📋 Copy Jobs"]
    HUB --> MLV["🧱 Materialized Lake View Runs"]
```

| Feature | Description |
|---------|-------------|
| **View** | All activities you have access to, across workspaces |
| **Filter** | By item type, status, time range, submitter, location (workspace) |
| **Status** | Running, Succeeded, Failed, Cancelled |
| **Details** | Drill into individual run details, stages, errors; cancel in-progress runs |
| **Historical** | View past 30 days of activity |

---

### Monitoring Tools Beyond the Hub

| Tool | What it gives you | Retention |
|------|-------------------|-----------|
| **Workspace monitoring** *(preview)* | Logs & metrics from supported items (e.g., `ItemJobEventLogs`, Eventhouse, semantic models, warehouse, mirrored DBs) streamed into a read-only **Eventhouse KQL database** you query with KQL, dashboards, or alerts | **30 days** by default (editable retention/caching policies) |
| **Fabric Capacity Metrics app** | CU consumption per item/operation, smoothing, throttling, storage | **14 days** |
| **Query Insights** (Warehouse / SQL endpoint) | `queryinsights.*` views of historical queries | 30 days |
| **Mirrored database monitoring** | Replication status, rows replicated, errors, latency | — |
| **OneLake diagnostics** | Data-access activity on OneLake for a workspace | — |

> **Exam Caveat ⚠️:**
> - Workspace monitoring is enabled by a **workspace admin** (tenant setting "Workspace admins can turn on monitoring for their workspaces" required); it **doesn't backfill** earlier activity and consumes capacity
> - Need cross-run trends, KQL, or alerts on failures across many items? → **workspace monitoring**. Need one run's details? → **Monitoring hub**

---

### Monitor Data Ingestion

| What to Monitor | Where | Key Metrics |
|----------------|-------|-------------|
| **Pipeline Copy Activity** | Monitoring Hub → Pipeline runs | Rows read, rows written, duration, throughput (MB/s) |
| **Dataflow Gen2** | Monitoring Hub → Dataflow runs / refresh history | Rows processed, duration, refresh status, engine used (e.g., Fast Copy = CopyActivity) |
| **Copy job** | Copy job monitoring pane / Monitoring Hub | Rows read/written, duration, incremental state |
| **Mirroring** | Mirrored database item → Monitor replication | Replication status, rows replicated, errors |
| **Eventstream** | Eventstream monitoring tab | Events in/out, backlog, errors |

> **Exam Caveat ⚠️:** For **Pipeline Copy Activity**, check both **rows read** and **rows written** — a mismatch indicates data loss or filtering during copy.

---

### Monitor Data Transformation

| What to Monitor | Where | Key Metrics |
|----------------|-------|-------------|
| **Notebook / Spark jobs** | Monitoring Hub → Spark sessions | Duration, stages, tasks, shuffle read/write, spill |
| **Spark UI** | Linked from notebook run | DAG visualization, executor metrics, SQL tab |
| **T-SQL queries** | Dynamic Management Views (DMVs) | Query duration, resource usage |

**Key Spark UI tabs:**

| Tab | What It Shows |
|-----|--------------|
| **Jobs** | List of Spark jobs, their stages, duration |
| **Stages** | Task breakdown, shuffle metrics, data skew indicators |
| **SQL** | Physical query plan for Spark SQL operations |
| **Storage** | Cached RDDs/DataFrames, memory usage |
| **Environment** | Spark config, library versions |
| **Executors** | Per-executor metrics (memory, disk, shuffle, GC time) |

---

### Monitor Semantic Model Refresh

| Feature | Description |
|---------|-------------|
| **Monitoring Hub** | View refresh status, duration, error messages |
| **Refresh history** | Available on the semantic model item |
| **Scheduled refresh** | Configure in item settings |
| **On-demand refresh** | Manual trigger or API |

---

### Configure Alerts

| Alert Type | Description | Configuration |
|-----------|-------------|---------------|
| **Activator** | Alert or act on data conditions (e.g., value exceeds threshold) in Eventstreams, KQL querysets, Real-Time Dashboards, Power BI visuals | Activator item (shows as **Reflex**) — actions: email, Teams, run a Fabric item, Power Automate |
| **Fabric job / workspace events** | React when any item run fails or succeeds | Real-Time hub → Fabric events → set alert (Activator) |
| **Pipeline failure** | Notify on scheduled pipeline run failure | Schedule → **Failure notifications**, or Office 365 Outlook / Teams activities on the failure path |
| **Workspace monitoring alerts** | Alert from a saved KQL query over monitoring data | Monitoring item's Activator |
| **Capacity alerts** | Notify when CU consumption crosses a threshold | Admin portal → capacity settings → notifications |

---

## 🔧 3.2 Identify & Resolve Errors

### Pipeline Errors

```mermaid
flowchart TD
    ERR["❌ Pipeline Failed"] --> CHECK["📋 Check Monitoring Hub\n(Error message, failed activity)"]
    CHECK --> TYPE{Error type?}
    TYPE -->|"Connection"| CONN["🔌 Connection Error\n• Check credentials\n• Check network/firewall\n• Check gateway status"]
    TYPE -->|"Timeout"| TIMEOUT["⏱️ Timeout Error\n• Increase timeout setting\n• Reduce data volume\n• Add parallelism"]
    TYPE -->|"Data type"| DTYPE["📊 Data Type Mismatch\n• Check schema mapping\n• Fix source/dest types"]
    TYPE -->|"Permission"| PERM["🔒 Permission Error\n• Check workspace role\n• Check source credentials\n• Check item permissions"]
    TYPE -->|"Capacity"| CAP["⚡ Capacity Throttling\n• Check CU usage\n• Scale up capacity\n• Reduce parallelism"]
```

**Common pipeline errors and resolutions:**

| Error | Cause | Resolution |
|-------|-------|------------|
| **UserErrorInvalidDataSourceCredentials** | Expired or wrong credentials | Update connection credentials |
| **UserErrorCopyActivityFailed** | Schema mismatch, data type incompatibility | Check column mapping, fix data types |
| **UserErrorActivityTimeout** | Activity exceeded timeout limit | Increase timeout, optimize query |
| **UserErrorCapacityThrottled** | Capacity CU limit exceeded | Scale capacity or reduce parallelism |
| **UserErrorGatewayUnreachable** | On-prem data gateway offline | Restart gateway, check network |

---

### Dataflow Gen2 Errors

| Error | Cause | Resolution |
|-------|-------|------------|
| **Refresh failed: data source error** | Source unavailable or credentials expired | Check source connectivity, refresh credentials |
| **Evaluation timeout** | Transform too complex or data too large | Simplify steps, keep steps foldable, enable staging, reduce data |
| **Destination error** | Cannot write to target Lakehouse/Warehouse | Check destination exists, permissions, schema mapping |
| **Memory exceeded** | Dataflow ran out of memory | Stage the query, split heavy steps into referenced queries, reduce data volume |
| **"Require Fast Copy" failure** | Query marked *Require fast copy* but source/transforms aren't Fast Copy–compatible | Remove incompatible steps (stage first) or clear *Require fast copy* |
| **Gateway error** | On-premises data gateway offline or outdated | Restart/update the gateway, check the connection |
| **Duplicate column names** | Column name collision after join/merge | Rename columns before merge |

> **Exam Caveat ⚠️:** For heavy or memory-bound Dataflow Gen2 work, **stage** the data (staging is enabled **per query**, landing in the auto-created staging Lakehouse/Warehouse) and transform from the staged query — an ELT pattern. Fast Copy, the Modern Evaluator, and partitioned compute are the other levers.

---

### Notebook Errors

| Error | Cause | Resolution |
|-------|-------|------------|
| **OutOfMemoryError** | Data too large for driver/executor memory | Increase node size, repartition data, use broadcast joins carefully |
| **AnalysisException: Table not found** | Wrong table name or path | Verify table exists, check Lakehouse attachment |
| **Py4JJavaError** | Java/Spark internal error | Check Spark UI logs, review stack trace |
| **Session timeout** | Idle session expired | Re-run notebook, adjust session timeout settings |
| **Library not found** | Missing pip/conda package | Add library to Environment or `%pip install` |
| **FileNotFoundException** | File path incorrect or file deleted | Verify path, check if VACUUM removed the file |

---

### Eventhouse Errors

| Error | Cause | Resolution |
|-------|-------|------------|
| **Ingestion failure** | Schema mismatch, format error | Check ingestion mapping, validate source schema |
| **Query timeout** | Query too complex or data too large | Optimize KQL query, add filters, reduce time range |
| **Capacity exceeded** | Eventhouse CU limit | Scale capacity, optimize ingestion rate |
| **Streaming ingestion errors** | Batch size too large, throttling | Reduce batch size, check event schema |

---

### Eventstream Errors

| Error | Cause | Resolution |
|-------|-------|------------|
| **Source connection failure** | Event Hub unavailable or auth failed | Check connection string, SAS policy |
| **Deserialization error** | Event format doesn't match schema | Fix event schema, check serialization format |
| **Destination write failure** | Target unavailable or permission denied | Check destination status and permissions |
| **Backlog growing** | Processing slower than ingestion rate | Scale processing, simplify transforms |

---

### T-SQL Errors

| Error | Cause | Resolution |
|-------|-------|------------|
| **Insufficient permissions** | User lacks required role | Grant appropriate permissions (db_datareader, etc.) |
| **Object does not exist** | Table/view not found | Check schema name, verify object exists |
| **Query cancelled: timeout** | Query exceeded timeout | Optimize query, add indexes, reduce data scanned |
| **Data type conversion** | Implicit conversion failure | Explicit CAST/CONVERT in query |
| **Distributed query failed** | Cross-database query issue | Check permissions on all referenced databases |

---

### OneLake Shortcut Errors

| Error | Cause | Resolution |
|-------|-------|------------|
| **Shortcut target unavailable** | External source down or moved | Verify source path, check source connectivity |
| **Access denied** | Source credentials expired or permissions changed | Update connection credentials, check source IAM |
| **Schema mismatch** | Source data format changed | Update shortcut or schema mapping |
| **Performance degradation** | High latency on external shortcuts | Consider mirroring, or enable query acceleration (RTI) |

---

## 🚀 3.3 Optimize Performance

### Optimize a Lakehouse Table

```mermaid
flowchart TD
    START([Lakehouse Performance Issue]) --> Q1{Small files\nproblem?}
    Q1 -->|Yes| OPTIMIZE["⚡ OPTIMIZE\n(Compact small files)"]
    Q1 -->|No| Q2{Old files\naccumulating?}
    Q2 -->|Yes| VACUUM["🧹 VACUUM\n(Remove old files)"]
    Q2 -->|No| Q3{Slow filtered\nqueries?}
    Q3 -->|Yes| ZORDER["📊 Z-ORDER BY\n(Co-locate related data)"]
    Q3 -->|No| Q4{Read by Power BI\nDirect Lake / SQL?}
    Q4 -->|Yes| VORDER["⚡ V-Order\n(readHeavyForPBI profile\nor OPTIMIZE ... VORDER)"]
```

**Key optimization commands:**

| Command | Purpose | Syntax |
|---------|---------|--------|
| **OPTIMIZE** | Compact small files into larger ones | `OPTIMIZE table_name` |
| **OPTIMIZE + Z-ORDER** | Compact + co-locate data by columns | `OPTIMIZE table_name ZORDER BY (col1, col2)` |
| **OPTIMIZE + V-Order** | Compact and rewrite with V-Order | `OPTIMIZE table_name VORDER` (combinable with `ZORDER BY`) |
| **VACUUM** | Remove old files beyond retention | `VACUUM table_name RETAIN 168 HOURS` |
| **V-Order (session)** | Write new files with V-Order | **Off by default in new workspaces**; `spark.conf.set("spark.sql.parquet.vorder.default", "true")` |
| **V-Order (table)** | Default V-Order for one table | `ALTER TABLE t SET TBLPROPERTIES ("delta.parquet.vorder.enabled" = "true")` |
| **Resource profile** | Apply a tuned config set | `spark.conf.set("spark.fabric.resourceProfile", "readHeavyForPBI")` |

You can also run OPTIMIZE / V-Order / VACUUM without code from the Lakehouse **Table maintenance** option, or the pipeline **Lakehouse maintenance** activity.

**Write-time layout options:**

| Option | What it does | When |
|--------|-------------|------|
| **Optimize Write** | Bin-packs data before writing → fewer, larger files (adds a shuffle) | Partitioned tables, many small writes |
| **Auto compaction** | Compacts small files after writes | Frequent small ingestion batches |
| **Adaptive target file size** | Auto-picks the ideal file size per table | Most workloads (recommended, not default) |

```sql
-- Optimize a Delta table (SQL syntax)
OPTIMIZE lakehouse1.gold_sales;

-- Optimize with Z-Order for common filter columns
OPTIMIZE lakehouse1.gold_sales ZORDER BY (region, product_category);

-- Clean up old files (default 7-day retention)
VACUUM lakehouse1.gold_sales;

-- Clean up with custom retention (e.g., 3 days)
VACUUM lakehouse1.gold_sales RETAIN 72 HOURS;
```

> **Exam Caveat ⚠️:**
> - **V-Order** is **disabled by default in new workspaces** (`writeHeavy` profile). It's a write-time Parquet layout optimization: big gains for **Power BI Direct Lake** (~40–60% on cold cache), ~10% for SQL, **no inherent benefit for Spark reads** and slower writes
> - **Z-ORDER** is complementary to V-Order — it co-locates data across files based on specified columns (best for frequently filtered columns)
> - In Runtime 1.3+, the old `spark.sql.parquet.vorder.enabled` setting is **removed** — use `spark.sql.parquet.vorder.default`
> - OPTIMIZE and VACUUM are **Spark** commands — not supported from the SQL analytics endpoint or Warehouse editor
> - **VACUUM** does NOT compact files — use **OPTIMIZE** for that. VACUUM only deletes old, unreferenced files
> - Running VACUUM with retention < 7 days can break **time travel** queries

---

### Optimize a Pipeline

| Optimization | Description |
|-------------|-------------|
| **Parallelism** | Use ForEach with parallel batches for independent operations |
| **Copy activity tuning** | Adjust **intelligent throughput optimization** (Fabric's equivalent of ADF DIUs) and degree of copy parallelism; use partitioned reads for large source tables |
| **Staging** | Enable staging for large cross-region copies |
| **Selective columns** | Only copy needed columns, not entire tables |
| **Incremental copy** | Use watermark/CDC (or a Copy job) instead of full copy each run |
| **Pipeline structure** | Minimize sequential dependencies; parallelize where possible |
| **Right tool** | Move pure data movement into a **Copy job**; push heavy transforms to Spark or the Warehouse rather than chaining many Dataflow steps |

---

### Optimize a Data Warehouse

| Optimization | Description |
|-------------|-------------|
| **Statistics** | Auto-created and auto-updated for the query optimizer; can also be created manually (single-column) |
| **CTAS / INSERT…SELECT** | Set-based loads instead of row-by-row or singleton inserts (avoid many small transactions → small files) |
| **Data clustering** *(preview)* | Co-locate rows on frequently filtered columns at table creation → better file/rowgroup skipping |
| **Data types** | Use the smallest appropriate types (e.g., `varchar(n)` not `varchar(max)`) |
| **Avoid SELECT \*** | Select only needed columns — reduces I/O |
| **Result set caching** | **On by default** for Warehouses and SQL endpoints; disable per item (`ALTER DATABASE … SET RESULT_SET_CACHING OFF`) or per query (`OPTION (USE HINT ('DISABLE_RESULT_SET_CACHE'))`) |
| **Cold cache awareness** | First run after idle may read from OneLake (`data_scanned_remote_storage_mb` in Query Insights) — benchmark warm runs |

> **Exam Caveat ⚠️:** Fabric Warehouse does **not** support **materialized views**, user-defined **indexes**, or table **partitioning** — use data clustering, statistics, and result-set caching instead. (Materialized views *do* exist in **Eventhouse/KQL** — see below.)

**Key DMVs for Warehouse monitoring:**

| DMV | Purpose |
|-----|---------|
| `sys.dm_exec_requests` | Currently running queries |
| `sys.dm_exec_sessions` | Active sessions |
| `queryinsights.exec_requests_history` | Historical query performance |
| `queryinsights.long_running_queries` | Queries exceeding duration threshold |
| `queryinsights.frequently_run_queries` | Most frequently executed queries |

```sql
-- Find long-running queries in Fabric Warehouse
SELECT *
FROM queryinsights.long_running_queries
ORDER BY start_time DESC;

-- Find top resource-consuming queries
SELECT
    command,
    total_elapsed_time_ms,
    data_processed_mb
FROM queryinsights.exec_requests_history
ORDER BY total_elapsed_time_ms DESC;
```

---

### Optimize Eventstream and Eventhouse

**Eventstream optimization:**

| Optimization | Description |
|-------------|-------------|
| **Partition key** | Choose a good partition key for parallel processing |
| **Batch size** | Tune ingestion batch size for throughput vs latency |
| **Schema** | Keep events lean — avoid unnecessary fields |

**Eventhouse optimization:**

| Optimization | Description |
|-------------|-------------|
| **Retention policy** | Set appropriate data retention to manage storage |
| **Caching policy** | Configure hot cache duration for frequently queried data |
| **Materialized views** | Pre-aggregate frequently queried patterns |
| **Update policy** | Transform data on ingestion (ETL at ingest time) |
| **Partitioning** | Partition by time for time-series workloads |

```kql
// Check table size and extents
.show table SensorReadings extents
| summarize ExtentCount = count(), TotalRows = sum(RowCount), TotalSize = sum(OriginalSize)

// Set hot cache to 30 days
.alter table SensorReadings policy caching hot = 30d

// Set retention to 365 days
.alter table SensorReadings policy retention softdelete = 365d

// Create a materialized view for hourly aggregates
.create materialized-view HourlySensorAgg on table SensorReadings {
    SensorReadings
    | summarize AvgTemp = avg(Temperature), MaxTemp = max(Temperature)
      by bin(Timestamp, 1h), DeviceId
}
```

---

### Optimize Spark Performance

```mermaid
flowchart TD
    SLOW["🐌 Slow Spark Job"] --> DIAG{Diagnose in\nSpark UI}
    DIAG -->|"Shuffle too high"| SHUFFLE["📦 Reduce Shuffle\n• Broadcast small tables\n• Coalesce instead of repartition\n• Avoid unnecessary groupBy"]
    DIAG -->|"Data skew"| SKEW["⚖️ Fix Data Skew\n• Salt skewed keys\n• Use adaptive query execution\n• Repartition by key"]
    DIAG -->|"Small files"| SMALL["📁 Small File Problem\n• OPTIMIZE Delta tables\n• Coalesce before write\n• Auto-optimize settings"]
    DIAG -->|"Spill to disk"| SPILL["💾 Spill to Disk\n• Increase executor memory\n• Reduce partition count\n• Broadcast small tables"]
    DIAG -->|"Slow scan"| SCAN["🔍 Slow Table Scan\n• Partition pruning\n• Z-ORDER on filter cols\n• Predicate pushdown"]
```

**Key Spark optimizations:**

| Optimization | How | When |
|-------------|-----|------|
| **Broadcast join** | `broadcast(small_df)` | Small table < 100MB joined with large table |
| **Adaptive Query Execution (AQE)** | Enabled by default in Fabric | Automatic optimization at runtime |
| **Predicate pushdown** | Filter early in the query | Always — reduces data scanned |
| **Partition pruning** | Partition data by frequently filtered columns | Date-based queries on time-partitioned tables |
| **Coalesce** | `df.coalesce(n)` before write | Reduce output file count without full shuffle |
| **Cache** | `df.cache()` or `df.persist()` | Reuse intermediate results multiple times |
| **Native execution engine** | Enable in workspace Spark settings | Supported operations get up to 4x speedup |
| **Dynamic allocation** | Auto-scale executors based on workload | Variable workloads, avoid over-provisioning |

```python
# Broadcast join for small dimension table
from pyspark.sql.functions import broadcast

df_result = df_large.join(broadcast(df_small), "customer_id")

# Coalesce to reduce output files
df_result.coalesce(10).write.format("delta").mode("overwrite").saveAsTable("output_table")

# Cache for repeated use
df_intermediate = df.filter(col("year") == 2026).cache()
count1 = df_intermediate.count()
count2 = df_intermediate.filter(col("region") == "EU").count()
```

> **Exam Caveat ⚠️:**
> - **AQE (Adaptive Query Execution)** is enabled by default in Fabric — it handles skew joins, coalescing shuffle partitions, and converting sort-merge joins to broadcast automatically
> - **`coalesce()` vs `repartition()`**: coalesce reduces partitions without a full shuffle (faster); repartition does a full shuffle (use when you need even distribution)
> - **Broadcast join** fails with OOM if the "small" table is too large — keep it under ~100MB

---

### Optimize Query Performance

**General query optimization principles:**

| Principle | Description |
|-----------|-------------|
| **Filter early** | Push WHERE clauses as early as possible |
| **Select only needed columns** | Avoid SELECT * — reduces I/O |
| **Use appropriate joins** | Broadcast small tables, hash join for large-large |
| **Leverage partitioning** | Query only relevant partitions |
| **Use statistics** | Ensure up-to-date table statistics (auto in Fabric Warehouse) |
| **Avoid UDFs** | PySpark UDFs are slow — use built-in functions |
| **Materialize** | Pre-compute expensive aggregations |

---

## 📊 Quick-Reference Scenario Table

| Scenario | Requirement | Answer |
|----------|-------------|--------|
| Many small Parquet files in Lakehouse | File compaction | **OPTIMIZE** |
| Old Delta file versions consuming storage | Storage cleanup | **VACUUM** |
| Slow filtered queries on specific columns | Query performance | **Z-ORDER BY** on those columns |
| Pipeline copy activity is slow | Throughput | **Raise intelligent throughput optimization / parallelism, partitioned reads** |
| Dataflow Gen2 runs out of memory | Memory issue | **Stage the query, then transform from staging** |
| Dataflow Gen2 bulk load with no transforms is slow | Data movement | **Fast Copy** |
| Notebook OOM error on large join | Spark memory | **Broadcast small table, increase executor memory** |
| Find long-running Warehouse queries | Diagnostics | **queryinsights.long_running_queries** |
| KQL queries slow on old data | Hot cache miss | **Extend caching policy** |
| Spark job has data skew | Uneven partitions | **Salt keys, enable AQE** |
| Eventstream backlog growing | Processing lag | **Scale processing, simplify transforms** |
| Reduce output file count from Spark | Write optimization | **coalesce() before write** |
| Slow Power BI Direct Lake reports on new Delta tables | Read speed | **Enable V-Order (`readHeavyForPBI`) + OPTIMIZE** |
| Fast ingestion matters more than read speed | Write speed | **`writeHeavy` profile (default, V-Order off)** |
| Pre-aggregate KQL query results | Repeated aggregation | **Materialized views (Eventhouse)** |
| Warehouse query scans too much data | I/O reduction | **Data clustering on filter columns, avoid SELECT \*** |
| Find which item is burning capacity | CU analysis | **Fabric Capacity Metrics app** |
| KQL-queryable run history and failure alerts for all items in a workspace | Observability | **Workspace monitoring** |

---

[← 02 — Ingest & Transform Data](/dp-700-study-notes/02-ingest-transform-data/) | [04 — Quick Reference Cheatsheet →](/dp-700-study-notes/04-quick-reference-cheatsheet/)