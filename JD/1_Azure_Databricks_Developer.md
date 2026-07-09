# 🚀 Complete Azure Databricks Developer Interview Preparation Roadmap

## How to Use This Guide

This roadmap is divided into **8 Stages**, progressing from fundamentals to advanced topics. Each stage includes:
- 📖 **Concepts & Explanations**
- 🛠️ **Hands-On Exercises**
- 🎯 **Interview-Focused Topics**
- ❓ **Practice Interview Questions with Answers**

**Estimated Timeline**: 8-12 weeks (assuming 3-4 hours/day)

---

---

# STAGE 1: FOUNDATIONS — Data Engineering & Cloud Basics (Week 1)

---

## 1.1 What is Data Engineering?

Data engineering is the discipline of designing, building, and maintaining systems that collect, store, transform, and serve data. Think of it as building the **plumbing** that moves and processes data so that analysts, data scientists, and business users can use it.

### Key Concepts

**Data Pipeline**: A series of steps that move data from a source (e.g., a database, API, file) to a destination (e.g., a data warehouse, dashboard). Think of it like an assembly line in a factory.

```
[Source Systems] → [Extract] → [Transform] → [Load] → [Destination]
     (APIs,           (Pull       (Clean,        (Write     (Data
   Databases,         data)       reshape,       data)    Warehouse,
    Files)                       aggregate)                Lake)
```

**ETL vs ELT**:
- **ETL (Extract, Transform, Load)**: Data is transformed *before* loading into the destination. Traditional approach.
- **ELT (Extract, Load, Transform)**: Data is loaded raw first, then transformed *inside* the destination. Modern cloud approach (what Databricks uses).

```
ETL (Traditional):
Source → Extract → Transform (external engine) → Load → Warehouse

ELT (Modern / Databricks approach):
Source → Extract → Load (raw into Lake) → Transform (inside Databricks) → Serve
```

**Why ELT is preferred in modern cloud**:
- Cloud storage is cheap (store everything raw)
- Cloud compute is elastic (scale up for heavy transformations)
- You preserve raw data for re-processing

**Batch vs Real-Time (Streaming) Processing**:
| Aspect | Batch | Streaming |
|--------|-------|-----------|
| When data is processed | At scheduled intervals (hourly, daily) | Continuously, as it arrives |
| Latency | Minutes to hours | Seconds to minutes |
| Example | Nightly sales report | Real-time fraud detection |
| Tools | Spark batch jobs | Spark Structured Streaming, Kafka |

**Data Warehouse vs Data Lake vs Data Lakehouse**:

```
Data Warehouse (Structured Only):
┌─────────────────────────┐
│  Structured Data Only    │
│  Schema-on-Write         │
│  SQL-based queries       │
│  Example: Azure Synapse  │
└─────────────────────────┘

Data Lake (All Data Types, Less Structure):
┌─────────────────────────┐
│  Structured + Semi +     │
│  Unstructured Data       │
│  Schema-on-Read          │
│  Cheap storage           │
│  Example: Azure Data Lake│
│  Problem: "Data Swamp"   │
└─────────────────────────┘

Data Lakehouse (Best of Both - THIS IS DATABRICKS):
┌─────────────────────────┐
│  All Data Types          │
│  ACID Transactions       │
│  Schema Enforcement      │
│  SQL + ML + Streaming    │
│  Built on Delta Lake     │
│  Example: Databricks     │
└─────────────────────────┘
```

The **Lakehouse** is the architecture Databricks champions — you get the flexibility of a data lake with the reliability and performance of a data warehouse.

---

## 1.2 Cloud Computing Basics

### What is Cloud Computing?
Instead of owning physical servers, you rent computing resources (servers, storage, databases, networking) from a provider (Microsoft Azure, AWS, GCP) and pay only for what you use.

### Key Cloud Concepts

**IaaS, PaaS, SaaS**:
```
┌─────────────────────────────────────────────┐
│              Cloud Service Models             │
├──────────┬──────────────┬───────────────────┤
│   IaaS   │     PaaS     │       SaaS        │
│ (Infra)  │ (Platform)   │   (Software)      │
├──────────┼──────────────┼───────────────────┤
│ You      │ You manage   │ Everything        │
│ manage   │ only your    │ managed for you   │
│ VMs, OS, │ code & data  │                   │
│ storage  │              │                   │
├──────────┼──────────────┼───────────────────┤
│ Example: │ Example:     │ Example:          │
│ Azure VM │ Databricks   │ Office 365        │
│          │ Azure        │ Power BI Service  │
│          │ Functions    │                   │
└──────────┴──────────────┴───────────────────┘
```

Databricks is **PaaS** — you focus on writing data pipelines, not managing servers.

### Azure Basics You Must Know

| Azure Service | Purpose | Analogy |
|---------------|---------|---------|
| **Azure Data Lake Storage (ADLS) Gen2** | Scalable storage for big data | A giant, organized filing cabinet |
| **Azure Databricks** | Big data processing & analytics platform | Your workbench for building pipelines |
| **Azure DevOps** | CI/CD, source control, project management | Your build & deploy automation tool |
| **Azure Key Vault** | Securely store secrets, keys, certificates | A digital safe |
| **Azure Active Directory (Entra ID)** | Identity & access management | The security guard at the door |
| **Azure Service Bus** | Message broker for async communication | A postal service for apps |
| **Azure Functions** | Serverless compute (run code without servers) | An automatic trigger/action bot |
| **Azure Log Analytics** | Centralized logging & monitoring | A CCTV system for your apps |

### Setting Up Your Free Learning Environment

1. **Create a free Azure account**: https://azure.microsoft.com/free/ (you get $200 credit for 30 days)
2. **Create a Databricks Community Edition** (FREE, no Azure needed for learning): https://community.cloud.databricks.com/
3. **Azure DevOps**: https://dev.azure.com/ (free for up to 5 users)

---

## 🛠️ Hands-On Exercise 1.1

1. Sign up for Databricks Community Edition
2. Create a new cluster (it auto-configures for you)
3. Create a notebook, select Python, and run:
```python
print("Hello, Data Engineering!")
```
4. In the same notebook, switch a cell to SQL and run:
```sql
SELECT "I am learning Databricks!" AS message
```
5. Explore the Databricks UI: Workspace, Clusters, Data, Jobs tabs

---

## 🎯 Interview Topics for Stage 1

- Explain what a data pipeline is
- ETL vs ELT — when to use which
- Batch vs Streaming processing
- Data Lake vs Data Warehouse vs Lakehouse
- What is Azure Databricks at a high level?
- Basic cloud concepts (IaaS, PaaS, SaaS)

## ❓ Practice Interview Questions — Stage 1

**Q1: What is the difference between ETL and ELT? Which approach does Databricks favor and why?**

> **A**: ETL extracts data, transforms it externally, then loads it. ELT extracts and loads raw data first, then transforms it inside the destination. Databricks favors ELT because:
> 1. Cloud storage (ADLS) is inexpensive — store raw data cheaply
> 2. Databricks provides powerful distributed compute (Spark) to transform data in-place
> 3. Raw data is preserved for re-processing or different transformations
> 4. Delta Lake adds ACID transactions to the lake, making ELT reliable

**Q2: What is a Data Lakehouse, and how does it differ from a Data Lake?**

> **A**: A Data Lakehouse combines the best features of data lakes and data warehouses. A data lake stores all types of data cheaply but lacks reliability features (no transactions, schema enforcement) — leading to "data swamps." A lakehouse adds ACID transactions, schema enforcement, indexing, and caching on top of a data lake. Databricks implements this through Delta Lake, giving you warehouse-like reliability on lake storage.

**Q3: Why would you use Azure Databricks instead of just writing Python scripts on a VM?**

> **A**: Databricks provides distributed computing (Apache Spark), which can process terabytes of data across many machines in parallel. A single VM would run out of memory or take days. Databricks also provides: cluster management, notebook collaboration, built-in MLflow, Delta Lake for reliable storage, job scheduling, integration with Azure services, and auto-scaling — none of which you get from a plain VM.

**Q4: What is the difference between Batch and Streaming processing? Give examples of when you'd use each.**

> **A**: Batch processing handles large volumes of data at scheduled intervals (e.g., daily sales aggregation). Streaming processing handles data continuously as it arrives (e.g., real-time fraud detection). In Databricks, batch uses `spark.read` and streaming uses `spark.readStream` with Structured Streaming. Many modern architectures use both — streaming for low-latency needs and batch for heavy historical analysis.

---

---

# STAGE 2: Python & SQL for Data Engineering (Weeks 1-2)

---

## 2.1 Python Essentials for Data Engineering

You don't need to know ALL of Python — focus on what data engineers use daily.

### Core Python Concepts to Master

**1. Data Types & Data Structures**
```python
# Strings
name = "pipeline_daily_sales"

# Lists (ordered, mutable) — like a dynamic array
sources = ["database", "api", "file"]
sources.append("stream")

# Dictionaries (key-value pairs) — used EVERYWHERE in configs
config = {
    "source": "adls",
    "path": "/raw/sales/",
    "format": "parquet",
    "mode": "overwrite"
}
print(config["path"])  # /raw/sales/

# Tuples (ordered, immutable) — for fixed data
schema_version = (2, 1, 0)  # major, minor, patch

# Sets (unique values)
unique_countries = {"US", "UK", "DE", "US"}  # {"US", "UK", "DE"}
```

**2. Control Flow**
```python
# If-Else (used for pipeline branching)
data_format = "parquet"
if data_format == "parquet":
    df = spark.read.parquet(path)
elif data_format == "csv":
    df = spark.read.csv(path, header=True)
else:
    raise ValueError(f"Unsupported format: {data_format}")

# For Loops (iterate over tables, files, etc.)
tables = ["customers", "orders", "products"]
for table in tables:
    print(f"Processing table: {table}")
    # process each table...

# List Comprehension (Pythonic way to create lists)
parquet_files = [f for f in all_files if f.endswith('.parquet')]
```

**3. Functions (reusable pipeline components)**
```python
def read_data(path: str, format: str = "parquet") -> "DataFrame":
    """Read data from ADLS in the specified format."""
    if format == "parquet":
        return spark.read.parquet(path)
    elif format == "csv":
        return spark.read.csv(path, header=True, inferSchema=True)
    elif format == "json":
        return spark.read.json(path)
    else:
        raise ValueError(f"Unsupported format: {format}")

# Usage
df = read_data("/mnt/raw/sales/2024/", "parquet")
```

**4. Error Handling (critical for production pipelines)**
```python
from datetime import datetime

def run_pipeline(table_name: str):
    try:
        print(f"[{datetime.now()}] Starting pipeline for {table_name}")
        df = spark.read.parquet(f"/raw/{table_name}/")
        transformed = df.filter(df["amount"] > 0)
        transformed.write.mode("overwrite").parquet(f"/curated/{table_name}/")
        print(f"[{datetime.now()}] Pipeline completed for {table_name}")
    except FileNotFoundError:
        print(f"ERROR: Source file not found for {table_name}")
        raise
    except Exception as e:
        print(f"ERROR: Pipeline failed for {table_name}: {str(e)}")
        raise
    finally:
        print(f"[{datetime.now()}] Pipeline execution ended for {table_name}")
```

**5. Working with Files and JSON (common in configs)**
```python
import json

# Reading a config file
with open("pipeline_config.json", "r") as f:
    config = json.load(f)

# Creating config dynamically
pipeline_config = {
    "tables": ["sales", "customers"],
    "target_path": "/curated/",
    "write_mode": "overwrite"
}
config_json = json.dumps(pipeline_config, indent=2)
```

**6. Classes (used in modular pipeline design)**
```python
class DataPipeline:
    def __init__(self, source_path: str, target_path: str):
        self.source_path = source_path
        self.target_path = target_path
        self.spark = SparkSession.builder.getOrCreate()

    def extract(self):
        self.df = self.spark.read.parquet(self.source_path)
        return self

    def transform(self):
        self.df = self.df.filter(self.df["status"] == "active")
        self.df = self.df.dropDuplicates(["id"])
        return self

    def load(self):
        self.df.write.mode("overwrite").format("delta").save(self.target_path)
        return self

    def run(self):
        self.extract().transform().load()
        print("Pipeline completed successfully!")

# Usage
pipeline = DataPipeline("/raw/customers/", "/curated/customers/")
pipeline.run()
```

---

## 2.2 SQL Essentials for Data Engineering

SQL is the **most critical skill** for this role. You'll use it both in Databricks SQL notebooks and inside PySpark.

### Core SQL Concepts

**1. Basic Queries**
```sql
-- Select specific columns
SELECT customer_id, customer_name, country, total_purchases
FROM customers
WHERE country = 'US'
  AND total_purchases > 1000
ORDER BY total_purchases DESC
LIMIT 100;

-- Aggregations
SELECT 
    country,
    COUNT(*) AS customer_count,
    SUM(total_purchases) AS total_revenue,
    AVG(total_purchases) AS avg_revenue,
    MAX(total_purchases) AS max_revenue
FROM customers
GROUP BY country
HAVING COUNT(*) > 10
ORDER BY total_revenue DESC;
```

**2. JOINs (very important for interviews)**
```
Tables:
orders (order_id, customer_id, product_id, amount, order_date)
customers (customer_id, name, country)

INNER JOIN: Only matching rows from both tables
LEFT JOIN:  All rows from left + matching from right (NULLs if no match)
RIGHT JOIN: All rows from right + matching from left
FULL OUTER JOIN: All rows from both (NULLs where no match)
```

```sql
-- INNER JOIN: Only customers who have placed orders
SELECT c.name, o.order_id, o.amount
FROM customers c
INNER JOIN orders o ON c.customer_id = o.customer_id;

-- LEFT JOIN: All customers, even those without orders
SELECT c.name, o.order_id, COALESCE(o.amount, 0) AS amount
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id;

-- Self JOIN: Compare each order with the previous order
SELECT 
    a.order_id,
    a.amount AS current_amount,
    b.amount AS previous_amount,
    a.amount - b.amount AS difference
FROM orders a
LEFT JOIN orders b ON a.customer_id = b.customer_id 
    AND a.order_date = DATE_ADD(b.order_date, 1);
```

