# 01 — Azure Platform Services

Covers: ADLS Gen2 · Storage Accounts · Azure Networking · Managed Identity · Key Vault · Azure Monitor · RBAC · Log Analytics · Service Bus · Azure Functions

---

## 1. Azure Data Lake Storage Gen2 (ADLS Gen2)

### Quick Theory
ADLS Gen2 is Microsoft's cloud storage built for big data. It's Azure Blob Storage with a **hierarchical namespace** turned on — meaning it behaves like a real folder structure (directories, sub-directories) instead of Blob Storage's flat "container + key" model. It's the default storage layer under almost every Azure Databricks Lakehouse.

- **Why used**: Cheap, virtually unlimited storage; separates storage from compute (you can shut down clusters but keep data); hierarchical namespace makes directory operations (rename, delete a folder) fast and atomic instead of simulated.
- **Where used**: Landing zone for raw data (Bronze), and storage for Silver/Gold Delta tables.
- **Benefits**: Massive scale, low cost, integrates natively with Databricks (`abfss://` paths), supports fine-grained POSIX-like ACLs.
- **Limitations**: Not a database — no built-in transactions/indexing (that's what Delta Lake adds on top); throughput throttling exists per storage account/partition and can bottleneck huge parallel jobs if not planned for.

### Real-Life Analogy
Think of ADLS Gen2 as a giant **warehouse (Amazon fulfillment center)**. Blob storage (without hierarchical namespace) is like one giant room where every box has a barcode but no aisles — finding "all boxes in aisle 4" means scanning every barcode. ADLS Gen2 adds actual **aisles and shelves** (folders), so "move aisle 4 to aisle 9" is one instruction, not moving every box individually.

### Technical Explanation
```
Storage Account
   └── Container (Filesystem)
         └── /raw/trading/orders/2026/07/09/*.parquet
         └── /bronze/...
         └── /silver/...
         └── /gold/...
```
- Access via `abfss://container@account.dfs.core.windows.net/path`
- Security: Azure AD + RBAC at storage-account level, or POSIX ACLs at folder/file level for finer control.
- Redundancy options: LRS, ZRS, GRS, RA-GRS — trade-off between cost and disaster-recovery guarantees.
- Best practice: partition folder structure by date (`/yyyy/mm/dd/`) so downstream jobs can prune easily; avoid millions of tiny files (the "small file problem") — compact with Delta `OPTIMIZE`.
- Common mistake: using flat Blob Storage (no hierarchical namespace) for a Lakehouse — loses atomic rename and slows Spark job commits.

### Interview Q&A

**Beginner**
1. *What is ADLS Gen2 and how is it different from Blob Storage?*
   ADLS Gen2 is Blob Storage with hierarchical namespace enabled, so it supports real directories with atomic rename/delete, POSIX-style ACLs, and is optimized for analytics workloads — whereas plain Blob Storage is a flat key-value store.
2. *Why is ADLS Gen2 used with Databricks instead of a normal database?*
   Because Databricks needs to process huge, often semi-structured datasets cheaply at scale; a traditional database can't hold petabytes cost-effectively or scale compute independently of storage.
3. *What access protocol does Databricks use to talk to ADLS Gen2?*
   `abfss://` (Azure Blob File System Secure), which is the Databricks/Spark connector protocol for ADLS Gen2.

**Intermediate**
4. *How do you secure access from Databricks to ADLS Gen2 without hardcoding keys?*
   Using a Managed Identity or a Service Principal registered in Azure AD, combined with either Azure RBAC roles (e.g., Storage Blob Data Contributor) or ADLS POSIX ACLs, often mounted via Unity Catalog external locations/credentials rather than storing keys in notebooks.
5. *What's the "small file problem" and how do you fix it in ADLS Gen2?*
   Many small files (e.g., from micro-batch streaming) create huge metadata overhead and slow scans. Fixed by compacting files with Delta Lake's `OPTIMIZE` command or by controlling write batch sizes.
6. *What redundancy option would you choose for a critical trading dataset and why?*
   Likely GRS or RA-GRS for cross-region durability given financial data criticality, balanced against cost — I'd confirm the actual RPO/RTO requirement with the architecture team before deciding.

**Advanced / Architecture**
7. *How would you design the folder/container structure for a multi-client SaaS platform (as mentioned in this JD)?*
   Typically isolate by client at the container or top-level folder (`/client_id/bronze/...`), then by medallion layer, then by date partition — this supports per-client access control via ACLs/RBAC and per-client cost tracking.
8. *How does ADLS Gen2 support Delta Lake's ACID guarantees?*
   It doesn't provide ACID itself — Delta Lake layers a transaction log (`_delta_log`) on top of plain Parquet files in ADLS Gen2, using atomic file operations the hierarchical namespace supports to guarantee consistency.
9. *A downstream job says "file not found" intermittently reading from ADLS. What do you check?*
   Whether it's reading directly from ADLS paths during an active write (partial/uncommitted files) instead of reading through Delta Lake's transaction log which only exposes committed files — classic reason to always read via Delta, not raw Parquet folders.

**Scenario**
10. *Your storage costs suddenly spiked. How do you investigate?*
    Check for retained old snapshots/versions (Delta Lake `VACUUM` not running), redundant duplicate copies across environments, or unexpectedly large Bronze retention with no lifecycle policy — set up ADLS lifecycle management rules to auto-tier or delete old raw data.

### Practical Examples
- **Trading platform**: raw market tick data lands in `/bronze/trading/ticks/`, gets deduplicated/cleaned into `/silver/trading/orders/`, aggregated into `/gold/trading/daily_pnl/`.
- **Banking**: regulatory transaction logs stored with strict ACLs so only compliance teams can read raw PII data.
- **Retail**: clickstream events land hourly, partitioned by `/yyyy/mm/dd/hh/`.

### Common Interview Mistakes
- Confusing ADLS Gen2 with "a database" — it's storage, not a query engine.
- Saying "we use SAS tokens for everything" — modern best practice is Managed Identity / service principals + Unity Catalog credentials, not shared keys.
- Forgetting to mention the transaction log / Delta Lake link when asked "how do you get ACID on a file system?"

### One-Line Revision
**ADLS Gen2 = Blob Storage + real folders, and it's the "disk" that Delta Lake turns into a database.**

---

## 2. Azure Storage Accounts (general)

### Quick Theory
A Storage Account is the top-level Azure resource that provides access to Blob, File, Queue, and Table storage services, including ADLS Gen2 (Blob with hierarchical namespace). It's the billing/security boundary and holds the account keys, endpoints, and redundancy settings.

### Real-Life Analogy
If ADLS Gen2 is the warehouse, the **Storage Account is the whole shipping company** — the warehouse (Blob/ADLS) is one of its services, alongside message queues (Queue storage) and file shares (Azure Files).

### Technical Explanation
- Performance tiers: Standard vs Premium.
- Access tiers (for cost): Hot, Cool, Archive — move infrequently-read Bronze history to Cool/Archive to save cost.
- Security: Firewall rules, Private Endpoints (to keep traffic off the public internet), Shared Access Signatures (SAS), Azure AD auth.

### Interview Q&A
1. *What's the difference between Hot, Cool, and Archive tiers?* — Hot for frequently accessed data, Cool for infrequent access with lower storage cost but higher retrieval cost, Archive for rarely-accessed data with cheapest storage but slow (hours) rehydration — used for compliance-retained old trading records.
2. *How do you restrict a storage account to only be reachable from your VNet?* — Configure a Private Endpoint and disable public network access, so traffic stays on Microsoft's backbone network instead of the public internet.
3. *When would you use Standard vs Premium performance tier?* — Premium (SSD-backed) for latency-sensitive high-IOPS workloads; Standard for general Lakehouse batch/streaming workloads where throughput, not per-request latency, matters most.

### One-Line Revision
**Storage Account = the parent container that hosts ADLS Gen2, with tiers and network rules controlling cost and access.**

---

## 3. Azure Networking (for Databricks)

### Quick Theory
Databricks workspaces can be deployed in your own **VNet (VNet Injection)** so that cluster nodes get private IPs and can be locked down with Network Security Groups (NSGs), Private Link, and firewalls — essential for regulated industries like trading/finance.

### Real-Life Analogy
Public Databricks deployment is like a shop on the open street — anyone can walk by. VNet injection puts the shop **inside a gated business park** with a guard checking every car (NSG rules) before it reaches the shop.

### Technical Explanation
- **VNet Injection**: deploy Databricks control plane's data plane (clusters) into customer VNet subnets (public + private subnet pair).
- **NSGs**: control inbound/outbound traffic at subnet/NIC level.
- **Private Link/Endpoint**: keeps traffic between Databricks and ADLS/Key Vault off the public internet.
- **Common mistake**: forgetting Databricks needs specific outbound rules (to its control plane, PyPI, etc.) — over-locking down NSGs breaks cluster startup.

### Interview Q&A
1. *Why would a bank/trading company insist on VNet injection for Databricks?* — Compliance and security: it keeps all cluster traffic within a private network boundary, enables private connectivity to storage/Key Vault, and satisfies audit requirements that data planes not traverse the public internet.
2. *What's the difference between NSG and firewall?* — NSG operates at the subnet/NIC level with simple allow/deny rules based on IP/port/protocol; Azure Firewall is a managed, stateful, centralized service with more advanced rules (FQDN filtering, threat intelligence) typically used at the hub in a hub-spoke topology.
3. *A cluster fails to start after network changes. What do you check first?* — Outbound NSG rules to required endpoints (Databricks control plane, storage, PyPI/Maven if libraries are installed), DNS resolution, and whether a Private Endpoint route is missing from route tables.

### One-Line Revision
**Networking wraps Databricks in a private, audited perimeter — critical for finance-grade compliance.**

---

## 4. Managed Identity

### Quick Theory
A Managed Identity is an auto-managed Azure AD identity assigned to a resource (like a Databricks cluster or Azure Function) so it can authenticate to other Azure services **without any stored password/key/secret**. Two types: System-assigned (tied to one resource's lifecycle) and User-assigned (standalone, reusable across resources).

### Real-Life Analogy
Like an **employee ID badge** issued by the company itself — you don't carry a separate password for every door; the badge (identity) is trusted because the company (Azure AD) vouches for it, and it's automatically revoked when you leave (resource deleted).

### Interview Q&A
1. *Why use Managed Identity instead of storing a storage account key in a notebook?* — Avoids secret sprawl and rotation headaches; access is tied to Azure AD identity and can be scoped/audited via RBAC, and there's no credential to leak if code is shared or committed accidentally.
2. *System-assigned vs User-assigned — when would you pick each?* — System-assigned when the identity's life should match one resource exactly (deleted with it); User-assigned when multiple resources (e.g., several Databricks clusters or Functions) need to share the same identity and permission set.
3. *How does Unity Catalog typically use Managed Identity?* — Unity Catalog storage credentials are often backed by an Azure Managed Identity/Access Connector that's granted RBAC roles on ADLS Gen2, so all workspace access to external data flows through one governed identity instead of per-user keys.

### One-Line Revision
**Managed Identity = a badge Azure gives a resource so it never needs to remember a password.**

---

## 5. Key Vault

### Quick Theory
Azure Key Vault stores secrets, keys, and certificates centrally with strict access control and auditing. Databricks integrates via **Key Vault-backed secret scopes**, so notebooks reference secrets (`dbutils.secrets.get(scope, key)`) without ever seeing the actual value in plaintext in code.

### Real-Life Analogy
A **bank vault with a logbook**: every time someone withdraws (reads) a secret, it's logged — who, when, what. Nobody carries the vault key around; they request access through a controlled process (RBAC/access policy).

### Technical Explanation
```
Databricks Secret Scope ──► backed by ──► Key Vault
      │
notebook code: dbutils.secrets.get("kv-scope", "db-password")
      │
returns the value at runtime, redacted in notebook output as [REDACTED]
```
Best practice: never print secrets, never hardcode connection strings; rotate secrets via Key Vault and let apps pick up new values automatically.

### Interview Q&A
1. *How do you use a Key Vault secret inside a Databricks notebook?* — Create a Databricks secret scope backed by Key Vault, then call `dbutils.secrets.get(scope="kv-scope", key="secret-name")`; Databricks automatically redacts the value if it's printed to output.
2. *What happens if you print a secret value in a Databricks notebook?* — Databricks detects known secret values and redacts them as `[REDACTED]` in the output, though this isn't foolproof (e.g., if you manipulate the string first), so it's still bad practice to print secrets at all.
3. *How do you rotate a database password with zero downtime for pipelines?* — Update the secret in Key Vault; since pipelines fetch it at runtime rather than hardcoding it, the next run automatically picks up the new value — no code deployment needed.

### One-Line Revision
**Key Vault is the audited, centralized vault; Databricks secret scopes are the window notebooks use to peek in without touching the key.**

---

## 6. Azure Monitor & Log Analytics

### Quick Theory
Azure Monitor is the umbrella observability service; **Log Analytics** is its query/storage backend (using KQL — Kusto Query Language) where logs and metrics from Databricks, ADF, Storage, etc. are collected for dashboards, alerts, and troubleshooting.

### Real-Life Analogy
Azure Monitor is the **hospital's central monitoring station**; Log Analytics is the patient records database the nurses query ("show me all patients whose heart rate spiked in the last hour" = a KQL query over metrics).

### Technical Explanation
```
Databricks Cluster ──► Diagnostic Settings ──► Log Analytics Workspace
                                                     │
                                              KQL queries / Dashboards / Alerts
```
- Databricks doesn't send logs to Log Analytics by default — you configure **cluster init scripts / diagnostic logging** (via the Databricks-Azure Monitor integration or Spark listener) to forward event logs, driver/executor logs, and cluster metrics.
- Alerts can trigger Azure Functions/Logic Apps/emails/Teams messages when a job fails or a metric crosses a threshold.

### Interview Q&A
1. *How do you monitor Databricks job failures proactively instead of finding out from users?* — Ship cluster/job event logs to Log Analytics via diagnostic settings, write a KQL query/alert rule for failed job runs or specific error patterns, and wire the alert to email/Teams/PagerDuty.
2. *What's the difference between a metric and a log in Azure Monitor?* — Metrics are lightweight numeric time-series data (CPU %, active clusters) good for real-time dashboards; logs are structured/unstructured event records queried with KQL for deeper root-cause analysis.
3. *A job has been slow for the past 3 days — how do you find out why using Log Analytics?* — Query cluster metrics (CPU, memory, shuffle read/write) over that time window, correlate with Spark UI stage-level data if exported, and check for any recent code/data-volume changes logged alongside job run history.

### One-Line Revision
**Azure Monitor watches, Log Analytics remembers and lets you ask "why" in KQL.**

---

## 7. Azure RBAC

### Quick Theory
Role-Based Access Control assigns **roles** (collections of permissions) to identities (users, groups, service principals, managed identities) **scoped** to a resource, resource group, subscription, or management group — following least-privilege principles.

### Real-Life Analogy
Like a **hospital badge system**: a nurse's badge opens patient wards (scope) but not the pharmacy vault; a doctor's badge opens both. The badge type = role; where it works = scope.

### Interview Q&A
1. *What's the difference between RBAC and ACLs in ADLS Gen2?* — RBAC controls access at the Azure resource level (e.g., "Storage Blob Data Reader" on the whole storage account/container) via Azure AD; POSIX ACLs give finer-grained control down to individual files/folders, and both can apply together (most restrictive wins in practice for overlapping scope).
2. *How would you grant a data engineering team read-only access to production Gold tables but not Bronze?* — Assign a custom RBAC role or Unity Catalog grant scoped to the Gold schema/catalog only, following least privilege, rather than granting workspace-wide or storage-account-wide access.
3. *What's the principle of least privilege and why does it matter for a trading platform?* — Grant only the minimum permissions needed for a task; for trading data, this limits blast radius of a compromised credential and is often a regulatory/audit requirement (SOX, PCI, etc.).

### One-Line Revision
**RBAC = badges (roles) that open specific doors (scopes), never more than needed.**

---

## 8. Azure Service Bus

### Quick Theory
Service Bus is an enterprise message broker supporting **queues** (point-to-point) and **topics/subscriptions** (pub-sub), used for reliable, ordered, decoupled communication between systems — a common entry point for event-driven data ingestion into Databricks.

### Real-Life Analogy
A **post office sorting system**: a queue is one mailbox that a single recipient checks; a topic is a bulletin board where multiple subscribers each get their own copy of every notice posted.

### Technical Explanation
```
Upstream Trading System ──► Service Bus Topic ──┬──► Subscription A ──► Databricks (Structured Streaming)
                                                  ├──► Subscription B ──► Azure Function
                                                  └──► Subscription C ──► Audit/Archive
```
Databricks consumes Service Bus messages typically via a connector library or by routing through Event Hubs (Service Bus itself isn't natively read by Spark Structured Streaming as easily as Event Hubs/Kafka, so many real designs put Event Hubs in front for the Databricks side, with Service Bus used for transactional/ordered business events elsewhere).

### Interview Q&A
1. *When would you choose Service Bus over Event Hubs for event-driven ingestion?* — Service Bus is better for lower-throughput, transactional messaging needing guaranteed ordering, dead-lettering, and duplicate detection (e.g., "trade confirmation" business events); Event Hubs is built for very high-throughput telemetry/streaming ingestion (e.g., millions of tick events/sec) that Databricks Structured Streaming consumes natively.
2. *What is a dead-letter queue and why does it matter in a trading pipeline?* — Messages that repeatedly fail processing are moved to a dead-letter sub-queue instead of being lost or blocking the main queue, so a bad trade message doesn't halt processing of all other messages, and it can be inspected/replayed later.
3. *How do you guarantee a message isn't processed twice?* — Use Service Bus's built-in duplicate detection window plus idempotent processing logic downstream (e.g., upserts keyed by message/trade ID rather than blind inserts).

### One-Line Revision
**Service Bus is the reliable, ordered postal service for business events feeding into the pipeline.**

---

## 9. Azure Functions

### Quick Theory
Azure Functions is Azure's serverless compute — small pieces of code that run in response to triggers (HTTP call, Service Bus message, timer, Blob creation) without managing servers, often used as the "glue" that kicks off or reacts to Databricks jobs.

### Real-Life Analogy
Like a **motion-sensor light**: it does nothing until triggered (someone walks by / a message arrives), then runs briefly and switches off — you don't pay for it sitting idle.

### Interview Q&A
1. *How might Azure Functions be used alongside Databricks in this JD's pipeline?* — As an event trigger — e.g., a Function fires when a file lands in ADLS or a Service Bus message arrives, and calls the Databricks Jobs API to trigger a specific notebook/workflow run, or as a lightweight step for notifications/alerts after a job completes.
2. *What's the difference between Consumption plan and Premium plan for Functions?* — Consumption is pay-per-execution with cold-start latency and scale-to-zero; Premium keeps warm instances (no cold start) and supports VNet integration, better suited for latency-sensitive trading-adjacent triggers.
3. *Why not just put all logic inside Databricks instead of using Functions?* — Functions are cheaper and faster for small, simple, event-triggered glue logic (like calling an API or sending a notification) than spinning up a Databricks cluster, which has startup latency and cost overhead unsuited for trivial tasks.

### One-Line Revision
**Azure Functions is the lightweight trigger/glue that starts or reacts to the real work happening in Databricks.**

---

## Section Wrap-Up: How these connect in the JD's likely architecture

```
Upstream Trading System
        │ (event/message)
        ▼
  Service Bus / Event Hub ──► Azure Function (trigger) ──► Databricks Job API
        │                                                        │
        ▼                                                        ▼
  ADLS Gen2 (raw landing) ───────────────────────────► Databricks Cluster
        │                                             (Managed Identity auth,
        │                                              secrets from Key Vault)
        ▼
  Bronze → Silver → Gold Delta Tables (Unity Catalog governed, RBAC scoped)
        │
        ▼
  Downstream systems / Power BI
        │
  All the above monitored via Log Analytics/Azure Monitor,
  deployed via Azure DevOps CI/CD,
  network-isolated via VNet injection + NSGs.
```

Next file: `02_databricks_and_delta_lake.md`
