# 07 — HR/Behavioral Questions + Final Revision Materials

---

## PART A: HR & Behavioral Questions

General approach: use the **STAR method** (Situation, Task, Action, Result) for anything asking about a past experience. Keep each answer to 60–90 seconds. Since you're newer to hands-on Data Engineering, be honest but confident — frame answers around what you *have* done (labs, personal projects, prior role responsibilities) and your deep conceptual preparation, rather than fabricating years of experience you don't have.

### 1. Tell me about yourself.
**Structure**: current situation → relevant background → why this role.
> "I'm a data engineer with a strong foundation in Python, SQL, and cloud data platforms, and I've spent recent time going deep on Azure Databricks — building pipelines with PySpark and Delta Lake, understanding the medallion architecture, and learning how CI/CD ties into data engineering through Azure DevOps. I'm particularly drawn to this role because it's a SaaS data layer for a trading platform, which means the pipelines need to be both performant and extremely reliable — exactly the kind of problem I enjoy working on. I'm looking for a team where I can keep building that expertise hands-on."

### 2. Explain your current/most recent project.
Pick a real project (even a personal/learning project if needed) and describe it using: what data, what pipeline stages, what tools, what problem it solved, one challenge you overcame. Be ready for 2-3 follow-up technical questions on it — only mention details you can defend.

### 3. What's the biggest challenge you've faced in a data engineering context?
Pick something specific and technical (e.g., "a job that kept failing intermittently and I had to trace it back to data skew") rather than a vague answer like "learning new things is challenging" — specificity signals real experience.

### 4. Tell me about a production issue you solved.
Use STAR: describe the symptom, your diagnostic process (what you checked, in what order), the root cause, the fix, and what you changed to prevent recurrence (this last part — prevention — is what separates senior-sounding answers from junior ones).

### 5. Tell me about a conflict with a teammate.
Focus on a disagreement over a *technical approach*, not a personality conflict. Show you listened to the other view, found common ground or a data-driven way to decide, and preserved the relationship. Avoid blaming anyone.

### 6. How do you handle deadline pressure?
> "I prioritize by identifying what's actually blocking others or has the highest business impact, communicate early if something's at risk rather than going silent, and I'm willing to have an honest conversation about scope versus timeline rather than silently over-promising and delivering something broken."

### 7. Why Databricks?
> "Because it unifies data engineering, and increasingly analytics and ML, in one platform with strong governance through Unity Catalog and excellent performance through Delta Lake and Photon — it removes a lot of the operational overhead of managing Spark clusters yourself, and it's become the de facto standard for Lakehouse architectures on Azure."

### 8. Why Azure (over AWS/GCP)?
> "Azure has particularly strong enterprise/identity integration (Azure AD, Managed Identity, RBAC) which matters a lot for regulated industries like finance, plus Azure Databricks is a first-party integrated offering, not a bolt-on marketplace product — that tight integration simplifies security and networking design significantly."

### 9. Why should we hire you?
Focus on: genuine enthusiasm for the specific problem domain (trading platform data reliability), strong fundamentals you can demonstrate in the interview itself, and being a fast, hands-on learner who goes deep rather than surface-level. Don't oversell false experience — let your technical answers in the interview do that proving for you.

### 10. Where do you see yourself in 3-5 years?
> "Growing from strong hands-on pipeline development into broader architecture and design decisions — this role's mention of producing architectural diagrams and technical documentation is exactly the kind of growth path I'm looking for, moving from implementing to also designing solutions."

### 11. How do you keep your skills current with fast-changing cloud technology?
Mention specific habits: following Databricks/Azure release notes, hands-on experimentation (Community Edition, personal projects), and reading real production post-mortems/case studies rather than only official docs.

### 12. Describe a time you had to learn a new technology quickly.
Use a concrete example with STAR — what forced the fast learning, how you approached it (docs, hands-on practice, asking for help appropriately), and the outcome.

---

## PART B: The One-Day Revision Sheet

*(Use this the day before the interview — read once fully, then again skimming just the bolded lines.)*

### Morning Block (2 hrs) — Databricks + Delta Lake
- Databricks = Control Plane (managed) + Data Plane (your subscription).
- Job clusters for production (isolated, cost-attributed); pools for fast startup; autoscaling for variable load.
- Unity Catalog = account-wide governance: `catalog.schema.table`, RBAC-style grants, lineage, audit.
- **Delta Lake = Parquet + `_delta_log` transaction log** → gives ACID, time travel, schema enforcement.
- MERGE = upsert; OPTIMIZE = compact small files; ZORDER = cluster data for skipping; VACUUM = physically delete old files (breaks time travel beyond retention).
- Medallion: Bronze (raw, replayable) → Silver (clean, conformed) → Gold (business-ready aggregates).

