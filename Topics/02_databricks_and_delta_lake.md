# 02 — Azure Databricks Platform & Delta Lake (highest-weight topic — study this hardest)

---

## 1. Databricks Workspace & Architecture

### Quick Theory
Databricks is a **unified analytics platform** built around Apache Spark, split into a **Control Plane** (managed by Databricks — UI, job scheduler, notebooks, cluster manager) and a **Data Plane** (runs in your Azure subscription — the actual VMs/clusters that process your data). Your data never leaves your data plane unless you explicitly move it.

- **Why used**: combines data engineering, data science, and analytics in one collaborative platform with notebooks, managed Spark clusters, and governance (Unity Catalog) built in.
- **Where used**: ETL pipelines, streaming, ML training, BI serving layers.
- **Benefits**: no cluster ops overhead, autoscaling, collaborative notebooks, native Delta Lake, strong Azure integration.
- **Limitations**: cost can spiral without cluster policies; vendor-specific features (Photon, DLT) create some lock-in.

### Real-Life Analogy
Databricks is like an **airport**: the Control Plane is air-traffic control (schedules flights/jobs, manages logistics) sitting in a central tower, while the Data Plane is the actual planes and runways (clusters) that live at your airport (your Azure subscription) and do the real flying (data processing).

### Technical Explanation
```
        DATABRICKS CONTROL PLANE (Databricks-managed)
   ┌───────────────────────────────────────────────┐
   │  Web UI · Notebooks · Job Scheduler · REST API │
   │  Unity Catalog metadata · Cluster Manager       │
   └───────────────────┬───────────────────────────┘
                        │ orchestrates
                        ▼
        YOUR AZURE SUBSCRIPTION (Data Plane)
   ┌───────────────────────────────────────────────┐
   │  VMs (Driver + Executors) in your VNet          │
   │  Reads/writes ADLS Gen2 directly                │
   └───────────────────────────────────────────────┘
```

### Interview Q&A
1. *What's the difference between Control Plane and Data Plane in Databricks?* — Control Plane is Databricks-managed infrastructure (UI, scheduling, notebooks, metadata); Data Plane is the compute (clusters/VMs) that runs in your own Azure subscription and actually touches your data, so your data stays within your network boundary.
2. *Why would a company choose Databricks over plain open-source Spark on Azure HDInsight/AKS?* — Managed cluster lifecycle, autoscaling, collaborative notebooks, built-in governance (Unity Catalog), Delta Lake/Photon performance, and tight Azure AD/DevOps integration — reduces operational burden versus self-managing Spark.
3. *What are the main personas Databricks serves and how does that show up in the product?* — Data engineers (Jobs, Workflows, Delta Live Tables), data scientists/ML engineers (MLflow, notebooks), and analysts (SQL warehouses/Databricks SQL) — reflected in the different "personas" toggle in the workspace UI.

### One-Line Revision
**Databricks Control Plane schedules and coordinates; the Data Plane in your Azure subscription does the actual data work.**

---

## 2. Clusters, Pools & Autoscaling

### Quick Theory
A **cluster** is a set of VMs (1 driver + N executors) that runs your Spark code. **Pools** are pre-warmed sets of idle VMs that clusters can grab instantly instead of waiting for Azure to provision new ones (cuts cluster startup time from minutes to seconds). **Autoscaling** adds/removes executor nodes automatically based on workload.

- **Job clusters**: created for a single job run, then terminated — cheaper, isolated, best practice for production ETL.
- **All-purpose/interactive clusters**: long-running, shared, used for development/notebooks — more expensive if left idle.

### Real-Life Analogy
A cluster is a **restaurant kitchen crew** for the night. A pool is like having a few chefs already in uniform and warmed up in the break room, so when a rush order comes in, they jump straight to the line instead of clocking in from scratch. Autoscaling is calling in extra cooks only when the dinner rush hits, and sending them home once it's quiet.

### Technical Explanation
- Cluster config choices: instance type (memory-optimized vs compute-optimized vs storage-optimized), Spark version/runtime (DBR — Databricks Runtime), autoscaling min/max workers, spot vs on-demand instances (spot = cheaper but can be reclaimed by Azure).
- **Cluster policies**: admin-defined templates that restrict what users can configure (e.g., max node count, allowed instance types) — critical for cost governance in an enterprise.
- Common mistake: using one big all-purpose cluster shared by the whole team for production jobs — creates noisy-neighbor problems and makes cost attribution impossible. Best practice: **job clusters per pipeline**.

