# 04 — ETL/ELT Patterns & End-to-End Architecture

---

## 1. ETL vs ELT

### Quick Theory
**ETL** (Extract, Transform, Load): transform data *before* loading into the target system — traditional pattern when target compute was expensive/limited. **ELT** (Extract, Load, Transform): load raw data first, transform *inside* the powerful target system (like Databricks/Spark) — the modern Lakehouse pattern, since compute is now cheap and elastic and you want to preserve raw data for reprocessing.

### Real-Life Analogy
ETL is like **washing and chopping vegetables before they enter the kitchen** (at the delivery truck). ELT is **bringing the whole crate of raw vegetables into the kitchen first**, then washing/chopping there, using the kitchen's own equipment — more flexible if the recipe changes later, since you still have the raw ingredients on hand.

### Interview Q&A
1. *Why has ELT become more popular than ETL in modern Lakehouse architectures?* — Because storage is cheap and Spark/Databricks compute is elastic and powerful, it's more flexible to load raw data first (preserving it for reprocessing) and transform it in-platform, rather than transforming before load and losing access to the untransformed original.
2. *Does this JD's architecture use ETL or ELT?* — ELT — the Medallion architecture explicitly loads raw data into Bronze first (Extract+Load), then transforms it into Silver/Gold within Databricks (Transform), which is the textbook definition of ELT.

### One-Line Revision
**ETL transforms before loading; ELT loads raw first and transforms inside the powerful target — Lakehouses are ELT.**

---

## 2. Incremental Load vs Full Load

### Quick Theory
**Full load**: reprocess/reload the entire source dataset every run — simple but wasteful and slow at scale. **Incremental load**: only process new/changed records since the last successful run — efficient, but requires tracking "what's new" (via watermarks, CDC, or high-water-mark columns).

### Interview Q&A
1. *When would you still choose a full load over incremental, even though it's less efficient?* — For small reference/dimension tables where the overhead of tracking incremental state isn't worth it, or when source data doesn't reliably expose a change-tracking column/CDC feed, or for periodic full reconciliation to catch any incremental-logic bugs/missed records.
2. *What's needed to safely implement incremental load?* — A reliable way to identify "what changed" — typically a `last_modified`/`updated_at` timestamp column, a CDC feed, or an auto-incrementing ID — plus a durable record of the last successfully processed watermark, stored somewhere the pipeline can read on its next run.
3. *What's the danger of incremental load if the watermark tracking has a bug?* — Silent data loss (records skipped forever) or duplicate processing — because incremental pipelines don't naturally self-correct the way a full reload does, bugs can go unnoticed far longer; periodic full reconciliation runs help catch this.

### One-Line Revision
**Full load = reread everything, safe but slow; incremental load = read only what changed, fast but needs a trustworthy watermark.**

---

## 3. CDC (Change Data Capture)

### Quick Theory
CDC captures **row-level inserts/updates/deletes** from a source system as they happen (often via database transaction logs, e.g., SQL Server CDC/CT, Debezium for MySQL/Postgres) and streams them downstream — enabling near-real-time incremental pipelines without expensive full-table scans or comparisons.

### Real-Life Analogy
Like a **security camera recording every time someone enters/exits/moves an item in a warehouse**, versus walking the whole warehouse every hour to manually spot what changed. CDC is the camera feed; full-table comparison is the manual walk-through.

### Technical Explanation
```
Source DB transaction log ──► CDC tool (Debezium / native CDC) ──► Event stream (Kafka/Event Hub)
                                                                          │
                                                                          ▼
                                                          Databricks Structured Streaming
                                                                          │
                                                                          ▼
                                                          Delta Lake MERGE (upsert) into Silver
```

### Interview Q&A
1. *How is CDC different from just comparing snapshots to find changes?* — CDC reads changes directly from the source's transaction log as they happen (efficient, near-real-time, captures deletes too), while snapshot comparison requires pulling and diffing full datasets repeatedly — much heavier and can miss intermediate changes/deletes between snapshots.
2. *How would you apply CDC events (insert/update/delete) into a Delta table?* — With a MERGE statement handling all three operation types: `WHEN MATCHED AND op='D' THEN DELETE`, `WHEN MATCHED AND op='U' THEN UPDATE`, `WHEN NOT MATCHED AND op='I' THEN INSERT` — driven by an operation-type column in the CDC event payload.
3. *What challenge does out-of-order CDC event delivery create, and how do you handle it?* — A later-arriving "insert" event could arrive after its corresponding "update," causing the update to be lost if applied naively; handle it by comparing event timestamps/sequence numbers in the MERGE condition so only strictly-newer events are applied (similar to the `whenMatchedUpdateAll(condition="source.updated_at > target.updated_at")` pattern).