**3. Window Functions (heavily tested in interviews)**
```sql
-- ROW_NUMBER: Assign unique row numbers within groups
SELECT 
    customer_id,
    order_date,
    amount,
    ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY order_date DESC) AS rn
FROM orders;

-- Use ROW_NUMBER to get the latest order per customer
SELECT * FROM (
    SELECT 
        customer_id,
        order_date,
        amount,
        ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY order_date DESC) AS rn
    FROM orders
) WHERE rn = 1;

-- RANK vs DENSE_RANK
-- RANK: Skips numbers after ties (1, 2, 2, 4)
-- DENSE_RANK: No gaps (1, 2, 2, 3)

-- Running total
SELECT 
    order_date,
    amount,
    SUM(amount) OVER (ORDER BY order_date) AS running_total,
    AVG(amount) OVER (ORDER BY order_date ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) AS moving_avg_7day
FROM orders;

-- LAG/LEAD: Access previous/next row
SELECT 
    order_date,
    amount,
    LAG(amount, 1) OVER (ORDER BY order_date) AS prev_amount,
    LEAD(amount, 1) OVER (ORDER BY order_date) AS next_amount,
    amount - LAG(amount, 1) OVER (ORDER BY order_date) AS daily_change
FROM orders;
```

**4. CTEs (Common Table Expressions) and Subqueries**
```sql
-- CTE: Makes complex queries readable
WITH monthly_sales AS (
    SELECT 
        DATE_TRUNC('month', order_date) AS month,
        SUM(amount) AS total_sales
    FROM orders
    GROUP BY DATE_TRUNC('month', order_date)
),
ranked_months AS (
    SELECT 
        month,
        total_sales,
        RANK() OVER (ORDER BY total_sales DESC) AS sales_rank
    FROM monthly_sales
)
SELECT * FROM ranked_months WHERE sales_rank <= 3;
```

**5. Data Manipulation (DML for Delta Lake)**
```sql
-- INSERT
INSERT INTO customers VALUES (101, 'John Doe', 'US');

-- UPDATE
UPDATE customers SET country = 'UK' WHERE customer_id = 101;

-- DELETE
DELETE FROM customers WHERE customer_id = 101;

-- MERGE (UPSERT — extremely important for Databricks/Delta Lake)
MERGE INTO target_table AS target
USING source_table AS source
ON target.id = source.id
WHEN MATCHED THEN
    UPDATE SET target.name = source.name, target.amount = source.amount
WHEN NOT MATCHED THEN
    INSERT (id, name, amount) VALUES (source.id, source.name, source.amount)
WHEN NOT MATCHED BY SOURCE THEN
    DELETE;
```

**6. Important SQL Functions**
```sql
-- String functions
SELECT 
    UPPER(name),
    LOWER(name),
    TRIM(name),
    SUBSTRING(name, 1, 3),
    CONCAT(first_name, ' ', last_name),
    REPLACE(phone, '-', '')
FROM customers;

-- Date functions (critical for data pipelines)
SELECT 
    CURRENT_DATE(),
    CURRENT_TIMESTAMP(),
    DATE_ADD(order_date, 30) AS due_date,
    DATEDIFF(CURRENT_DATE(), order_date) AS days_since_order,
    DATE_FORMAT(order_date, 'yyyy-MM') AS year_month,
    YEAR(order_date),
    MONTH(order_date)
FROM orders;

-- NULL handling
SELECT 
    COALESCE(phone, email, 'No Contact') AS contact_info,
    IFNULL(discount, 0) AS discount,
    NULLIF(amount, 0) AS amount_or_null  -- returns NULL if amount = 0
FROM orders;

-- CASE statements
SELECT 
    order_id,
    amount,
    CASE 
        WHEN amount >= 10000 THEN 'High Value'
        WHEN amount >= 1000 THEN 'Medium Value'
        ELSE 'Low Value'
    END AS order_category
FROM orders;
```

---

## 🛠️ Hands-On Exercise 2.1

In Databricks Community Edition, create a notebook and run:

```python
# Create sample data
data = [
    (1, "Alice", "US", 5000, "2024-01-15"),
    (2, "Bob", "UK", 12000, "2024-01-20"),
    (3, "Charlie", "US", 300, "2024-02-10"),
    (4, "Diana", "DE", 8000, "2024-02-15"),
    (5, "Eve", "US", 15000, "2024-03-01"),
    (6, "Frank", "UK", 900, "2024-03-10"),
    (7, "Grace", "DE", 20000, "2024-03-15"),
    (8, "Hank", "US", 4500, "2024-04-01"),
]

columns = ["customer_id", "name", "country", "total_purchases", "signup_date"]
df = spark.createDataFrame(data, columns)
df.createOrReplaceTempView("customers")
```

Now practice these SQL queries in a SQL cell:
```sql
-- 1. Find all US customers with purchases > 1000
-- 2. Count customers per country, sort by count descending
-- 3. Rank customers by total_purchases within each country
-- 4. Find the customer with the highest purchase in each country
-- 5. Calculate a running total of purchases ordered by signup_date
```

---

## ❓ Practice Interview Questions — Stage 2

**Q1: Write a SQL query to find the second highest salary in an employees table.**

> ```sql
> -- Method 1: Using DENSE_RANK
> SELECT salary FROM (
>     SELECT salary, DENSE_RANK() OVER (ORDER BY salary DESC) AS rnk
>     FROM employees
> ) WHERE rnk = 2;
>
> -- Method 2: Using LIMIT/OFFSET
> SELECT DISTINCT salary 
> FROM employees 
> ORDER BY salary DESC 
> LIMIT 1 OFFSET 1;
> ```

**Q2: What is the difference between WHERE and HAVING?**

> **A**: `WHERE` filters individual rows *before* grouping. `HAVING` filters groups *after* the GROUP BY aggregation. Example: `WHERE salary > 5000` filters rows first, then `HAVING COUNT(*) > 3` keeps only groups with more than 3 members.

**Q3: Explain the MERGE statement and why it's important in Databricks.**

> **A**: MERGE (also called UPSERT) combines INSERT, UPDATE, and DELETE in a single atomic operation. It matches rows from a source table against a target table using a condition. If matched, it updates; if not matched, it inserts; optionally, if not matched by source, it deletes. This is critical in Databricks for:
> - Incremental data loads (only process new/changed records)
> - Type 2 Slowly Changing Dimensions
> - Avoiding duplicate data
> - Delta Lake supports MERGE natively with ACID transactions

**Q4: In Python, how would you handle a pipeline that processes 10 tables, where one might fail?**

> ```python
> tables = ["sales", "customers", "products", "inventory", "orders", 
>           "returns", "shipments", "payments", "reviews", "categories"]
> 
> results = {"success": [], "failed": []}
> 
> for table in tables:
>     try:
>         df = spark.read.parquet(f"/raw/{table}/")
>         df.write.mode("overwrite").format("delta").save(f"/curated/{table}/")
>         results["success"].append(table)
>     except Exception as e:
>         print(f"ERROR processing {table}: {str(e)}")
>         results["failed"].append((table, str(e)))
> 
> print(f"Success: {len(results['success'])}, Failed: {len(results['failed'])}")
> if results["failed"]:
>     raise Exception(f"Pipeline partially failed: {results['failed']}")
> ```

---

---

# STAGE 3: Apache Spark & PySpark Deep Dive (Weeks 2-3)

---

## 3.1 Understanding Apache Spark

Apache Spark is the **engine** behind Databricks. Understanding it is critical.

### What is Apache Spark?

Spark is a distributed computing engine that processes large datasets across a cluster of machines in parallel. Databricks is a managed platform built on top of Spark.

### Spark Architecture

```
┌─────────────────────────────────────────────────┐
│                  DRIVER PROGRAM                  │
│  (Your notebook code / main application)         │
│  - Contains SparkSession                         │
│  - Creates execution plan                        │
│  - Coordinates workers                           │
└──────────────────────┬──────────────────────────┘
                       │ Distributes work
          ┌────────────┼────────────┐
          ▼            ▼            ▼
   ┌──────────┐ ┌──────────┐ ┌──────────┐
   │ EXECUTOR │ │ EXECUTOR │ │ EXECUTOR │
   │ (Worker) │ │ (Worker) │ │ (Worker) │
   │          │ │          │ │          │
   │ ┌──────┐ │ │ ┌──────┐ │ │ ┌──────┐ │
   │ │Task 1│ │ │ │Task 3│ │ │ │Task 5│ │
   │ │Task 2│ │ │ │Task 4│ │ │ │Task 6│ │
   │ └──────┘ │ │ └──────┘ │ │ └──────┘ │
   │          │ │          │ │          │
   │ [Cache]  │ │ [Cache]  │ │ [Cache]  │
   └──────────┘ └──────────┘ └──────────┘
```

**Key concepts**:
- **Driver**: The "brain" — your notebook code runs here. It plans and coordinates.
- **Executors**: The "workers" — they do the actual data processing in parallel.
- **Tasks**: Units of work assigned to executors (1 task per partition per core).
- **Cluster Manager**: Allocates resources (in Databricks, this is managed for you).

### Spark Core Concepts

**Lazy Evaluation**: Spark doesn't execute anything until you call an *action*. It builds up a plan (DAG) of *transformations* and optimizes it before executing.

```python
# These are TRANSFORMATIONS (lazy - nothing happens yet)
df = spark.read.parquet("/data/sales/")      # Transformation
df_filtered = df.filter(df.amount > 100)     # Transformation
df_grouped = df_filtered.groupBy("country").sum("amount")  # Transformation

# This is an ACTION (triggers execution of entire plan)
df_grouped.show()     # Action — NOW Spark executes everything
df_grouped.count()    # Action
df_grouped.collect()  # Action
df_grouped.write.save("/output/")  # Action
```

**Why lazy evaluation?** Spark can optimize the entire chain. For example, if you filter and then select columns, Spark can push the filter down to read fewer rows from disk and only read needed columns — even though you wrote them in a different order.

**Transformations vs Actions:**
| Transformations (Lazy) | Actions (Trigger Execution) |
|------------------------|-----------------------------|
| `select()`, `filter()` | `show()`, `display()` |
| `groupBy()`, `agg()` | `count()`, `collect()` |
| `join()`, `union()` | `write.save()`, `write.parquet()` |
| `withColumn()`, `drop()` | `first()`, `take(n)` |
| `orderBy()`, `distinct()` | `toPandas()` |

**Narrow vs Wide Transformations:**
```
NARROW Transformations (no shuffle — fast):
Each input partition contributes to only ONE output partition
Examples: filter(), select(), map(), withColumn()

Partition 1 ──→ Partition 1
Partition 2 ──→ Partition 2
Partition 3 ──→ Partition 3

WIDE Transformations (shuffle required — expensive):
Each input partition contributes to MANY output partitions
Examples: groupBy(), join(), orderBy(), distinct(), repartition()

Partition 1 ──→ Partition 1
          ╲  ╱
           ╳
          ╱  ╲
Partition 2 ──→ Partition 2
          ╲  ╱
           ╳
          ╱  ╲
Partition 3 ──→ Partition 3

Shuffle = Data movement across executors over the network = SLOW
```

Understanding shuffle is KEY to performance tuning.

---

## 3.2 PySpark DataFrame API

### Creating DataFrames

```python
# From a list
data = [
    (1, "Alice", "Sales", 75000),
    (2, "Bob", "Engineering", 95000),
    (3, "Charlie", "Sales", 68000),
    (4, "Diana", "Engineering", 110000),
    (5, "Eve", "Marketing", 72000),
]
columns = ["emp_id", "name", "department", "salary"]
df = spark.createDataFrame(data, columns)

# From a file
df_parquet = spark.read.parquet("/mnt/data/employees/")
df_csv = spark.read.csv("/mnt/data/employees.csv", header=True, inferSchema=True)
df_json = spark.read.json("/mnt/data/employees.json")
df_delta = spark.read.format("delta").load("/mnt/data/employees_delta/")

# With explicit schema (best practice for production)
from pyspark.sql.types import StructType, StructField, StringType, IntegerType, DoubleType

schema = StructType([
    StructField("emp_id", IntegerType(), nullable=False),
    StructField("name", StringType(), nullable=False),
    StructField("department", StringType(), nullable=True),
    StructField("salary", DoubleType(), nullable=True),
])

df = spark.read.schema(schema).csv("/data/employees.csv", header=True)
```

### Essential DataFrame Operations

```python
from pyspark.sql.functions import col, lit, when, concat, upper, lower
from pyspark.sql.functions import sum, avg, count, max, min
from pyspark.sql.functions import current_date, datediff, date_format
from pyspark.sql.functions import row_number, rank, dense_rank, lag, lead
from pyspark.sql.window import Window

# ---- SELECTING & FILTERING ----
df.select("name", "salary").show()
df.select(col("name"), col("salary") * 1.1).show()
df.filter(col("salary") > 80000).show()
df.filter((col("department") == "Sales") & (col("salary") > 70000)).show()
df.where(col("name").like("A%")).show()
df.where(col("department").isin("Sales", "Marketing")).show()

# ---- ADDING/MODIFYING COLUMNS ----
df = df.withColumn("bonus", col("salary") * 0.1)
df = df.withColumn("salary_category", 
    when(col("salary") >= 100000, "High")
    .when(col("salary") >= 70000, "Medium")
    .otherwise("Low")
)
df = df.withColumn("name_upper", upper(col("name")))
df = df.withColumnRenamed("emp_id", "employee_id")

# ---- DROPPING ----
df = df.drop("bonus")
df = df.dropDuplicates(["name", "department"])
df = df.dropna(subset=["salary"])  # Drop rows with null salary
df = df.fillna({"salary": 0, "department": "Unknown"})  # Fill nulls

# ---- AGGREGATIONS ----
df.groupBy("department").agg(
    count("*").alias("emp_count"),
    sum("salary").alias("total_salary"),
    avg("salary").alias("avg_salary"),
    max("salary").alias("max_salary"),
    min("salary").alias("min_salary")
).show()

# ---- JOINS ----
departments = spark.createDataFrame([
    ("Sales", "New York"),
    ("Engineering", "San Francisco"),
    ("Marketing", "Chicago"),
], ["department", "location"])

# Inner join
df.join(departments, "department", "inner").show()

# Left join
df.join(departments, "department", "left").show()

# Join on different column names
df.join(departments, df.department == departments.department, "left").show()

# ---- WINDOW FUNCTIONS in PySpark ----
window_spec = Window.partitionBy("department").orderBy(col("salary").desc())

df_ranked = df.withColumn("rank", rank().over(window_spec))
df_ranked = df_ranked.withColumn("dense_rank", dense_rank().over(window_spec))
df_ranked = df_ranked.withColumn("row_num", row_number().over(window_spec))

# Running total
window_running = Window.partitionBy("department").orderBy("salary").rowsBetween(
    Window.unboundedPreceding, Window.currentRow
)
df = df.withColumn("running_total", sum("salary").over(window_running))

# LAG/LEAD
window_order = Window.partitionBy("department").orderBy("salary")
df = df.withColumn("prev_salary", lag("salary", 1).over(window_order))
df = df.withColumn("next_salary", lead("salary", 1).over(window_order))

# ---- SORTING ----
df.orderBy(col("salary").desc()).show()
df.orderBy(col("department").asc(), col("salary").desc()).show()

# ---- UNION ----
df1.union(df2)       # Columns must match in order and count
df1.unionByName(df2) # Matches by column name (safer)
```