### Interview Q&A
1. *Job cluster vs all-purpose cluster — which for production ETL and why?* — Job clusters, because they spin up only for the duration of the job and terminate automatically, giving cost isolation, no "noisy neighbor" issues from shared interactive use, and cleaner cost attribution per pipeline.
2. *How does a Pool reduce cluster startup latency?* — It keeps a set of idle, pre-provisioned VM instances ready; when a cluster requests nodes, it "checks out" from the pool instead of waiting on Azure to allocate and boot new VMs, cutting startup from minutes to seconds.
3. *What's the risk of using Spot/low-priority instances for a production trading pipeline?* — Azure can reclaim spot capacity with short notice when demand rises, causing executor loss mid-job; acceptable for fault-tolerant, restartable batch jobs, but risky for latency-sensitive streaming jobs unless you architect for graceful recovery (checkpointing).
4. *How do you decide min/max workers for autoscaling?* — Based on typical vs peak data volume and job SLAs — set min high enough to avoid slow scale-up mid-job for steady workloads, and max capped to control cost, informed by monitoring actual executor utilization in the Spark UI over time.
5. *What's a cluster policy and why would an enterprise enforce one?* — An admin-controlled template restricting allowed cluster configurations (instance types, max size, auto-termination settings) to prevent cost overruns and enforce security/compliance standards across all teams.
6. *A cluster keeps failing to start. What do you check?* — Quota limits for the VM family/region in the subscription, NSG/network rules blocking control-plane communication, invalid init scripts, or an incompatible library installed at cluster scope.

### One-Line Revision
**Clusters do the work, pools make them start fast, autoscaling matches muscle to workload, and job clusters are the production default.**

---

## 3. Jobs & Workflows

### Quick Theory
**Jobs** in Databricks are scheduled/triggered executions of a notebook, JAR, Python script, or SQL. **Workflows** (formerly "Jobs" in newer UI terminology, now more powerful) let you chain multiple tasks with dependencies, conditional logic, and retries — essentially Databricks' native orchestrator, competing with Azure Data Factory for pipeline scheduling.

### Real-Life Analogy
A single Job is one **relay runner**; a Workflow is the **whole relay race** — defining who runs after whom, what happens if a runner drops the baton (task failure → retry/alert), and the finish-line conditions.

### Technical Explanation
```
Workflow: "daily_trading_pipeline"
   Task 1: ingest_raw  ──► Task 2: clean_bronze_to_silver ──► Task 3: aggregate_silver_to_gold
                                                                       │
                                                          Task 4: notify_on_failure (conditional)
```
- Supports task dependencies (`depends_on`), retries with backoff, timeout, and triggering by schedule (CRON), file arrival, or another job's completion.
- Alerts can be wired to email/Slack/Teams on failure or duration threshold breach.

### Interview Q&A
1. *How do you handle task failure in a multi-step Workflow?* — Configure per-task retry policies (count + backoff), set a timeout, and add a conditional "on failure" task (e.g., send alert, run a cleanup/rollback task) so failures are handled gracefully instead of leaving partial state.
2. *Databricks Workflows vs Azure Data Factory — when would you use each?* — Workflows are ideal when the entire pipeline logic lives in Databricks/Spark/Delta and you want native task chaining without leaving the platform; ADF is preferred for broader orchestration across many non-Databricks Azure services (Copy Activity from on-prem SQL, multiple heterogeneous systems) or when a company standardizes orchestration in ADF and calls Databricks notebooks as one activity within a larger ADF pipeline.
3. *How would you trigger a Workflow automatically when a new file lands in ADLS?* — Use file arrival triggers (available natively in Databricks Workflows) or an Event Grid/Azure Function listening to Blob-created events that calls the Jobs API to start the run.

### One-Line Revision
**Workflows chain Jobs into a dependable, retry-aware pipeline — Databricks' built-in orchestrator.**

---

## 4. Unity Catalog

### Quick Theory
Unity Catalog is Databricks' unified governance layer providing a **three-level namespace** (`catalog.schema.table`), fine-grained access control (table/column/row level), data lineage, and audit logging — across all workspaces in an account, instead of siloed per-workspace permissions.

### Real-Life Analogy
Before Unity Catalog, each Databricks workspace was like a **separate branch bank with its own vault and rules**. Unity Catalog is like the **head office's central ledger and security system** — one place that governs who can see what, across every branch, with a full audit trail.

