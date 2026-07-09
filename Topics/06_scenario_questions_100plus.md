# 06 — 100+ Scenario-Based Interview Questions & Answers

These are the "what would you actually do" questions that separate candidates who've memorized definitions from candidates who've thought about production reality. Answer style: name the likely cause(s), your diagnostic steps, and the fix — in that order, out loud, in under 60 seconds.

---

## A. Pipeline Performance & Slowness (1–14)

1. **A pipeline that normally takes 20 minutes suddenly takes 3 hours. What do you check?**
   Check the Spark UI for a skewed stage (one long-running task), compare today's input data volume against historical norms (maybe upstream sent a much larger batch), check if cluster autoscaling actually kicked in, and check for any recent code change or a cluster/runtime version change deployed around the same time.

2. **One task in a stage is taking 10x longer than every other task. What's happening?**
   Data skew — one partition (often tied to a specific join/group-by key) has disproportionately more data. Fix with salting the key, enabling AQE skew join handling, or repartitioning by a better key.

3. **A join between a 2TB table and a 5MB table is slow. What would you check?**
   Whether Spark is actually broadcasting the small table — check the query plan (`EXPLAIN`) for a `BroadcastHashJoin`; if it's doing a shuffle join instead, force it with `broadcast(df)` or check if `autoBroadcastJoinThreshold` is misconfigured.

4. **A streaming job's processing time per micro-batch keeps creeping upward day over day.** 
   Likely accumulating small files (small file problem) in the sink table with no OPTIMIZE running, or growing streaming state (e.g., an unbounded aggregation without a watermark) — schedule regular OPTIMIZE/VACUUM and verify watermarking is configured.

5. **A job runs fine in Dev but times out in Prod with 100x the data.**
   Dev testing likely didn't validate at production scale — check for operations that don't scale linearly (e.g., a `.collect()` on the driver, a Python UDF instead of a native function, or a broadcast join where the "small" table is actually large in Prod).

6. **CPU utilization across the cluster is very low (10-20%) but the job is still slow.**
   Likely too few partitions for the cluster size (parallelism bottleneck) or the job is I/O bound / waiting on network shuffle rather than CPU bound — check partition count vs total cores, and check shuffle read/write metrics.

7. **Query plan shows a full table scan when you expected partition pruning.**
   The filter predicate might not be on the partition column, or it's wrapped in a function (e.g., `WHERE YEAR(trade_date) = 2026` prevents pruning vs `WHERE trade_date >= '2026-01-01'`), or file-level statistics are stale/missing.

8. **After migrating a table to Delta Lake, queries are still slow.**
   Migration alone doesn't optimize file layout — run `OPTIMIZE` (and `ZORDER` on commonly filtered columns) after migration; also verify statistics were computed (`ANALYZE TABLE`) since the optimizer relies on them.

9. **A notebook runs fast interactively but the same logic is slow when scheduled as a Job.**
   Check if the Job cluster has different (smaller/cheaper) configuration than the interactive cluster used for testing, or if the interactive cluster had cached/warm data that the fresh Job cluster doesn't.

10. **Executors keep getting killed and restarted mid-job.**
    Likely OOM on executors — check for skew, an oversized broadcast, or too few executor cores/memory for the partition size; also check if Spot instances are being reclaimed by Azure if using low-priority VMs.

11. **A groupBy aggregation over billions of rows is slow despite AQE being enabled.**
    Check whether the aggregation key itself is skewed (a few keys with disproportionate row counts) — AQE's shuffle partition coalescing helps with partition sizing but doesn't fully fix key skew without its skew-join feature (which applies mainly to joins, not standalone aggregations) — consider a two-phase aggregation (partial aggregate then combine) or salting.

12. **A batch job that reads from ADLS Gen2 is bottlenecked on I/O, not compute.**
    Check the number of files vs partitions being read (too few large files limits read parallelism; too many tiny files add overhead), and consider whether the storage account itself is being throttled (check request-rate limits) if running many parallel jobs against the same account.

13. **Adding more worker nodes didn't speed up the job at all.**
    The job may already be limited by too few partitions (more nodes can't help if there aren't enough parallel units of work to hand out), or the bottleneck is actually the driver (e.g., a `.collect()`), or a single skewed partition is the critical path regardless of total cluster size.

