# 05 — Azure DevOps, CI/CD, Performance Tuning, Monitoring & Security

---

## PART A: Azure DevOps & CI/CD

## 1. Git, Branching, Pull Requests

### Quick Theory
Git is the version control system underpinning Azure DevOps Repos. **Branching strategies** (e.g., trunk-based, GitFlow, feature-branch) define how teams isolate work-in-progress from stable code. **Pull Requests (PRs)** are the review gate before merging changes into a shared branch (usually `main`/`develop`).

### Real-Life Analogy
Branches are like **separate draft copies of a shared document** — you edit your copy without touching the master, then submit it for review (PR) before your edits get merged into the master copy everyone relies on.

### Interview Q&A
1. *What branching strategy would you recommend for a small data engineering team shipping frequent small changes?* — Trunk-based development with short-lived feature branches, each merged via PR quickly (within a day or two) — reduces merge conflicts and keeps `main` always close to production-ready, versus long-lived GitFlow branches which suit larger release-train style teams.
2. *Why require a PR instead of pushing directly to main?* — Enforces code review (catching bugs/style issues before production), enables automated CI checks to run before merge, and keeps a clear audit trail of who approved what change — especially important for regulated financial pipelines.
3. *How do you handle a merge conflict in a Databricks notebook (which is JSON/source-formatted)?* — Databricks Repos supports notebooks as `.py` files with cell markers (`# COMMAND ----------`) when using the "source" format, which git-diffs far more cleanly than the old `.ipynb`-style JSON — resolve conflicts like any text file, then re-verify the notebook still runs correctly.

### One-Line Revision
**Branches isolate work, PRs gate quality before it reaches everyone else's copy.**

---

## 2. Azure Pipelines (CI/CD) & YAML

### Quick Theory
Azure Pipelines automates build/test/deploy steps, defined in a YAML file checked into the repo alongside code (pipeline-as-code). For Databricks, a typical pipeline runs unit tests on PySpark logic, then uses the Databricks CLI/REST API to deploy notebooks and Job/Workflow definitions to each environment (Dev → Test → Prod).

### Technical Explanation
```yaml
# azure-pipelines.yml (simplified)
trigger:
  branches:
    include: [main]

stages:
- stage: Test
  jobs:
  - job: RunUnitTests
    pool:
      vmImage: 'ubuntu-latest'
    steps:
    - script: pip install -r requirements.txt --break-system-packages
      displayName: 'Install dependencies'
    - script: pytest tests/ --junitxml=test-results.xml
      displayName: 'Run PySpark unit tests'
    - task: PublishTestResults@2
      inputs:
        testResultsFiles: 'test-results.xml'

- stage: DeployToProd
  dependsOn: Test
  condition: succeeded()
  jobs:
  - job: DeployNotebooksAndJobs
    steps:
    - task: Bash@3
      inputs:
        targetType: 'inline'
        script: |
          databricks configure --token
          databricks workspace import_dir ./notebooks /Shared/prod_pipeline
          databricks jobs create --json-file ./jobs/prod_job_config.json
      env:
        DATABRICKS_HOST: $(prodWorkspaceUrl)
        DATABRICKS_TOKEN: $(prodDatabricksToken)   # stored as a secret variable, backed by Key Vault
```
- `trigger` defines which branch changes kick off the pipeline.
- `stages`/`jobs`/`steps` structure the pipeline hierarchically; `dependsOn` + `condition` enforce that deployment only happens after tests pass.
- Secrets (`DATABRICKS_TOKEN`) should be pulled from a Key Vault–linked variable group, never hardcoded in YAML.

### Interview Q&A
1. *Why is pipeline-as-code (YAML) preferred over the old classic (UI-based) Azure Pipelines?* — YAML pipelines are version-controlled alongside the code they build/deploy, reviewable via PR like any other code change, and easily reproducible/auditable — the classic UI editor's configuration isn't naturally tracked in git history.
2. *How do you prevent secrets (like a Databricks PAT token) from being exposed in pipeline logs?* — Store them as secret variables in a variable group linked to Azure Key Vault, reference them via pipeline variables (never hardcode), and Azure Pipelines automatically masks secret variable values in log output.
3. *How would you structure a multi-environment (Dev/Test/Prod) deployment pipeline for Databricks Workflows?* — Use environment-specific variable groups/parameter files (different workspace URLs, cluster configs, catalog names per environment), a single YAML pipeline parameterized by stage, with approval gates (manual or automated) before the Prod stage runs.
4. *What would you include in the "Test" stage of a Databricks CI/CD pipeline besides basic unit tests?* — Data quality/schema validation tests, integration tests against a small sample dataset in a dev workspace, and possibly a linting step (e.g., `flake8`/`black`) for code style consistency.

### One-Line Revision
**YAML pipelines turn "click through the UI to deploy" into version-controlled, reviewable, repeatable automation.**

---

## 3. Release Strategy & Deployment Patterns