### One-Line Revision
**CDC streams every row-level change directly from the source's log — no need to compare whole snapshots.**

---

## 4. SCD Type 1 & Type 2

### Quick Theory
**SCD (Slowly Changing Dimension)** describes how to handle changes to dimension attributes over time:
- **Type 1**: overwrite the old value — no history kept (e.g., correcting a typo in a customer's name).
- **Type 2**: keep full history — insert a new row for each change, with `effective_date`/`end_date`/`is_current` flags, preserving what the value *was* at any point in time.

### Real-Life Analogy
Type 1 is **editing a Wikipedia page in place** — old wording is gone. Type 2 is **Wikipedia's full revision history** — you can see every past version and exactly when it changed, important for "what did we know/have on file at the time of this trade" audit questions.

### Interview Q&A
1. *Why would a trading platform need SCD Type 2 instead of Type 1 for client account details?* — Regulatory/audit requirements often need to know what account details (address, risk tier, KYC status) were in effect *at the time* a specific trade occurred — Type 1 destroys that history, Type 2 preserves it.
2. *Write the logic (conceptually) for implementing SCD Type 2 with a Delta MERGE.* — On a matched record with changed attributes: expire the old row (`end_date = current_date`, `is_current = false`); then insert a new row with the new attribute values (`effective_date = current_date`, `end_date = null`, `is_current = true`) — often done as a two-step MERGE or a single MERGE with `WHEN MATCHED THEN UPDATE` (to expire) followed by a separate `INSERT` of new versions using a staging/source-derived dataset.
3. *What's the storage trade-off of Type 2 vs Type 1?* — Type 2 grows the table over time (one row per historical version per entity) versus Type 1's constant row count — acceptable trade-off for compliance-critical dimensions, wasteful for high-churn low-value attributes.

### One-Line Revision
**Type 1 overwrites and forgets; Type 2 remembers everything with a timeline of versions.**

---

## 5. Watermarking (Streaming)

### Quick Theory
In streaming (Structured Streaming), a **watermark** tells Spark how long to wait for late-arriving data before considering a time window "final" and discarding state for it — balances correctness (waiting for stragglers) against unbounded memory growth (state can't be kept forever).

### Interview Q&A
1. *Why does Structured Streaming need watermarks at all?* — Streaming aggregations (e.g., windowed counts) must eventually finalize and emit results, but events can arrive late (network delays, upstream buffering); a watermark defines an acceptable lateness threshold so Spark knows when it's safe to close a window and free its state, instead of holding state forever.
2. *What happens to data that arrives later than the watermark threshold?* — It's dropped from that windowed aggregation (the state for that window has already been finalized/cleared) — this is a deliberate trade-off between correctness and resource usage, and must be an accepted business risk or handled by a separate late-data path.
3. *How would you set a watermark for trading tick data with occasional 5-minute-delayed feeds?* — Set the watermark threshold slightly above the expected max delay (e.g., `withWatermark("event_time", "10 minutes")`) to safely capture the known 5-minute delay pattern with margin, balanced against how much extra memory/latency that buffer costs.

### One-Line Revision
**A watermark is streaming's "how long do we wait for stragglers before closing the books" rule.**

---

## 6. Idempotency

### Quick Theory
An **idempotent** pipeline produces the *same end result* no matter how many times it's re-run with the same input — critical for safe retries after failures (a very common real-world need, since jobs *will* fail and need to be re-run).

### Real-Life Analogy
Pressing an elevator call button repeatedly doesn't summon multiple elevators — one press or ten presses, the outcome is identical (one elevator arrives). That's idempotency; contrast with a "add item to cart" button that adds a new item every click (non-idempotent).

### Interview Q&A
1. *Why is idempotency especially critical for a pipeline processing financial trade data?* — A retried or duplicated pipeline run must never double-count a trade or duplicate a financial position — non-idempotent pipelines risk serious financial reporting errors on retry, which is unacceptable in a regulated trading environment.
2. *How does Delta Lake's MERGE naturally support idempotent pipeline design?* — Re-running the same MERGE with the same source batch and key-based match condition produces the same end state — matched rows just get updated to the same values again, not duplicated, unlike a naive `INSERT` which would duplicate rows on every retry.
3. *What design choice makes a streaming pipeline idempotent even with at-least-once delivery from the source?* — Using checkpointing (Structured Streaming's built-in exactly-once processing guarantees when paired with idempotent sinks like Delta Lake MERGE) combined with a stable, unique event/business key for deduplication downstream.

### One-Line Revision
**Idempotent = safe to retry as many times as needed, always landing on the same correct result.**

---

## 7. Data Validation, Error Handling & Retry Logic

### Quick Theory
Production pipelines need explicit strategies for: validating incoming data quality (schema checks, null checks, business-rule checks), handling failures gracefully (quarantine bad records vs failing the whole job), and retrying transient failures automatically (network blips, throttling) without retrying on non-transient errors (bad data, logic bugs).

### Interview Q&A
1. *How would you validate incoming trading data before it enters Silver?* — Schema validation (expected columns/types present), null/completeness checks on required fields (e.g., `trade_id`, `amount`), business-rule checks (e.g., `amount > 0`, valid `account_id` exists in reference data), and routing failing records to a quarantine table for review rather than silently dropping or crashing the whole batch.
2. *Should a single bad record fail the entire pipeline run?* — Generally no — best practice is to isolate and quarantine bad records (write them to a "rejected" table/location with the validation failure reason) while allowing valid records to proceed, unless the failure indicates a systemic issue (e.g., schema completely changed) that should halt the whole run.
3. *What's the difference between a transient and a permanent failure, and how does that affect retry logic?* — Transient failures (network timeout, temporary throttling, a brief service outage) are safe to retry automatically with backoff; permanent failures (malformed data, a logic bug, invalid credentials) will fail identically on every retry and should alert a human instead of retrying endlessly.
4. *How do you implement retry with exponential backoff conceptually?* — On failure, wait a short interval before retrying, doubling (or similar backoff) the wait time on each subsequent failure up to a max number of attempts/max wait, to avoid hammering a struggling downstream system while still recovering automatically from brief blips.

### One-Line Revision
**Validate early, quarantine bad rows instead of crashing everything, and only auto-retry failures that are actually temporary.**

---

## 8. Batch vs Streaming & Event-Driven Architecture

### Quick Theory
**Batch**: process data in scheduled chunks (hourly/daily) — simpler, higher latency, easier to reason about and reprocess. **Streaming**: process data continuously as it arrives (Structured Streaming, micro-batch or continuous mode) — lower latency, more operational complexity (state management, watermarking, exactly-once semantics). **Event-driven architecture**: systems react to events (messages) rather than polling on a schedule, decoupling producers and consumers.

### Interview Q&A
1. *When would you choose streaming over batch for this trading platform?* — When downstream consumers need near-real-time visibility (e.g., live position monitoring, risk alerts) where daily/hourly batch latency is unacceptable; batch remains appropriate for end-of-day reporting/reconciliation where latency doesn't matter as much and simplicity/cost is preferred.
2. *What's the trade-off of choosing streaming when it's not strictly necessary?* — Added operational complexity — checkpointing, watermark tuning, state store management, and generally higher always-on compute cost — versus batch's simpler "runs, finishes, done" model; don't over-engineer with streaming if hourly/daily batch actually meets the SLA.
3. *How does Databricks Structured Streaming provide exactly-once processing guarantees?* — Through checkpointing (tracking exactly which offsets/data have been processed) combined with idempotent sinks like Delta Lake, so even if a micro-batch is retried after failure, the same data is reprocessed deterministically without duplication.

### One-Line Revision
**Batch is simple and scheduled; streaming is continuous and complex — use streaming only when real latency actually demands it.**

---

## 9. Data Warehouse vs Data Lake vs Lakehouse

### Quick Theory
**Data Warehouse**: structured, schema-on-write, optimized for fast BI/SQL queries, historically expensive to scale for unstructured/huge data. **Data Lake**: cheap storage for any data type (structured/semi/unstructured), schema-on-read, flexible but historically lacked ACID/reliability guarantees. **Lakehouse**: combines both — cheap object storage (ADLS Gen2) + a reliability/performance layer (Delta Lake) that gives warehouse-like guarantees (ACID, schema enforcement, fast BI queries) directly on lake storage, without needing to duplicate data into a separate warehouse.

### Interview Q&A
1. *Why would a company choose a Lakehouse architecture instead of maintaining both a separate data lake and data warehouse?* — Avoids maintaining and syncing two separate systems (extra cost, complexity, and data-freshness lag between lake and warehouse); the Lakehouse gives one system with warehouse-grade reliability/performance directly over cheap lake storage, simplifying architecture and reducing duplicate data copies.
2. *What specifically does Delta Lake add to a plain data lake to make it "Lakehouse-grade"?* — ACID transactions, schema enforcement/evolution, time travel, and performance features (Z-order, data skipping, OPTIMIZE) — the reliability and performance characteristics previously exclusive to data warehouses.

### One-Line Revision
**Lakehouse = data lake's cheap flexible storage + data warehouse's reliability and speed, in one system.**

---

## 10. End-to-End Architecture Diagrams

### 10.1 Batch Pipeline (Trading Platform Example)
```
Upstream Trading Systems (order mgmt, market data feeds)
        │  nightly extract / file drop
        ▼
   ADLS Gen2 /raw/ (landing zone)
        │
        ▼
   Databricks Job Cluster (scheduled via Workflows)
        │  read raw → validate → append
        ▼
   BRONZE Delta Table  (raw, append-only, full history)
        │  dedupe, clean, join reference data, apply SCD Type 2 where needed
        ▼
   SILVER Delta Table  (conformed, one row per business entity)
        │  aggregate (daily P&L, position summaries)
        ▼
   GOLD Delta Table  (business-ready, Unity Catalog governed)
        │
        ▼
   Power BI / downstream client reporting systems
```

### 10.2 Streaming Pipeline
```
Trading Tick Feed ──► Event Hub / Service Bus ──► Databricks Structured Streaming
                                                          │  (readStream, checkpointing)
                                                          ▼
                                                  BRONZE Delta Table (streaming append)
                                                          │  (streaming MERGE / foreachBatch)
                                                          ▼
                                                  SILVER Delta Table (near-real-time)
                                                          │
                                                          ▼
                                                  Live dashboard / risk alert system
```

### 10.3 CI/CD Flow (ties into file 05)
```
Developer commits notebook/code change ──► Azure DevOps Repo (feature branch)
        │  Pull Request + code review
        ▼
   Azure Pipelines: run tests (pytest/unit tests on PySpark logic)
        │  merge to main
        ▼
   Release Pipeline: deploy notebooks/Jobs config to DEV → TEST → PROD workspaces
        (using Databricks CLI/REST API + environment-specific parameters)
        │
        ▼
   Databricks Workflow scheduled/triggered in PROD
```

### 10.4 Monitoring Flow
```
Databricks Cluster/Job Events ──► Diagnostic Settings ──► Log Analytics Workspace
        │
        ▼
   KQL Queries / Dashboards
        │
        ▼
   Alert Rules ──► Azure Monitor Action Group ──► Email / Teams / PagerDuty
```

### Interview Q&A on Architecture
1. *Walk me through the end-to-end flow of trading data in this platform, from source to report.* — Data lands raw in ADLS Gen2 from upstream trading systems (batch file drop or streaming event feed), gets ingested into Bronze via a Databricks job/streaming pipeline, cleaned and conformed into Silver with deduplication and SCD handling, aggregated into business-ready Gold tables governed by Unity Catalog, and finally consumed by Power BI or downstream client-facing systems — with the whole pipeline deployed via Azure DevOps CI/CD and monitored through Log Analytics/Azure Monitor.
2. *Where would you add a data quality checkpoint in this architecture, and why there specifically?* — Between Bronze and Silver — since Bronze should stay a faithful raw copy for replay, Silver is where enforcement (schema, nulls, business rules) belongs, quarantining bad records before they can pollute conformed/aggregated layers that downstream systems and clients rely on.
3. *How would this architecture change if a client needed sub-second latency instead of end-of-day batch?* — Replace scheduled batch ingestion with Structured Streaming reading from Event Hub/Service Bus, apply watermarking and streaming MERGE (via `foreachBatch`) into Bronze/Silver, and serve the dashboard directly off the near-real-time Silver table instead of waiting for a Gold batch aggregation job.

---

Next file: `05_devops_cicd_performance_monitoring_security.md`