### Writing DataFrames

```python
# Write as Parquet
df.write.mode("overwrite").parquet("/output/employees_parquet/")

# Write as Delta (preferred in Databricks)
df.write.mode("overwrite").format("delta").save("/output/employees_delta/")

# Write modes:
# "overwrite" — Replace existing data
# "append"    — Add to existing data
# "ignore"    — Don't write if data exists
# "error"     — Throw error if data exists (default)

# Partitioning (important for performance)
df.write \
    .mode("overwrite") \
    .partitionBy("department", "year") \
    .format("delta") \
    .save("/output/employees_partitioned/")

# This creates folder structure:
# /output/employees_partitioned/
#   ├── department=Sales/year=2024/
#   ├── department=Engineering/year=2024/
#   └── department=Marketing/year=2024/
```

### Partitioning Deep Dive

```
Why Partition Data?
Without partitioning: Query scans ALL data
With partitioning: Query only reads relevant folders

Example: Query WHERE department = 'Sales' AND year = 2024
- Without partitioning: Scans 100GB (all data)
- With partitioning by department, year: Scans only 5GB (Sales/2024 folder)
This is called PARTITION PRUNING

Best Practices:
- Partition by LOW CARDINALITY columns (country, year, month)
- DON'T partition by HIGH CARDINALITY columns (user_id, timestamp)
- Ideal: Each partition = 100MB to 1GB of data
- Too many small partitions = "small file problem" = poor performance
```

---

## 3.3 Spark SQL vs PySpark DataFrame API

In Databricks, you can write the same logic in SQL or PySpark. They compile to the same execution plan.

```python
# PySpark way
result = df.filter(col("department") == "Sales") \
           .groupBy("name") \
           .agg(sum("salary").alias("total_salary")) \
           .orderBy(col("total_salary").desc())

# SQL way (after registering as temp view)
df.createOrReplaceTempView("employees")

result = spark.sql("""
    SELECT name, SUM(salary) AS total_salary
    FROM employees
    WHERE department = 'Sales'
    GROUP BY name
    ORDER BY total_salary DESC
""")

# Both produce IDENTICAL execution plans
```

---

## 3.4 Spark Performance & Optimization Basics

### Key Concepts for Interviews

**1. Catalyst Optimizer**
Spark's query optimizer that automatically optimizes your code:
- Predicate Pushdown: Pushes filters as close to the data source as possible
- Column Pruning: Only reads columns you actually use
- Constant Folding: Pre-computes constant expressions
- Join Reordering: Optimizes the order of joins

**2. Partitions & Parallelism**
```python
# Check number of partitions
print(df.rdd.getNumPartitions())  # e.g., 200

# Repartition (increases or decreases, triggers shuffle)
df = df.repartition(8)  # Use for increasing partitions

# Coalesce (only decreases, no shuffle — more efficient)
df = df.coalesce(4)  # Use for decreasing partitions before write

# Best practice: Coalesce before writing to avoid small files
df.coalesce(10).write.mode("overwrite").format("delta").save("/output/")
```

**3. Caching / Persisting**
```python
# Cache a DataFrame that you'll use multiple times
df.cache()  # Stores in memory
df.count()  # Triggers the cache

# Later operations on df will use the cached version
df.filter(col("department") == "Sales").count()
df.filter(col("department") == "Engineering").count()

# Unpersist when done
df.unpersist()
```

**4. Broadcast Joins**
```python
from pyspark.sql.functions import broadcast

# When joining a large table with a small table
# Broadcast the small table to all executors (avoids shuffle of large table)
large_df.join(broadcast(small_df), "key")

# Databricks auto-broadcasts tables < 10MB by default
# Configure threshold:
spark.conf.set("spark.sql.autoBroadcastJoinThreshold", 50 * 1024 * 1024)  # 50MB
```

**5. Explain Plans**
```python
# See the execution plan
df.explain()           # Simple
df.explain(True)       # Extended (shows all stages)
df.explain("formatted") # Formatted version

# In SQL
# EXPLAIN SELECT * FROM table WHERE ...
```

---

## 🛠️ Hands-On Exercise 3.1

In Databricks Community Edition:

```python
# Create sample data
from pyspark.sql.functions import *

# Generate a larger dataset
from pyspark.sql.types import *
import random

data = [(i, f"Employee_{i}", random.choice(["Sales", "Engineering", "Marketing", "Finance"]),
         random.randint(40000, 150000), f"2024-{random.randint(1,12):02d}-{random.randint(1,28):02d}")
        for i in range(1, 10001)]

schema = StructType([
    StructField("emp_id", IntegerType()),
    StructField("name", StringType()),
    StructField("department", StringType()),
    StructField("salary", IntegerType()),
    StructField("hire_date", StringType()),
])

df = spark.createDataFrame(data, schema)
df = df.withColumn("hire_date", to_date(col("hire_date")))

# Tasks:
# 1. Find the top 3 highest-paid employees per department (use window functions)
# 2. Calculate the average salary by department and compare each employee's 
#    salary to their department average
# 3. Write the data as Delta format, partitioned by department
# 4. Read it back and verify partition pruning with .explain()
# 5. Cache the DataFrame and compare execution times of repeated operations
```

---

## ❓ Practice Interview Questions — Stage 3

**Q1: Explain lazy evaluation in Spark with an example.**

> **A**: Lazy evaluation means Spark doesn't execute transformations immediately. It records them as a Directed Acyclic Graph (DAG) of operations. Execution only happens when an *action* (show, count, write, collect) is called. This allows Spark's Catalyst optimizer to optimize the entire plan — for example, pushing filters before joins, eliminating unnecessary columns, and combining operations. Example:
> ```python
> df = spark.read.parquet("/data/")    # Lazy
> df2 = df.filter(col("x") > 10)      # Lazy
> df3 = df2.select("x", "y")          # Lazy
> df3.count()  # ACTION — now Spark executes, reading only columns x,y and filtering x>10 at the source
> ```

**Q2: What is the difference between repartition() and coalesce()?**

> **A**: Both change the number of partitions, but:
> - `repartition(n)` can increase or decrease partitions, performs a **full shuffle** (expensive), distributes data evenly
> - `coalesce(n)` can only decrease partitions, does **NOT shuffle** (just combines adjacent partitions), more efficient
> - Use `repartition` when you need more partitions or even distribution (e.g., before a join)
> - Use `coalesce` when reducing partitions (e.g., before writing to avoid small files)

**Q3: When would you use a broadcast join?**

> **A**: When joining a large table with a small table (reference/lookup table). Broadcasting copies the small table to every executor, so the large table doesn't need to be shuffled across the network. This dramatically speeds up joins. Databricks auto-broadcasts tables under 10MB, but you can increase this threshold or explicitly hint with `broadcast()`. Don't broadcast large tables — it can cause out-of-memory errors on executors.

**Q4: How do you handle the "small files problem" in Spark?**

> **A**: Small files problem occurs when data is written into too many small files, making reads slow (more metadata overhead, more file opens). Solutions:
> 1. Use `coalesce()` before writing to reduce output files
> 2. Use Delta Lake's `OPTIMIZE` command to compact small files
> 3. Enable Auto Optimize in Delta: `delta.autoOptimize.optimizeWrite = true`
> 4. Choose partition columns carefully (avoid high cardinality)
> 5. Use `maxRecordsPerFile` option when writing

**Q5: What is a shuffle in Spark, and why is it expensive?**

> **A**: A shuffle occurs when data needs to be redistributed across partitions — typically during groupBy, join, orderBy, or distinct operations. It's expensive because:
> 1. Data must be serialized, written to disk, transferred over the network, then deserialized
> 2. It creates a "stage boundary" — all tasks before the shuffle must complete before tasks after can start
> 3. It can cause out-of-memory errors and disk spills
> To minimize shuffles: use broadcast joins for small tables, pre-partition data by join keys, use bucketing, avoid unnecessary sorts/distinct operations.

---

---

# STAGE 4: Azure Databricks Deep Dive (Weeks 3-5)

---

## 4.1 Databricks Architecture

```
┌───────────────────────────────────────────────────┐
│                DATABRICKS PLATFORM                 │
│                                                    │
│  ┌──────────────────────────────────────────────┐ │
│  │           CONTROL PLANE (Managed)             │ │
│  │  • Web UI / Notebooks                        │ │
│  │  • Jobs Scheduler                            │ │
│  │  • Cluster Manager                           │ │
│  │  • Unity Catalog                             │ │
│  │  • REST APIs                                 │ │
│  │  • Databricks Repos                          │ │
│  └──────────────────────────────────────────────┘ │
│                       │                            │
│  ┌──────────────────────────────────────────────┐ │
│  │           DATA PLANE (Your Azure Sub)         │ │
│  │  • Clusters (VMs running Spark)              │ │
│  │  • ADLS Gen2 (Your data)                     │ │
│  │  • Azure VNet (Networking)                   │ │
│  │  • Azure Key Vault (Secrets)                 │ │
│  └──────────────────────────────────────────────┘ │
└───────────────────────────────────────────────────┘
```

**Control Plane** (managed by Databricks): Web app, notebook management, job orchestration, cluster management, user management. This runs in Databricks' Azure subscription.

**Data Plane** (your Azure subscription): The actual compute (VMs/clusters) and storage (ADLS) run in YOUR subscription. Your data never leaves your Azure environment.

### Cluster Types

```
┌─────────────────────────────────┬──────────────────────────────┐
│       ALL-PURPOSE CLUSTER        │        JOB CLUSTER           │
├─────────────────────────────────┼──────────────────────────────┤
│ Interactive / Development        │ Automated / Production       │
│ Stays running until terminated   │ Auto-starts, auto-terminates │
│ Multiple users can share         │ Created per job run          │
│ More expensive (always on)       │ Cheaper (runs only when needed)│
│ Use for: development, debugging  │ Use for: scheduled pipelines │
└─────────────────────────────────┴──────────────────────────────┘
```

**Cluster Configuration Best Practices**:
```
• Auto-scaling: Set min and max workers (e.g., 2-8 workers)
  - Databricks adds/removes workers based on load
• Auto-termination: Set idle timeout (e.g., 30 mins for dev, 10 mins for jobs)
• Spot/Preemptible instances: Use for workers (up to 60-80% cheaper)
  - Never use spot for driver node
• Choose right VM size based on workload:
  - Memory-optimized: For caching, large joins
  - Compute-optimized: For heavy transformations
  - Storage-optimized: For data-intensive I/O
• Databricks Runtime versions:
  - Standard: Basic Spark + Delta Lake
  - ML Runtime: Adds ML libraries (TensorFlow, PyTorch, etc.)
  - Photon Runtime: C++ engine, 2-8x faster for SQL/Delta workloads
```

---

## 4.2 Delta Lake — The Heart of Databricks

Delta Lake is an **open-source storage layer** that brings reliability (ACID transactions) to data lakes. It's the foundation of the Lakehouse architecture.

### Why Delta Lake?

```
Traditional Parquet Problems:        Delta Lake Solutions:
─────────────────────────────       ──────────────────────────────
No ACID transactions           →    Full ACID transactions
No schema enforcement          →    Schema enforcement & evolution
No versioning                  →    Time travel (version history)
No updates/deletes             →    UPDATE, DELETE, MERGE support
Small files problem            →    OPTIMIZE (auto-compaction)
No audit trail                 →    Full audit history
Inconsistent reads during      →    Consistent snapshot reads
writes (dirty reads)
```

### Delta Lake Architecture

```
Delta Table on ADLS:
/delta_table/
├── _delta_log/                  ← Transaction log (JSON files)
│   ├── 00000000000000000000.json  ← Version 0
│   ├── 00000000000000000001.json  ← Version 1
│   ├── 00000000000000000002.json  ← Version 2
│   └── 00000000000000000010.checkpoint.parquet  ← Checkpoint (every 10 versions)
├── part-00000-...snappy.parquet   ← Data files (standard Parquet)
├── part-00001-...snappy.parquet
└── part-00002-...snappy.parquet
```

The **transaction log** (`_delta_log/`) is what makes Delta special. Every operation (write, update, delete) is recorded as a JSON entry in the log. This provides:
- ACID transactions
- Time travel
- Audit history

### Core Delta Lake Operations

```python
# ---- CREATE ----
# Write DataFrame as Delta table
df.write.format("delta").mode("overwrite").save("/mnt/delta/employees")

# Create managed table (metadata in Unity Catalog/Hive metastore)
df.write.format("delta").saveAsTable("catalog.schema.employees")

# ---- READ ----
df = spark.read.format("delta").load("/mnt/delta/employees")
df = spark.table("catalog.schema.employees")

# ---- UPDATE ----
from delta.tables import DeltaTable

delta_table = DeltaTable.forPath(spark, "/mnt/delta/employees")

# Update salaries for Engineering department
delta_table.update(
    condition="department = 'Engineering'",
    set={"salary": "salary * 1.10"}
)

# ---- DELETE ----
delta_table.delete("salary < 30000")

# ---- MERGE (UPSERT) — Most Important Operation ----
# Scenario: You receive daily updates. Some are new records, some are updates.

# New/updated data
updates_df = spark.read.parquet("/incoming/employee_updates/")

delta_table.alias("target").merge(
    updates_df.alias("source"),
    "target.emp_id = source.emp_id"  # Match condition
).whenMatchedUpdate(
    set={
        "name": "source.name",
        "salary": "source.salary",
        "department": "source.department"
    }
).whenNotMatchedInsert(
    values={
        "emp_id": "source.emp_id",
        "name": "source.name",
        "salary": "source.salary",
        "department": "source.department"
    }
).execute()
```

```sql
-- SQL equivalent of MERGE
MERGE INTO employees AS target
USING employee_updates AS source
ON target.emp_id = source.emp_id
WHEN MATCHED THEN
    UPDATE SET 
        target.name = source.name,
        target.salary = source.salary
WHEN NOT MATCHED THEN
    INSERT (emp_id, name, salary, department)
    VALUES (source.emp_id, source.name, source.salary, source.department);
```

### Time Travel