### Midday Block (2 hrs) — PySpark + SQL
- Transformations = lazy (build plan); Actions = trigger execution; DAG = the optimized execution graph.
- Narrow transformation = no shuffle (filter, select); Wide = shuffle (groupBy, join).
- Repartition = full shuffle, even; Coalesce = merge partitions, cheap, uneven.
- Broadcast join = small table copied to every executor; use for big-small joins.
- Window functions: `PARTITION BY` + `ORDER BY` keep rows but add per-row aggregate/rank context.
- Prefer built-in functions > Pandas UDF > row-at-a-time UDF, for performance.
- SQL logical execution order: FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY.

### Afternoon Block (2 hrs) — ETL Patterns + Architecture
- ELT (not ETL) is the modern Lakehouse pattern: load raw, transform inside Databricks.
- CDC = row-level change stream from source transaction log (vs comparing full snapshots).
- SCD Type 1 = overwrite (no history); SCD Type 2 = new row per change + effective/end dates (full history).
- Watermarking = how long streaming waits for late data before finalizing a window.
- Idempotency = safe to re-run without duplicating/corrupting results — critical for financial data.
- Batch = simple, scheduled; Streaming = continuous, complex, needed only when real-time latency is required.

### Evening Block (1.5 hrs) — DevOps, Security, Scenarios
- YAML pipelines = version-controlled CI/CD; test stage before deploy stage; secrets from Key Vault, never hardcoded.
- Skew = one task way slower than peers → salting or AQE skew join fix.
- Small file problem → fixed by OPTIMIZE.
- Managed Identity = passwordless auth for resources; Key Vault = centralized secret storage; RBAC = least-privilege scoped roles.
- Re-skim 10-15 scenario questions from file 06, focusing on ones you found hardest.

---

## PART C: The 30-Minute Last-Minute Revision Guide

*(Read this right before the interview — just before the call or in the waiting room.)*

1. **Your one-sentence positioning**: "I build and run Databricks pipelines that pull trading data into a Bronze/Silver/Gold Lakehouse using PySpark and Delta Lake, ship them through Azure DevOps CI/CD, and keep them secure and observable with Key Vault, RBAC, and Log Analytics."
2. **Delta Lake in one breath**: "Parquet files plus a transaction log (`_delta_log`) that gives ACID transactions, time travel, and safe upserts via MERGE."
3. **Medallion in one breath**: "Bronze is raw and replayable, Silver is clean and conformed, Gold is business-ready aggregates."
4. **The universal scenario-answer shape**: name the likely cause → say where you'd look (Spark UI / DESCRIBE HISTORY / Log Analytics / query plan) → state the fix → mention prevention.
5. **If asked something you don't know**: say what you *do* know that's adjacent, state your reasoning process, and be honest you'd verify/research the specific detail — don't guess confidently and get caught out; interviewers respect calibrated honesty far more than bluffing.
6. **Breathe. Speak slower than feels natural.** Most candidates rush when nervous, which makes technical explanations harder to follow — you have more time than you think.

---

## PART D: Cheat Sheet (quick reference table)

| Concept | One-line definition |
|---|---|
| ADLS Gen2 | Blob storage + hierarchical namespace = real folders for big data |
| Managed Identity | Password-less Azure AD identity for a resource |
| Key Vault | Centralized, audited secret/key/cert store |
| RBAC | Roles scoped to resources, least-privilege access |
| Log Analytics | KQL-queryable log/metric store behind Azure Monitor |
| Service Bus | Ordered, transactional messaging (queues/topics) |
| Azure Functions | Serverless, event-triggered glue code |
| Databricks Control Plane | Databricks-managed: UI, scheduler, metadata |
| Databricks Data Plane | Your subscription: actual VMs/clusters processing data |
| Job Cluster | Spins up per job, terminates after — production default |
| Pool | Pre-warmed VMs for fast cluster startup |
| Unity Catalog | Account-wide governance: `catalog.schema.table`, grants, lineage |
| Delta Lake | Parquet + transaction log = ACID, time travel, schema enforcement |
| MERGE | Upsert: update matched, insert unmatched |
| OPTIMIZE | Compacts small files into larger ones |
| ZORDER | Clusters data by column for faster data skipping |
| VACUUM | Physically deletes old unreferenced files (breaks time travel beyond retention) |
| Medallion | Bronze (raw) → Silver (clean) → Gold (aggregated) |
| Transformation | Lazy — builds the plan (filter, select, groupBy) |
| Action | Triggers execution (count, show, write) |
| Narrow transformation | No shuffle (filter, select) |
| Wide transformation | Shuffle required (groupBy, join, distinct) |
| Repartition | Full shuffle, evenly redistributes |
| Coalesce | Merges existing partitions, cheap, no shuffle, can be uneven |
| Broadcast Join | Small table copied to every executor — no shuffle of big table |
| AQE | Runtime re-optimization: fixes skew, join strategy, partition sizing |
| Window Function | Per-row aggregate/rank without collapsing rows (`OVER (PARTITION BY...)`) |
| Pandas UDF | Vectorized (batch) UDF — faster than row-at-a-time Python UDF |
| ELT | Load raw first, transform inside the powerful engine (modern Lakehouse pattern) |
| CDC | Row-level change stream from source transaction log |
| SCD Type 1 | Overwrite — no history kept |
| SCD Type 2 | New row per change — full history with effective/end dates |
| Watermark | How long streaming waits for late data before finalizing a window |
| Idempotency | Safe to re-run without producing a different/duplicated result |
| YAML Pipeline | Version-controlled CI/CD definition (Azure Pipelines) |
| Blue-Green / Canary | Deployment strategies to limit blast radius of a bad release |
| Data Skew | Uneven data distribution causing a slow straggler task |
| Small File Problem | Too many tiny files → excessive metadata/scheduling overhead |

