---
layout: default
title: "04 — Quick Reference Cheatsheet"
nav_order: 6
description: "DP-700 quick reference cheatsheet: key numbers, decision matrices, service comparisons, exam traps, common scenarios, and pre-exam checklist for the Microsoft Fabric Data Engineer Associate exam."
permalink: /04-quick-reference-cheatsheet/
mermaid: true
---

# 04 — Quick Reference Cheatsheet
> 📁 [← Back to Home](/dp-700-study-notes/)

---

## ⚡ Key Numbers to Remember

| Item | Value | Notes |
|------|-------|-------|
| Passing score | **700 / 1000** | Scaled score — not 70% |
| Exam duration | **100 minutes** | 120 min seat time total |
| Domains | **3** | Equally weighted (30–35% each) |
| VACUUM default retention | **7 days** (168 hours) | Shorter retention breaks time travel |
| Delta checkpoint interval | **Every 10 versions** | Transaction log compaction |
| Deployment pipeline stages | **Up to 10** | Typically Dev → Test → Prod |
| Broadcast join limit | **~100 MB** | Larger tables cause OOM |
| Eventhouse hot cache default | **Unlimited** *(policy-configurable)* | Set explicitly for cost control |
| Spark vCores per CU | **2** (3× burst) | F64 = 128 vCores, up to 384 with burst |
| Spark session timeout | **20 min** default | Pool deallocates 2 min after session expiry |
| Airflow starter pool idle shutdown | **20 min** | Custom pool = always on |
| Audit log (Purview, Standard) | **180 days** | Fabric activity log: 30 days |
| Monitoring hub / workspace monitoring | **30 days** | Capacity Metrics app: **14 days** |
| Shortcut cache retention | **1–28 days** | GCS, S3, S3-compatible, gateway shortcuts only |
| OneLake shortcut sources | **Internal + 9 external types** | ADLS Gen2, Blob, S3, S3-compatible, GCS, Dataverse, OneDrive/SharePoint, Iceberg, on-prem (gateway) |

---

## 🧭 Master Decision Trees

### "Where Should I Store This Data?"

```mermaid
flowchart TD
    START([Data Storage Decision]) --> Q1{Primary query\nengine?}
    Q1 -->|"T-SQL read+write\nstored procs"| WH["🏢 Warehouse"]
    Q1 -->|"Spark + SQL\nmixed workloads"| LH["🏠 Lakehouse"]
    Q1 -->|"KQL\nreal-time analytics"| EH["⚡ Eventhouse"]
    Q1 -->|"Don't copy\njust reference"| SC["🔗 Shortcut"]
    Q1 -->|"Near real-time\nreplica needed"| MR["🪞 Mirroring"]
```

### "How Should I Move This Data?"

```mermaid
flowchart TD
    START([Data Movement Decision]) --> Q1{Need?}
    Q1 -->|"Read-only replica of an\noperational database"| MR["🪞 Mirroring\n(free core replication)"]
    Q1 -->|"Bulk / incremental / CDC copy\ninto your own tables"| CJ["📋 Copy job"]
    Q1 -->|"Copy inside a larger workflow\nor metadata-driven"| CA["🔧 Pipeline Copy activity"]
    Q1 -->|"Continuous event streams"| ES["⚡ Eventstream"]
    Q1 -->|"No movement at all"| SC["🔗 Shortcut"]
```

### "How Should I Transform This Data?"

```mermaid
flowchart TD
    START([Transform Decision]) --> Q1{Who is doing it?}
    Q1 -->|"Analyst\nno-code"| DF["🔄 Dataflow Gen2\n(Power Query M)"]
    Q1 -->|"Data engineer\ncode-first"| Q2{Scale?}
    Q2 -->|"Large scale\ncomplex logic"| NB["📓 Notebook\n(PySpark)"]
    Q2 -->|"SQL preferred"| Q3{Where's the data?}
    Q3 -->|"Warehouse"| TSQL["T-SQL in Warehouse"]
    Q3 -->|"Lakehouse"| SPARKSQL["Spark SQL in Notebook"]
    Q1 -->|"Real-time\nstreaming"| Q4{Style?}
    Q4 -->|"No-code routing"| ES["⚡ Eventstream"]
    Q4 -->|"Code, joins with batch"| SSS["📓 Spark Structured Streaming"]
    Q4 -->|"KQL analytics"| KQL["KQL in Eventhouse"]
```

### "How Should I Orchestrate This?"