### Technical Explanation
```
Metastore (account-level)
   └── Catalog: "trading_prod"
         └── Schema: "silver"
               └── Table: "orders"
                     Grants: SELECT to role "data_analysts"
                             MODIFY to role "data_engineers"
```
- Storage credentials + external locations connect Unity Catalog to ADLS Gen2 using Managed Identity/Access Connector (no per-user keys).
- Provides **data lineage** automatically (which notebook/job read/wrote which table) and **row/column-level security** via dynamic views or attribute-based access control.

### Interview Q&A
1. *What problem does Unity Catalog solve that plain Hive Metastore didn't?* — Hive Metastore was scoped per-workspace with coarser permissions; Unity Catalog gives one account-wide governance layer with fine-grained (table/row/column) access control, cross-workspace data sharing, and built-in lineage/audit — critical for multi-team, compliance-heavy environments like trading platforms.
2. *How would you restrict a junior analyst to see only non-PII columns of a customer table?* — Create a dynamic view in Unity Catalog that masks/excludes PII columns based on the querying user's group membership, and grant the analyst SELECT only on the view, not the base table.
3. *What is data lineage and why does a SaaS trading platform care about it?* — Automatic tracking of which tables/notebooks/jobs produced or consumed a given dataset; critical for auditing "where did this number in the client's report come from" and for impact analysis before changing a shared table's schema.

### One-Line Revision
**Unity Catalog is the account-wide security guard and record-keeper for every table across every workspace.**

---

## 5. Notebooks & Repos

### Quick Theory
Notebooks are Databricks' interactive coding environment (supports Python/SQL/Scala/R in one notebook via `%sql`, `%python` magic commands). **Repos** integrate Git (Azure DevOps, GitHub) directly into the workspace so notebooks are version-controlled like real code, not just autosaved files.

### Interview Q&A
1. *Why are Repos important for a production data engineering team, versus just saving notebooks in the workspace?* — Repos bring proper Git workflows (branching, PRs, code review, CI/CD triggers) to Databricks code, which is essential for production reliability, rollback capability, and collaboration — plain workspace notebooks have only basic revision history, not real version control.
2. *How do you structure a notebook-based project for CI/CD with Azure DevOps?* — Keep notebooks in a Repo backed by an Azure DevOps repo, use branches per feature, open PRs for review, and have the DevOps pipeline run tests (e.g., via `dbx` or Databricks CLI/REST API) and deploy notebooks/jobs to higher environments on merge.
3. *What's a common mistake teams make when going from notebooks to production?* — Leaving hardcoded paths/parameters and manual "Run All" workflows instead of parameterizing notebooks (via widgets/job parameters) and orchestrating them through Jobs/Workflows with proper environment configs (dev/test/prod).

### One-Line Revision
**Notebooks are where you write and explore; Repos make sure that work is real, versioned, reviewable code.**

---

## 6. MLflow (basics — nice-to-have per JD)

### Quick Theory
MLflow is an open-source platform (created by Databricks) for managing the ML lifecycle: experiment tracking, model packaging, model registry, and deployment. Even in a mostly-ETL role, it may come up if the "AI-based analytics" nice-to-have is probed.

### Interview Q&A
1. *What does MLflow Tracking do?* — Logs parameters, metrics, and artifacts for each model training run, so you can compare experiments and reproduce results later.
2. *What's the Model Registry for?* — A centralized store for versioned models with stage transitions (Staging → Production → Archived) and approval workflows, decoupling model training from deployment.

### One-Line Revision
**MLflow tracks experiments and manages model versions the way Git manages code versions.**

---

## 7. Photon

### Quick Theory
Photon is Databricks' native, vectorized query engine (written in C++) that replaces the JVM-based Spark execution engine for many SQL/DataFrame operations, giving significant speedups (often 2–8x) with no code changes.

### Interview Q&A
1. *What is Photon and do you need to change your code to use it?* — A native vectorized execution engine that accelerates SQL and DataFrame operations; you simply enable it at the cluster/SQL-warehouse level — no code rewrite needed since it operates under the same Spark API.
2. *When would Photon not help much?* — Workloads dominated by custom Python UDFs or RDD-level operations that fall outside Photon's supported operator set won't see much benefit since execution falls back to the regular JVM engine for those parts.

### One-Line Revision
**Photon is a faster engine under the same steering wheel — turn it on, don't rewrite your code.**

---

## 8. DBFS (Databricks File System)