---

## PART E: Mind Map

See the mind map in `00_START_HERE_index_and_study_plan.md` — re-read it once tonight and once before the call. The shape to remember:

**Compute (Databricks/clusters) + Data (Delta Lake/Medallion) + Operations (DevOps/Monitoring/Security)** — every JD requirement fits into one of those three buckets.

---

## PART F: Top 100 Most Likely Interview Questions — Ranked

*(Ranked by how directly the JD emphasizes the underlying skill. "Must have" JD items rank highest.)*

### Tier 1 — Near-certain to be asked (Databricks + PySpark + Delta core, JD's #1 emphasis)
1. What is Delta Lake and how does it differ from plain Parquet?
2. Explain the Medallion Architecture (Bronze/Silver/Gold) and why each layer exists.
3. What is MERGE (upsert) in Delta Lake and when do you use it?
4. What's the difference between OPTIMIZE and VACUUM?
5. Explain lazy evaluation and the difference between transformations and actions.
6. What's a broadcast join and when does Spark choose one automatically?
7. What's the difference between repartition and coalesce?
8. Explain Delta Lake's ACID guarantees and how the transaction log provides them.
9. What is Time Travel in Delta Lake and give a use case.
10. Walk me through how you'd design an end-to-end pipeline in Azure Databricks for [trading data / a given scenario].
11. What's data skew and how do you detect and fix it?
12. Explain window functions and how they differ from groupBy.
13. What is Unity Catalog and what problem does it solve?
14. Job cluster vs all-purpose cluster — which for production and why?
15. Explain incremental load vs full load, and how you'd implement incremental.
16. What is CDC (Change Data Capture) and how would you apply CDC events to a Delta table?
17. Explain SCD Type 1 vs Type 2 with an example.
18. How do you handle late-arriving data and what is watermarking?
19. What makes a pipeline idempotent, and why does it matter?
20. How would you troubleshoot a slow Spark job? (walk through your process)

### Tier 2 — Very likely (Azure DevOps, PySpark depth, SQL)
21. Explain your Git branching strategy for a data engineering team.
22. Walk me through a typical Azure Pipelines YAML CI/CD flow for Databricks.
23. How do you manage secrets (like Databricks tokens) securely in a CI/CD pipeline?
24. What's Adaptive Query Execution (AQE) and what problems does it solve?
25. What's the small file problem and how do you prevent/fix it?
26. Explain SQL execution order (FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY).
27. Write SQL to get the latest record per key using window functions.
28. What's a Pandas UDF and why is it faster than a regular UDF?
29. Explain caching/persist and when you'd use it.
30. What's the difference between LEFT JOIN, LEFT SEMI JOIN, and LEFT ANTI JOIN?
31. How would you design retry logic for a pipeline — what should and shouldn't be retried?
32. How do you validate data quality between pipeline stages?
33. Explain schema enforcement vs schema evolution in Delta Lake.
34. What's Z-ORDER and how do you choose which column to Z-order by?
35. What's the difference between ETL and ELT, and which does a Lakehouse use?
36. Explain batch vs streaming and when you'd choose each.
37. How do you handle duplicate records arriving into a pipeline?
38. What's a CTE and when would you use one over a subquery?
39. Explain Managed Identity and why it's preferred over storage account keys.
40. What's Azure Key Vault and how does Databricks integrate with it?