```python
# Read a specific version
df_v0 = spark.read.format("delta").option("versionAsOf", 0).load("/mnt/delta/employees")
df_v2 = spark.read.format("delta").option("versionAsOf", 2).load("/mnt/delta/employees")

# Read as of a timestamp
df_yesterday = spark.read.format("delta") \
    .option("timestampAsOf", "2024-01-15") \
    .load("/mnt/delta/employees")

# View history
delta_table = DeltaTable.forPath(spark, "/mnt/delta/employees")
delta_table.history().show()
```

```sql
-- SQL time travel
SELECT * FROM employees VERSION AS OF 0;
SELECT * FROM employees TIMESTAMP AS OF '2024-01-15';

-- View history
DESCRIBE HISTORY employees;
```

**Use cases for Time Travel**:
- Audit: "What did this table look like on Jan 1st?"
- Debugging: "What changed that broke the dashboard?"
- Rollback: "Undo the bad update from yesterday"
- Reproducibility: "Run ML model on last month's data snapshot"

### Schema Enforcement & Evolution

```python
# Schema Enforcement (default): Rejects writes that don't match schema
# If you try to write a DataFrame with extra/missing columns → ERROR

# Schema Evolution: Allow schema changes
df_with_new_column.write \
    .format("delta") \
    .mode("append") \
    .option("mergeSchema", "true") \  # Allows adding new columns
    .save("/mnt/delta/employees")

# Or set at table level
spark.conf.set("spark.databricks.delta.schema.autoMerge.enabled", "true")
```

### Delta Lake Optimization

```sql
-- OPTIMIZE: Compacts small files into larger ones (solves small files problem)
OPTIMIZE delta.`/mnt/delta/employees`;
OPTIMIZE employees;  -- For managed tables

-- OPTIMIZE with Z-ORDER: Co-locates related data for faster queries
-- Use Z-ORDER on columns you frequently filter on
OPTIMIZE employees ZORDER BY (department, hire_date);

-- VACUUM: Removes old/stale data files no longer referenced by the transaction log
-- Default retention: 7 days (168 hours)
VACUUM employees;              -- Default retention
VACUUM employees RETAIN 720 HOURS; -- Custom retention (30 days)

-- WARNING: VACUUM permanently deletes data files — Time Travel won't work 
-- for versions older than the retention period
```

```
OPTIMIZE with Z-ORDER Visualization:

Before Z-ORDER (data scattered):
File 1: [Sales, Engineering, Marketing, Finance, ...]
File 2: [Engineering, Sales, Finance, Marketing, ...]
File 3: [Marketing, Finance, Sales, Engineering, ...]
Query: WHERE department = 'Sales' → Must read ALL files

After Z-ORDER by department:
File 1: [Engineering, Engineering, Engineering, ...]
File 2: [Finance, Finance, Marketing, Marketing, ...]
File 3: [Sales, Sales, Sales, Sales, ...]
Query: WHERE department = 'Sales' → Reads only File 3 (data skipping!)
```

### Auto Optimize & Auto Compact

```python
# Table properties for auto optimization
spark.sql("""
    ALTER TABLE employees SET TBLPROPERTIES (
        'delta.autoOptimize.optimizeWrite' = 'true',
        'delta.autoOptimize.autoCompact' = 'true'
    )
""")
```

- **Optimize Write**: Dynamically optimizes partition sizes during writes
- **Auto Compact**: Automatically compacts small files after writes

---

## 4.3 Databricks Medallion Architecture (Bronze-Silver-Gold)

This is THE standard architecture pattern for Databricks data pipelines. You WILL be asked about this.

```
┌─────────────────────────────────────────────────────────────────┐
│                   MEDALLION ARCHITECTURE                         │
│                                                                  │
│  ┌─────────────┐    ┌──────────────┐    ┌────────────────┐     │
│  │   BRONZE     │    │    SILVER     │    │     GOLD       │     │
│  │  (Raw Data)  │───▶│ (Cleaned)    │───▶│ (Business)     │     │
│  │             │    │              │    │                │     │
│  │ • Raw ingest│    │ • Deduplicate│    │ • Aggregations │     │
│  │ • All data  │    │ • Type cast  │    │ • KPIs         │     │
│  │ • As-is from│    │ • Validate   │    │ • Dimensions   │     │
│  │   source    │    │ • Standardize│    │ • Facts         │     │
│  │ • Append    │    │ • Join refs  │    │ • Ready for     │     │
│  │   only      │    │ • Filter bad │    │   dashboards   │     │
│  │             │    │   records    │    │   & ML models  │     │
│  │ Delta Table │    │ Delta Table  │    │ Delta Table    │     │
│  └─────────────┘    └──────────────┘    └────────────────┘     │
│                                                                  │
│   Data Sources                              Consumers            │
│   ┌──────────┐                              ┌──────────┐        │
│   │ APIs     │                              │ Power BI │        │
│   │ DBs      │───▶  Bronze ─▶ Silver ─▶ Gold ──▶│ ML Models│        │
│   │ Files    │                              │ Reports  │        │
│   │ Streams  │                              │ APIs     │        │
│   └──────────┘                              └──────────┘        │
└─────────────────────────────────────────────────────────────────┘
```

### Complete Medallion Architecture Example

```python
# ====== BRONZE LAYER: Raw Ingestion ======
# Goal: Land raw data as-is, add metadata (ingestion time, source)

bronze_df = (spark.read
    .format("csv")
    .option("header", "true")
    .option("inferSchema", "true")
    .load("/mnt/raw/sales/2024-01-15/")
)

# Add metadata columns
from pyspark.sql.functions import current_timestamp, lit, input_file_name

bronze_df = bronze_df \
    .withColumn("_ingestion_timestamp", current_timestamp()) \
    .withColumn("_source_file", input_file_name()) \
    .withColumn("_source_system", lit("crm_system"))

# Write to Bronze (always APPEND — never overwrite raw data)
bronze_df.write \
    .format("delta") \
    .mode("append") \
    .save("/mnt/bronze/sales/")

# ====== SILVER LAYER: Cleaned & Standardized ======
# Goal: Clean, deduplicate, type-cast, validate, join reference data

silver_df = spark.read.format("delta").load("/mnt/bronze/sales/")

# Clean and transform
silver_df = silver_df \
    .dropDuplicates(["order_id"]) \
    .filter(col("order_id").isNotNull()) \
    .filter(col("amount") > 0) \
    .withColumn("amount", col("amount").cast("double")) \
    .withColumn("order_date", to_date(col("order_date"), "yyyy-MM-dd")) \
    .withColumn("country", upper(trim(col("country")))) \
    .drop("_source_file")

# Data quality checks
bad_records = silver_df.filter(col("customer_id").isNull())
print(f"Bad records: {bad_records.count()}")
# Optionally write bad records to a quarantine table
bad_records.write.format("delta").mode("append").save("/mnt/quarantine/sales/")

silver_df = silver_df.filter(col("customer_id").isNotNull())

# Write to Silver (MERGE for incremental updates)
from delta.tables import DeltaTable

if DeltaTable.isDeltaTable(spark, "/mnt/silver/sales/"):
    delta_silver = DeltaTable.forPath(spark, "/mnt/silver/sales/")
    delta_silver.alias("target").merge(
        silver_df.alias("source"),
        "target.order_id = source.order_id"
    ).whenMatchedUpdateAll() \
     .whenNotMatchedInsertAll() \
     .execute()
else:
    silver_df.write.format("delta").save("/mnt/silver/sales/")

# ====== GOLD LAYER: Business-Ready Aggregations ======
# Goal: Create business-level aggregations, KPIs, dimensional models

gold_daily_sales = spark.sql("""
    SELECT 
        order_date,
        country,
        COUNT(DISTINCT order_id) AS total_orders,
        COUNT(DISTINCT customer_id) AS unique_customers,
        SUM(amount) AS total_revenue,
        AVG(amount) AS avg_order_value
    FROM silver_sales
    GROUP BY order_date, country
""")

gold_daily_sales.write \
    .format("delta") \
    .mode("overwrite") \
    .save("/mnt/gold/daily_sales_summary/")
```

---

## 4.4 Unity Catalog

Unity Catalog is Databricks' unified governance solution for all data and AI assets.

```
Unity Catalog Hierarchy:
┌─────────────────────────────┐
│        METASTORE            │  (Top-level container, one per region)
│  ┌──────────────────────┐   │
│  │      CATALOG          │  │  (e.g., "production", "development")
│  │  ┌────────────────┐   │  │
│  │  │    SCHEMA       │  │  │  (e.g., "bronze", "silver", "gold")
│  │  │  ┌───────────┐  │  │  │
│  │  │  │  TABLE     │  │  │  │  (e.g., "sales", "customers")
│  │  │  │  VIEW      │  │  │  │
│  │  │  │  FUNCTION  │  │  │  │
│  │  │  └───────────┘  │  │  │
│  │  └────────────────┘   │  │
│  └──────────────────────┘   │
└─────────────────────────────┘

Three-level namespace: catalog.schema.table
Example: production.gold.daily_sales_summary
```

```sql
-- Create catalog and schema
CREATE CATALOG IF NOT EXISTS production;
CREATE SCHEMA IF NOT EXISTS production.bronze;
CREATE SCHEMA IF NOT EXISTS production.silver;
CREATE SCHEMA IF NOT EXISTS production.gold;

-- Create table in specific schema
CREATE TABLE production.bronze.raw_sales (
    order_id STRING,
    customer_id STRING,
    amount DOUBLE,
    order_date DATE
) USING DELTA;

-- Grant permissions
GRANT USAGE ON CATALOG production TO `data_engineers`;
GRANT SELECT ON SCHEMA production.gold TO `data_analysts`;
GRANT ALL PRIVILEGES ON TABLE production.bronze.raw_sales TO `etl_service_principal`;
```

Key Unity Catalog features:
- **Centralized access control** across workspaces
- **Data lineage** (track where data came from and where it goes)
- **Audit logging** (who accessed what data and when)
- **Data discovery** (search for tables, columns, descriptions)
- **Row-level and column-level security**

---

## 4.5 Databricks Workflows & Jobs

```
Databricks Workflow:
┌─────────────────────────────────────────────┐
│                  JOB                         │
│  ┌───────┐    ┌───────┐    ┌───────┐       │
│  │Task 1 │───▶│Task 2 │───▶│Task 3 │       │
│  │Bronze │    │Silver │    │ Gold  │       │
│  │Ingest │    │Clean  │    │Agg    │       │
│  └───────┘    └───────┘    └───┬───┘       │
│                                │            │
│                           ┌────▼────┐       │
│                           │ Task 4  │       │
│                           │Notify   │       │
│                           └─────────┘       │
│                                              │
│  Schedule: Daily at 2:00 AM                 │
│  Cluster: Job cluster (auto-terminate)      │
│  Alerts: Email on failure                   │
│  Retries: 2 retries with 5-min delay        │
└─────────────────────────────────────────────┘
```

```python
# In a notebook, you can pass parameters using widgets
dbutils.widgets.text("date", "2024-01-15", "Processing Date")
dbutils.widgets.dropdown("env", "dev", ["dev", "staging", "prod"], "Environment")

# Retrieve parameter values
processing_date = dbutils.widgets.get("date")
environment = dbutils.widgets.get("env")

print(f"Processing data for {processing_date} in {environment}")
```

---

## 4.6 Databricks Notebooks & Utilities

### dbutils — Your Swiss Army Knife

```python
# ---- File System Operations ----
dbutils.fs.ls("/mnt/data/")                    # List files
dbutils.fs.head("/mnt/data/file.txt")          # Preview file content
dbutils.fs.cp("/source/", "/dest/", True)      # Copy (recursive)
dbutils.fs.mv("/source/", "/dest/")            # Move
dbutils.fs.rm("/old_data/", True)              # Remove (recursive)
dbutils.fs.mkdirs("/new_folder/")              # Create directory

# ---- Secrets Management (Azure Key Vault integration) ----
# First, set up Key Vault-backed secret scope in Databricks
storage_key = dbutils.secrets.get(scope="my-key-vault-scope", key="storage-account-key")
db_password = dbutils.secrets.get(scope="my-key-vault-scope", key="database-password")

# ---- Notebook Orchestration ----
# Run another notebook (useful for modular pipeline design)
result = dbutils.notebook.run("./silver_transform", timeout_seconds=600, 
                                arguments={"date": "2024-01-15"})
print(f"Notebook returned: {result}")

# Exit notebook with a return value
dbutils.notebook.exit("Pipeline completed successfully")

# ---- Widget Parameters ----
dbutils.widgets.text("input_path", "/default/path/")
dbutils.widgets.dropdown("mode", "overwrite", ["overwrite", "append"])
path = dbutils.widgets.get("input_path")
```

### Mounting ADLS in Databricks

```python
# Mount Azure Data Lake Storage Gen2
configs = {
    "fs.azure.account.auth.type": "OAuth",
    "fs.azure.account.oauth.provider.type": "org.apache.hadoop.fs.azurebfs.oauth2.ClientCredsTokenProvider",
    "fs.azure.account.oauth2.client.id": dbutils.secrets.get("kv-scope", "client-id"),
    "fs.azure.account.oauth2.client.secret": dbutils.secrets.get("kv-scope", "client-secret"),
    "fs.azure.account.oauth2.client.endpoint": "https://login.microsoftonline.com/<tenant-id>/oauth2/token"
}

dbutils.fs.mount(
    source="abfss://container@storageaccount.dfs.core.windows.net/",
    mount_point="/mnt/data",
    extra_configs=configs
)

# Now you can access data like a local path:
df = spark.read.parquet("/mnt/data/raw/sales/")

# Modern approach: Unity Catalog External Locations (recommended over mounts)
# No mounting needed — access storage directly with governed permissions
```

---

## 🛠️ Hands-On Exercise 4.1

Build a complete mini Medallion Architecture pipeline in Databricks:

```python
# STEP 1: Create sample raw data (simulating a source system)
raw_data = [
    ("ORD001", "C001", 150.00, "2024-01-15", "COMPLETED"),
    ("ORD002", "C002", -50.00, "2024-01-15", "COMPLETED"),   # Bad: negative amount
    ("ORD003", None, 200.00, "2024-01-16", "COMPLETED"),      # Bad: null customer
    ("ORD004", "C001", 300.00, "2024-01-16", "COMPLETED"),
    ("ORD001", "C001", 150.00, "2024-01-15", "COMPLETED"),    # Duplicate
    ("ORD005", "C003", 500.00, "2024-01-17", "PENDING"),
    ("ORD006", "C004", 75.00, "2024-01-17", "COMPLETED"),
]
columns = ["order_id", "customer_id", "amount", "order_date", "status"]
raw_df = spark.createDataFrame(raw_data, columns)

# STEP 2: Bronze Layer
# Add metadata, write as Delta (append mode)

# STEP 3: Silver Layer
# Remove duplicates, filter bad records (negative amounts, null customer_ids)
# Cast types, standardize formats
# Write clean records as Delta, quarantine bad records

# STEP 4: Gold Layer
# Create daily summary: total orders, total revenue, unique customers per day
# Write as Delta

# STEP 5: Verify with Time Travel
# Make a change, then read the previous version

# STEP 6: Run OPTIMIZE and DESCRIBE HISTORY
```

---

## ❓ Practice Interview Questions — Stage 4

**Q1: What is Delta Lake, and what problems does it solve?**

> **A**: Delta Lake is an open-source storage layer that adds ACID transactions, schema enforcement, time travel, and data versioning on top of data lakes (like ADLS). It solves key data lake problems:
> 1. **No transactions** → Delta provides ACID compliance; concurrent reads/writes are safe
> 2. **Schema issues** → Delta enforces schemas on write and supports schema evolution
> 3. **No versioning** → Transaction log enables time travel to any previous version
> 4. **No updates/deletes** → Delta supports UPDATE, DELETE, and MERGE on Parquet data
> 5. **Small files** → OPTIMIZE command compacts files; Z-ORDER improves query performance
> 6. **Inconsistent reads** → Snapshot isolation ensures consistent reads during concurrent writes

**Q2: Explain the Medallion Architecture (Bronze/Silver/Gold). Why is it used?**

> **A**: The Medallion Architecture is a layered approach to data organization:
> - **Bronze (Raw)**: Raw data ingested as-is from sources. Append-only. Preserves full history. Added metadata (ingestion timestamp, source).
> - **Silver (Cleaned)**: Deduplicated, validated, type-cast, standardized data. Bad records quarantined. Conforms to business rules.
> - **Gold (Business)**: Aggregated, business-ready datasets. KPIs, dimensional models, denormalized tables optimized for specific use cases.
>
> Benefits: Clear data quality progression, easy debugging (check each layer), raw data preserved for reprocessing, separation of concerns, enables different teams to work at different layers.

**Q3: What is the difference between OPTIMIZE and VACUUM in Delta Lake?**

> **A**: 
> - **OPTIMIZE**: Compacts small Parquet files into larger ones (target ~1GB each). Optional Z-ORDER reorganizes data for faster queries on specified columns. Does NOT delete old files.
> - **VACUUM**: Permanently deletes old data files that are no longer referenced by the transaction log (older than retention period, default 7 days). After VACUUM, you cannot time travel to versions that referenced those files.
> - Use OPTIMIZE regularly to maintain read performance. Use VACUUM to reclaim storage space.

**Q4: How does the Delta Lake transaction log work?**

> **A**: Every operation on a Delta table (write, update, delete, merge) creates a new JSON file in the `_delta_log/` directory. Each JSON file records:
> - Which Parquet files were added
> - Which Parquet files were removed
> - Schema changes, metadata changes
> 
> Every 10 commits, a checkpoint file (Parquet format) is created to speed up log replay. When reading a Delta table, Spark reads the transaction log to determine which Parquet files are the "current" version. This is what enables ACID transactions, time travel, and consistent reads.

**Q5: What is Unity Catalog and why is it important?**

> **A**: Unity Catalog is Databricks' unified governance layer providing:
> - Three-level namespace (catalog.schema.table) for organizing data
> - Centralized access control (GRANT/REVOKE on catalogs, schemas, tables)
> - Data lineage tracking (upstream/downstream dependencies)
> - Audit logging (who accessed what, when)
> - Cross-workspace governance
> - Row-level and column-level security
> - It's important for compliance (GDPR, HIPAA), data discoverability, and managing access at enterprise scale.

**Q6: How would you handle schema evolution in a production pipeline?**

> **A**: 
> 1. Enable `mergeSchema` for additive changes (new columns): `.option("mergeSchema", "true")`
> 2. For breaking changes (renaming/removing columns), create a new version of the table
> 3. Use Unity Catalog to track schema versions
> 4. Implement schema validation in the Silver layer — reject non-conforming records
> 5. Use Delta Lake's schema enforcement as a safety net
> 6. Communicate schema changes through documentation and alerting

---

---

# STAGE 5: Real-Time Streaming & Event-Driven Architecture (Weeks 5-6)

---

## 5.1 Spark Structured Streaming

Structured Streaming treats a live data stream as a table that is being continually appended to.

```
Structured Streaming Mental Model:

          UNBOUNDED INPUT TABLE (continuously appended)
          ┌─────────────────────────────────────┐
Time T=1: │ Row 1 │ Row 2 │ Row 3 │            │
Time T=2: │ Row 1 │ Row 2 │ Row 3 │ Row 4 │ R5 │
Time T=3: │ Row 1 │ Row 2 │ Row 3 │ Row 4 │ R5 │ R6 │ R7 │
          └─────────────────────────────────────┘
                              │
                    ┌─────────▼──────────┐
                    │   Query / Transform │
                    │   (same as batch!)  │
                    └─────────┬──────────┘
                              │
          OUTPUT TABLE (Result of query)
          ┌─────────────────────────────────────┐
          │   Continuously updated results      │
          └─────────────────────────────────────┘
```

### Basic Streaming Pipeline

```python
# ---- READING A STREAM ----

# From a file source (Auto Loader / cloudFiles is Databricks-specific)
stream_df = (spark.readStream
    .format("cloudFiles")        # Auto Loader format
    .option("cloudFiles.format", "json")
    .option("cloudFiles.schemaLocation", "/mnt/schema/orders/")
    .load("/mnt/raw/orders/")
)

# From a Delta table (as source)
stream_df = spark.readStream.format("delta").load("/mnt/bronze/orders/")

# From Kafka / Event Hubs
stream_df = (spark.readStream
    .format("kafka")
    .option("kafka.bootstrap.servers", "host:9092")
    .option("subscribe", "orders-topic")
    .option("startingOffsets", "latest")
    .load()
)

# ---- TRANSFORMING (same as batch!) ----
transformed = stream_df \
    .filter(col("amount") > 0) \
    .withColumn("processed_time", current_timestamp()) \
    .select("order_id", "customer_id", "amount", "processed_time")

# ---- WRITING A STREAM ----
query = (transformed.writeStream
    .format("delta")
    .outputMode("append")          # append / complete / update
    .option("checkpointLocation", "/mnt/checkpoints/orders/")
    .trigger(processingTime="1 minute")  # How often to process
    .start("/mnt/silver/orders/")
)

# Wait for termination (in production, the job keeps running)
query.awaitTermination()
```

### Output Modes

```
APPEND MODE (default):
- Only NEW rows are written to the output
- Used for: Simple transforms, filters (no aggregations)
- Most common in data pipeline scenarios

COMPLETE MODE:
- Entire result table is written every trigger
- Used for: Aggregations (the complete aggregated result changes each time)
- Example: Running COUNT per category

UPDATE MODE:
- Only CHANGED rows are written
- Used for: Aggregations where you only want the updated aggregates
- More efficient than COMPLETE for large result sets
```

### Trigger Types

```python
# Process every 1 minute
.trigger(processingTime="1 minute")

# Process as fast as possible (continuous)
.trigger(processingTime="0 seconds")

# Process once (useful for batch-like processing of new files)
.trigger(once=True)  # Deprecated

# Available Files trigger (modern replacement for once=True)
.trigger(availableNow=True)  # Process all available data, then stop
```

### Checkpointing

```
Checkpointing is HOW Structured Streaming tracks progress:

/mnt/checkpoints/orders/
├── offsets/       ← What data has been read (e.g., Kafka offsets, file list)
├── commits/       ← What data has been successfully written
├── metadata       ← Query metadata
└── sources/       ← Source-specific state

WHY it's critical:
1. Fault tolerance: If the stream fails and restarts, it picks up where it left off
2. Exactly-once processing: Combined with Delta Lake, ensures no duplicates
3. Must use a RELIABLE storage (ADLS, not local disk)
4. Each stream needs its OWN checkpoint location (don't share!)
```

---

## 5.2 Auto Loader (cloudFiles)

Auto Loader is Databricks' recommended way to incrementally ingest files from cloud storage. It's more efficient than listing files manually.

```python
# Auto Loader automatically:
# 1. Discovers new files as they arrive in the source folder
# 2. Tracks which files have been processed (via checkpointing)
# 3. Handles schema inference and evolution

stream_df = (spark.readStream
    .format("cloudFiles")
    .option("cloudFiles.format", "json")
    .option("cloudFiles.schemaLocation", "/mnt/schema/events/")
    .option("cloudFiles.inferColumnTypes", "true")
    .option("cloudFiles.schemaEvolutionMode", "addNewColumns")
    .load("/mnt/landing/events/")
)

# Process and write
(stream_df
    .withColumn("_ingestion_time", current_timestamp())
    .writeStream
    .format("delta")
    .outputMode("append")
    .option("checkpointLocation", "/mnt/checkpoints/events_bronze/")
    .trigger(availableNow=True)  # Process available files, then stop
    .start("/mnt/bronze/events/")
)
```

**Auto Loader vs Traditional File Listing:**
```
Traditional (spark.read):
- Lists ALL files every time
- YOU track which are new
- Slow for millions of files

Auto Loader (cloudFiles):
- Uses Azure Event Grid notifications (file arrival events)
- OR incremental listing (efficient for smaller directories)
- Automatically tracks processed files
- Handles schema evolution
- Scales to millions of files
```

---

## 5.3 Delta Live Tables (DLT)

DLT is Databricks' declarative pipeline framework. You define WHAT you want (tables and their transformations), and DLT handles the HOW (orchestration, error handling, monitoring).

```python
# DLT uses decorators to define tables
import dlt

# Bronze: Ingest raw data
@dlt.table(
    comment="Raw orders from source system"
)
def bronze_orders():
    return (
        spark.readStream
        .format("cloudFiles")
        .option("cloudFiles.format", "json")
        .load("/mnt/landing/orders/")
    )

# Silver: Clean data with expectations (data quality rules)
@dlt.table(
    comment="Cleaned orders"
)
@dlt.expect_or_drop("valid_amount", "amount > 0")
@dlt.expect_or_drop("valid_customer", "customer_id IS NOT NULL")
@dlt.expect("valid_date", "order_date IS NOT NULL")  # Track but don't drop
def silver_orders():
    return (
        dlt.read_stream("bronze_orders")
        .select("order_id", "customer_id", "amount", "order_date", "status")
        .dropDuplicates(["order_id"])
    )

# Gold: Business aggregation
@dlt.table(
    comment="Daily sales summary"
)
def gold_daily_sales():
    return (
        dlt.read("silver_orders")
        .groupBy("order_date")
        .agg(
            count("order_id").alias("total_orders"),
            sum("amount").alias("total_revenue")
        )
    )
```

**DLT Data Quality Expectations:**
```python
@dlt.expect("rule_name", "condition")          # Log violations, keep data
@dlt.expect_or_drop("rule_name", "condition")  # Drop violating rows
@dlt.expect_or_fail("rule_name", "condition")  # Fail pipeline on violation
```

---

## 5.4 Event-Driven Architecture with Azure Services

```
Event-Driven Architecture for the Trading Platform:

┌──────────────┐     ┌─────────────────┐     ┌──────────────┐
│ Trading       │────▶│  Azure Event    │────▶│  Azure       │
│ System        │     │  Hubs / Service │     │  Databricks  │
│ (Source)      │     │  Bus            │     │  (Streaming) │
└──────────────┘     └─────────────────┘     └──────┬───────┘
                                                      │
                                              ┌───────▼───────┐
                                              │  Delta Lake    │
                                              │  Bronze/Silver │
                                              │  /Gold         │
                                              └───────┬───────┘
                                                      │
                                              ┌───────▼───────┐
                                              │  Downstream    │
                                              │  Systems /     │
                                              │  Dashboards    │
                                              └───────────────┘
```

**Azure Event Hubs** = Kafka-compatible message broker for high-throughput streaming
**Azure Service Bus** = Enterprise message broker for reliable message delivery

```python
# Reading from Azure Event Hubs (Kafka-compatible API)
connection_string = dbutils.secrets.get("kv-scope", "eventhub-connection")

kafka_options = {
    "kafka.bootstrap.servers": "your-namespace.servicebus.windows.net:9093",
    "subscribe": "orders",
    "kafka.sasl.mechanism": "PLAIN",
    "kafka.security.protocol": "SASL_SSL",
    "kafka.sasl.jaas.config": f'org.apache.kafka.common.security.plain.PlainLoginModule required username="$ConnectionString" password="{connection_string}";',
}

stream_df = (spark.readStream
    .format("kafka")
    .options(**kafka_options)
    .load()
    .select(
        col("key").cast("string"),
        from_json(col("value").cast("string"), schema).alias("data")
    )
    .select("data.*")
)
```

---

## 5.5 Streaming + Delta Lake = Exactly-Once Processing

```
                    Streaming with Delta Lake:
                    
                    ┌─── Idempotent Write ───┐
                    │                        │
Source ──▶ readStream ──▶ Transform ──▶ writeStream (Delta)
              │                               │
              │        Checkpoint              │
              └──────── tracks ────────────────┘
                       progress
                       
Key guarantees:
1. Checkpoint tracks what's been read (offsets)
2. Delta Lake's transaction log ensures atomic writes
3. If a micro-batch fails and retries, Delta prevents duplicate writes
4. Result: EXACTLY-ONCE end-to-end semantics
```

---

## 🛠️ Hands-On Exercise 5.1

```python
# Simulate streaming with a rate source (built-in for testing)
stream_df = (spark.readStream
    .format("rate")               # Generates data automatically
    .option("rowsPerSecond", 10)  # 10 rows per second
    .load()
)

# The rate source generates:
# timestamp (TimestampType) - time of generation
# value (LongType) - incrementing number

# Add some transformations
from pyspark.sql.functions import *

enriched = stream_df \
    .withColumn("category", when(col("value") % 3 == 0, "A")
                           .when(col("value") % 3 == 1, "B")
                           .otherwise("C")) \
    .withColumn("amount", (col("value") * 10.5).cast("double"))

# Write to Delta table
query = (enriched.writeStream
    .format("delta")
    .outputMode("append")
    .option("checkpointLocation", "/tmp/checkpoint/rate_stream/")
    .trigger(processingTime="10 seconds")
    .start("/tmp/delta/streaming_output/")
)

# Let it run for ~30 seconds, then:
# query.stop()

# Read the output Delta table
result = spark.read.format("delta").load("/tmp/delta/streaming_output/")
result.show()
print(f"Total records: {result.count()}")

# Check streaming aggregation
agg_query = (enriched
    .withWatermark("timestamp", "10 seconds")
    .groupBy(
        window("timestamp", "10 seconds"),
        "category"
    )
    .agg(
        count("*").alias("count"),
        sum("amount").alias("total_amount")
    )
    .writeStream
    .format("console")
    .outputMode("update")
    .trigger(processingTime="10 seconds")
    .start()
)
```