### Quick Theory
DBFS is a distributed file system abstraction layered over cloud storage (ADLS Gen2 etc.), mounted into the workspace so notebooks can use simple paths (`/dbfs/...` or `dbfs:/...`) instead of full cloud URIs. Modern best practice (with Unity Catalog) favors direct cloud paths/Volumes over legacy DBFS mounts for governance reasons.

### Interview Q&A
1. *Why is DBFS root not recommended for production data storage today?* — It's a workspace-level, less-governed storage location without Unity Catalog's fine-grained access control and lineage — production data should live in governed external locations/Volumes instead.
2. *What replaced DBFS mounts in the Unity Catalog world?* — Unity Catalog **Volumes** and **external locations**, which apply the same governed access control as tables to non-tabular files.

### One-Line Revision
**DBFS was the old convenient shortcut; Unity Catalog Volumes are the governed, modern replacement.**

---

## 9. Delta Lake — the heart of this JD

### Quick Theory
Delta Lake is an open-source storage layer that adds **ACID transactions, schema enforcement, time travel, and performance optimizations** on top of plain Parquet files sitting in ADLS Gen2. It turns a "data lake" (files) into a "Lakehouse" (files that behave like a reliable database table).

- **Why used**: Plain Parquet has no transaction guarantees — concurrent writers can corrupt data, readers can see partial writes, there's no easy way to update/delete individual rows.
- **Where used**: every layer of the medallion architecture (Bronze/Silver/Gold tables).
- **Benefits**: ACID safety, time travel (query historical versions), efficient upserts (`MERGE`), file compaction (`OPTIMIZE`), fast data skipping (`Z-ORDER`).
- **Limitations**: still batch/micro-batch oriented under the hood even for "streaming"; small-file and metadata overhead if not maintained (`VACUUM`/`OPTIMIZE` need to be scheduled).

### Real-Life Analogy
Plain Parquet files in a data lake are like a **library where anyone can add or remove books from the shelves at any time, with no librarian** — you might read a shelf mid-reorganization and get a nonsensical partial result. Delta Lake adds the **librarian and a logbook**: every change is recorded in order (`_delta_log`), readers always see a consistent, complete snapshot, and you can ask "show me the shelf exactly as it looked yesterday at 3pm" (time travel).

### Technical Explanation
```
/gold/orders/
   ├── part-0001.snappy.parquet
   ├── part-0002.snappy.parquet
   └── _delta_log/
         ├── 00000000000000000000.json   ← version 0 (initial write)
         ├── 00000000000000000001.json   ← version 1 (a MERGE)
         └── 00000000000000000002.json   ← version 2 (an OPTIMIZE)
```
- Every write creates a new JSON (and periodically Parquet "checkpoint") entry in `_delta_log` describing which files are now part of the table version — this log IS the transaction mechanism providing ACID.
- **MERGE (upsert)**: match on a key, update existing rows, insert new ones — the backbone of CDC/incremental pipelines.
- **OPTIMIZE**: compacts small files into larger ones for faster reads.
- **Z-ORDER**: co-locates related data within files by column(s) so queries filtering on that column skip more files (data skipping).
- **VACUUM**: physically deletes old, no-longer-referenced data files past a retention window (default 7 days) — needed to actually reclaim storage after `OPTIMIZE`/deletes, but breaks time travel beyond the retention period if run too aggressively.
- **Time Travel**: `SELECT * FROM table VERSION AS OF 5` or `TIMESTAMP AS OF '2026-07-01'` — query any historical version directly from the log.

### Interview Q&A

**Beginner**
1. *What is Delta Lake in one sentence?* — An open storage format that adds ACID transactions, schema enforcement, and time travel on top of Parquet files in a data lake.
2. *How does Delta Lake achieve ACID transactions on plain files?* — Through the `_delta_log` transaction log — every write is recorded as an atomic, ordered JSON commit describing exactly which files are added/removed, and readers only see files referenced by a fully-committed log entry.
3. *What is Time Travel and why is it useful?* — The ability to query a table as it existed at a past version or timestamp; useful for auditing ("what did this look like before the bad load"), reproducing reports, and recovering from accidental bad writes by rolling back.