```mermaid
flowchart TD
    START([Orchestration Decision]) --> Q1{Single transform\nor multi-step?}
    Q1 -->|"Single transform"| Q2{Type?}
    Q2 -->|"No-code"| DF["🔄 Dataflow Gen2"]
    Q2 -->|"Code"| NB["📓 Notebook"]
    Q1 -->|"Multi-step workflow"| PIPE["🔧 Pipeline"]
    Q1 -->|"Python DAGs / Airflow"| AIR["🌬️ Apache Airflow job"]
    Q1 -->|"Notebook DAG in\none Spark session"| RM["📓 notebookutils\n.notebook.runMultiple()"]
    PIPE --> ACT["Activities:\n• Copy → Notebook → Dataflow\n• ForEach / If / Until\n• Parameters & expressions"]
```

---

## 🔒 Security Layer Reference

```mermaid
graph TD
    T["🏢 Tenant Level\n(Admin portal)"] --> W["📁 Workspace Level\n(Admin, Member, Contributor, Viewer)"]
    W --> I["📄 Item Level\n(Read, ReadAll, ReadData, Write, Reshare)"]
    I --> D["📊 Data Level"]
    D --> RLS["🔐 RLS — Row-Level Security\n(T-SQL security policy / DAX filter)"]
    D --> CLS["🔐 CLS — Column-Level Security\n(T-SQL GRANT/DENY on columns)"]
    D --> OLS["🔐 OLS — Object-Level Security\n(T-SQL grants / OneLake roles / model roles)"]
    D --> DDM["🎭 DDM — Dynamic Data Masking\n(Mask functions on columns)"]
    D --> FLS["📁 Folder/File-Level\n(OneLake security roles)"]
```

| Security Layer | Where Supported | Method |
|---------------|----------------|--------|
| **Workspace roles** | All items | Admin, Member, Contributor, Viewer |
| **Item permissions** | All items | Share dialog / API (Read, ReadData, ReadAll, Write, Reshare) |
| **RLS** | Warehouse, SQL Endpoint, Semantic Model, Lakehouse (OneLake security) | T-SQL security policy / DAX / OneLake role row filter |
| **CLS** | Warehouse, SQL Endpoint, Lakehouse (OneLake security) | T-SQL GRANT/DENY / OneLake role column *Hide* |
| **OLS** | Warehouse, SQL Endpoint, Lakehouse (OneLake security), Semantic Model | T-SQL GRANT/DENY / OneLake role table grants / model role `none` |
| **DDM** | Warehouse, SQL Endpoint | T-SQL masked column |
| **Folder/file** | Lakehouse | **OneLake security roles** (Read / ReadWrite) |

> 🔑 OneLake security roles only restrict **Viewers / item Read** users — Admin, Member, and Contributor always see all data.

---

## 🔄 Lakehouse vs Warehouse — Quick Compare