---

## ❓ Practice Interview Questions — Stage 5

**Q1: What is Structured Streaming, and how does it relate to batch processing in Spark?**

> **A**: Structured Streaming is Spark's stream processing engine built on the Spark SQL engine. It treats a live data stream as an unbounded table being continuously appended to. You can express streaming computations using the same DataFrame/Dataset API as batch queries. The engine handles incremental execution, fault tolerance, and exactly-once guarantees. Key difference: `spark.read` → `spark.readStream`, `df.write` → `df.writeStream`.

**Q2: What is a checkpoint in Spark Streaming, and why is it critical?**

> **A**: A checkpoint stores the streaming query's progress — specifically, which data has been read (source offsets) and which results have been committed (sink commits). It's stored in a reliable distributed storage (like ADLS). It's critical because:
> 1. **Fault tolerance**: If the stream fails and restarts, it resumes from the last checkpoint
> 2. **Exactly-once semantics**: Prevents reprocessing already-committed data
> 3. **Each query must have a unique checkpoint path** — sharing causes data corruption

**Q3: What is Auto Loader, and how is it different from a regular file read?**

> **A**: Auto Loader (`cloudFiles` format) efficiently discovers and processes new files arriving in cloud storage. Unlike `spark.read` which requires you to track which files are new, Auto Loader automatically tracks processed files via checkpointing. It uses Azure Event Grid notifications for near-instant file discovery (or incremental listing for smaller directories). It also handles schema inference and evolution. This is the recommended approach for incrementally loading data into Bronze layer in Databricks.

**Q4: Explain the difference between the three output modes in Structured Streaming.**

> **A**: 
> - **Append**: Only new rows are output. Used for simple filters/maps with no aggregation. Cannot be used with aggregations unless using watermarks.
> - **Complete**: The entire updated result table is output every trigger. Used for aggregations (e.g., running counts). Result table must fit in memory.
> - **Update**: Only rows that changed since the last trigger are output. More efficient than complete for aggregations. Most commonly used with Delta Lake sinks.

**Q5: How does Delta Live Tables (DLT) differ from regular Databricks notebooks for pipeline building?**

> **A**: DLT is declarative — you define WHAT tables and transformations you want, not HOW to orchestrate them. Benefits over notebooks:
> 1. Automatic dependency management (DLT figures out execution order)
> 2. Built-in data quality with `expectations` (expect, expect_or_drop, expect_or_fail)
> 3. Automatic error handling and recovery
> 4. Built-in monitoring dashboard
> 5. Manages streaming and batch in the same framework
> 6. Handles checkpointing, incremental processing, and schema evolution automatically
> Trade-off: Less control and flexibility compared to custom notebook pipelines.

---

---

# STAGE 6: Azure DevOps, CI/CD & Infrastructure (Week 6-7)

---

## 6.1 Azure DevOps Overview

Azure DevOps is Microsoft's platform for planning, developing, delivering, and monitoring software.

```
Azure DevOps Services:
┌────────────────────────────────────────────────┐
│                                                 │
│  ┌──────────┐  ┌───────┐  ┌───────────────┐  │
│  │  Boards   │  │ Repos │  │  Pipelines    │  │
│  │ (Agile    │  │ (Git  │  │ (CI/CD)       │  │
│  │  tracking)│  │  repos)│  │               │  │
│  └──────────┘  └───────┘  └───────────────┘  │
│                                                 │
│  ┌──────────────┐  ┌───────────────────────┐  │
│  │  Test Plans   │  │  Artifacts            │  │
│  │  (Testing)    │  │  (Package management) │  │
│  └──────────────┘  └───────────────────────┘  │
│                                                 │
└────────────────────────────────────────────────┘
```

### Azure Repos — Version Control for Databricks

```
Branching Strategy for Data Pipelines:

main (production)
  │
  ├── develop (integration/staging)
  │     │
  │     ├── feature/add-customer-pipeline
  │     ├── feature/fix-sales-dedup
  │     └── bugfix/null-handling
  │
  └── release/v2.1
```

```bash
# Basic Git commands
git clone https://dev.azure.com/org/project/_git/databricks-pipelines
git checkout -b feature/add-customer-pipeline
git add .
git commit -m "Add customer pipeline bronze layer"
git push origin feature/add-customer-pipeline
# Create Pull Request in Azure DevOps UI
```

### Databricks Repos Integration

Databricks has built-in Git integration:
1. In Databricks, go to **Repos**
2. Connect to Azure DevOps repository
3. Clone the repo into Databricks
4. Work on feature branches
5. Commit and push from Databricks UI
6. Create Pull Requests in Azure DevOps

---

## 6.2 CI/CD for Databricks

CI/CD automates testing and deployment of your data pipelines.

```
CI/CD Pipeline Flow:

Developer pushes code to feature branch
         │
         ▼
┌─────────────────────────┐
│   CONTINUOUS INTEGRATION │
│   (CI Pipeline)          │
│                          │
│   1. Code checkout       │
│   2. Run linting/format  │
│   3. Run unit tests      │
│   4. Run integration     │
│      tests               │
│   5. Build artifacts     │
│      (wheel files,       │
│       configs)           │
└────────────┬────────────┘
             │ If all pass...
             ▼
┌─────────────────────────┐
│   CONTINUOUS DEPLOYMENT  │
│   (CD Pipeline)          │
│                          │
│   DEV Environment:       │
│   1. Deploy to dev       │
│   2. Run smoke tests     │
│                          │
│   STAGING Environment:   │
│   3. Deploy to staging   │
│   4. Run full tests      │
│   5. Approval gate       │
│                          │
│   PRODUCTION Environment:│
│   6. Deploy to prod      │
│   7. Run health checks   │
│   8. Monitor             │
└─────────────────────────┘
```

### Azure DevOps Pipeline YAML (azure-pipelines.yml)

```yaml
# CI Pipeline: Triggered on pull request to main
trigger:
  branches:
    include:
      - main
      - develop

pr:
  branches:
    include:
      - main

pool:
  vmImage: 'ubuntu-latest'

variables:
  pythonVersion: '3.10'
  databricksHost: '$(DATABRICKS_HOST)'
  databricksToken: '$(DATABRICKS_TOKEN)'

stages:
  # ===== CI STAGE =====
  - stage: Build
    displayName: 'Build and Test'
    jobs:
      - job: Test
        displayName: 'Run Tests'
        steps:
          # Install Python
          - task: UsePythonVersion@0
            inputs:
              versionSpec: '$(pythonVersion)'

          # Install dependencies
          - script: |
              pip install -r requirements.txt
              pip install pytest databricks-cli
            displayName: 'Install dependencies'

          # Run linting
          - script: |
              pip install flake8 black
              flake8 src/ --max-line-length 120
              black --check src/
            displayName: 'Lint and Format Check'

          # Run unit tests
          - script: |
              pytest tests/unit/ -v --junitxml=test-results.xml
            displayName: 'Run Unit Tests'

          # Publish test results
          - task: PublishTestResults@2
            inputs:
              testResultsFormat: 'JUnit'
              testResultsFiles: '**/test-results.xml'

  # ===== CD STAGE: Deploy to Dev =====
  - stage: DeployDev
    displayName: 'Deploy to Dev'
    dependsOn: Build
    condition: succeeded()
    jobs:
      - deployment: DeployDev
        environment: 'dev'
        strategy:
          runOnce:
            deploy:
              steps:
                - script: |
                    # Configure Databricks CLI
                    databricks configure --token <<EOF
                    $(DATABRICKS_HOST_DEV)
                    $(DATABRICKS_TOKEN_DEV)
                    EOF
                    
                    # Deploy notebooks to dev workspace
                    databricks workspace import_dir ./notebooks /Repos/dev/pipelines --overwrite
                    
                    # Deploy/update job definitions
                    python scripts/deploy_jobs.py --env dev
                  displayName: 'Deploy to Dev Databricks'

  # ===== CD STAGE: Deploy to Prod (with approval) =====
  - stage: DeployProd
    displayName: 'Deploy to Production'
    dependsOn: DeployDev
    condition: and(succeeded(), eq(variables['Build.SourceBranch'], 'refs/heads/main'))
    jobs:
      - deployment: DeployProd
        environment: 'production'  # Has manual approval gate configured
        strategy:
          runOnce:
            deploy:
              steps:
                - script: |
                    databricks configure --token <<EOF
                    $(DATABRICKS_HOST_PROD)
                    $(DATABRICKS_TOKEN_PROD)
                    EOF
                    
                    databricks workspace import_dir ./notebooks /Repos/prod/pipelines --overwrite
                    python scripts/deploy_jobs.py --env prod
                  displayName: 'Deploy to Production Databricks'
```

### Deploying Databricks Jobs via API

```python
# scripts/deploy_jobs.py
import requests
import json
import sys

def deploy_job(host, token, job_config):
    headers = {"Authorization": f"Bearer {token}"}
    
    # Check if job exists
    response = requests.get(
        f"{host}/api/2.1/jobs/list",
        headers=headers,
        params={"name": job_config["name"]}
    )
    
    jobs = response.json().get("jobs", [])
    
    if jobs:
        # Update existing job
        job_id = jobs[0]["job_id"]
        job_config["job_id"] = job_id
        requests.post(
            f"{host}/api/2.1/jobs/reset",
            headers=headers,
            json={"job_id": job_id, "new_settings": job_config}
        )
        print(f"Updated job: {job_config['name']} (ID: {job_id})")
    else:
        # Create new job
        response = requests.post(
            f"{host}/api/2.1/jobs/create",
            headers=headers,
            json=job_config
        )
        job_id = response.json()["job_id"]
        print(f"Created job: {job_config['name']} (ID: {job_id})")

# Job configuration
job_config = {
    "name": "Daily_Sales_Pipeline",
    "tasks": [
        {
            "task_key": "bronze_ingest",
            "notebook_task": {
                "notebook_path": "/Repos/prod/pipelines/bronze/ingest_sales"
            },
            "new_cluster": {
                "spark_version": "14.3.x-scala2.12",
                "node_type_id": "Standard_DS3_v2",
                "num_workers": 2
            }
        },
        {
            "task_key": "silver_transform",
            "depends_on": [{"task_key": "bronze_ingest"}],
            "notebook_task": {
                "notebook_path": "/Repos/prod/pipelines/silver/transform_sales"
            },
            "existing_cluster_id": "0123-456789-abcdef"
        }
    ],
    "schedule": {
        "quartz_cron_expression": "0 0 2 * * ?",  # Daily at 2 AM
        "timezone_id": "UTC"
    },
    "email_notifications": {
        "on_failure": ["team@company.com"]
    }
}
```

---

## 6.3 Testing Databricks Pipelines

### Unit Testing PySpark Code

```python
# tests/unit/test_transformations.py
import pytest
from pyspark.sql import SparkSession
from pyspark.sql.functions import col
from src.transformations import clean_sales_data  # Your transformation function

@pytest.fixture(scope="session")
def spark():
    """Create a SparkSession for testing."""
    return SparkSession.builder \
        .master("local[2]") \
        .appName("unit-tests") \
        .getOrCreate()

def test_clean_sales_removes_negative_amounts(spark):
    """Test that negative amounts are filtered out."""
    data = [
        ("ORD1", "C1", 100.0),
        ("ORD2", "C2", -50.0),  # Should be removed
        ("ORD3", "C3", 200.0),
    ]
    df = spark.createDataFrame(data, ["order_id", "customer_id", "amount"])
    
    result = clean_sales_data(df)
    
    assert result.count() == 2
    assert result.filter(col("amount") < 0).count() == 0

def test_clean_sales_removes_null_customers(spark):
    """Test that null customer_ids are filtered out."""
    data = [
        ("ORD1", "C1", 100.0),
        ("ORD2", None, 200.0),  # Should be removed
    ]
    df = spark.createDataFrame(data, ["order_id", "customer_id", "amount"])
    
    result = clean_sales_data(df)
    
    assert result.count() == 1

def test_clean_sales_removes_duplicates(spark):
    """Test that duplicate order_ids are removed."""
    data = [
        ("ORD1", "C1", 100.0),
        ("ORD1", "C1", 100.0),  # Duplicate
        ("ORD2", "C2", 200.0),
    ]
    df = spark.createDataFrame(data, ["order_id", "customer_id", "amount"])
    
    result = clean_sales_data(df)
    
    assert result.count() == 2
```

```python
# src/transformations.py
from pyspark.sql import DataFrame
from pyspark.sql.functions import col

def clean_sales_data(df: DataFrame) -> DataFrame:
    """Clean sales data: remove negatives, nulls, duplicates."""
    return (df
        .filter(col("amount") > 0)
        .filter(col("customer_id").isNotNull())
        .dropDuplicates(["order_id"])
    )
```

---

## 6.4 Azure Log Analytics & Monitoring

```
Monitoring Stack for Databricks:
┌─────────────────────────────────────────┐
│              Azure Monitor               │
│  ┌──────────────┐  ┌────────────────┐  │
│  │Log Analytics  │  │  Azure Alerts   │  │
│  │  Workspace    │  │                 │  │
│  │              │  │ • Job failures   │  │
│  │ • Cluster    │  │ • Long run time  │  │
│  │   logs       │  │ • Data quality   │  │
│  │ • Job logs   │  │   thresholds     │  │
│  │ • Spark logs │  │ • Cluster costs  │  │
│  │ • Audit logs │  │                 │  │
│  └──────────────┘  └────────────────┘  │
│                                          │
│  Databricks sends diagnostic logs to     │
│  Log Analytics via diagnostic settings   │
└─────────────────────────────────────────┘
```