**Intermediate**
4. *Explain MERGE (upsert) with an example use case.* — `MERGE INTO target USING source ON target.id = source.id WHEN MATCHED THEN UPDATE ... WHEN NOT MATCHED THEN INSERT ...` — used for CDC-style incremental loads where a daily batch of changed trading orders needs to update existing rows and insert new ones without duplicating.
5. *What does OPTIMIZE do and when should you run it?* — Compacts many small files into fewer, larger ones to speed up reads; run it periodically (e.g., nightly) especially after high-frequency streaming writes that produce many small files.
6. *What does Z-ORDER do and how do you choose the column?* — Physically clusters data within files by the specified column(s) so range/equality filters on that column can skip more files entirely; choose high-cardinality columns commonly used in WHERE clauses (e.g., `trade_date`, `client_id`).
7. *What's the risk of running VACUUM with a very short retention period?* — It permanently deletes files still referenced by recent table versions, breaking time travel to those versions and potentially causing "file not found" errors for any long-running query or downstream job still reading an older snapshot.

**Advanced**
8. *How does Delta Lake handle concurrent writers?* — Via optimistic concurrency control — each writer reads the current log version, prepares its change, and attempts to commit the next version number; if another writer already committed that version, the second writer retries by re-checking for conflicts (row-level conflict detection varies by operation type).
9. *What's schema enforcement vs schema evolution in Delta Lake?* — Schema enforcement rejects writes that don't match the table's existing schema (prevents accidental corruption); schema evolution (`mergeSchema` option) explicitly allows new columns to be added when writing, for controlled, intentional schema changes.
10. *How would you implement SCD Type 2 using Delta Lake MERGE?* — Use MERGE with conditions that, on a matched record with changed attributes, expire the old row (set `end_date`, `is_current = false`) and insert a new row with the updated attributes and `is_current = true`, all within one atomic MERGE statement using `WHEN MATCHED ... THEN UPDATE` and `WHEN NOT MATCHED ... THEN INSERT`.
11. *What's a Delta table "checkpoint" file and why does it exist?* — Periodically (every 10 commits by default), Delta writes a Parquet checkpoint summarizing the full table state, so readers don't need to replay every single JSON log entry from version 0 — speeds up log reconstruction for tables with long histories.

**Scenario / Debugging**
12. *A Delta MERGE job is taking much longer than usual. What do you check?* — Whether the source (incoming changes) has grown unexpectedly large, whether the target table has too many small files (needs OPTIMIZE), whether the join/match condition lacks a good partition/Z-order alignment causing a full table scan, and whether concurrent writers are causing conflict retries.
13. *A downstream consumer complains data "disappeared" after you ran VACUUM. What happened and how do you prevent it in future?* — VACUUM deleted files still needed by a version the consumer's long-running query or cached read was referencing (beyond default 7-day retention, or a shorter interval was configured); prevent by keeping retention adequate for the slowest downstream consumer and coordinating VACUUM scheduling with consumer SLAs.
14. *How would you detect and handle duplicate records arriving in a Bronze-to-Silver load?* — Use MERGE keyed on a natural/business key (or a hash of key columns) with `WHEN NOT MATCHED THEN INSERT` only, or deduplicate the incoming micro-batch first with `dropDuplicates`/window-function row-numbering before the merge, especially important with at-least-once delivery from upstream messaging systems.
15. *Table is corrupted / a bad batch got written to Gold — how do you recover?* — Use Time Travel to identify the last known-good version, then either restore the whole table (`RESTORE TABLE ... TO VERSION AS OF n`) or read that version and selectively re-merge/backfill affected records, then investigate the pipeline for the root cause before re-enabling it.

### Practical Examples
- **Trading platform**: Gold-layer `daily_positions` table uses MERGE nightly to apply position changes, Z-ordered on `account_id` since that's the most common filter in downstream reports.
- **Banking**: SCD Type 2 on `customer_profile` table to preserve full history of address/KYC changes for audit.
- **Retail**: OPTIMIZE + Z-ORDER on `sales_transactions` by `store_id, sale_date` for fast BI dashboard queries.

### Hands-on Example
```python
# MERGE (upsert) example — incremental load of trading orders into Silver
from delta.tables import DeltaTable

silver_table = DeltaTable.forName(spark, "trading_prod.silver.orders")

silver_table.alias("target").merge(
    source=incoming_orders_df.alias("source"),
    condition="target.order_id = source.order_id"
).whenMatchedUpdateAll(
    condition="source.last_updated > target.last_updated"   # only update if source is newer
).whenNotMatchedInsertAll(
).execute()
```
- `DeltaTable.forName` loads the existing Delta table as a mergeable object.
- `.merge(source, condition)` defines the join key between incoming and existing data.
- `whenMatchedUpdateAll(condition=...)` updates all columns only if the incoming row is actually newer — prevents out-of-order/late data from overwriting fresher records.
- `whenNotMatchedInsertAll()` inserts brand-new order IDs not yet in the table.
- `.execute()` runs it as one atomic transaction, logged in `_delta_log`.

