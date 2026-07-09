# Azure Databricks Developer — Interview Prep Master Index

## How this prep kit is organized

You have 8 files. Read them **in this order** — each builds on the last:

| # | File | Covers | Study time |
|---|------|--------|-----------|
| 1 | `01_azure_platform_services.md` | ADLS Gen2, Storage Accounts, Networking, Managed Identity, Key Vault, Azure Monitor, RBAC, Log Analytics, Service Bus, Azure Functions | 3–4 hrs |
| 2 | `02_databricks_and_delta_lake.md` | Workspace, Clusters, Pools, Jobs/Workflows, Unity Catalog, Notebooks, Repos, MLflow, Photon, DBFS, Autoscaling, Delta Lake (ACID, time travel, merge, optimize, vacuum, Z-order), Medallion Architecture | 5–6 hrs |
| 3 | `03_pyspark_and_sql.md` | DataFrame API, transformations/actions, lazy eval, DAG, partitioning, joins, window functions, caching, UDFs, performance + SQL joins/CTE/views/indexes/window functions/execution order | 5–6 hrs |
| 4 | `04_etl_patterns_and_architecture.md` | Incremental/full load, CDC, SCD Type 1 & 2, watermarking, idempotency, validation, error handling, retries, batch vs streaming, event-driven design, end-to-end + trading-platform architecture diagrams | 4–5 hrs |
| 5 | `05_devops_cicd_performance_monitoring_security.md` | Git/branching/PRs, Azure DevOps pipelines, YAML, release strategy, shuffle/skew/broadcast/AQE, Spark UI, cluster tuning, Log Analytics, RBAC, Key Vault, encryption, governance | 4 hrs |
| 6 | `06_scenario_questions_100plus.md` | 100+ real "what would you do if..." questions | 3 hrs |
| 7 | `07_hr_behavioral_and_final_revision.md` | HR/behavioral answers + 1-day revision sheet + 30-min last-minute guide + cheat sheet + top 100 ranked questions | 2 hrs |
| 8 | This file | Index, mind map, study plan | 15 min |

**Total realistic prep time: ~3–4 full days** if you're starting from basics. If you have less time, jump straight to file 7 (final revision) after skimming 2 and 3, since Databricks + PySpark carry the most interview weight for this JD.

---

## Why this order?

The JD is for a **trading platform SaaS data layer** built almost entirely on **Azure Databricks pipelines**. The interviewer's mental model will be:

```
Data arrives (upstream systems / trading feeds)
        │
        ▼
   Azure Databricks (PySpark + Delta Lake)  ◄── 70% of interview weight
        │
        ▼
   Bronze → Silver → Gold (Medallion Architecture)
        │
        ▼
   Downstream systems / reporting
        │
   CI/CD via Azure DevOps ──► deploys the above
   Log Analytics / Monitor ──► watches the above
   Key Vault / RBAC ────────► secures the above
```

So: learn Databricks + PySpark + Delta Lake first (the actual product being built), then ETL patterns (how data moves through it), then DevOps/Security/Monitoring (how it's shipped and kept healthy), then drill scenarios and HR.

---

## The One Mind Map (memorize this shape, not the words)

```
                         ┌─────────────────────────┐
                         │   AZURE DATABRICKS       │
                         │   DEVELOPER ROLE         │
                         └────────────┬─────────────┘
                 ┌────────────────────┼────────────────────┐
                 │                    │                     │
          ┌──────▼──────┐    ┌────────▼────────┐   ┌────────▼────────┐
          │   COMPUTE    │    │      DATA        │   │   OPERATIONS    │
          │  Databricks  │    │  Delta Lake +     │   │  DevOps, CI/CD, │
          │  Clusters,   │    │  ADLS Gen2 +      │   │  Monitoring,    │
          │  Jobs,       │    │  Medallion        │   │  Security       │
          │  Pools,      │    │  (Bronze/Silver/  │   │                 │
          │  Photon,     │    │  Gold)            │   │                 │
          │  Autoscale   │    │                   │   │                 │
          └──────┬───────┘    └─────────┬─────────┘   └────────┬────────┘
                 │                      │                       │
          ┌──────▼──────┐      ┌────────▼────────┐    ┌─────────▼────────┐
          │  PROCESSING  │      │   PATTERNS      │    │   PLUMBING       │
          │  PySpark:    │      │  CDC, SCD1/2,   │    │  Service Bus,    │
          │  DataFrame,  │      │  Incremental,   │    │  Functions,      │
          │  joins,      │      │  Watermarking,  │    │  Key Vault,      │
          │  window fns, │      │  Idempotency    │    │  Managed ID,     │
          │  UDFs,       │      │                 │    │  RBAC, Log       │
          │  caching     │      │                 │    │  Analytics       │
          └──────────────┘      └─────────────────┘    └──────────────────┘
```

**One sentence to remember the whole role:**
> "I build and run Databricks pipelines that pull trading data into a Bronze/Silver/Gold Lakehouse using PySpark and Delta Lake, ship those pipelines through Azure DevOps CI/CD, and keep them secure and observable with Key Vault, RBAC, and Log Analytics."

Say a version of that sentence in your first answer to "tell me about your experience" — it signals you understand the whole system, not just isolated tools.

---

## Study tips specific to you (beginner → interview-ready)

1. **Don't memorize definitions. Memorize the analogy + the one-line revision** at the end of each topic. Definitions crumble under follow-up questions; mental models don't.
2. For every "Interview Answer" in these files, **read it out loud once**. Interviews are spoken, not written — reading silently gives false confidence.
3. When you hit a code example, **run it yourself** in a free Databricks Community Edition or local PySpark shell if you can. Even 30 minutes of hands-on beats hours of reading.
4. The JD explicitly mentions **10+ years experience, but you're a beginner** — so your honest positioning in interview should be: strong hands-on fundamentals + rapid learner + you've *studied the architecture deeply* even if your production hours are fewer. Don't lie about years of experience; instead over-index on depth of understanding. Interviewers can tell the difference between memorized answers and real understanding within 2 follow-up questions — these files train you for the follow-ups too.
5. Re-read file 7 (final revision) the morning of the interview. Re-read the 30-minute guide in the waiting room / just before the call.

Now go to `01_azure_platform_services.md`.