```python
# In Databricks: Send custom metrics/logs to Azure Monitor
# Or use Databricks built-in monitoring:

# Check job run status programmatically
import requests

def check_pipeline_health(host, token, job_id):
    headers = {"Authorization": f"Bearer {token}"}
    response = requests.get(
        f"{host}/api/2.1/jobs/runs/list",
        headers=headers,
        params={"job_id": job_id, "limit": 5}
    )
    runs = response.json().get("runs", [])
    
    for run in runs:
        status = run["state"]["result_state"]
        duration = run.get("run_duration", 0) / 1000 / 60  # minutes
        print(f"Run {run['run_id']}: {status}, Duration: {duration:.1f} min")
        
        if status == "FAILED":
            print(f"  Error: {run['state'].get('state_message', 'Unknown')}")
```

---

## 6.5 Azure Networking Concepts (Nice-to-Have)

```
VNet-Injected Databricks Workspace:
┌─────────────────────────────────────────────┐
│              Azure VNet                      │
│  ┌────────────────┐  ┌────────────────────┐ │
│  │ Public Subnet   │  │ Private Subnet     │ │
│  │ (Driver nodes)  │  │ (Worker nodes)     │ │
│  │                 │  │                    │ │
│  └────────────────┘  └────────────────────┘ │
│                                              │
│  ┌────────────────────────────────────────┐ │
│  │ Network Security Groups (NSGs)         │ │
│  │ • Control inbound/outbound traffic     │ │
│  └────────────────────────────────────────┘ │
│                                              │
│  ┌────────────────────────────────────────┐ │
│  │ Private Endpoints                      │ │
│  │ • ADLS ← Private connection (no public)│ │
│  │ • Key Vault ← Private connection       │ │
│  │ • SQL DB ← Private connection          │ │
│  └────────────────────────────────────────┘ │
└─────────────────────────────────────────────┘

Key concepts:
• VNet Injection: Databricks cluster runs in YOUR VNet (more control)
• Private Endpoints: Access Azure services without going over public internet
• NSGs: Firewall rules for subnets
• Service Endpoints: Route traffic to Azure services via Microsoft backbone
```

---

## 🛠️ Hands-On Exercise 6.1

1. **Create an Azure DevOps project**: https://dev.azure.com/
2. **Initialize a Git repository** with this structure:
```
databricks-pipelines/
├── notebooks/
│   ├── bronze/
│   │   └── ingest_sales.py
│   ├── silver/
│   │   └── transform_sales.py
│   └── gold/
│       └── aggregate_sales.py
├── src/
│   └── transformations.py
├── tests/
│   └── unit/
│       └── test_transformations.py
├── configs/
│   ├── dev.json
│   └── prod.json
├── azure-pipelines.yml
├── requirements.txt
└── README.md
```
3. **Connect the repo to Databricks Repos**
4. **Create a simple CI pipeline** that runs unit tests

---

## ❓ Practice Interview Questions — Stage 6

**Q1: How would you set up CI/CD for Databricks pipelines?**

> **A**: 
> 1. **Version Control**: Store notebooks and code in Azure DevOps Repos. Use Databricks Repos for Git integration.
> 2. **Branching**: Use feature branches, develop, and main. PRs require code review.
> 3. **CI Pipeline**: On PR, run linting (flake8/black), unit tests (pytest with local Spark), and integration tests.
> 4. **CD Pipeline**: On merge to main, deploy notebooks/jobs to dev environment first, run smoke tests. After approval, deploy to staging, then production.
> 5. **Deployment**: Use Databricks REST API or CLI to deploy notebooks (`workspace import_dir`), create/update jobs (`jobs/create` or `jobs/reset`), and manage clusters.
> 6. **Secrets**: Store tokens and connection strings in Azure DevOps variable groups linked to Azure Key Vault.
> 7. **Environment-specific configs**: Use parameter files (dev.json, prod.json) for different storage paths, cluster sizes, etc.

**Q2: How do you promote code from development to production in Databricks?**

> **A**: 
> 1. Developer works on a feature branch in Databricks Repos
> 2. Code is tested locally (unit tests) and in dev workspace
> 3. Developer creates a Pull Request to the develop/main branch
> 4. Code review by peers
> 5. CI pipeline runs automated tests
> 6. After approval and merge, CD pipeline deploys to staging
> 7. Integration/smoke tests run in staging
> 8. Manual approval gate for production deployment
> 9. CD pipeline deploys to production workspace
> 10. Monitoring confirms successful deployment

**Q3: How do you handle secrets and credentials in Databricks pipelines?**

> **A**: 
> 1. Store all secrets in **Azure Key Vault**
> 2. Create a **Key Vault-backed secret scope** in Databricks
> 3. In code, access secrets using `dbutils.secrets.get(scope, key)` — secrets are never displayed in logs or notebook output (redacted as `[REDACTED]`)
> 4. In CI/CD pipelines, use Azure DevOps **variable groups** linked to Key Vault
> 5. Never hardcode credentials in notebooks or config files
> 6. Use **Service Principals** (not personal tokens) for production automation

---

---

# STAGE 7: Data Governance, Security & Best Practices (Week 7-8)

---

## 7.1 Data Governance

```
Data Governance Framework:
┌─────────────────────────────────────────────────────┐
│                   DATA GOVERNANCE                    │
│                                                      │
│  ┌───────────────┐  ┌──────────────┐  ┌───────────┐│
│  │ Data Quality   │  │ Data Lineage │  │ Data      ││
│  │               │  │              │  │ Catalog   ││
│  │ • Completeness│  │ • Where data │  │ • Table   ││
│  │ • Accuracy    │  │   came from  │  │   metadata││
│  │ • Consistency │  │ • How it was │  │ • Column  ││
│  │ • Timeliness  │  │   transformed│  │   descriptions│
│  │ • Uniqueness  │  │ • Who uses it│  │ • Tags    ││
│  └───────────────┘  └──────────────┘  └───────────┘│
│                                                      │
│  ┌───────────────┐  ┌──────────────┐  ┌───────────┐│
│  │ Access Control │  │ Compliance   │  │ Auditing  ││
│  │               │  │              │  │           ││
│  │ • RBAC        │  │ • GDPR       │  │ • Who     ││
│  │ • Column-level│  │ • HIPAA      │  │   accessed││
│  │ • Row-level   │  │ • SOX        │  │   what    ││
│  │ • Dynamic     │  │ • PCI-DSS    │  │ • When    ││
│  │   data masking│  │              │  │ • Changes ││
│  └───────────────┘  └──────────────┘  └───────────┘│
└─────────────────────────────────────────────────────┘
```

### Data Quality Framework in Databricks

```python
# Implementing data quality checks
from pyspark.sql.functions import col, count, when, isnan, isnull

def run_data_quality_checks(df, table_name):
    """Run comprehensive data quality checks."""
    results = {}
    total_rows = df.count()
    results["total_rows"] = total_rows
    
    # 1. Completeness: Check for nulls in critical columns
    critical_columns = ["order_id", "customer_id", "amount"]
    for col_name in critical_columns:
        null_count = df.filter(col(col_name).isNull()).count()
        null_pct = (null_count / total_rows * 100) if total_rows > 0 else 0
        results[f"{col_name}_null_pct"] = null_pct
        if null_pct > 5:  # Alert if > 5% nulls
            print(f"⚠️ WARNING: {col_name} has {null_pct:.1f}% null values")
    
    # 2. Uniqueness: Check for duplicates
    unique_keys = df.select("order_id").distinct().count()
    dup_count = total_rows - unique_keys
    results["duplicate_count"] = dup_count
    if dup_count > 0:
        print(f"⚠️ WARNING: {dup_count} duplicate order_ids found")
    
    # 3. Validity: Check for invalid values
    invalid_amounts = df.filter(col("amount") <= 0).count()
    results["invalid_amounts"] = invalid_amounts
    
    # 4. Freshness: Check latest data timestamp
    latest_date = df.agg({"order_date": "max"}).collect()[0][0]
    results["latest_date"] = str(latest_date)
    
    # Log results
    print(f"\n📊 Data Quality Report for {table_name}:")
    for key, value in results.items():
        print(f"  {key}: {value}")
    
    # Write quality metrics to a monitoring table
    from pyspark.sql import Row
    metrics_df = spark.createDataFrame([Row(**results, table_name=table_name, 
                                             check_time=current_timestamp())])
    metrics_df.write.format("delta").mode("append").save("/mnt/monitoring/data_quality/")
    
    return results
```

### Using DLT Expectations for Data Quality

```python
import dlt

@dlt.table
@dlt.expect("valid_order_id", "order_id IS NOT NULL")
@dlt.expect("valid_amount", "amount > 0")
@dlt.expect_or_drop("valid_customer", "customer_id IS NOT NULL")
@dlt.expect_or_fail("reasonable_amount", "amount < 1000000")  # Pipeline fails if violated
def silver_orders():
    return dlt.read_stream("bronze_orders")
```

---

## 7.2 Security Best Practices

### Authentication & Authorization

```
Azure Databricks Security Layers:

1. IDENTITY (Who are you?)
   • Azure Active Directory (Entra ID) authentication
   • Service Principals for automation
   • Personal Access Tokens (PATs) — avoid in production

2. ACCESS CONTROL (What can you do?)
   • Unity Catalog — GRANT/REVOKE on catalogs, schemas, tables
   • Workspace-level permissions (Admin, User, Reader)
   • Cluster policies (limit VM types, auto-termination)
   • Token-based access for APIs

3. DATA PROTECTION (Protecting data at rest and in transit)
   • Encryption at rest (Azure Storage Service Encryption)
   • Encryption in transit (TLS/SSL)
   • Column-level encryption for sensitive data
   • Dynamic data masking
   • Row-level security

4. NETWORK SECURITY
   • VNet injection (deploy in your own VNet)
   • Private endpoints (no public internet access)
   • IP access lists (whitelist specific IPs)
   • NSGs (network security groups)

5. AUDITING
   • Azure Diagnostic Logs → Log Analytics
   • Unity Catalog audit logs
   • Databricks audit logging
```

```sql
-- Unity Catalog Access Control Examples

-- Grant read access to analysts on gold tables
GRANT USAGE ON CATALOG production TO `data_analysts`;
GRANT USAGE ON SCHEMA production.gold TO `data_analysts`;
GRANT SELECT ON SCHEMA production.gold TO `data_analysts`;

-- Grant full access to engineers on all layers
GRANT ALL PRIVILEGES ON CATALOG production TO `data_engineers`;

-- Dynamic views for row-level security
CREATE VIEW production.gold.sales_by_region AS
SELECT * FROM production.gold.sales
WHERE region = current_user_region();

-- Column masking for PII
CREATE FUNCTION mask_email(email STRING) 
RETURNS STRING
RETURN CONCAT(LEFT(email, 2), '***@***', SUBSTRING(email, INSTR(email, '.'), LEN(email)));
```

---

## 7.3 Performance Optimization Checklist

```
Performance Optimization Checklist for Production:

DATA STORAGE:
□ Use Delta format for all tables
□ Partition by low-cardinality columns (date, region)
□ Run OPTIMIZE regularly (or enable Auto Optimize)
□ Use Z-ORDER on frequently filtered columns
□ VACUUM old files to reclaim storage
□ Avoid small files (coalesce before writes)

QUERIES:
□ Push filters early (predicate pushdown)
□ Select only needed columns (column pruning)
□ Use broadcast joins for small tables
□ Avoid collect() on large datasets
□ Cache DataFrames used multiple times
□ Use explain() to verify execution plans

CLUSTERS:
□ Right-size cluster (don't over/under provision)
□ Enable auto-scaling
□ Use spot instances for workers
□ Use Photon runtime for SQL-heavy workloads
□ Set auto-termination
□ Use job clusters for scheduled pipelines (cheaper)

SPARK CONFIG:
□ spark.sql.shuffle.partitions = ~2-3x number of cores
□ spark.sql.adaptive.enabled = true (AQE)
□ spark.sql.autoBroadcastJoinThreshold (adjust for your data)
□ Enable dynamic partition pruning
```

### Adaptive Query Execution (AQE) — Automatically Optimizes at Runtime

```python
# Enabled by default in Databricks
spark.conf.set("spark.sql.adaptive.enabled", "true")

# AQE features:
# 1. Coalescing post-shuffle partitions (reduces small partitions)
# 2. Converting sort-merge join to broadcast join (if runtime statistics show small table)
# 3. Optimizing skewed joins (splits skewed partitions)
```

---

## 7.4 Architectural Design & Technical Documentation

Since the JD mentions "Producing Architectural design diagram/Technical Documentation":

```
Data Pipeline Technical Document Template:

1. OVERVIEW
   - Pipeline name, purpose, owner
   - Business context

2. ARCHITECTURE DIAGRAM
   - Source systems → Ingestion → Bronze → Silver → Gold → Consumers
   - Include all Azure services involved

3. DATA FLOW
   - Source: What data, what format, how often
   - Transformations: Business rules applied at each layer
   - Target: Where data lands, who consumes it

4. SCHEMA
   - Table schemas (columns, types, descriptions)
   - Partitioning strategy
   - Data retention policy

5. SCHEDULING
   - Trigger type (scheduled, event-driven, streaming)
   - Schedule (cron expression)
   - Dependencies (upstream/downstream)

6. ERROR HANDLING
   - Retry strategy
   - Dead letter queue
   - Alerting (who gets notified, how)

7. SECURITY
   - Access controls
   - Encryption
   - PII handling

8. MONITORING
   - KPIs (records processed, latency, error rate)
   - Dashboards
   - Alerts

9. OPERATIONAL RUNBOOK
   - How to restart failed pipelines
   - How to backfill data
   - Contact information
```

---

## ❓ Practice Interview Questions — Stage 7

**Q1: How do you ensure data quality in your pipelines?**