14. **The first run of a job each morning is always slower than subsequent runs.**
    Classic cold-start: cluster provisioning + JVM/Spark session startup + no warm cache; mitigate with cluster pools to reduce startup latency, or keep a small always-on cluster if the SLA truly requires consistently fast first-run performance.

---

## B. Delta Table Issues & Data Corruption (15–26)

15. **A Delta table query fails with "file not found" referencing a file that should exist.**
    Likely someone ran `VACUUM` with too short a retention period while a long-running query/consumer was still referencing an older table version — align VACUUM retention with the slowest known consumer's needs.

16. **A downstream consumer reports numbers "changed" for a report that was supposedly final.**
    Check Delta Lake's Time Travel/history (`DESCRIBE HISTORY`) to see if a later write (a correction, a re-run, or a bad job) modified the table after the report was generated — trace which job/user made that commit.

17. **A batch job crashed halfway through writing to a Delta table. Is the table now corrupted?**
    No — Delta Lake's transaction log only exposes fully-committed writes; a crashed job's partial files simply never get referenced by a completed log entry, so readers still see the last consistent, complete version.

18. **Two jobs wrote to the same Delta table at the same time and one failed with a concurrency error.**
    Delta Lake uses optimistic concurrency control — if both writers' changes conflict (e.g., overlapping updates on the same files), the second commit fails and must retry after re-reading the latest table state; design pipelines to catch this exception and retry, or avoid concurrent writers on the same table region when possible.

19. **You need to recover a Gold table to how it looked yesterday before a bad job ran.**
    Use `RESTORE TABLE gold.table TO TIMESTAMP AS OF '2026-07-08 00:00:00'` (or `VERSION AS OF n`), assuming the relevant version hasn't already been vacuumed past retention.

20. **A schema change upstream (a new column added) is now breaking your Bronze ingestion job.**
    If using `mergeSchema` / schema evolution appropriately, a new column should just be added automatically; if the job is failing instead, likely schema enforcement is blocking an unexpected type mismatch (not just a new column) — investigate the actual error and confirm whether it's a genuinely new column or a changed data type for an existing one.

21. **A Delta table's read performance has degraded significantly over 6 months with no code changes.**
    Almost certainly the small-file problem accumulating from ongoing writes with no OPTIMIZE scheduled — check file count/sizes and run OPTIMIZE (+ ZORDER) and set up a recurring maintenance job.

22. **You need to permanently delete a specific customer's records for compliance (GDPR-style) from a Delta table.**
    Run `DELETE FROM table WHERE customer_id = X`, then run `VACUUM` to physically purge the now-unreferenced files (being mindful this also removes the ability to time-travel to versions containing that data, which is usually the compliance intent).

23. **A MERGE operation is failing with "multiple source rows matched a single target row."**
    The source (incoming) batch has duplicate keys for at least one match — dedupe the source with a window function (`ROW_NUMBER()` keeping the latest per key) before the MERGE.

24. **Someone accidentally ran `DROP TABLE` on a production Delta table.**
    If Unity Catalog is being used, check if the table was actually a managed table (data deleted) vs external (metadata dropped but underlying files may remain, at least until any subsequent VACUUM/cleanup); recovery may involve restoring from a table snapshot/backup process if the underlying data was truly deleted, which is why RBAC should heavily restrict DROP TABLE permissions in Prod.

25. **A table's `_delta_log` directory has grown huge and log operations (like DESCRIBE HISTORY) are slow.**
    Check if checkpointing is happening as expected (every 10 commits by default) — if not, or if log retention is set very high with no cleanup, the log itself becomes a bottleneck; Delta automatically manages checkpoints, but log retention settings and very high-frequency streaming commits can still bloat it.

26. **A time-travel query to a version from 3 months ago fails.**
    The underlying files for that version were likely already removed by a `VACUUM` run using the default 7-day retention — time travel is only reliable within the configured retention window.

---

## C. Duplicate Records & Late-Arriving Data (27–36)

27. **Duplicate trade records are showing up in the Silver table.**
    Check the ingestion/MERGE logic — likely doing a plain `INSERT` (or an unkeyed append) instead of a MERGE keyed on a unique business ID, or the source batch itself contains duplicates that weren't deduped before merging.