### Tier 3 — Likely (architecture, monitoring, security, scenario depth)
41. Walk me through the Databricks Control Plane vs Data Plane.
42. How would you design multi-client data isolation on a shared SaaS platform?
43. What's RBAC and how does it differ from ADLS ACLs?
44. How do you monitor Databricks jobs in production (Log Analytics/alerts)?
45. What's a cluster policy and why would an enterprise enforce one?
46. How would you design a streaming pipeline from Event Hub/Service Bus into Delta Lake?
47. What's the role of Azure Functions in an event-driven Databricks architecture?
48. How would you roll back a bad production deployment?
49. Explain Photon and when it helps (or doesn't).
50. What is DBFS and why are Unity Catalog Volumes preferred now?
51. How would you approach root cause analysis for a recurring job failure?
52. Explain the difference between Data Warehouse, Data Lake, and Lakehouse.
53. How would you design a Gold table for a client-facing daily P&L report?
54. What's a dead-letter queue and why does it matter for financial messaging?
55. How do you decide the right partition column for a huge Delta table?
56. What's the difference between a metric and a log in Azure Monitor?
57. How would you handle a "right to be forgotten" data deletion request across Bronze/Silver/Gold?
58. Explain data lineage and why Unity Catalog's lineage matters for audits.
59. What's the difference between Blue-Green and Canary deployment strategies?
60. How would you secure Databricks-to-storage network traffic (VNet injection, Private Link)?

### Tier 4 — Possible / nice-to-have areas & depth probes
61. What is MLflow and what are its main components?
62. How would you use Power BI with a Databricks Gold table?
63. What's the difference between System-assigned and User-assigned Managed Identity?
64. Explain Delta Live Tables — how is it different from a regular Workflow?
65. What's the difference between Hot, Cool, and Archive storage tiers?
66. When would you use Service Bus over Event Hub for ingestion?
67. What's exponential backoff and why not just retry immediately?
68. How do you handle out-of-order CDC events?
69. Explain the difference between RANK() and DENSE_RANK().
70. What's the difference between a view and a materialized view?
71. How does Delta Lake handle concurrent writers (optimistic concurrency)?
72. What's a checkpoint file in Delta Lake's transaction log?
73. How would you test a schema change before deploying to production?
74. What's the difference between customer-managed and Microsoft-managed encryption keys?
75. How do you decide memory-optimized vs compute-optimized cluster instances?

### Tier 5 — Deep scenario/debugging probes (draw from file 06 as needed)
76. A Delta table query fails with "file not found" — what happened?
77. A job that ran fine in Dev times out in Prod at 100x scale — why?
78. Two jobs write to the same Delta table concurrently and one fails — what do you do?
79. A streaming job's micro-batch processing time keeps creeping up — why?
80. A shared cluster is randomly slow at unpredictable times — what's the likely cause?
81. A job "succeeds" but silently produces zero rows — how do you catch that?
82. A client asks for proof their data was never accessible to another client — how do you demonstrate it?
83. A security review finds overly broad service principal permissions — how do you remediate?
84. Data costs tripled with no business growth — how do you investigate?
85. A downstream report shows unexpectedly different numbers than before — how do you trace it?

### Tier 6 — HR/Behavioral (always asked in some form)
86. Tell me about yourself.
87. Why this role / why this company?
88. Why Databricks? Why Azure?
89. Tell me about a production issue you solved.
90. Tell me about a time you disagreed with a teammate.
91. How do you handle tight deadlines?
92. Describe your current/most recent project in detail.
93. What's your biggest technical weakness and how are you addressing it?
94. How do you stay current with fast-moving cloud technology?
95. Where do you see yourself in 3-5 years?
96. Why should we hire you?
97. Describe a time you had to learn something new quickly under pressure.
98. How do you handle being told your approach is wrong?
99. Tell me about a time you had to explain something technical to a non-technical stakeholder.
100. Do you have any questions for us? *(Always have 2-3 genuine ones ready — e.g., "What does the current CI/CD maturity look like for the Databricks pipelines?" or "How is the team currently handling data quality validation?" — questions that show you're already thinking like someone on the team.)*

---

## Final Note

You asked for material to help you **crack this interview**, not just learn concepts — so here's the honest meta-point: interviewers for a "10+ years" JD staffing a beginner-level candidate will almost certainly probe hard on **depth of understanding over breadth of buzzwords**. The structure of this whole prep kit (analogy → technical → follow-ups → scenarios) exists specifically to get you past the first surface-level answer into the follow-up questions, because that's where real signal comes from in these interviews. Good luck.