### Quick Theory
Common patterns: **Blue-Green** (two identical environments, switch traffic once new version is verified), **Canary** (roll out to a small subset first, expand gradually), **Rolling** (gradual replacement). For data pipelines specifically, "deployment" often means promoting notebook code + Job/Workflow configuration across environments rather than swapping live traffic.

### Interview Q&A
1. *How would a canary-style rollout apply to a Databricks pipeline change?* — Deploy the updated pipeline logic to process a small subset of data/clients first (or run in parallel writing to a shadow table), validate output correctness against the old logic, then gradually expand to full production traffic once confidence is established.
2. *What approval gates would you put before a production deployment for a trading platform?* — Automated test pass requirement, a manual approval step from a lead/architect (especially for schema or business-logic changes affecting financial calculations), and a scheduled deployment window avoiding peak trading hours.
3. *How do you roll back a bad Databricks Workflow deployment?* — Redeploy the previous known-good notebook/Job configuration version from git history (since it's version-controlled), and if bad data was already written, use Delta Lake's `RESTORE TABLE ... TO VERSION AS OF` to roll back affected tables to their pre-deployment state.

### One-Line Revision
**Release strategy is about controlling blast radius — validate small before going wide, and always keep a rollback path.**

---

## PART B: Performance Optimization

## 4. Shuffle, Skew, Broadcast Join, AQE

*(Shuffle and Broadcast Join are covered in depth in file 03 — this section adds skew and AQE specifics.)*

### Quick Theory
**Skew**: uneven distribution of data across partitions (e.g., one `account_id` has 10x more trades than others), causing one task to take far longer than its peers — a classic straggler problem. **Adaptive Query Execution (AQE)**: Spark 3.x+ feature that re-optimizes the physical plan at runtime based on actual data statistics observed during execution — can auto-handle skew (splitting skewed partitions), auto-switch join strategies, and auto-coalesce shuffle partitions.

### Real-Life Analogy
Skew is like a **grocery store where one checkout lane gets 10x more customers** than the others because everyone thinks it's fastest — that one lane (task) becomes the bottleneck for the whole store to close (job completion), even though every other lane finished long ago.

### Interview Q&A
1. *How do you detect data skew in a Spark job?* — In the Spark UI, look at the stage's task duration distribution — a skewed stage shows most tasks finishing quickly while one or a few tasks take dramatically longer (visible as a long tail in the task timeline).
2. *What are two ways to fix a skewed join?* — (1) **Salting** — add a random suffix to the skewed key on both sides of the join to spread it across more partitions, then aggregate results back; (2) enable **AQE's skew join optimization** (`spark.sql.adaptive.skewJoin.enabled`), which automatically detects and splits oversized partitions at runtime.
3. *What does AQE's "coalesce shuffle partitions" feature do?* — After a shuffle, if the resulting partitions turn out much smaller than expected (based on actual runtime data size, not static estimates), AQE automatically merges them into fewer, larger partitions to avoid excessive small-task overhead.
4. *Is AQE enabled by default in modern Databricks Runtime?* — Yes, in recent Databricks Runtime versions AQE is enabled by default, though it's worth confirming/tuning specific thresholds (like the skew join detection thresholds) for particularly skewed production workloads.

### One-Line Revision
**Skew makes one task the bottleneck for everyone; AQE (or manual salting) spreads the load back out.**

---

## 5. File Size & the Small File Problem

### Quick Theory
Spark performs best with a moderate number of reasonably-large files (roughly 128MB–1GB range per file, not a hard rule) — too many tiny files create excessive metadata/scheduling overhead per task; too few giant files limit parallelism.

### Interview Q&A
1. *Why is having millions of tiny files bad for a Delta table's performance?* — Each file requires separate metadata tracking and a task to open/read it — with millions of tiny files, the overhead of managing and scheduling all those tiny tasks can dwarf the actual data processing time, and listing/planning against the transaction log becomes slower too.
2. *What Delta Lake feature directly addresses the small file problem?* — `OPTIMIZE`, which compacts many small files into fewer, appropriately-sized larger files; can be combined with `ZORDER` for improved data skipping on top of compaction.
3. *What streaming pattern tends to create the small file problem in the first place?* — Frequent micro-batch writes (e.g., a streaming job writing every few seconds) each produce a new small file per partition — mitigated by tuning trigger intervals or scheduling periodic `OPTIMIZE` jobs to compact the accumulated small files.

### One-Line Revision
**Too many tiny files drown Spark in overhead — OPTIMIZE compaction is the fix.**

---

## 6. Spark UI & Cluster Tuning

### Quick Theory
The Spark UI (accessible per-cluster in Databricks) shows **Jobs, Stages, Tasks, Storage, Executors, and SQL** tabs — the primary tool for diagnosing slow or failing Spark jobs. Cluster tuning involves right-sizing driver/executor memory and cores, choosing appropriate instance types, and setting shuffle partition counts appropriately for data volume.

### Interview Q&A
1. *A job is failing with "OutOfMemoryError." Where do you look first and what do you check?* — Spark UI's Executors tab for memory usage patterns, and Stages tab to identify which stage triggered it — common causes include a skewed join/groupBy exploding one partition's data, an oversized broadcast join, or simply undersized executor memory for the data volume; also check for excessive caching holding onto memory unnecessarily.
2. *How do you decide the right number of shuffle partitions (`spark.sql.shuffle.partitions`)?* — Base it on data volume and cluster size — too few creates large, slow partitions that don't parallelize well; too many creates excessive small-task scheduling overhead; with AQE enabled, Spark can often auto-tune this at runtime rather than relying purely on a static config.
3. *What's the difference between driver and executor memory, and what happens if the driver runs out?* — The driver coordinates the job and collects final results (and runs `.collect()`/`.toPandas()` results into its own memory); executors do the actual distributed processing. If the driver runs out of memory — commonly from calling `.collect()` on a huge dataset — the whole job crashes since there's only one driver, unlike executor OOMs which might only fail one task/partition.

### One-Line Revision
**Spark UI is the black-box flight recorder — always check Stages/Executors before guessing at a fix.**

---

## PART C: Monitoring, Logging, Alerting

## 7. Monitoring Databricks Jobs in Production

### Quick Theory
Beyond the Azure Monitor/Log Analytics integration covered in file 01, Databricks itself provides **Job Run history** (success/failure, duration trends per run) and can trigger **email/webhook notifications** on job start/success/failure directly from the Workflow configuration — often the first line of monitoring before deeper Log Analytics/KQL investigation.

### Interview Q&A
1. *What's the simplest way to get notified when a production Databricks job fails, without setting up Log Analytics?* — Configure email/webhook notifications directly on the Job/Workflow definition (on-failure alerts), which Databricks sends natively without needing external log shipping — good for a quick first layer of alerting.
2. *How would you build a dashboard showing job duration trends over the last 30 days?* — Query the Databricks Jobs API (or the exported job-run logs in Log Analytics) for historical run durations, and visualize with Databricks SQL dashboards or Power BI connected to that data — helps catch gradual performance degradation before it becomes a hard failure.
3. *What's "root cause analysis" and how would you approach it for a recurring intermittent job failure?* — Systematically investigating the underlying reason behind a failure rather than just fixing the symptom — start with the actual error/stack trace in the driver log, correlate timing with data volume or upstream system changes, check for patterns (same time of day, same data source), and verify whether it's transient (retry logic can handle it) or a genuine bug/capacity issue needing a permanent fix.

### One-Line Revision
**Start with Databricks' own job-failure notifications; escalate to Log Analytics/KQL for deeper trend and root-cause analysis.**

---

## PART D: Security & Governance

## 8. Encryption

### Quick Theory
Data should be encrypted **at rest** (ADLS Gen2 encrypts by default using Microsoft-managed or customer-managed keys via Key Vault) and **in transit** (TLS between Databricks, ADLS, and other Azure services).

### Interview Q&A
1. *What's the difference between Microsoft-managed keys and customer-managed keys for encryption at rest?* — Microsoft-managed keys are handled entirely by Azure with no customer involvement (simplest, default); customer-managed keys (stored in the customer's own Key Vault) give the customer control over key rotation and the ability to revoke access by disabling the key — often a compliance requirement for regulated industries like finance.
2. *Is data encrypted in transit by default between Databricks and ADLS Gen2?* — Yes, communication uses TLS/HTTPS by default via the `abfss://` protocol, which is the secure variant of the ADLS Gen2 connection protocol.