28. **The same event arrives twice from Service Bus due to at-least-once delivery.**
    Expected behavior of at-least-once messaging — the pipeline itself must be idempotent (MERGE keyed on a unique message/event ID) so processing the same event twice produces the same end state, not a duplicate row.

29. **Late-arriving data (a trade confirmed 2 days after the fact) needs to update an already-finalized daily summary.**
    If using a batch Gold aggregation, either re-run the affected day's aggregation job (idempotent MERGE-based Gold logic re-processes cleanly) or, for streaming, ensure the watermark threshold accounts for expected lateness — if data is later than the watermark, it needs a separate correction/backfill process.

30. **How do you decide the right watermark threshold for a streaming job dealing with occasionally delayed feeds?**
    Analyze historical lateness distribution from the source (e.g., 95th/99th percentile delay) and set the watermark to comfortably cover that, balancing the memory/state cost of a longer watermark window against the risk of dropping legitimately late data.

31. **A CDC pipeline received an "update" event before the corresponding "insert" event (out of order).**
    Handle via sequence/timestamp-based conditional MERGE logic (`WHEN NOT MATCHED THEN INSERT` for the "insert" event even if logically it's really an update scenario, combined with comparing event sequence numbers) so out-of-order arrival doesn't silently drop data.

32. **How would you detect duplicate records already sitting in a table (not just during ingestion)?**
    Use a window function (`ROW_NUMBER() OVER (PARTITION BY business_key ORDER BY ingestion_time)`) to find rows with `row_number > 1` for the same key, then investigate/clean up as a one-time remediation, and fix the root-cause ingestion logic.

33. **A daily reconciliation report shows more records in Gold than expected for a given day.**
    Check for duplicate ingestion (same file processed twice, e.g., due to a retry that didn't check "already processed" state), or a fan-out join upstream (a join that unexpectedly multiplied rows due to a non-unique join key).

34. **How do you make a file-based ingestion job safe to re-run without duplicating already-processed files?**
    Track processed file names/paths (or use Databricks Auto Loader's built-in checkpointing which tracks exactly which files have been ingested) so re-running the job skips already-ingested files rather than reprocessing them.

35. **A "corrected" trade record needs to overwrite a previous version, but you also need to preserve the original for audit.**
    This is the SCD Type 2 pattern — expire the old row (set `end_date`, `is_current=false`) and insert the corrected row as a new current version, preserving both in history.

36. **How would you handle a batch where 5% of incoming records fail validation, without blocking the other 95%?**
    Split the batch during validation — write passing records through the normal pipeline path and write failing records to a quarantine/rejected table with the failure reason, alerting on quarantine volume if it exceeds a threshold.

---

## D. Schema Evolution (37–44)

37. **Upstream added a new column to the source feed. What happens to your Bronze ingestion?**
    With Delta Lake's schema evolution (`mergeSchema` option) enabled appropriately, the new column is automatically added to the table schema on write; without it, schema enforcement would reject the write — decide deliberately whether evolution should be automatic or reviewed.

38. **A column's data type changed upstream (e.g., an ID field went from Integer to String).**
    Schema evolution generally does *not* auto-handle type changes safely — this needs explicit handling (a controlled migration, casting logic, or failing the pipeline with an alert for manual review), since silently reinterpreting a type change risks data corruption.

39. **How do you evolve a Gold table's schema without breaking downstream BI dashboards relying on it?**
    Add new columns without removing/renaming existing ones (additive-only changes), communicate schema changes to downstream consumers in advance, and consider versioned or "as-of" table naming if a breaking change is unavoidable.

40. **A required column is missing from an incoming batch. Should the pipeline fail or default the value?**
    Depends on business criticality — for financial-critical columns (e.g., `trade_amount`), fail/quarantine rather than silently defaulting, since a wrong default could be worse than a delayed pipeline; for non-critical optional columns, a sensible default with logging may be acceptable.

41. **How would you enforce schema contracts between an upstream team and your pipeline?**
    Define an explicit expected schema (and validate incoming data against it at Bronze ingestion), use schema enforcement rather than blanket auto-evolution, and establish a change-notification process/contract with the upstream team before they alter feed structure.

42. **A nested/complex JSON structure from an upstream API keeps changing shape slightly between calls.**
    Consider ingesting the raw JSON as a string/variant column in Bronze (preserving whatever structure arrives without failing), then parse/flatten with more tolerant logic (checking for field existence) in Silver.

43. **How do you test that a schema change won't break a production Delta Live Tables/Workflow pipeline before deploying?**
    Deploy and run the change first in a Dev/Test environment against a representative sample (or full copy) of production-like data, verify downstream queries/dashboards still resolve correctly, and include schema assertions as part of automated pipeline tests.

44. **Should Bronze tables generally use strict schema enforcement or permissive schema evolution?**
    Often more permissive at Bronze (to avoid losing/failing on raw data even if it's messy) with stricter validation/enforcement applied at the Silver layer where you actively shape and guarantee the conformed schema.

---

## E. Cluster / Memory Issues (45–54)

45. **A job fails with `java.lang.OutOfMemoryError: Java heap space` on the driver.**
    Likely a `.collect()`, `.toPandas()`, or broadcast join pulling too much data to the driver — check the code for driver-side data collection and either avoid it, sample/limit the data, or increase driver memory if genuinely needed.

46. **Executors are failing with OOM but the driver is fine.**
    Check for data skew (one partition too large for its executor's memory), an oversized/incorrectly-forced broadcast join, or simply undersized executor memory relative to per-partition data volume — consider repartitioning or resizing the cluster.

47. **How do you decide between a memory-optimized vs compute-optimized cluster instance type?**
    Memory-optimized for workloads with large shuffles, wide joins, or big in-memory caching needs; compute-optimized for CPU-heavy transformation logic (e.g., complex UDFs, ML feature engineering) with relatively smaller data footprints per task.

48. **A cluster takes 8 minutes to start every time a job runs, hurting your SLA.**
    Use a cluster pool to keep pre-warmed instances ready, reducing startup time significantly, or consider a smaller always-on cluster if the job runs frequently enough to justify the standing cost.

49. **Should you use Spot (low-priority) instances for a nightly batch trading reconciliation job?**
    Reasonable for cost savings if the job is fault-tolerant and can retry/checkpoint through a lost node without corrupting results (e.g., idempotent Delta MERGE-based steps), but risky if the job has tight SLA windows with no slack for retries from reclaimed capacity.

50. **How would you troubleshoot a cluster that repeatedly fails to reach "Running" state?**
    Check cloud provider quota limits for the requested VM family/region, network/NSG rules blocking required outbound endpoints, invalid init scripts, or an incompatible library installed at cluster scope causing startup failure.

51. **Autoscaling isn't kicking in even though the job clearly needs more executors.**
    Check the configured max worker limit (may already be at max), verify the workload actually has enough pending/queued tasks to trigger scale-up (Databricks scales based on task backlog, not just raw data size), and check cluster policy restrictions.

52. **A shared all-purpose cluster used by multiple teams is randomly slow at unpredictable times.**
    Classic "noisy neighbor" problem — one team's heavy interactive query is consuming shared cluster resources; recommend moving production jobs to dedicated job clusters and reserving the shared cluster strictly for lightweight interactive development.

53. **How would you right-size a cluster for a job you've never run before?**
    Start with a reasonable baseline based on estimated data volume, run with autoscaling enabled and a generous max, monitor actual resource utilization (Spark UI/Ganglia metrics) during the first few runs, then tighten min/max bounds based on observed real usage.

54. **A Python library installed at the cluster level is causing version conflicts with another team's notebook on the same cluster.**
    This is a strong argument for job clusters (isolated per pipeline, no shared library conflicts) or notebook-scoped library installs (`%pip install` within a single notebook session) instead of cluster-wide library installation on shared clusters.

---

## F. Job Failures & Retry Strategy (55–64)

55. **A scheduled job failed overnight with a generic timeout error. How do you triage in the morning?**
    Check the Databricks Job run's driver logs and error stack trace first, correlate the failure time with any upstream system outage or unusually large data volume, and check whether it's a one-off transient issue or a recurring pattern from job history.

56. **A job fails intermittently — succeeds 8 out of 10 runs with the same code and similar data volume.**
    Suspect a transient external dependency issue (network blip, upstream API rate limiting, a race condition with a concurrent writer on the same table) rather than a deterministic code bug; add retry logic with backoff for the transient case and add logging to capture more detail on the next failure.

57. **Should every task in a Workflow have the same retry policy?**
    No — transient-failure-prone tasks (e.g., an API call, a network-dependent read) benefit from retries with backoff, while tasks where retrying the same bad input would just fail identically (e.g., a genuine data validation failure) shouldn't blindly retry — they should alert a human instead.

58. **A job's retry succeeded, but now you're worried it double-processed some data.**
    This is exactly why idempotent pipeline design (MERGE-based upserts keyed on unique identifiers, not blind appends) matters — verify the pipeline's write logic is idempotent so a retry naturally produces the same correct end state without duplication.

59. **How do you prevent a single failing task from blocking an entire Workflow indefinitely?**
    Set explicit timeouts on each task and a maximum retry count, and configure "on failure" alerting/conditional branches so the Workflow fails fast and notifies someone rather than hanging.

60. **A downstream job depends on an upstream job's success, but the upstream job is delayed (not failed) today.**
    If using Workflow task dependencies or file-arrival triggers, the downstream task should simply wait rather than run against stale/incomplete data — confirm the dependency/trigger mechanism doesn't have a hardcoded time-based schedule that could fire before upstream truly finishes.

61. **How would you design alerting so the team knows about a failure within minutes, not hours?**
    Configure Job-level failure notifications (email/webhook) directly in the Workflow, wired to a fast-response channel (Teams/Slack/PagerDuty) rather than relying solely on someone manually checking the Databricks UI or a daily log review.

62. **A job failed because of a transient Azure storage throttling error. What's the right response?**
    Automatic retry with exponential backoff — throttling errors are explicitly transient and expected to resolve after a brief wait, so an immediate blind retry (without backoff) could make throttling worse, while manual intervention is unnecessary for a well-understood transient cause.

63. **How do you distinguish between a job that "failed" vs one that's still legitimately running long?**
    Compare current runtime against historical baseline duration and set a reasonable timeout threshold with alerting, rather than assuming any long-running job is stuck — check Spark UI for active task progress to confirm it's actually making progress, not hung.

64. **A critical job failed at 2am and no one was paged. How do you fix the process, not just the immediate issue?**
    Add proper on-call alerting (PagerDuty/Teams integration) tied to job-failure notifications for critical pipelines specifically (not just email, which people don't check at 2am), and review whether the failure could have been caught earlier by upstream data-quality validation.

---

## G. CI/CD Deployment Failures (65–72)

65. **A deployment to Prod succeeded in the pipeline but the Databricks Job is now failing with "cluster policy violation."**
    The Prod environment likely has a stricter cluster policy than Dev/Test that the deployed Job config doesn't comply with (e.g., a disallowed instance type or missing tag) — parameterize Job configs per environment and test against each environment's actual policy before promoting.

66. **A CI/CD pipeline's unit tests pass, but the deployed pipeline breaks on real Prod data.**
    Unit tests likely only cover logic in isolation with sample/mocked data, not integration behavior against real data scale/quality — add integration tests against a representative Prod-like dataset in a Test/Staging environment before Prod deployment.

67. **How do you avoid deploying a broken change directly to Prod on a Friday afternoon?**
    Enforce a manual approval gate before the Prod stage in the pipeline, avoid deploying_critical changes right before a low-monitoring window (weekend), and consider deployment windows/freeze periods for high-risk-timing changes.

68. **The DevOps pipeline's Databricks CLI deployment step fails with an authentication error.**
    Likely the service principal/PAT token used for deployment has expired or lacks sufficient permissions in the target workspace — verify the credential (ideally backed by Key Vault, with rotation tracked) and its assigned Unity Catalog/workspace permissions.

69. **How would you roll back a bad Prod deployment quickly?**
    Redeploy the last known-good notebook/Job configuration from git history (since everything is version-controlled), and if data was already affected, use Delta Lake's time travel/RESTORE to revert affected tables.

70. **Two developers' changes conflict when both try to deploy to the same Job configuration.**
    This points to insufficient environment/branch isolation — enforce that Job configuration changes go through the same PR + CI/CD process as code, with a single pipeline responsible for deployment (no manual out-of-band changes to Prod Job configs).

71. **How do you test infrastructure/cluster configuration changes safely before applying them to Prod?**
    Apply the change first to a Dev/Test workspace cluster with similar (scaled-down) configuration, validate job behavior and cost impact, then promote the configuration change through the same CI/CD pipeline used for code.

72. **A deployment succeeded, but nobody documented what changed, and now there's a Prod issue with no clear cause.**
    This highlights the value of PR descriptions, commit history, and a changelog/release notes process as part of the CI/CD pipeline — every Prod deployment should be traceable to a specific reviewed PR explaining the "why."

---

## H. Data Quality Issues (73–82)

73. **A downstream client reports their daily P&L numbers look wrong.**
    Trace the number back through Gold → Silver → Bronze using lineage (Unity Catalog), check for recent schema/logic changes, verify no duplicate/missing records for that client/day, and use Delta time travel to compare against a previous known-good version.

74. **How would you catch data quality issues before they reach a client-facing report?**
    Build validation checks (schema, null/completeness, business-rule checks like `amount > 0`) between Bronze and Silver, and add reconciliation checks between Silver and Gold (e.g., row counts, sum totals matching expected ranges) with automated alerts on failure.

75. **A reference/lookup table (e.g., currency codes) wasn't updated, causing failed joins downstream.**
    This highlights the need for validating reference data freshness as part of the pipeline (e.g., alert if reference table hasn't been refreshed within expected SLA) and possibly failing/quarantining records with unmatched lookups rather than silently dropping them.

76. **You're asked to add automated data quality checks to an existing pipeline that has none. Where do you start?**
    Start with the highest-risk checks first: schema/type validation, null checks on required business-critical fields, and row-count/volume sanity checks (e.g., alert if today's volume is 50% below the historical average) — expand to more nuanced business-rule checks over time.

77. **A "silent" data quality issue existed for 3 weeks before anyone noticed.**
    This is a strong argument for proactive automated alerting (volume/anomaly checks) rather than relying on someone manually noticing wrong numbers — retroactively, also assess whether affected downstream reports need correction/reprocessing once the root cause is fixed.

78. **How do you validate that Bronze-to-Silver transformation logic hasn't introduced bugs after a code change?**
    Compare aggregate metrics (row counts, sums, distinct key counts) between the old and new logic's output on the same input data in a Test environment, ideally with an automated regression test suite covering known edge cases.

79. **A field that should never be null suddenly has nulls in 2% of records.**
    Investigate whether this is a genuine upstream data issue (a new optional field, a source system bug) versus a transformation bug in your own pipeline (e.g., a join that unexpectedly produced unmatched/null rows) — quarantine affected records and alert the appropriate team.

80. **How would you set up an automated "row count sanity check" between two pipeline stages?**
    Compare the row count of Silver's output for a given batch against Bronze's input count (accounting for expected filtering/deduplication logic), and alert if the difference falls outside an expected tolerance range.

81. **A data quality check is generating too many false-positive alerts, and the team has started ignoring them.**
    Tune the check's thresholds based on actual historical variability (rather than an arbitrary fixed number) and consider a severity tiering (warning vs critical) so the team doesn't get "alert fatigue" from noisy low-priority checks drowning out real critical alerts.

82. **How do you communicate a data quality incident to a business/client stakeholder?**
    Clearly explain what happened, the scope of impact (which data/reports/time period), the root cause once known, the remediation steps taken (including any reprocessing), and preventive measures going forward — without over-promising a timeline before root cause is confirmed.

---

## I. Scale, Partitioning & Architecture Decisions (83–96)

83. **You need to process billions of records nightly. What architectural choices matter most?**
    Efficient partitioning aligned with common query/filter patterns, avoiding small-file accumulation (regular OPTIMIZE), choosing appropriately-sized clusters with autoscaling, minimizing shuffles (broadcast joins where applicable), and using incremental rather than full processing wherever possible.

84. **How do you choose a good partition column for a multi-terabyte Delta table?**
    Choose a column commonly used in WHERE-clause filtering with moderate cardinality (e.g., `trade_date`, not `trade_id`) — too high cardinality creates the small-file problem, too low cardinality doesn't help pruning enough; combine with Z-ORDER on secondary high-cardinality filter columns.

85. **A table partitioned by `customer_id` (thousands of distinct values) has performance issues.**
    Classic over-partitioning — creates too many small partitions/files; better to partition by a lower-cardinality column (e.g., `date`) and use Z-ORDER by `customer_id` instead for query acceleration without the file-count explosion.

86. **How would you design a pipeline to handle a client onboarding with 10x more data volume than existing clients?**
    Test the pipeline against a representative sample of that volume in advance, verify partitioning strategy still holds up (may need adjustment for the new scale), confirm cluster autoscaling limits are sufficient, and check for any per-client isolation assumptions in the architecture that might not scale (e.g., single shared table vs schema-per-client).

87. **When would you choose a job cluster with a fixed size instead of autoscaling?**
    When workload size is highly predictable and consistent run-to-run — autoscaling adds a small amount of latency/overhead for scale-up decisions that isn't worth it for a stable, well-understood workload; autoscaling shines more for variable/unpredictable volume.

88. **How would you decide whether to use Databricks SQL Warehouses vs a general-purpose cluster for a reporting workload?**
    Databricks SQL Warehouses are optimized specifically for concurrent SQL/BI query workloads (with Photon, caching, and auto-stop/scaling tuned for that pattern) — preferred for Power BI/dashboard-serving; general-purpose clusters suit custom ETL/ML notebook workloads.

89. **A pipeline that worked fine for 6 months is now failing to keep up as data volume has grown 5x.**
    Revisit whether the original partitioning/cluster sizing assumptions still hold at the new scale, check whether the workload has shifted from batch-appropriate to needing streaming, and profile the job fresh (Spark UI) rather than assuming last year's tuning still applies.

90. **How would you architect a system to support both near-real-time client dashboards and heavy nightly batch reconciliation on the same underlying data?**
    Use a Lakehouse pattern where Silver is updated near-real-time via streaming MERGE for dashboard freshness, while a separate scheduled batch job builds Gold-layer reconciliation aggregates — same source of truth (Silver), different consumption patterns layered on top.

91. **A multi-client SaaS platform needs strict data isolation but also wants to minimize infrastructure duplication.**
    Use a shared Databricks workspace/cluster infrastructure (cost efficiency) but enforce logical isolation via separate catalogs/schemas per client in Unity Catalog with RBAC grants, rather than fully separate infrastructure per client (which would eliminate sharing benefits) or a single shared table relying only on row-level filtering (higher risk of isolation bugs).

92. **How do you estimate the cost of a new pipeline before building it?**
    Estimate expected data volume and processing frequency, prototype against a representative sample to measure actual cluster time/size needed, and extrapolate — plus account for storage costs (including retained history) and ongoing maintenance job costs (OPTIMIZE/VACUUM).

93. **A cost review shows Databricks spend has grown 3x in 6 months with no corresponding business growth.**
    Investigate for cluster sprawl (too many long-running all-purpose clusters vs job clusters), missing auto-termination settings, redundant/duplicate pipelines, or a lack of cluster policies enforcing reasonable size limits — audit actual utilization vs allocated capacity.

94. **How would you migrate an existing pipeline from a traditional data warehouse to a Databricks Lakehouse without downtime?**
    Run both systems in parallel for a validation period (dual-write or backfill the Lakehouse from the same sources), reconcile outputs between old and new systems until confidence is established, then cut over consumers gradually (starting with lower-risk reports) before fully decommissioning the old system.

95. **How would you handle a requirement for sub-second query latency on a Gold table serving a live trading dashboard?**
    Consider whether Databricks SQL Warehouses with caching/Photon meet the latency need, or whether the requirement actually calls for a purpose-built serving layer (e.g., a low-latency database/cache fed by the Lakehouse) since Spark/Delta, even optimized, is generally not designed for true sub-second single-row-lookup latency at high concurrency.

96. **How would you decide whether a new requirement should be built as a batch Workflow or a Delta Live Tables pipeline?**
    Delta Live Tables suits declarative, quality-constraint-driven pipelines with built-in lineage/monitoring and less custom orchestration code; a custom Workflow suits pipelines needing complex custom control flow, external system integration, or logic that doesn't fit DLT's declarative model as cleanly.

---

## J. Monitoring & Security Incidents (97–108)

97. **You discover a Databricks job has been silently failing for a week with no alert.**
    Add job-failure notifications immediately as a stopgap, then investigate why existing monitoring didn't catch it (missing alert configuration, alert routed to an unmonitored inbox, or the job "succeeding" technically while silently producing wrong/empty output — which points to needing output-validation checks, not just success/failure status).

98. **How would you set up alerting to catch a job that "succeeds" but produces zero rows of output (a silent failure)?**
    Add a post-processing validation step checking output row count against an expected minimum/historical baseline, and fail/alert the job explicitly if the output is suspiciously empty or far outside normal range, rather than relying solely on the job's technical success/failure status.

99. **A security review flags that several service principals have overly broad permissions ("Owner" on the whole subscription).**
    Apply least-privilege remediation — scope each service principal down to only the specific resource group/resource/RBAC role it actually needs, and set up periodic access reviews to catch permission creep going forward.

100. **You suspect a credential (e.g., a Databricks PAT token) may have been leaked in a public commit.**
     Immediately revoke/rotate the credential, audit recent activity under that identity for any unauthorized access, and add secret-scanning to the CI/CD pipeline/pre-commit hooks to catch this before it happens again.

101. **How would you detect unusual/potentially malicious access patterns to a sensitive trading data table?**
     Rely on Unity Catalog's audit logs feeding into Log Analytics, alerting on anomalies like access from an unexpected identity/location, unusually large data exports, or access outside normal business hours for that dataset.

102. **A client asks for proof that their data was never accessible to another client on the shared platform.**
     Use Unity Catalog's lineage and audit logs to demonstrate exactly which identities/roles had grants on that client's schema/catalog and confirm no cross-client access occurred, backed by the RBAC/grant configuration history.

103. **How do you handle a situation where a well-meaning engineer manually modified production data directly (bypassing the pipeline) to "fix" an issue?**
     Treat it as a process gap to address (restrict direct write access to Prod tables via RBAC so only the pipeline's service identity can write), document the manual change for audit purposes, and verify it didn't introduce inconsistency that the next pipeline run might overwrite or conflict with.

104. **A production alert fires at 3am for a job failure, but it turns out to be a known, harmless transient issue.**
     Rather than just ignoring future alerts (alert fatigue risk), fix the underlying noise — add automatic retry-with-backoff for that specific known transient case so it self-heals without paging anyone, reserving alerts for genuinely actionable failures.

105. **How would you design monitoring so that a failure in a critical trading pipeline is distinguished from a failure in a low-priority internal report pipeline?**
     Tag/categorize pipelines by criticality and route alerts accordingly — critical pipelines page on-call immediately (PagerDuty/phone), lower-priority pipelines can go to a shared Teams channel or daily digest email, avoiding both under- and over-alerting.

106. **A Key Vault-backed secret rotation broke a pipeline because the code had a hardcoded old value cached somewhere.**
     This indicates the pipeline wasn't actually fetching the secret fresh at runtime — fix by ensuring secrets are always retrieved live via the secret scope/Key Vault reference rather than cached/hardcoded, and test rotation in Dev/Test before relying on it in Prod.

107. **How would you investigate whether a recent Azure networking change (NSG rule update) caused a pipeline outage?**
     Correlate the outage's start time precisely with the change deployment timestamp (via Azure Activity Log/change history), check whether the failure symptoms match a connectivity issue (timeouts, DNS failures) vs a data/logic issue, and test connectivity from an equivalent cluster after temporarily reverting the suspected rule change in a controlled way.

108. **A compliance audit asks you to demonstrate that access to a specific sensitive Gold table is properly restricted.**
     Pull the Unity Catalog grants for that table (who/what roles have SELECT/MODIFY), cross-reference against the intended access policy, and provide audit logs showing actual access history matches the expected authorized users only.

---

## How to Use This Section

Don't memorize all 108 verbatim. Instead, notice the **repeating diagnostic pattern** across almost every answer:
1. Name the most likely cause(s) first (don't just say "I'd investigate").
2. State the specific tool/place you'd look (Spark UI, `DESCRIBE HISTORY`, Log Analytics, cluster policy, etc.).
3. State the fix, and if relevant, the prevention for next time.

If you internalize that 3-step shape, you can improvise a confident answer to almost any scenario question the interviewer invents on the spot — even ones not listed here.

Next file: `07_hr_behavioral_and_final_revision.md`