> **A**: Multi-layered approach:
> 1. **Schema enforcement** at Bronze layer (Delta Lake rejects non-conforming writes)
> 2. **Validation rules** at Silver layer (null checks, range checks, format checks)
> 3. **DLT Expectations** for declarative quality rules (expect, expect_or_drop, expect_or_fail)
> 4. **Quarantine tables** for bad records (don't silently drop data)
> 5. **Data quality monitoring** — write metrics to a monitoring table, set up alerts for anomalies
> 6. **Reconciliation** — compare record counts between source and target
> 7. **Automated tests** — unit tests for transformation logic, integration tests for end-to-end pipelines

**Q2: How do you handle PII (Personally Identifiable Information) in Databricks?**

> **A**: 
> 1. **Identify PII columns** (names, emails, SSNs, etc.) and tag them in Unity Catalog
> 2. **Column-level access control** via Unity Catalog — restrict who can see PII columns
> 3. **Dynamic data masking** — show masked values to unauthorized users
> 4. **Encryption** — encrypt sensitive columns using Azure Key Vault managed keys
> 5. **Row-level security** — use dynamic views to filter data based on user identity
> 6. **Data retention** — implement VACUUM policies and TTL for compliance (e.g., GDPR right to be forgotten)
> 7. **Audit logging** — track all access to PII data via Unity Catalog audit logs

**Q3: How would you optimize a slow Databricks pipeline?**

> **A**: Systematic approach:
> 1. **Identify bottleneck**: Use Spark UI to find slow stages, check for data skew, shuffles
> 2. **Data-level fixes**: 
>    - Add partitioning on filter columns
>    - Run OPTIMIZE + Z-ORDER on Delta tables
>    - Fix small files problem
> 3. **Query-level fixes**:
>    - Push filters early
>    - Use broadcast joins for small tables
>    - Avoid unnecessary shuffles (e.g., use `coalesce` instead of `repartition`)
>    - Cache repeatedly used DataFrames
> 4. **Cluster-level fixes**:
>    - Use Photon runtime
>    - Enable AQE (Adaptive Query Execution)
>    - Right-size cluster (monitor CPU/memory utilization)
>    - Use memory-optimized VMs for heavy joins
> 5. **Architecture fixes**:
>    - Pre-aggregate data (move aggregations to Gold layer)
>    - Use incremental processing (don't reprocess everything)
>    - Materialize expensive joins

**Q4: Describe how you would design a data pipeline for a trading platform.**

> **A**: 
> - **Sources**: Trading systems generate orders, trades, positions, market data
> - **Ingestion**: 
>   - Real-time: Azure Event Hubs → Structured Streaming → Bronze (for trade executions, market data)
>   - Batch: File drops to ADLS → Auto Loader → Bronze (for daily reconciliation, reference data)
> - **Bronze Layer**: Raw data as-is, append-only, with ingestion metadata
> - **Silver Layer**: Validated trades (amount > 0, valid instruments, no duplicates), enriched with reference data (counterparty info, instrument details), deduplication
> - **Gold Layer**: Position calculations, P&L aggregations, regulatory reports (MiFID, Dodd-Frank), risk metrics
> - **Consumers**: Power BI for dashboards, downstream systems via Delta Sharing, regulatory reporting systems
> - **Security**: VNet injection, private endpoints, PII masking, Unity Catalog access controls, audit logging
> - **Monitoring**: Data quality checks at each layer, latency monitoring for streaming, alert on trade processing failures

---

---

# STAGE 8: Interview Preparation & Mock Scenarios (Weeks 8+)

---

## 8.1 Common Interview Question Categories

### Category 1: Tell Me About Yourself / Experience

> **Template**: "I have X years of experience in software development, with the last Y years focused on data engineering. I've worked on building data pipelines using [technologies]. Most recently, I [describe a relevant project]. I'm experienced with Azure Databricks, PySpark, Delta Lake, and CI/CD with Azure DevOps. I'm excited about this role because [specific reasons related to the JD — trading platform, SaaS data layer, real-time streaming]."

### Category 2: Technical Deep Dives

**Q: Walk me through how you would design a data pipeline from scratch.**

> **A (structured answer)**:
> 1. **Understand requirements**: What's the source? What's the frequency? What's the SLA? Who consumes the data?
> 2. **Design architecture**: Choose Medallion Architecture (Bronze/Silver/Gold). Draw the architecture diagram.
> 3. **Ingestion**: 
>    - Batch: Auto Loader for file-based sources
>    - Streaming: Structured Streaming + Event Hubs for real-time
> 4. **Storage**: ADLS Gen2 with Delta Lake format
> 5. **Transformation**: PySpark for complex logic, SQL for simple aggregations
> 6. **Orchestration**: Databricks Workflows (Jobs with task dependencies)
> 7. **Quality**: DLT expectations or custom validation at Silver layer
> 8. **Security**: Unity Catalog for access control, Key Vault for secrets
> 9. **CI/CD**: Azure DevOps pipelines for automated testing and deployment
> 10. **Monitoring**: Log Analytics, custom quality metrics, alerts
> 11. **Documentation**: Architecture diagram, data flow documentation, runbook

**Q: You have a pipeline that takes 4 hours to run. How would you reduce it?**

> **A**:
> 1. **Profile**: Check Spark UI for stages, tasks, shuffles, skew
> 2. **Quick wins**: 
>    - Are we reading more data than needed? (Add partition pruning, filter pushdown)
>    - Are there unnecessary shuffles? (Review joins, groupBys)
>    - Small files? (Run OPTIMIZE on source tables)
> 3. **Join optimization**: 
>    - Broadcast small tables
>    - Skew handling (salt keys)
>    - Pre-partition data by join keys
> 4. **Compute**: 
>    - Increase cluster size
>    - Use Photon runtime
>    - Enable AQE
> 5. **Architecture**: 
>    - Switch from full refresh to incremental processing
>    - Pre-materialize expensive subqueries
>    - Parallelize independent tasks
> 6. **Measure**: Run the pipeline again and compare metrics

### Category 3: Scenario-Based Questions

**Q: A downstream team reports that their dashboard shows wrong numbers after yesterday's pipeline run. How do you investigate?**

> **A**:
> 1. **Time Travel**: Use Delta Lake time travel to compare yesterday's data with the version before the pipeline ran — identify what changed
> 2. **Check logs**: Review Databricks job run logs for errors/warnings
> 3. **Data Quality**: Check quality metrics table — did any quality checks fail?
> 4. **Lineage**: Trace data lineage from Gold → Silver → Bronze → Source
> 5. **Source**: Did the source system send bad data? Compare source file/counts with Bronze layer
> 6. **Transformation**: Review the Silver transformation logic — any recent code changes?
> 7. **Rollback if needed**: Use Delta Lake's `RESTORE TABLE employees VERSION AS OF X` to restore the last known good version
> 8. **Root cause fix**: Fix the issue, add test cases to prevent recurrence
> 9. **Communication**: Update the downstream team with findings and remediation

**Q: You need to process data from 50 different client environments (SaaS). How would you architect this?**

> **A (relevant to the JD — SaaS Data Layer)**:
> 1. **Multi-tenant design**: Use parameterized pipelines where client_id is a parameter
> 2. **Data isolation**: 
>    - Option A: Separate Delta schemas per client (catalog.client_A.table, catalog.client_B.table)
>    - Option B: Shared tables with client_id column + row-level security via Unity Catalog
> 3. **Pipeline template**: Create a generic pipeline notebook that takes client config as input
> 4. **Orchestration**: Databricks Workflows with dynamic task generation — loop through clients
> 5. **Configuration**: Store client-specific configs (source paths, schemas, business rules) in a metadata table
> 6. **Scaling**: Use job clusters that auto-scale per client workload
> 7. **Monitoring**: Per-client data quality metrics and latency tracking
> 8. **Security**: Separate access controls per client via Unity Catalog

### Category 4: Behavioral Questions

**Q: Tell me about a time you handled a production incident.**

> **STAR Format**:
> - **Situation**: "Our daily pipeline that feeds the trading dashboard failed at 3 AM, and traders wouldn't have updated positions by market open."
> - **Task**: "I was on-call and needed to diagnose and fix the issue before 7 AM market open."
> - **Action**: "I checked the job logs and found a schema change in the source system — a new column was added that broke our schema validation. I used Delta Lake time travel to verify the Gold tables still had yesterday's data. I added mergeSchema=true to handle the new column, tested in staging, and reran the pipeline."
> - **Result**: "Pipeline completed by 5:30 AM, well before market open. I then added automated schema change detection alerts and updated our runbook."

---

## 8.2 System Design Interview Practice

**Design a Real-Time Trade Processing System on Azure Databricks**

```
                    SYSTEM DESIGN ANSWER

┌──────────┐     ┌────────────┐     ┌──────────────────────┐
│ Trading   │────▶│ Azure Event│────▶│ Databricks           │
│ Systems   │     │ Hubs       │     │ Structured Streaming │
│ (Source)  │     │ (Buffer)   │     │                      │
└──────────┘     └────────────┘     │ ┌──────────────────┐ │
                                     │ │ Bronze Layer     │ │
                                     │ │ Raw trades       │ │
                                     │ │ (append-only)    │ │
                                     │ └────────┬─────────┘ │
                                     │          │            │
                                     │ ┌────────▼─────────┐ │
                                     │ │ Silver Layer     │ │
                                     │ │ Validated trades │ │
                                     │ │ Enriched with    │ │
                                     │ │ reference data   │ │
                                     │ └────────┬─────────┘ │
                                     │          │            │
                                     │ ┌────────▼─────────┐ │
┌───────────┐                        │ │ Gold Layer       │ │
│ Power BI  │◀───────────────────────│ │ Positions, P&L,  │ │
│ Dashboards│                        │ │ Risk Metrics     │ │
└───────────┘                        │ └──────────────────┘ │
                                     └──────────────────────┘
┌───────────┐
│ Regulatory│◀── Daily batch export from Gold
│ Reporting │
└───────────┘

Design Decisions:
• Event Hubs: Handles millions of events/sec, Kafka-compatible
• Structured Streaming: Near real-time processing (micro-batch: 10-30 sec)
• Delta Lake: ACID transactions ensure trade data integrity
• Bronze: Raw trades for audit trail (regulatory requirement)
• Silver: Validated, deduplicated, enriched trades
• Gold: Pre-computed positions and P&L for fast dashboard queries
• Unity Catalog: Client-level data isolation (multi-tenant)
• Monitoring: Alert on trade processing latency > threshold
```

---

## 8.3 Quick Reference — Key Things to Remember

### Azure Databricks Key Features
```
• Apache Spark-based distributed processing
• Delta Lake for reliable data storage (ACID, time travel, schema evolution)
• Unity Catalog for governance
• Structured Streaming for real-time
• Auto Loader for incremental file ingestion
• Delta Live Tables for declarative pipelines
• MLflow for ML experiment tracking
• Photon engine for fast SQL
• Databricks Workflows for orchestration
• Repos for Git integration
```

### Common SQL Interview Patterns
```
1. Find Nth highest salary → DENSE_RANK() or LIMIT OFFSET
2. Deduplicate → ROW_NUMBER() PARTITION BY key ORDER BY timestamp DESC, WHERE rn=1
3. Running total → SUM() OVER(ORDER BY date)
4. Year-over-year comparison → LAG(metric, 12) OVER(ORDER BY month)
5. Consecutive days → Date arithmetic + ROW_NUMBER trick
6. Pivoting → CASE WHEN + GROUP BY
7. Top N per group → ROW_NUMBER() PARTITION BY group ORDER BY metric DESC
```

### PySpark Common Patterns
```python
# Read Delta
df = spark.read.format("delta").load(path)

# Incremental load with MERGE
delta_table.alias("t").merge(source.alias("s"), "t.id = s.id") \
    .whenMatchedUpdateAll().whenNotMatchedInsertAll().execute()

# Write with optimization
df.coalesce(10).write.format("delta").mode("overwrite") \
    .option("overwriteSchema", "true").partitionBy("date").save(path)

# Streaming
spark.readStream.format("cloudFiles") \
    .option("cloudFiles.format", "json") \
    .load(path).writeStream.format("delta") \
    .option("checkpointLocation", cp_path) \
    .trigger(availableNow=True).start(target_path)
```

---

## 8.4 Certification Path

To strengthen your profile:

1. **Microsoft Certified: Azure Data Engineer Associate (DP-203)**
   - Covers: Azure data services, batch/stream processing, data security
   - Study: Microsoft Learn free modules
   - Duration: ~4-6 weeks of preparation

2. **Databricks Certified Data Engineer Associate**
   - Covers: Databricks, Delta Lake, ELT, pipelines
   - Study: Databricks Academy (free courses)
   - Duration: ~3-4 weeks of preparation

---

## 8.5 Final Checklist Before the Interview

```
TECHNICAL KNOWLEDGE:
□ Can explain Databricks architecture and Spark internals
□ Can write PySpark transformations and SQL queries fluently
□ Can explain Delta Lake (ACID, time travel, MERGE, OPTIMIZE, VACUUM)
□ Can design Medallion Architecture (Bronze/Silver/Gold)
□ Can explain Structured Streaming concepts
□ Can describe CI/CD pipeline for Databricks
□ Can discuss data governance and security best practices
□ Can optimize slow pipelines (Spark UI, partitioning, broadcast, AQE)

PRACTICAL SKILLS:
□ Written at least 5 PySpark pipeline notebooks
□ Implemented MERGE/UPSERT logic
□ Used Delta Lake features (time travel, schema evolution)
□ Built a streaming pipeline (even a simple one)
□ Set up a Databricks Workflow with multiple tasks
□ Connected Databricks to Azure DevOps Repos

INTERVIEW PREPARATION:
□ Prepared "Tell me about yourself" answer
□ Prepared 3-4 STAR stories (production incident, optimization, collaboration)
□ Practiced SQL window function questions
□ Practiced system design (trade processing pipeline)
□ Researched the company and trading platform domain
□ Prepared thoughtful questions to ask the interviewer
```

---

## 8.6 Questions to Ask the Interviewer

1. "Can you describe the current data architecture and what layer of the Medallion Architecture the team focuses on?"
2. "What does the typical development lifecycle look like — from feature request to production deployment?"
3. "How many client environments does the SaaS platform currently support, and how do you handle multi-tenancy?"
4. "What streaming technologies are currently in use for real-time data processing?"
5. "What's the team's approach to data quality and monitoring?"
6. "What professional development and Azure certifications does the company support?"

---

# 📚 Recommended Learning Resources

| Resource | Type | What You'll Learn |
|----------|------|-------------------|
| [Databricks Academy](https://www.databricks.com/learn) | Free courses | Databricks platform, Delta Lake, DLT |
| [Microsoft Learn — DP-203](https://learn.microsoft.com/en-us/training/paths/data-engineer/) | Free modules | Azure Data Engineering concepts |
| [Spark: The Definitive Guide](https://www.oreilly.com/library/view/spark-the-definitive/9781491912201/) | Book | Deep Spark understanding |
| [Databricks Documentation](https://docs.databricks.com/) | Documentation | Platform reference |
| [Azure DevOps Labs](https://azuredevopslabs.com/) | Hands-on labs | CI/CD pipelines |
| [LeetCode (SQL)](https://leetcode.com/problemset/database/) | Practice | SQL interview questions |
| [Databricks Community Edition](https://community.cloud.databricks.com/) | Free platform | Hands-on practice |

---

**Good luck with your interview! Follow this roadmap systematically, practice hands-on, and you'll be well-prepared to demonstrate your skills as an Azure Databricks Developer. 🚀**