| | Lakehouse | Warehouse |
|-|-----------|-----------|
| **Engine** | Spark (primary) + SQL Endpoint (read-only data) | T-SQL (read/write) |
| **Format** | Delta Lake | Delta Lake |
| **Write via SQL** | ❌ No (SQL Endpoint is read-only for data) | ✅ Yes |
| **Views / functions / stored procs** | ✅ Yes, in the SQL Endpoint (can't modify data) | ✅ Yes |
| **Files section** | ✅ Yes (unstructured data) | ❌ No |
| **Cross-DB queries** | ✅ From SQL Endpoint (same workspace) | ✅ Native |
| **Best persona** | Data engineer, data scientist | SQL analyst, BI developer |

---

## 📦 Delta Lake Operations Cheatsheet

| Operation | Purpose | Command |
|-----------|---------|---------|
| **OPTIMIZE** | Compact small files | `OPTIMIZE table_name` |
| **Z-ORDER** | Co-locate data by columns | `OPTIMIZE table_name ZORDER BY (col)` |
| **VACUUM** | Remove old files | `VACUUM table_name RETAIN 168 HOURS` |
| **V-Order** | Read optimization (esp. Direct Lake) | **Off by default in new workspaces**; `OPTIMIZE t VORDER`, `spark.sql.parquet.vorder.default=true`, or `readHeavyForPBI` profile |
| **MERGE** | Upsert (insert/update) | Delta `MERGE INTO ... WHEN MATCHED ... WHEN NOT MATCHED` |
| **Time Travel** | Query past versions | `SELECT * FROM table VERSION AS OF 5` |
| **Schema Evolution** | Add columns on write | `.option("mergeSchema", "true")` |
| **Change Data Feed** | Stream changes from Delta | `.option("readChangeFeed", "true")` |

---

## ⚡ Spark Performance Cheatsheet

| Problem | Solution |
|---------|----------|
| Slow join (small + large table) | **Broadcast join** — `broadcast(small_df)` |
| Data skew in join/group | **Salt keys** or rely on **AQE** (auto) |
| Too many small output files | **`coalesce(n)`** before write |
| Too many shuffle partitions | AQE auto-coalesces; or set `spark.sql.shuffle.partitions` |
| Slow scan on filtered column | **Z-ORDER** on that column |
| Slow Direct Lake / SQL reads | Enable **V-Order** (off by default) + **OPTIMIZE** |
| Write-heavy ingestion | Keep **`writeHeavy`** profile (default) |
| Repeated intermediate use | **`df.cache()`** or **`df.persist()`** |
| Slow UDFs | Replace with **built-in PySpark functions** |
| Cold start delay | **Starter pools** (best-effort) or **custom live pools** (~5 sec, scheduled) |
| Need 4x faster on supported ops | **Native execution engine** |
| HTTP 430 on notebook start | Capacity Spark cores exhausted — cancel jobs, scale SKU, or retry |

---

## 🌊 Streaming Quick Reference

| Feature | Eventstream | Spark Structured Streaming | KQL |
|---------|-------------|---------------------------|-----|
| **Code** | No-code + SQL operator | PySpark | KQL |
| **Latency** | Seconds | Seconds–min | Seconds |
| **Windowing** | Tumbling, hopping, sliding (+ session, snapshot in SQL) | Tumbling, sliding, session | `bin()` + KQL window functions |
| **Joins with batch** | Limited (stream-to-stream Join) | ✅ Full | ✅ Via shortcuts/materialized views |
| **Destination** | Eventhouse, Lakehouse, Activator, custom endpoint, derived stream | Lakehouse (Delta) | KQL Database |
| **Best for** | Routing, simple transforms | Complex transforms, ML | Real-time analytics, dashboards |

---

## 🚨 Top Exam Traps

1. **Lakehouse SQL Endpoint is READ-ONLY for data** — you cannot INSERT/UPDATE/DELETE via T-SQL on a Lakehouse (use Spark or a Warehouse). It **can** still host views, functions, and stored procedures.

2. **OPTIMIZE ≠ VACUUM** — OPTIMIZE compacts files; VACUUM deletes old files. They solve different problems.

3. **VACUUM retention < 7 days breaks time travel** — the default 168 hours exists for a reason.

4. **V-Order is NOT Z-Order** — V-Order optimizes within files (Parquet layout); Z-Order optimizes across files (data co-location). And **V-Order is OFF by default** in new workspaces (`writeHeavy` profile).

5. **Mirroring ≠ Shortcuts** — Mirroring copies data (CDC-based); shortcuts point to data without copying.

6. **Git syncs definitions, NOT data** — Git integration version-controls item metadata, not the actual data in tables.

7. **Deployment pipelines deploy definitions, NOT data** — same as Git — item definitions are promoted, not data.

8. **DDM is not a security boundary** — users with UNMASK permission see the real data. Don't rely on DDM for sensitive access control.

9. **Workspace Viewer ≠ data access** — Viewer role lets you see items but may not grant access to the underlying data.

10. **Dataflow Gen2 memory/timeout issues → stage, then transform** — staging is set **per query** (not a workspace setting); pair it with Fast Copy for bulk loads.

11. **AQE is enabled by default** — don't manually tune what AQE handles automatically (skew joins, partition coalescing).

12. **Broadcast join OOM** — if the "small" table is > ~100MB, broadcast will fail. Check size first.

13. **Shortcut data is NOT affected by OPTIMIZE/VACUUM** — only native Lakehouse data is optimized.

14. **KQL query acceleration on shortcuts adds CU cost** — it caches data for performance but consumes additional capacity.

15. **Eventstream has no Warehouse destination** — destinations are Eventhouse, Lakehouse, Activator, custom endpoint, derived stream (and Spark notebook in preview).

16. **Idle Spark pools aren't billed** — starter and custom pools charge only for active sessions. (Apache Airflow **custom** pools, by contrast, are always on and always billed.)

17. **OneLake security doesn't restrict Contributors** — roles only limit Viewers / item Read users; Admin, Member, and Contributor always have full data access.

18. **Fabric Warehouse has no materialized views** — those are Eventhouse (KQL) objects. Lakehouses have **materialized lake views** (Spark SQL).

19. **Watermark incremental loads miss deletes** — use CDC (Copy job or Mirroring) when deletes must propagate.

20. **`mssparkutils` → `notebookutils`** — same API, new name; `runMultiple()` runs notebooks in parallel/as a DAG inside one Spark session.

---

## 📋 Pre-Exam Checklist

- [ ] **Lakehouse vs Warehouse** — Can you explain when to use each and the SQL Endpoint limitation?
- [ ] **Loading patterns** — Full load, incremental (watermark, CDC), MERGE for upserts?
- [ ] **Security layers** — Workspace, item, RLS, CLS, OLS, DDM, folder/file-level, **OneLake security roles**?
- [ ] **Workspace settings** — Spark pools (starter / custom / live), resource profiles, OneLake shortcut cache, **Apache Airflow starter vs custom pools**?
- [ ] **Orchestration** — Pipeline vs Dataflow Gen2 vs Notebook — when to use which? Schedules vs event triggers?
- [ ] **Data movement** — Mirroring vs Copy job vs Copy activity vs Eventstream vs Shortcut?
- [ ] **Delta Lake** — OPTIMIZE, VACUUM, Z-ORDER, V-Order (off by default!), MERGE, time travel?
- [ ] **Spark optimization** — Broadcast joins, AQE, coalesce, predicate pushdown, caching?
- [ ] **Streaming** — Eventstream vs Spark Structured Streaming vs KQL, windowing types?
- [ ] **Mirroring vs Shortcuts** — Copy vs point, CDC vs reference, latency trade-offs?
- [ ] **Lifecycle management** — Git integration, deployment pipelines, database projects, variable libraries?
- [ ] **Monitoring** — Monitoring Hub, workspace monitoring, Capacity Metrics app, Spark UI tabs, DMVs, query insights?
- [ ] **Error resolution** — Common errors in pipelines, notebooks, Dataflow Gen2, T-SQL?
- [ ] **KQL basics** — `summarize`, `bin()`, `ago()`, materialized views, caching policy?

---

## 📊 Full Scenario Decision Table

| Scenario | Key Constraint | Answer |
|----------|---------------|--------|
| SQL analysts write stored procedures | T-SQL read/write | **Warehouse** |
| Data engineers use PySpark + SQL | Mixed engine | **Lakehouse** |
| Access ADLS Gen2 without copying | No data movement | **OneLake Shortcut** |
| Near real-time replica of Azure SQL | Automatic CDC | **Mirroring** |
| No-code CSV-to-table transform | Citizen developer | **Dataflow Gen2** |
| Complex PySpark ETL on 500M rows | Code-first, large scale | **Notebook** |
| Multi-step: Copy → Transform → Refresh | Orchestration | **Pipeline** |
| Incremental / CDC copy with no pipeline | Managed data movement | **Copy job** |
| Production Airflow DAGs on a 24/7 schedule | Always-on Airflow | **Apache Airflow custom pool** |
| Viewer sees only some lakehouse rows/columns everywhere | Data-plane security | **OneLake security role (RLS/CLS)** |
| Stage-specific connection values across Dev/Test/Prod | CI/CD config | **Variable library** |
| Real-time IoT event routing | Stream processing | **Eventstream → Eventhouse** |
| 5-min tumbling window aggregation | Time-series | **KQL `bin()` or Spark `window()`** |
| Upsert changed rows into Delta | Incremental load | **Delta MERGE** |
| Compact small files for read perf | File management | **OPTIMIZE** |
| Remove old unreferenced files | Storage cleanup | **VACUUM** |
| Speed up filtered queries | Query optimization | **Z-ORDER BY** |
| Restrict rows by user's region | Data security | **RLS (T-SQL security policy)** |
| Hide column from specific users | Column security | **CLS (T-SQL GRANT/DENY)** |
| Mask email in query output | Data masking | **DDM** |
| Promote Dev → Test → Prod | CI/CD | **Deployment Pipelines** |
| Version-control notebooks | Source control | **Git Integration** |
| Slow Dataflow Gen2, OOM | Memory issue | **Stage the query, transform from staging** |
| Spark join slow, small + large table | Join optimization | **Broadcast join** |
| Too many small output files | Write optimization | **coalesce() before write** |
| Slow KQL on historical data | Cache miss | **Extend caching policy / query acceleration** |

---

[← 03 — Monitor & Optimize an Analytics Solution](/dp-700-study-notes/03-monitor-optimize-analytics-solution/) | [Back to Home →](/dp-700-study-notes/)