### One-Line Revision
**Encrypt at rest and in transit by default; use customer-managed keys when compliance demands direct control.**

---

## 9. Data Governance (tying RBAC + Key Vault + Unity Catalog together)

### Quick Theory
Governance combines: **who can access what** (RBAC/Unity Catalog grants), **how secrets are managed** (Key Vault), **what happened to the data** (lineage/audit logs), and **compliance policies** (retention, PII handling, regional data residency) — all critical in a regulated trading platform.

### Interview Q&A
1. *How would you design governance for a multi-client SaaS trading data platform where clients must never see each other's data?* — Isolate at the catalog or schema level per client in Unity Catalog, enforce RBAC grants scoped per client, use dynamic views/row-level security if any shared infrastructure exists, and rely on Unity Catalog's audit logs to prove isolation during compliance reviews.
2. *What role does data lineage play in a security incident investigation?* — It lets you quickly trace exactly which jobs/notebooks/users touched a given table or record, critical for scoping the impact of a potential breach or data quality incident and for demonstrating compliance to auditors.
3. *How would you handle a request to permanently delete a specific customer's data across Bronze/Silver/Gold (e.g., a "right to be forgotten" request)?* — Use Delta Lake's `DELETE` command across all three layers where that customer's data exists, followed by running `VACUUM` (after evaluating retention/time-travel implications) to physically purge the underlying files, and verify via lineage that no downstream aggregate/derived table still retains that data.

### One-Line Revision
**Governance = RBAC (who), Key Vault (secrets), lineage (what happened), and clear retention/deletion policy (compliance) working together.**

---

Next file: `06_scenario_questions_100plus.md`