```sql
-- Time travel: compare today's totals against yesterday's snapshot for a data quality check
SELECT SUM(amount) FROM trading_prod.gold.daily_pnl VERSION AS OF 42;
SELECT SUM(amount) FROM trading_prod.gold.daily_pnl; -- current version
```

```sql
-- Maintenance job run nightly
OPTIMIZE trading_prod.silver.orders ZORDER BY (account_id, trade_date);
VACUUM trading_prod.silver.orders RETAIN 168 HOURS; -- 7 days, the safe default
```

### Common Interview Mistakes
- Saying "Delta Lake is just Parquet" — it's Parquet **plus the transaction log**; that log is the entire point.
- Confusing OPTIMIZE (compaction) with VACUUM (deletion) — they solve different problems and get asked about together often.
- Not mentioning idempotency/late-data handling when asked about MERGE-based pipelines — interviewers probe this heavily for financial data.
- Forgetting that VACUUM breaks time travel beyond retention — a very common "gotcha" follow-up.

### Follow-up Questions (sample chain)
- Q: "How does MERGE guarantee no duplicates?" → A: "It doesn't automatically — the merge condition (matching key) and whether you dedupe the source batch first is what guarantees it; MERGE just gives you atomic update/insert semantics."
- Follow-up: "What if the same order_id appears twice in the source batch itself?" → A: "MERGE will fail or produce ambiguous results if multiple source rows match one target row in some engines' strict mode — best practice is to dedupe/window-function the source batch to one row per key before merging."

### One-Line Revision
**Delta Lake = Parquet files + a transaction log, turning your data lake into a database with ACID, time travel, and safe upserts.**

---

## 10. Medallion Architecture (Bronze / Silver / Gold)

### Quick Theory
A layered data design pattern: **Bronze** (raw, as-is ingested data, append-only), **Silver** (cleaned, deduplicated, conformed, joined data), **Gold** (business-level aggregates ready for reporting/ML). Each layer is a Delta table; quality and structure increase as data moves up.

### Real-Life Analogy
Like **refining crude oil**: Bronze is crude oil straight from the well (unrefined, exactly as extracted); Silver is refined fuel (cleaned, standardized, usable); Gold is the specific blended product sold at the pump (ready for the end customer — a dashboard, a report, a downstream system).

### Technical Explanation
```
Upstream Systems
      │  (raw files/streams, exactly as received — no transformation)
      ▼
  BRONZE  (append-only, schema-on-read, keeps raw history for replay/audit)
      │  (dedupe, clean nulls/types, apply schema, join reference data)
      ▼
  SILVER  (conformed, queryable, business-entity level — e.g., one row per order)
      │  (aggregate, business logic, star-schema/denormalize for BI)
      ▼
  GOLD    (curated, aggregated — e.g., daily P&L per account, ready for Power BI)
```
- Best practice: Bronze should be as close to lossless/raw as possible — you want to be able to reprocess Silver/Gold from Bronze if logic changes, without re-pulling from the (possibly unavailable) original source.
- Common mistake: doing heavy business logic directly in Bronze, which then can't be safely replayed/reprocessed for new requirements.

### Interview Q&A
1. *Why keep raw data in Bronze instead of transforming immediately?* — It acts as a durable, replayable source of truth — if transformation logic in Silver/Gold has a bug or requirements change, you can reprocess from Bronze without needing to re-fetch from a possibly-unavailable or rate-limited upstream source.
2. *What kind of transformations belong in Silver vs Gold?* — Silver handles data quality: deduplication, type casting, null handling, joining reference/dimension data, standardizing schema; Gold handles business aggregation: rollups, KPIs, denormalized tables tailored to specific reporting/dashboard needs.
3. *How does Medallion Architecture support reprocessing after a bug is found in transformation logic?* — Since Bronze retains raw untransformed data, you can simply re-run the (fixed) Silver/Gold transformation logic against existing Bronze data — no need to re-ingest from source systems.
4. *In this JD's trading platform, what might a Gold table look like?* — Something like `daily_account_pnl` or `client_position_summary` — aggregated, denormalized, and directly queryable by the client-facing reporting layer or Power BI without further joins.

### One-Line Revision
**Bronze is raw truth you can always replay from, Silver is clean and trustworthy, Gold is business-ready.**

---

Next file: `03_pyspark_and_sql.md`
