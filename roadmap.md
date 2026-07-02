# 🚀 Complete Azure Data Engineer Roadmap: Zero to Job-Ready

## Your Learning Philosophy
> **Every concept below is explained like you're explaining to a friend at a coffee shop, then refined so you can confidently explain it in an interview.**

---

# 📅 PHASE 1: THE FOUNDATION (Week 1-2)
## "You can't build a house without understanding bricks"

---

## Module 1: What is Data Engineering? (Day 1)

### Real-Life Example
Imagine a **massive shopping mall** (like Reliance or Amazon warehouse).
- Customers buy products → **raw data is generated**
- Someone needs to **collect** all bills from all counters
- **Organize** them by date, product category, payment method
- **Store** them neatly in filing cabinets
- **Deliver** summarized reports to the mall manager every morning

👉 **That "someone" is a Data Engineer.**

### Quick Theory for Interview

```
"A Data Engineer is responsible for building and maintaining the infrastructure
that collects, stores, transforms, and delivers data so that analysts and
data scientists can make decisions from it.

Think of it as: Data Engineer builds the ROADS,
               Data Analyst drives the CAR,
               Data Scientist builds the GPS."
```

### The Data Engineering Lifecycle

```
┌─────────────┐    ┌─────────────┐    ┌──────────────┐    ┌─────────────┐
│   COLLECT    │───▶│    STORE     │───▶│  TRANSFORM   │───▶│   DELIVER   │
│  (Ingest)    │    │  (Database/  │    │  (Clean/     │    │ (Dashboard/ │
│              │    │   Storage)   │    │   Process)   │    │   Report)   │
└─────────────┘    └─────────────┘    └──────────────┘    └─────────────┘
  Swiggy order       Save in DB        Calculate total     Show to restaurant
  placed             warehouse         delivery time       owner on app
```

---

## Module 2: Understanding Data Basics (Day 1-2)

### 2.1 Types of Data

#### Real-Life Examples

```
📊 STRUCTURED DATA (Organized in rows & columns - like an Excel sheet)
┌──────────┬─────────┬────────┬───────────┐
│ Order_ID │ Customer│ Amount │ Date      │
├──────────┼─────────┼────────┼───────────┤
│ 001      │ Rahul   │ ₹500   │ 2025-01-01│
│ 002      │ Priya   │ ₹1200  │ 2025-01-02│
└──────────┴─────────┴────────┴───────────┘
Example: Bank transactions, Employee records, Exam marks

📝 SEMI-STRUCTURED DATA (Has some organization but flexible)
{
  "customer": "Rahul",
  "orders": [
    {"item": "Pizza", "price": 500},
    {"item": "Coke", "price": 60}
  ]
}
Example: JSON from APIs, XML files, Email headers

🎵 UNSTRUCTURED DATA (No fixed format at all)
- Photos you upload on Instagram
- Voice messages on WhatsApp
- YouTube videos
- PDF documents
```

#### Interview Answer

```
"Structured data fits neatly into tables like SQL databases — think bank 
statements. Semi-structured has some organization but is flexible — like 
JSON from a REST API. Unstructured has no predefined format — like images, 
videos, or free-text documents. As a data engineer, I need to handle all 
three types and often convert unstructured or semi-structured data into 
structured formats for analysis."
```

### 2.2 Data Formats You'll Work With

```
FORMAT     │ WHAT IT IS                    │ REAL-LIFE ANALOGY
───────────┼───────────────────────────────┼──────────────────────────
CSV        │ Comma-separated values        │ Excel export file
JSON       │ JavaScript Object Notation    │ Data from any mobile app API
Parquet    │ Columnar storage format       │ Compressed, fast-read warehouse file
Avro       │ Row-based format with schema  │ Streaming data from Kafka
Delta      │ Parquet + versioning + ACID   │ Parquet with "undo" button
XML        │ Markup language format        │ Old banking/insurance systems
```

#### Why Parquet Matters (Interview Favorite!)

```
CSV (Row-based):                    Parquet (Column-based):
┌────┬───────┬────────┬─────┐      ┌────┬────┬────┬────┐  ← All IDs together
│ ID │ Name  │ Amount │ City│      │ 1  │ 2  │ 3  │ 4  │
├────┼───────┼────────┼─────┤      ├────────────────────┤
│ 1  │ Rahul │ 500    │ Delhi│     │Rahul│Priya│...│    │  ← All Names together
│ 2  │ Priya │ 1200   │ Mumbai│    ├────────────────────┤
│ 3  │ ...   │ ...    │ ...  │     │500 │1200│...│     │  ← All Amounts together
└────┴───────┴────────┴─────┘      └────────────────────┘

If you need SUM(Amount) for 1 billion rows:
- CSV: Must read EVERY column of EVERY row → SLOW
- Parquet: Reads ONLY the Amount column → 100x FASTER
```

#### Interview Answer

```
"Parquet is a columnar storage format. When I need to aggregate a single 
column across billions of rows — say, calculating total revenue — Parquet 
only reads that one column, skipping everything else. This dramatically 
reduces I/O and speeds up queries. It also supports efficient compression 
because similar data types are stored together. That's why it's the 
default format in most modern data lakes."
```

---

## Module 3: Database Fundamentals (Day 2-4)

### 3.1 What is a Database?

#### Real-Life Example

```
Think of a LIBRARY:
📚 Library = Database
📖 Each Shelf/Section = Table  
📄 Each Book's Catalog Card = Row/Record
📝 Book Title, Author, ISBN = Columns/Fields

The LIBRARIAN = Database Management System (DBMS)
- Knows where everything is
- Helps you find books quickly
- Makes sure two people don't check out the same book
```

### 3.2 SQL - The Language of Databases

#### Real-Life Analogy

```
SQL is like talking to the librarian:

YOU: "Show me all books by J.K. Rowling" 
SQL: SELECT * FROM books WHERE author = 'J.K. Rowling';

YOU: "How many science books do we have?"
SQL: SELECT COUNT(*) FROM books WHERE category = 'Science';

YOU: "Add this new book to the collection"
SQL: INSERT INTO books (title, author) VALUES ('New Book', 'New Author');
```

### Essential SQL You MUST Know (with examples)

```sql
-- ═══════════════════════════════════════════════════════
-- 1. CREATE TABLE - Building a new shelf in the library
-- ═══════════════════════════════════════════════════════
CREATE TABLE customers (
    customer_id     INT PRIMARY KEY,        -- Unique library card number
    name            VARCHAR(100),           -- Customer name
    email           VARCHAR(100),           -- Contact email
    city            VARCHAR(50),            -- City they live in
    signup_date     DATE,                   -- When they joined
    total_purchases DECIMAL(10,2)           -- How much they've spent
);

-- Real-life: Zomato creating a table to store all customer info

-- ═══════════════════════════════════════════════════════
-- 2. INSERT - Adding new books to the shelf
-- ═══════════════════════════════════════════════════════
INSERT INTO customers VALUES 
(1, 'Rahul Sharma', 'rahul@email.com', 'Delhi', '2024-01-15', 15000.50),
(2, 'Priya Patel', 'priya@email.com', 'Mumbai', '2024-03-20', 28000.00),
(3, 'Amit Kumar', 'amit@email.com', 'Delhi', '2024-02-10', 5000.75);

-- ═══════════════════════════════════════════════════════
-- 3. SELECT - Finding/Reading data
-- ═══════════════════════════════════════════════════════

-- Get everything
SELECT * FROM customers;

-- Get specific columns (don't read entire book, just title & author)
SELECT name, city, total_purchases FROM customers;

-- Filter with WHERE (only Delhi customers)
SELECT * FROM customers WHERE city = 'Delhi';

-- Multiple conditions
SELECT * FROM customers 
WHERE city = 'Delhi' AND total_purchases > 10000;

-- ═══════════════════════════════════════════════════════
-- 4. AGGREGATIONS - Summarize data (Manager's report)
-- ═══════════════════════════════════════════════════════

-- How many customers per city?
-- Like asking: "How many students in each classroom?"
SELECT city, COUNT(*) as customer_count
FROM customers
GROUP BY city;

-- Result:
-- Delhi  | 2
-- Mumbai | 1

-- Total purchases by city
SELECT city, 
       SUM(total_purchases) as total_revenue,
       AVG(total_purchases) as avg_revenue,
       MAX(total_purchases) as highest_spender
FROM customers
GROUP BY city;

-- HAVING = WHERE but for groups
-- Cities with more than 1 customer
SELECT city, COUNT(*) as customer_count
FROM customers
GROUP BY city
HAVING COUNT(*) > 1;

-- ═══════════════════════════════════════════════════════
-- 5. JOINS - Combining data from multiple tables
-- ═══════════════════════════════════════════════════════

-- Think of it like this:
-- Table 1 (customers): Student info (name, roll number)
-- Table 2 (orders): Exam results (roll number, marks)
-- JOIN: Combine to see "Student Name + Their Marks"

CREATE TABLE orders (
    order_id    INT,
    customer_id INT,          -- This LINKS to customers table
    product     VARCHAR(100),
    amount      DECIMAL(10,2),
    order_date  DATE
);

-- INNER JOIN: Only show customers WHO HAVE orders
-- (Students who appeared for the exam)
SELECT c.name, o.product, o.amount
FROM customers c
INNER JOIN orders o ON c.customer_id = o.customer_id;

-- LEFT JOIN: Show ALL customers, even those with NO orders  
-- (All students, even those absent for exam - marks will be NULL)
SELECT c.name, o.product, o.amount
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id;

-- ═══════════════════════════════════════════════════════
-- 6. SUBQUERIES - Query inside a query
-- ═══════════════════════════════════════════════════════

-- Customers who spent more than average
-- Like: "Students who scored above class average"
SELECT name, total_purchases
FROM customers
WHERE total_purchases > (SELECT AVG(total_purchases) FROM customers);

-- ═══════════════════════════════════════════════════════
-- 7. WINDOW FUNCTIONS (Interview Favorites!)
-- ═══════════════════════════════════════════════════════

-- RANK customers by purchases within each city
-- Like: Rank students by marks within each class section
SELECT 
    name,
    city,
    total_purchases,
    RANK() OVER (PARTITION BY city ORDER BY total_purchases DESC) as city_rank,
    ROW_NUMBER() OVER (ORDER BY total_purchases DESC) as overall_rank,
    SUM(total_purchases) OVER (PARTITION BY city) as city_total,
    LAG(total_purchases) OVER (ORDER BY signup_date) as prev_customer_purchase
FROM customers;

-- PARTITION BY city = "Do this separately for each city"
-- ORDER BY total_purchases DESC = "Highest spender gets rank 1"
-- LAG = "What was the previous row's value?" (useful for comparison)
```

### JOIN Types Visual Guide

```
INNER JOIN (Only matching):        LEFT JOIN (All from left):
   A     B                            A     B
  ┌───┐ ┌───┐                       ┌───┐ ┌───┐
  │   │█│   │                       │███│█│   │
  │   │█│   │                       │███│█│   │
  └───┘ └───┘                       └───┘ └───┘
  Only █ part                       All of A + matching B

RIGHT JOIN (All from right):       FULL OUTER JOIN (Everything):
   A     B                            A     B
  ┌───┐ ┌───┐                       ┌───┐ ┌───┐
  │   │█│███│                       │███│█│███│
  │   │█│███│                       │███│█│███│
  └───┘ └───┘                       └───┘ └───┘
  Matching A + all of B             Everything from both
```

### 3.3 OLTP vs OLAP

```
OLTP (Online Transaction Processing)     OLAP (Online Analytical Processing)
════════════════════════════════════     ════════════════════════════════════
📱 Swiggy app processing your order     📊 Swiggy analyzing "Which restaurant 
                                           had most orders this month?"
                                        
✅ INSERT one order                      ✅ SELECT across millions of orders
✅ UPDATE order status                   ✅ GROUP BY restaurant, city, month
✅ Fast for single record operations     ✅ Fast for scanning large datasets

🏪 Like a CASH REGISTER                 📈 Like a BUSINESS REPORT
   (quick individual transactions)          (summarize all transactions)

Examples:                                Examples:
- MySQL, PostgreSQL, SQL Server          - Azure Synapse, Snowflake, BigQuery
- Your bank's transaction system         - Your bank's monthly statement analysis
```

#### Interview Answer

```
"OLTP systems handle day-to-day transactions — like processing a payment 
or updating an order status. They're optimized for fast reads and writes 
of individual records. OLAP systems are designed for analytical queries 
across large volumes of historical data — like calculating monthly revenue 
trends across all stores. As a data engineer, I typically extract data 
from OLTP sources and load it into OLAP systems for analysis."
```

### 3.4 Normalization vs Denormalization

```
NORMALIZATION (Split tables to avoid repetition):
Like organizing your wardrobe - shirts in one drawer, pants in another

❌ Bad (Denormalized/Repeated):
┌──────────┬──────────┬──────────────┬────────────────┐
│ Order_ID │ Customer │ Customer_Phone│ Product       │
├──────────┼──────────┼──────────────┼────────────────┤
│ 1        │ Rahul    │ 9876543210   │ Pizza         │
│ 2        │ Rahul    │ 9876543210   │ Burger        │  ← Rahul's info repeated!
│ 3        │ Rahul    │ 9876543210   │ Coke          │  ← Again repeated!
└──────────┴──────────┴──────────────┴────────────────┘

✅ Good (Normalized - 3NF):
Customers Table:                Orders Table:
┌─────┬────────┬────────────┐  ┌──────────┬─────────────┬─────────┐
│ C_ID│ Name   │ Phone      │  │ Order_ID │ Customer_ID │ Product │
├─────┼────────┼────────────┤  ├──────────┼─────────────┼─────────┤
│ 1   │ Rahul  │ 9876543210 │  │ 1        │ 1           │ Pizza   │
└─────┴────────┴────────────┘  │ 2        │ 1           │ Burger  │
                                │ 3        │ 1           │ Coke    │
                                └──────────┴─────────────┴─────────┘
Rahul's info stored ONCE! Orders reference by ID.

DENORMALIZATION (Combine for faster reads - used in OLAP/Data Warehouses):
Sometimes you intentionally flatten data so analytical queries don't 
need complex JOINs. Speed of reading > storage efficiency.
```

---

## Module 4: Data Warehouse Concepts (Day 4-5)

### 4.1 What is a Data Warehouse?

#### Real-Life Example

```
Imagine FLIPKART has multiple departments:
🛒 Sales System → "Who bought what?"
🚚 Shipping System → "Where was it delivered?"  
💳 Payment System → "How did they pay?"
📞 Customer Service → "Any complaints?"

Each system has its OWN database. The CEO asks:
"Show me total revenue by city, by payment method, for customers 
who complained, for the last quarter"

❌ You can't easily answer this from 4 separate databases!

✅ Solution: DATA WAREHOUSE
   Pull data from ALL systems → Transform → Store in ONE place
   → Now the CEO can get any cross-system report instantly

   Sales DB ──────┐
   Shipping DB ───┼──→ [DATA WAREHOUSE] ──→ Reports/Dashboards
   Payment DB ────┤                          Power BI
   Support DB ────┘
```

### 4.2 Star Schema vs Snowflake Schema

```
STAR SCHEMA (Most Common - Interview Favorite):
Like a STAR ⭐ - One central FACT table surrounded by DIMENSION tables

                    ┌──────────────┐
                    │  DIM_DATE    │
                    │ date_key     │
                    │ day, month   │
                    │ quarter, year│
                    └──────┬───────┘
                           │
┌─────────────┐    ┌───────┴────────┐    ┌──────────────┐
│ DIM_CUSTOMER│    │  FACT_SALES    │    │ DIM_PRODUCT  │
│ cust_key    │◄───┤ cust_key (FK)  ├───►│ prod_key     │
│ name, city  │    │ prod_key (FK)  │    │ name, category│
│ segment     │    │ date_key (FK)  │    │ brand, price │
└─────────────┘    │ store_key (FK) │    └──────────────┘
                   │ quantity       │
                   │ total_amount   │    ┌──────────────┐
                   │ discount       │    │ DIM_STORE    │
                   └───────┬────────┘    │ store_key    │
                           └────────────►│ location,city│
                                         └──────────────┘

FACT TABLE = What happened (measurements/metrics: amount, quantity)
             Like: "The RECEIPT at a store"
             
DIMENSION TABLE = Context/Description (who, what, when, where)
                  Like: "Details about the customer, product, store"
```

```
SNOWFLAKE SCHEMA:
Same as Star, but dimensions are further normalized (broken down)

Instead of:
DIM_PRODUCT (prod_key, name, category_name, brand_name)

You have:
DIM_PRODUCT (prod_key, name, category_id, brand_id)
DIM_CATEGORY (category_id, category_name)
DIM_BRAND (brand_id, brand_name)

Like a snowflake ❄️ - more branches/tables
```

#### Interview Answer

```
"A star schema has a central fact table containing business measurements 
like sales amount and quantity, surrounded by denormalized dimension tables 
that provide context — who bought it, what product, when, and where. 
I prefer star schema for most analytics use cases because it minimizes 
JOINs and makes queries simpler and faster. Snowflake schema normalizes 
dimensions further, which saves storage but requires more JOINs."
```

### 4.3 Slowly Changing Dimensions (SCD) - Top Interview Topic!

```
PROBLEM: A customer moves from Delhi to Mumbai. How do you handle this?

Example: Rahul was in Delhi, now moved to Mumbai.

╔══════════════════════════════════════════════════════════════╗
║ SCD TYPE 1: OVERWRITE (Just update it, forget the past)     ║
╠══════════════════════════════════════════════════════════════╣

Before: │ 1 │ Rahul │ Delhi  │
After:  │ 1 │ Rahul │ Mumbai │  ← Delhi is GONE forever!

🍕 Like changing your phone number and telling no one your old number.
✅ Simple | ❌ Lose history

╔══════════════════════════════════════════════════════════════╗
║ SCD TYPE 2: ADD NEW ROW (Keep full history) ★MOST COMMON★   ║
╠══════════════════════════════════════════════════════════════╣

│ Key │ ID │ Name  │ City   │ Start_Date │ End_Date   │ Current │
│  1  │ 1  │ Rahul │ Delhi  │ 2023-01-01 │ 2025-06-01 │ N       │
│  2  │ 1  │ Rahul │ Mumbai │ 2025-06-01 │ 9999-12-31 │ Y       │

🍕 Like keeping your old diary AND starting a new one.
✅ Full history preserved | ❌ Table grows larger

╔══════════════════════════════════════════════════════════════╗
║ SCD TYPE 3: ADD NEW COLUMN (Keep limited history)            ║
╠══════════════════════════════════════════════════════════════╣

│ ID │ Name  │ Current_City │ Previous_City │
│ 1  │ Rahul │ Mumbai       │ Delhi         │

🍕 Like writing your new address on the same page.
✅ Quick comparison | ❌ Only stores ONE previous value
```

#### Interview Answer

```
"SCD Type 2 is what I use most. When a dimension attribute changes — say, 
a customer relocates — I don't overwrite the old record. Instead, I insert 
a new row with updated values, mark the old row as inactive with an 
end date, and flag the new row as current. This preserves full historical 
context. For example, if I need to analyze what revenue we earned from 
'Delhi customers' last year, I need to know Rahul was in Delhi at that 
time, even though he's now in Mumbai."
```

---

## Module 5: ETL vs ELT (Day 5)

### The Big Picture

```
ETL (Extract → Transform → Load):
Traditional approach - Transform BEFORE loading

🏭 Real-life analogy: COOKING AT HOME before going to a party
   1. Buy groceries (Extract from source)
   2. Cook at home (Transform - clean, cut, cook)
   3. Bring dish to party (Load into warehouse)

   Source DB → [Transformation Server] → Data Warehouse
   
   Used when: Limited warehouse compute power, 
              data needs heavy cleaning before storing

═══════════════════════════════════════════════════════════

ELT (Extract → Load → Transform):
Modern approach - Load raw data FIRST, transform later

🏭 Real-life analogy: ORDERING RAW INGREDIENTS to a restaurant kitchen
   1. Buy groceries (Extract from source)
   2. Deliver to restaurant (Load into data lake/warehouse)
   3. Chef cooks in restaurant kitchen (Transform using warehouse power)

   Source DB → Data Lake/Warehouse → [Transform inside warehouse]
   
   Used when: Cloud warehouses with massive compute (Synapse, BigQuery)
              You want to keep raw data for future unknown use cases

═══════════════════════════════════════════════════════════

In Azure:
ETL → Azure Data Factory (orchestrates) + Databricks (transforms)
ELT → ADF loads into ADLS → Synapse/Databricks transforms there
```

---

# 📅 PHASE 2: AZURE CLOUD FUNDAMENTALS (Week 2-3)
## "Moving from paper maps to Google Maps"

---

## Module 6: Cloud Computing & Azure Basics (Day 8-9)

### 6.1 What is Cloud Computing?

```
WITHOUT CLOUD (Old Way):
┌─────────────────────────────────┐
│ Your Office Basement            │
│ ┌─────┐ ┌─────┐ ┌─────┐       │
│ │ SRV1│ │ SRV2│ │ SRV3│       │  Buy servers: ₹50 Lakh
│ └─────┘ └─────┘ └─────┘       │  Hire IT team: ₹20 Lakh/year
│ 🔧 Maintenance                 │  Electricity: ₹5 Lakh/year
│ ❄️ AC                          │  Space: ₹10 Lakh/year
│ 🔒 Security                    │  
│ ⚡ Power backup                │  Total: ₹85 Lakh before you even start!
└─────────────────────────────────┘  And if business fails? Money wasted.

WITH CLOUD (New Way):
┌─────────────────────────────────┐
│ Microsoft Azure Data Center     │
│ (They manage everything)        │  Pay only for what you USE
│                                 │  Scale up during Diwali sale
│ Your data: 🔐 Secured          │  Scale down after sale
│ Your servers: 🖥️ Running 24/7   │  No upfront cost
│ Your backups: 💾 Automatic      │  Start with ₹1000/month
└─────────────────────────────────┘

🍕 Analogy: 
  Without cloud = Buying an entire pizza oven for your home
  With cloud = Ordering from Domino's (pay per pizza, they manage the oven)
```

### 6.2 Azure Core Services for Data Engineering

```
┌─────────────────────────────────────────────────────────────────┐
│                    AZURE DATA ENGINEERING STACK                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  📦 STORAGE                                                     │
│  ├── Azure Data Lake Storage Gen2 (ADLS Gen2) → File storage    │
│  └── Azure Blob Storage → Object/raw file storage               │
│                                                                 │
│  🔄 INGESTION & ORCHESTRATION                                   │
│  └── Azure Data Factory (ADF) → ETL/ELT pipeline tool           │
│                                                                 │
│  ⚡ PROCESSING & TRANSFORMATION                                 │
│  ├── Azure Databricks → Spark-based big data processing         │
│  └── Azure Synapse Analytics → Warehousing + Spark + SQL        │
│                                                                 │
│  🗄️ DATABASES                                                   │
│  ├── Azure SQL Database → Managed SQL Server                    │
│  └── Cosmos DB → NoSQL, globally distributed                    │
│                                                                 │
│  📊 VISUALIZATION                                               │
│  └── Power BI → Dashboards and reports                          │
│                                                                 │
│  🔐 GOVERNANCE & SECURITY                                       │
│  ├── Azure Purview (now Microsoft Purview) → Data catalog        │
│  ├── Azure Key Vault → Secret/password management               │
│  └── Azure Active Directory (Entra ID) → Access control          │
│                                                                 │
│  📡 STREAMING                                                   │
│  ├── Azure Event Hubs → Real-time event ingestion               │
│  └── Azure Stream Analytics → Real-time processing              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 6.3 Azure Resource Hierarchy

```
🏢 Tenant (Your Organization - like "Tata Group")
  └── 💳 Subscription (Billing unit - like "department budget")
       └── 📁 Resource Group (Logical container - like "project folder")
            ├── 🗄️ Storage Account
            ├── 🏭 Data Factory
            ├── 📊 Databricks Workspace
            └── 🗃️ SQL Database

Real-life:
Tenant = Your company "TechCorp"
Subscription = "Data Engineering Team Budget" (₹5 Lakh/month)
Resource Group = "Sales-Analytics-Project"
Resources = All Azure services for that project
```

---

## Module 7: Azure Data Lake Storage Gen2 (ADLS Gen2) (Day 9-11)

### What is it?

```
🏠 Real-Life Analogy:
ADLS Gen2 is like a MASSIVE WAREHOUSE with infinite shelves where you 
can store ANYTHING - Excel files, images, videos, JSON, Parquet...

Regular Database = A well-organized filing cabinet (structured only)
Data Lake = A warehouse that stores EVERYTHING in original form

ADLS Gen2 = Blob Storage + Hierarchical Namespace (folders!)
```

### Storage Structure

```
Storage Account: techcorpdata
├── Container: raw-data                    (Like a hard drive partition)
│   ├── sales/
│   │   ├── 2025/
│   │   │   ├── 01/
│   │   │   │   ├── sales_20250101.csv
│   │   │   │   └── sales_20250102.csv
│   │   │   └── 02/
│   │   │       └── sales_20250201.csv
│   │   └── customers.json
│   └── inventory/
│       └── stock_data.parquet
├── Container: processed-data              (Cleaned/transformed data)
│   └── sales/
│       └── 2025/
│           └── monthly_summary.parquet
└── Container: curated-data                (Ready for reporting)
    └── sales_dashboard_data.parquet
```

### The Medallion Architecture (Bronze-Silver-Gold) ⭐ TOP INTERVIEW TOPIC

```
RAW DATA ──→ BRONZE ──→ SILVER ──→ GOLD
            (Raw)     (Cleaned)   (Business-ready)

🥉 BRONZE LAYER (Raw Zone):
   "Dump everything as-is from source"
   - Raw CSV/JSON from APIs, databases
   - No cleaning, no transformation
   - Like: Raw groceries from the market (unwashed vegetables)
   
🥈 SILVER LAYER (Cleaned/Standardized Zone):
   "Clean, deduplicate, standardize"
   - Remove nulls, fix data types, deduplicate
   - Standardize date formats, currency
   - Like: Washed, chopped, ready-to-cook ingredients
   
🥇 GOLD LAYER (Business/Aggregated Zone):
   "Business-ready aggregated data"
   - Aggregated metrics, KPIs
   - Joined with dimensions
   - Like: Final cooked dish, ready to serve

Example Flow:
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐
│ BRONZE           │    │ SILVER            │    │ GOLD             │
│                  │    │                   │    │                  │
│ sales_raw.csv    │──→ │ sales_cleaned     │──→ │ monthly_revenue  │
│ - Has duplicates │    │ - No duplicates   │    │   _by_city.parquet│
│ - NULL values    │    │ - NULLs handled   │    │ - Aggregated     │
│ - Mixed formats  │    │ - Correct types   │    │ - Ready for BI   │
│ - 10M rows       │    │ - 9.5M rows       │    │ - 1000 rows      │
└─────────────────┘    └──────────────────┘    └─────────────────┘
```

#### Interview Answer

```
"I implement the Medallion architecture — Bronze, Silver, Gold — in 
Azure Data Lake Storage Gen2. Bronze stores raw ingested data with no 
transformations, preserving the source of truth. Silver applies 
data quality rules — deduplication, null handling, schema enforcement, 
and type casting. Gold contains business-level aggregates and curated 
datasets optimized for Power BI dashboards or ML models. This layered 
approach provides data lineage, reprocessing capability, and clear 
separation of concerns."
```

### Access Control

```
Two ways to secure data in ADLS Gen2:

1. RBAC (Role-Based Access Control):
   "Give John READ access to the entire storage account"
   - Coarse-grained (whole account/container level)
   - Like: Giving someone a KEY to the entire building

2. ACL (Access Control Lists):
   "Give John READ access to ONLY the /sales/2025/ folder"
   - Fine-grained (folder/file level)
   - Like: Giving someone a KEY to only Room 205

Best Practice: Use RBAC for management, ACL for data access
```

---

## Module 8: Azure Data Factory (ADF) (Day 11-16) ⭐ MOST IMPORTANT

### What is ADF?

```
🍕 Real-Life Analogy:
ADF is like a DELIVERY MANAGER at Domino's:
- Doesn't make pizzas (doesn't transform data itself)
- COORDINATES everything:
  "Get dough from warehouse A" (Extract)
  "Send to kitchen B for cooking" (Transform via Databricks/Dataflow)
  "Deliver to customer C" (Load)
- Schedules deliveries
- Tracks if delivery was successful or failed
- Retries if something goes wrong

ADF is an ORCHESTRATION tool. It moves and coordinates data.
```

### ADF Components Explained

```
┌─────────────────────────────────────────────────────────────────┐
│                    AZURE DATA FACTORY                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  📡 LINKED SERVICE (Connection string / Login credentials)       │
│     "Address book entry - HOW to connect to a system"           │
│     Example: SQL Server connection, ADLS credentials            │
│     Real-life: Saving a restaurant's phone number               │
│                                                                 │
│  📋 DATASET (Pointer to specific data)                          │
│     "Which specific table/file to use"                          │
│     Example: "The 'orders' table in SQL" or "sales.csv in ADLS" │
│     Real-life: "Page 5 of Menu" (specific item in the restaurant)│
│                                                                 │
│  🔄 ACTIVITY (Single task/action)                                │
│     "One specific thing to do"                                  │
│     Example: Copy data, run Databricks notebook, delete file    │
│     Real-life: "Make 1 pizza" or "Deliver 1 order"              │
│                                                                 │
│  🔗 PIPELINE (Sequence of activities)                           │
│     "A workflow of multiple tasks"                              │
│     Example: Copy → Transform → Load → Send Email               │
│     Real-life: Complete order process (take order → cook →       │
│                pack → deliver → confirm)                        │
│                                                                 │
│  ⏰ TRIGGER (When to run the pipeline)                          │
│     "Schedule or event that starts the pipeline"                │
│     - Schedule: "Run every day at 6 AM"                         │
│     - Event: "Run when a new file lands in ADLS"               │
│     - Manual: "Run when I click the button"                     │
│     Real-life: "Kitchen starts cooking at 10 AM daily"          │
│                                                                 │
│  🔗 INTEGRATION RUNTIME (Compute engine)                        │
│     "The actual machine/server that does the work"              │
│     - Azure IR: Cloud-to-cloud (Azure → Azure)                 │
│     - Self-hosted IR: On-premises to cloud                     │
│     - Azure-SSIS IR: Run legacy SSIS packages                  │
│     Real-life: "The delivery bike that physically carries food"  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### ADF Pipeline Example: End-to-End

```
SCENARIO: Every day, load sales data from SQL Server to Data Lake,
          clean it, and make it available for Power BI.

┌──────────┐     ┌──────────┐     ┌──────────┐     ┌──────────┐
│ Lookup   │────▶│  ForEach │────▶│  Copy    │────▶│ Databricks│
│ Activity │     │ Activity │     │ Activity │     │ Notebook  │
│          │     │          │     │          │     │ Activity  │
│"Get list │     │"Loop thru│     │"Copy each│     │"Clean &   │
│ of tables│     │ each     │     │ table to │     │ transform │
│ to copy" │     │ table"   │     │ ADLS"    │     │ the data" │
└──────────┘     └──────────┘     └──────────┘     └──────────┘
                                                        │
                                                        ▼
                                                  ┌──────────┐
                                                  │   If     │
                                                  │ Success  │
                                                  │ → Email ✅│
                                                  │ Failure  │
                                                  │ → Alert ❌│
                                                  └──────────┘
```

### Key ADF Activities You Must Know

```
ACTIVITY              │ WHAT IT DOES                  │ REAL-LIFE
══════════════════════╪═══════════════════════════════╪════════════════════
Copy Data             │ Move data from A to B         │ Photocopy a document
Data Flow             │ Visual data transformation    │ Chef cooking recipe
Lookup                │ Read a value/list from source │ Check a phone number
Get Metadata          │ Get file info (size, exists?) │ Check if letter arrived
If Condition          │ If-else logic                 │ If raining, take umbrella
ForEach               │ Loop through a list           │ Deliver to each address
Execute Pipeline      │ Run another pipeline          │ Call another department
Web Activity          │ Call REST API                 │ Check weather online
Databricks Notebook   │ Run Spark code                │ Send to expert chef
Stored Procedure      │ Run SQL stored procedure      │ Run standard recipe
Wait                  │ Pause for specified time      │ Wait 5 minutes
Set Variable          │ Set a pipeline variable       │ Write down a note
```

### Parameterization & Dynamic Content

```
❌ WITHOUT Parameters (Hardcoded - BAD):
Pipeline 1: Copy sales_jan.csv to ADLS
Pipeline 2: Copy sales_feb.csv to ADLS
Pipeline 3: Copy sales_mar.csv to ADLS
→ 12 pipelines for 12 months! 😱

✅ WITH Parameters (Dynamic - GOOD):
ONE Pipeline with parameter: @pipeline().parameters.month
→ File name: sales_@{pipeline().parameters.month}.csv
→ Reuse same pipeline, just pass different month value!

Common Dynamic Expressions:
@utcnow()                          → Current datetime
@formatDateTime(utcnow(),'yyyy')   → "2025"
@concat('sales_', pipeline().parameters.month, '.csv')  → "sales_jan.csv"
@pipeline().RunId                  → Unique run identifier
@activity('CopyData').output.rowsCopied → Rows copied in previous activity
```

#### Interview Answer for ADF

```
"Azure Data Factory is a cloud-based data integration and orchestration 
service. I use it to build ETL/ELT pipelines that extract data from 
various sources — SQL databases, APIs, flat files — and load it into 
Azure Data Lake or Synapse. 

I heavily use parameterization and dynamic content to make pipelines 
reusable. For example, instead of creating separate pipelines for each 
table, I create one generic pipeline that accepts table name as a 
parameter, loops through a metadata-driven list of tables, and copies 
each one dynamically. 

I also implement proper error handling with If Condition activities, 
retry policies, and alert notifications via Logic Apps or email."
```

---

# 📅 PHASE 3: BIG DATA PROCESSING (Week 3-5)
## "When Excel crashes because your file is too big"

---

## Module 9: Apache Spark & Azure Databricks (Day 17-25) ⭐ CRITICAL

### 9.1 Why Do We Need Spark?

```
PROBLEM:
You have 500 GB of sales data. 
- Excel: ❌ Can handle ~1 million rows (crashes)
- SQL Server: ⚠️ Takes 2 hours to process  
- Python (Pandas): ⚠️ Need 500 GB RAM (impossible on normal laptop)

SOLUTION: Apache Spark
- Distributes the work across MULTIPLE machines
- 500 GB split across 50 machines = each handles only 10 GB
- All work simultaneously = done in minutes!

🍕 Analogy: 
  Cooking for 1000 guests:
  - 1 chef = 3 days (Python/Pandas - one machine)
  - 50 chefs working together = 2 hours (Spark - distributed computing)
  
  Spark is the HEAD CHEF that divides work among 50 chefs.
```

### 9.2 How Spark Works Internally (Simplified)

```
┌─────────────────────────────────────────────────────────┐
│                    SPARK CLUSTER                         │
│                                                         │
│  ┌──────────────────┐                                   │
│  │   DRIVER NODE     │ ← The Manager/Head Chef          │
│  │   (Your code      │   Plans the work                 │
│  │    runs here)     │   Divides tasks                  │
│  └────────┬─────────┘   Collects results                │
│           │                                              │
│     ┌─────┼─────────────┐                               │
│     │     │             │                                │
│  ┌──▼──┐ ┌──▼──┐  ┌──▼──┐                              │
│  │WORKER│ │WORKER│  │WORKER│ ← The Chefs                │
│  │NODE 1│ │NODE 2│  │NODE 3│   Do the actual cooking    │
│  │      │ │      │  │      │                             │
│  │Part 1│ │Part 2│  │Part 3│ ← Each gets a PARTITION    │
│  │of data│ │of data│ │of data│  of the data              │
│  └──────┘ └──────┘  └──────┘                             │
│                                                         │
│  500GB data ÷ 3 workers = ~167GB each (in parallel!)    │
└─────────────────────────────────────────────────────────┘
```

### 9.3 Databricks = Spark Made Easy

```
Apache Spark alone = Bare engine (you need to set up everything)
Azure Databricks = Spark + Easy UI + Notebooks + Collaboration + Auto-scaling

Like:
Spark = A powerful car engine
Databricks = A complete car with engine + dashboard + GPS + comfy seats

Key Databricks Features:
✅ Notebooks (like Jupyter) - write code interactively
✅ Clusters - managed Spark compute (auto-scales up/down)
✅ Unity Catalog - data governance
✅ Delta Lake - reliable data storage
✅ Jobs - schedule notebooks to run automatically
✅ Collaborative - team can work on same notebook
```

### 9.4 PySpark - Python for Spark (What You'll Code Daily)

```python
# ═══════════════════════════════════════════════════════════
# PYSPARK COMPLETE GUIDE WITH EXAMPLES
# ═══════════════════════════════════════════════════════════

from pyspark.sql import SparkSession
from pyspark.sql.functions import *
from pyspark.sql.types import *
from pyspark.sql.window import Window

# ─── 1. CREATE SPARK SESSION (Starting the car engine) ───
spark = SparkSession.builder \
    .appName("SalesAnalysis") \
    .getOrCreate()

# ─── 2. READ DATA ────────────────────────────────────────
# From CSV
df = spark.read \
    .option("header", "true") \
    .option("inferSchema", "true") \
    .csv("/mnt/raw/sales_data.csv")

# From Parquet (most common in real projects)
df = spark.read.parquet("/mnt/bronze/sales/")

# From JSON
df = spark.read.json("/mnt/raw/api_response.json")

# From Delta Lake
df = spark.read.format("delta").load("/mnt/silver/sales/")

# From SQL Database (via JDBC)
df = spark.read \
    .format("jdbc") \
    .option("url", "jdbc:sqlserver://server.database.windows.net") \
    .option("dbtable", "dbo.sales") \
    .option("user", "admin") \
    .option("password", "password") \
    .load()

# ─── 3. EXPLORE DATA (First thing you always do) ─────────

df.show(5)              # Show first 5 rows
df.printSchema()        # Show column names and types
df.count()              # Total row count
df.columns              # List of column names
df.describe().show()    # Statistics (min, max, avg, count)
df.distinct().count()   # Count unique rows

# ─── 4. SELECT & FILTER ─────────────────────────────────

# Select specific columns
df.select("customer_name", "amount", "city").show()

# With alias (rename)
df.select(
    col("customer_name").alias("name"),
    col("amount"),
    (col("amount") * 0.18).alias("gst")
).show()

# Filter (WHERE equivalent)
df.filter(col("city") == "Delhi").show()
df.filter((col("city") == "Delhi") & (col("amount") > 1000)).show()
df.filter(col("city").isin("Delhi", "Mumbai")).show()
df.filter(col("customer_name").like("%Sharma%")).show()
df.filter(col("email").isNotNull()).show()

# ─── 5. ADD/MODIFY COLUMNS ──────────────────────────────

# Add new column
df = df.withColumn("gst_amount", col("amount") * 0.18)
df = df.withColumn("total", col("amount") + col("gst_amount"))

# Type casting
df = df.withColumn("amount", col("amount").cast("double"))
df = df.withColumn("order_date", to_date(col("order_date"), "yyyy-MM-dd"))

# Conditional column (CASE WHEN equivalent)
df = df.withColumn("customer_tier",
    when(col("amount") > 10000, "Premium")
    .when(col("amount") > 5000, "Standard")
    .otherwise("Basic")
)

# Rename column
df = df.withColumnRenamed("customer_name", "name")

# Drop column
df = df.drop("temp_column")

# ─── 6. HANDLE NULLS & DUPLICATES ───────────────────────

# Drop rows with any null
df_clean = df.dropna()

# Drop rows where specific column is null
df_clean = df.dropna(subset=["customer_name", "amount"])

# Fill nulls
df_clean = df.fillna({"city": "Unknown", "amount": 0})

# Remove duplicates
df_clean = df.dropDuplicates()
df_clean = df.dropDuplicates(["customer_id", "order_date"])

# ─── 7. AGGREGATIONS ────────────────────────────────────

# Group By (Like asking: "Total sales per city")
df.groupBy("city") \
  .agg(
      count("*").alias("order_count"),
      sum("amount").alias("total_revenue"),
      avg("amount").alias("avg_order_value"),
      max("amount").alias("highest_order"),
      min("amount").alias("lowest_order"),
      countDistinct("customer_id").alias("unique_customers")
  ) \
  .orderBy(col("total_revenue").desc()) \
  .show()

# ─── 8. JOINS ───────────────────────────────────────────

customers_df = spark.read.parquet("/mnt/silver/customers/")
orders_df = spark.read.parquet("/mnt/silver/orders/")

# Inner Join
result = orders_df.join(
    customers_df, 
    orders_df.customer_id == customers_df.customer_id, 
    "inner"
)

# Left Join
result = orders_df.join(customers_df, "customer_id", "left")

# Handle ambiguous columns after join
result = orders_df.alias("o").join(
    customers_df.alias("c"),
    col("o.customer_id") == col("c.customer_id"),
    "left"
).select("o.*", "c.customer_name", "c.city")

# ─── 9. WINDOW FUNCTIONS (Interview Favorite!) ──────────

windowSpec = Window.partitionBy("city").orderBy(col("amount").desc())

df_ranked = df.withColumn("rank_in_city", rank().over(windowSpec)) \
              .withColumn("dense_rank", dense_rank().over(windowSpec)) \
              .withColumn("row_num", row_number().over(windowSpec))

# Running total
windowSpec2 = Window.partitionBy("city") \
                    .orderBy("order_date") \
                    .rowsBetween(Window.unboundedPreceding, Window.currentRow)

df = df.withColumn("running_total", sum("amount").over(windowSpec2))

# Previous/Next value
df = df.withColumn("prev_amount", lag("amount", 1).over(windowSpec))
df = df.withColumn("next_amount", lead("amount", 1).over(windowSpec))

# ─── 10. WRITE DATA ─────────────────────────────────────

# Write as Parquet
df.write \
  .mode("overwrite") \     # overwrite, append, ignore, errorIfExists
  .partitionBy("year", "month") \  # Physical folder partitioning
  .parquet("/mnt/silver/sales/")

# Write as Delta (PREFERRED in Databricks)
df.write \
  .format("delta") \
  .mode("overwrite") \
  .save("/mnt/silver/sales_delta/")

# Write to SQL Database
df.write \
  .format("jdbc") \
  .option("url", "jdbc:sqlserver://...") \
  .option("dbtable", "dbo.sales_summary") \
  .mode("overwrite") \
  .save()

# Write modes:
# "overwrite" → Delete old data, write new (full refresh)
# "append"    → Add new data to existing (incremental)
# "ignore"    → Do nothing if data exists
# "error"     → Throw error if data exists (default)
```

### 9.5 Spark SQL (Alternative to PySpark DataFrame API)

```python
# Register DataFrame as a temporary view (like creating a temp table)
df.createOrReplaceTempView("sales")

# Now you can use SQL!
result = spark.sql("""
    SELECT 
        city,
        COUNT(*) as order_count,
        SUM(amount) as total_revenue,
        AVG(amount) as avg_order_value,
        RANK() OVER (ORDER BY SUM(amount) DESC) as revenue_rank
    FROM sales
    WHERE year = 2025
    GROUP BY city
    HAVING SUM(amount) > 100000
    ORDER BY total_revenue DESC
""")

result.show()
```

### 9.6 Transformations vs Actions (Interview Concept)

```
TRANSFORMATIONS (Lazy - just a plan, no execution):
Think: "Writing a recipe" - nothing is cooked yet

df2 = df.filter(col("city") == "Delhi")     # Lazy ✍️
df3 = df2.select("name", "amount")          # Lazy ✍️
df4 = df3.withColumn("tax", col("amount") * 0.18)  # Lazy ✍️

Spark says: "Noted. I'll do all this when you actually need results."

ACTIONS (Trigger actual computation):
Think: "Actually cooking the recipe" - fire is ON!

df4.show()      # Action 🔥 → NOW Spark executes all 3 steps
df4.count()     # Action 🔥
df4.collect()   # Action 🔥
df4.write.save  # Action 🔥

WHY LAZY?
Because Spark optimizes the plan before execution!
Like: Instead of going to 3 different shops separately,
Spark plans a route that covers all 3 in one trip.
This optimization is called the CATALYST OPTIMIZER.
```

#### Interview Answer

```
"Spark uses lazy evaluation. When I chain transformations like filter, 
select, and withColumn, Spark doesn't execute them immediately. Instead, 
it builds a logical execution plan called a DAG — Directed Acyclic Graph. 
When I trigger an action like show(), collect(), or write(), the Catalyst 
optimizer analyzes the entire DAG, optimizes it — for example, pushing 
filters down before joins to reduce data early — and then executes it 
across the cluster. This is much more efficient than executing each 
step individually."
```

---

## Module 10: Delta Lake (Day 25-28) ⭐ VERY IMPORTANT

### What is Delta Lake?

```
PROBLEM with regular Parquet files:
❌ No ACID transactions (partial writes can corrupt data)
❌ No updates/deletes (have to overwrite entire file)
❌ No versioning (can't go back to yesterday's data)
❌ No schema enforcement (bad data can sneak in)

DELTA LAKE = Parquet files + Transaction Log
✅ ACID transactions (all or nothing, no corruption)
✅ UPDATE, DELETE, MERGE operations on data lake
✅ Time Travel (query data as it was at any point in time)
✅ Schema enforcement (rejects bad data)
✅ Schema evolution (add new columns safely)

🍕 Analogy:
Parquet = Writing on paper (can't undo, can't erase)
Delta   = Writing on Google Docs (version history, undo, track changes)
```

### Delta Lake Operations

```python
# ─── CREATE DELTA TABLE ────────────────────────────────

# Method 1: Write DataFrame as Delta
df.write.format("delta").save("/mnt/silver/sales_delta/")

# Method 2: SQL
spark.sql("""
    CREATE TABLE IF NOT EXISTS silver.sales (
        order_id INT,
        customer_name STRING,
        amount DOUBLE,
        city STRING,
        order_date DATE
    )
    USING DELTA
    LOCATION '/mnt/silver/sales_delta/'
""")

# ─── READ DELTA TABLE ──────────────────────────────────
df = spark.read.format("delta").load("/mnt/silver/sales_delta/")
# OR
df = spark.table("silver.sales")

# ─── UPDATE (Impossible with plain Parquet!) ────────────
from delta.tables import DeltaTable

deltaTable = DeltaTable.forPath(spark, "/mnt/silver/sales_delta/")

# Update all Delhi orders - add 10% bonus
deltaTable.update(
    condition="city = 'Delhi'",
    set={"amount": "amount * 1.10"}
)

# ─── DELETE ─────────────────────────────────────────────
deltaTable.delete("amount < 100")  # Delete small orders

# ─── MERGE / UPSERT (⭐ Most Important - Interview Favorite!) ──

# SCENARIO: You get daily sales data. Some orders are NEW, 
# some are UPDATES to existing orders. How to handle both?

# New data arrives
new_data = spark.read.parquet("/mnt/bronze/daily_sales/2025-06-01/")

deltaTable.alias("target").merge(
    new_data.alias("source"),
    "target.order_id = source.order_id"  # Match condition
).whenMatchedUpdate(          # If order exists → UPDATE it
    set={
        "amount": "source.amount",
        "city": "source.city"
    }
).whenNotMatchedInsert(       # If order is new → INSERT it
    values={
        "order_id": "source.order_id",
        "customer_name": "source.customer_name",
        "amount": "source.amount",
        "city": "source.city",
        "order_date": "source.order_date"
    }
).execute()

# Think of MERGE like:
# "For each incoming order:
#   - If we already have it → update with latest info
#   - If it's new → add it to our records"
# Like a school updating student records at the start of each year.

# ─── TIME TRAVEL (Query historical data) ────────────────

# See all versions (history)
deltaTable.history().show()

# Query data as it was 2 versions ago
df_old = spark.read.format("delta") \
    .option("versionAsOf", 2) \
    .load("/mnt/silver/sales_delta/")

# Query data as it was at a specific timestamp
df_old = spark.read.format("delta") \
    .option("timestampAsOf", "2025-06-01 00:00:00") \
    .load("/mnt/silver/sales_delta/")

# RESTORE to a previous version (undo!)
deltaTable.restoreToVersion(2)

# ─── SCHEMA ENFORCEMENT & EVOLUTION ─────────────────────

# Schema Enforcement (default): Rejects data that doesn't match schema
# If your Delta table has columns [id, name, amount]
# and you try to write data with [id, name, amount, city]
# → It will REJECT and throw error!

# Schema Evolution: Allow adding new columns
df_with_new_column.write \
    .format("delta") \
    .mode("append") \
    .option("mergeSchema", "true") \  # ← Allow new columns
    .save("/mnt/silver/sales_delta/")

# ─── OPTIMIZE & VACUUM (Maintenance) ────────────────────

# OPTIMIZE: Compact small files into larger ones
spark.sql("OPTIMIZE silver.sales")

# Z-ORDER: Optimize for specific query patterns
spark.sql("OPTIMIZE silver.sales ZORDER BY (city, order_date)")
# Makes queries with WHERE city = 'Delhi' AND order_date = ... FAST

# VACUUM: Delete old files (versions) to save storage
spark.sql("VACUUM silver.sales RETAIN 168 HOURS")  # Keep 7 days
```

#### Interview Answer for Delta Lake

```
"Delta Lake provides ACID transactions on top of a data lake, solving 
the reliability problems of plain Parquet files. The key features I use 
daily are:

1. MERGE (upsert): For incremental loads, I match incoming records 
   against existing data — update if matched, insert if new. This is 
   essential for SCD Type 1 implementations.

2. Time Travel: If a pipeline produces incorrect results, I can query 
   the data as it existed before the bad run, and even restore to a 
   previous version.

3. Schema Enforcement: Prevents corrupt data from entering the Silver 
   or Gold layers by rejecting records that don't match the expected schema.

4. OPTIMIZE with Z-ORDER: I Z-ORDER on frequently filtered columns 
   like date and region, which co-locates related data and reduces 
   the amount of data Spark needs to scan."
```

---

# 📅 PHASE 4: AZURE SYNAPSE ANALYTICS (Week 5-6)

---

## Module 11: Azure Synapse Analytics (Day 29-35)

### What is Synapse?

```
🍕 Real-Life Analogy:
Synapse is like a FOOD COURT in a mall:
- Multiple restaurants (engines) under one roof
- You don't need to go to different places for different cuisines

                    ┌────────────────────────────┐
                    │    AZURE SYNAPSE ANALYTICS  │
                    │    (The Food Court)          │
                    ├────────────────────────────┤
                    │                             │
                    │  🍕 Dedicated SQL Pool       │
                    │     (Full data warehouse)    │
                    │     Like: Main restaurant    │
                    │                             │
                    │  🍔 Serverless SQL Pool      │
                    │     (Query data lake files)  │
                    │     Like: Food truck (pay    │
                    │     only when you eat)       │
                    │                             │
                    │  🍜 Spark Pool               │
                    │     (Big data processing)    │
                    │     Like: Specialty kitchen  │
                    │                             │
                    │  🔄 Pipelines                │
                    │     (Same as ADF!)           │
                    │     Like: Delivery service   │
                    │                             │
                    │  📊 Power BI Integration     │
                    │     Built-in dashboards      │
                    └────────────────────────────┘
```

### Dedicated SQL Pool vs Serverless SQL Pool

```
DEDICATED SQL POOL:
┌─────────────────────────────────────────────┐
│ Pre-provisioned compute (always running)     │
│ Data stored INSIDE the pool                  │
│ Like: Renting a restaurant space full-time  │
│                                             │
│ ✅ Fast, predictable performance            │
│ ✅ Great for heavy, frequent reporting      │
│ ❌ Pay even when not using (expensive!)     │
│                                             │
│ Pricing: DWU (Data Warehouse Units)         │
│ DW100c ~ ₹25,000/month                     │
│ DW1000c ~ ₹2,50,000/month                  │
│                                             │
│ Use when: Frequent dashboards, heavy joins,  │
│          need sub-second response times      │
└─────────────────────────────────────────────┘

SERVERLESS SQL POOL:
┌─────────────────────────────────────────────┐
│ No infrastructure to manage                  │
│ Data stays IN THE LAKE (not imported)        │
│ Like: Food truck (comes only when you order)│
│                                             │
│ ✅ Pay per TB of data scanned only          │
│ ✅ No setup, no maintenance                 │
│ ✅ Great for exploration & ad-hoc queries   │
│ ❌ Slower for complex repeated queries      │
│                                             │
│ Pricing: ~₹400 per TB scanned              │
│                                             │
│ Use when: Data exploration, ad-hoc analysis, │
│          creating views on data lake files   │
└─────────────────────────────────────────────┘
```

### Serverless SQL Pool - Query Data Lake Files Directly

```sql
-- ═══════════════════════════════════════════════════════
-- Query Parquet files directly in ADLS (no loading needed!)
-- ═══════════════════════════════════════════════════════

-- Query a specific file
SELECT * 
FROM OPENROWSET(
    BULK 'https://mydatalake.dfs.core.windows.net/raw/sales/2025/01/*.parquet',
    FORMAT = 'PARQUET'
) AS sales_data;

-- Query with column selection (only reads needed columns!)
SELECT customer_name, city, SUM(amount) as total
FROM OPENROWSET(
    BULK 'https://mydatalake.dfs.core.windows.net/silver/sales/**',
    FORMAT = 'PARQUET'
) AS sales_data
GROUP BY customer_name, city
ORDER BY total DESC;

-- ═══════════════════════════════════════════════════════
-- Create External Table (permanent view on data lake)
-- ═══════════════════════════════════════════════════════

-- First, create required objects
CREATE DATABASE silver_db;
GO
USE silver_db;
GO

CREATE EXTERNAL DATA SOURCE my_adls
WITH (
    LOCATION = 'https://mydatalake.dfs.core.windows.net/silver/'
);

CREATE EXTERNAL FILE FORMAT parquet_format
WITH (FORMAT_TYPE = PARQUET);

-- Now create external table
CREATE EXTERNAL TABLE sales (
    order_id INT,
    customer_name VARCHAR(100),
    amount DECIMAL(10,2),
    city VARCHAR(50),
    order_date DATE
)
WITH (
    LOCATION = 'sales/',
    DATA_SOURCE = my_adls,
    FILE_FORMAT = parquet_format
);

-- Now query like any normal table!
SELECT city, SUM(amount) as revenue
FROM sales
GROUP BY city;

-- ═══════════════════════════════════════════════════════
-- CETAS (Create External Table As Select)
-- Transform & save results back to lake
-- ═══════════════════════════════════════════════════════

CREATE EXTERNAL TABLE gold.monthly_revenue
WITH (
    LOCATION = 'gold/monthly_revenue/',
    DATA_SOURCE = my_adls,
    FILE_FORMAT = parquet_format
)
AS
SELECT 
    YEAR(order_date) as year,
    MONTH(order_date) as month,
    city,
    COUNT(*) as order_count,
    SUM(amount) as total_revenue
FROM sales
GROUP BY YEAR(order_date), MONTH(order_date), city;
```

### Dedicated SQL Pool - Key Concepts

```sql
-- ═══════════════════════════════════════════════════════
-- TABLE DISTRIBUTION (How data is spread across nodes)
-- Interview Favorite!
-- ═══════════════════════════════════════════════════════

-- 1. HASH Distribution
-- Rows are distributed based on hash of a column
-- Same column value → same node
-- Best for: Large fact tables, JOIN columns
CREATE TABLE fact_sales (
    order_id INT,
    customer_id INT,
    amount DECIMAL(10,2)
)
WITH (
    DISTRIBUTION = HASH(customer_id),  -- Distribute by customer_id
    CLUSTERED COLUMNSTORE INDEX        -- Compressed columnar storage
);

-- 2. ROUND ROBIN Distribution (Default)
-- Rows distributed evenly, randomly
-- Best for: Staging/temp tables, no clear distribution key
CREATE TABLE stg_sales (
    order_id INT,
    customer_id INT,
    amount DECIMAL(10,2)
)
WITH (
    DISTRIBUTION = ROUND_ROBIN
);

-- 3. REPLICATED
-- Full copy on EVERY node
-- Best for: Small dimension tables (< 2GB)
CREATE TABLE dim_city (
    city_id INT,
    city_name VARCHAR(50),
    state VARCHAR(50)
)
WITH (
    DISTRIBUTION = REPLICATE  -- Copy to all nodes
);

-- ═══════════════════════════════════════════════════════
-- WHY DISTRIBUTION MATTERS
-- ═══════════════════════════════════════════════════════

-- If fact_sales is distributed by customer_id
-- and dim_customer is REPLICATED:
-- JOIN happens LOCALLY on each node → FAST! (no data movement)

-- If fact_sales is distributed by customer_id
-- but you JOIN on order_date:
-- Data must SHUFFLE across nodes → SLOW! (data movement)

-- GOLDEN RULE: Distribute fact tables by the column you JOIN most.
```

```
VISUAL: Distribution Types

HASH (customer_id):           ROUND_ROBIN:              REPLICATE:
Node1: Cust 1,4,7            Node1: Row 1,4,7          Node1: ALL rows
Node2: Cust 2,5,8            Node2: Row 2,5,8          Node2: ALL rows  
Node3: Cust 3,6,9            Node3: Row 3,6,9          Node3: ALL rows

Same customer always          Even spread, but           Small tables,
on same node.                 random placement.          full copy everywhere.
JOIN-friendly!                Good for loading.          JOIN-friendly for
                                                         dimension tables!
```

#### Interview Answer

```
"In Synapse Dedicated SQL Pool, I choose distribution strategy based on 
query patterns. For large fact tables, I use HASH distribution on the 
column most frequently used in JOINs — typically a customer or date key. 
For small dimension tables under 2GB, I use REPLICATE so every compute 
node has a full copy, eliminating data movement during JOINs. For staging 
tables where there's no clear distribution key, I use ROUND_ROBIN for 
even data spread during loads. The goal is to minimize data movement 
across nodes, which is the biggest performance killer in distributed 
query processing."
```

---

# 📅 PHASE 5: STREAMING & REAL-TIME (Week 6)

---

## Module 12: Real-Time Data Processing (Day 36-40)

### 12.1 Batch vs Streaming

```
BATCH PROCESSING:
📦 Like postal mail:
- Collect all letters during the day
- Sort them all at night
- Deliver next morning
- Process data in chunks at scheduled intervals

Example: "Every night at 2 AM, process all of yesterday's sales data"
Tools: ADF, Databricks batch jobs, Synapse pipelines

═══════════════════════════════════════════════════════

STREAMING PROCESSING:
📱 Like WhatsApp messages:
- Message arrives → immediately processed → instantly delivered
- Process data as it arrives, continuously

Example: "Alert me the MOMENT a transaction over ₹1 Lakh happens"
         "Show live dashboard of orders being placed RIGHT NOW"
Tools: Event Hubs, Kafka, Spark Structured Streaming, Stream Analytics

═══════════════════════════════════════════════════════

MICRO-BATCH (Middle ground):
Process small batches every few seconds
Like: WhatsApp sending typing indicators every 2 seconds
Spark Structured Streaming uses this approach by default
```

### 12.2 Azure Event Hubs

```
🍕 Real-Life Analogy:
Event Hub is like a HIGHWAY TOLL BOOTH with multiple lanes:
- Millions of cars (events) pass through
- Multiple lanes (partitions) for parallel processing
- Cars are processed in order within their lane

┌──────────────┐     ┌───────────────────┐     ┌──────────────┐
│ IoT Sensors  │────▶│                   │────▶│ Spark        │
│ Mobile Apps  │────▶│   EVENT HUB       │────▶│ Stream       │
│ Web Clicks   │────▶│ (The Highway)     │────▶│ Analytics    │
│ Transactions │────▶│                   │────▶│ Azure Funcs  │
└──────────────┘     │ Partition 0 ────  │     └──────────────┘
                     │ Partition 1 ────  │
                     │ Partition 2 ────  │
                     │ Partition 3 ────  │
                     └───────────────────┘

Key Concepts:
- Event: Single data record (like one car passing)
- Partition: Ordered channel (like one lane of highway)
- Consumer Group: Different readers of same data
  (Like: Police checkpoint AND toll booth reading same cars)
- Throughput Units: Capacity (1 TU = 1 MB/s in, 2 MB/s out)
```

### 12.3 Spark Structured Streaming in Databricks

```python
# ═══════════════════════════════════════════════════════
# REAL-TIME SALES DASHBOARD EXAMPLE
# Process orders as they arrive and update metrics
# ═══════════════════════════════════════════════════════

# --- Read stream from Event Hub ---
raw_stream = spark.readStream \
    .format("eventhubs") \
    .options(**ehConf) \
    .load()

# Event Hub gives us binary data, decode it
from pyspark.sql.functions import from_json, col

schema = StructType([
    StructField("order_id", StringType()),
    StructField("customer_name", StringType()),
    StructField("amount", DoubleType()),
    StructField("city", StringType()),
    StructField("timestamp", TimestampType())
])

orders_stream = raw_stream \
    .select(from_json(col("body").cast("string"), schema).alias("data")) \
    .select("data.*")

# --- Real-time aggregation ---
# City-wise revenue updated every 1 minute using a 5-minute window
city_revenue = orders_stream \
    .withWatermark("timestamp", "10 minutes") \
    .groupBy(
        window("timestamp", "5 minutes"),  # 5-min tumbling window
        "city"
    ) \
    .agg(
        count("*").alias("order_count"),
        sum("amount").alias("total_revenue")
    )

# --- Write stream to Delta Table ---
city_revenue.writeStream \
    .format("delta") \
    .outputMode("append") \
    .option("checkpointLocation", "/mnt/checkpoints/city_revenue/") \
    .start("/mnt/gold/realtime_city_revenue/")

# --- Or write to console for testing ---
city_revenue.writeStream \
    .outputMode("complete") \
    .format("console") \
    .start()
```

### Streaming Concepts You Must Know

```
OUTPUT MODES:
┌─────────────┬───────────────────────────────────────────┐
│ Mode        │ What it does                              │
├─────────────┼───────────────────────────────────────────┤
│ Append      │ Only NEW rows since last trigger          │
│             │ Like: Only show new messages               │
├─────────────┼───────────────────────────────────────────┤
│ Complete    │ ALL results, recalculated every time      │
│             │ Like: Show complete updated leaderboard    │
├─────────────┼───────────────────────────────────────────┤
│ Update      │ Only CHANGED rows since last trigger      │
│             │ Like: Only show scores that changed        │
└─────────────┴───────────────────────────────────────────┘

WATERMARK:
"How late can data arrive and still be processed?"
.withWatermark("timestamp", "10 minutes")
= "If an event arrives more than 10 minutes late, drop it"
Like: "Restaurant kitchen closes at 11 PM. Orders after 11 PM rejected."

CHECKPOINT:
"Where to resume if stream crashes and restarts"
Like: A bookmark in a book. If you stop reading, start from bookmark.
```

---

# 📅 PHASE 6: SECURITY, GOVERNANCE & OPTIMIZATION (Week 6-7)

---

## Module 13: Security & Governance (Day 41-44)

### 13.1 Authentication & Authorization

```
AUTHENTICATION = "Who are you?" (Proving your identity)
🔑 Like: Showing your Aadhaar card at a bank

AUTHORIZATION = "What can you do?" (What permissions you have)
🎫 Like: Having a movie ticket but for SCREEN 3 only

Azure Tools:
┌──────────────────────────────────┐
│ Azure Active Directory (Entra ID)│ → Identity management
│ Service Principals               │ → App identities (non-human)
│ Managed Identity                 │ → Azure resource auto-identity
│ RBAC                            │ → Role-based permissions
│ ACLs                            │ → File-level permissions in ADLS
│ Azure Key Vault                 │ → Secure storage for secrets
└──────────────────────────────────┘
```

### 13.2 Key Vault (Secret Management)

```
PROBLEM: Your pipeline needs a database password.
❌ BAD: Hardcode "Password123" in your code
❌ BAD: Store password in a config file
✅ GOOD: Store in Azure Key Vault, reference securely

Key Vault is like a BANK LOCKER:
- You store your valuables (passwords, connection strings, keys)
- Only authorized people can open it
- Everything is encrypted
- Access is logged (audit trail)

In ADF:
1. Store password in Key Vault
2. Create Linked Service to Key Vault
3. In your SQL Linked Service, reference:
   @Microsoft.KeyVault(SecretUri=https://myvault.vault.azure.net/secrets/sqlPassword)
```

### 13.3 Data Governance with Microsoft Purview

```
🍕 Real-Life Analogy:
Purview is like a LIBRARIAN + CATALOG for your entire data estate:
- Knows what data exists across ALL your systems
- Where it came from (lineage)
- Who owns it
- Is it sensitive? (classification)
- How it flows from source to destination

┌─────────────────────────────────────────────┐
│           MICROSOFT PURVIEW                  │
├─────────────────────────────────────────────┤
│                                             │
│  📖 Data Catalog                            │
│     "What data do we have? Where is it?"    │
│     Searchable catalog of all data assets    │
│                                             │
│  🔍 Data Classification                     │
│     "Is this column a credit card number?"  │
│     Auto-detects PII, financial data        │
│                                             │
│  🗺️ Data Lineage                            │
│     "Where did this data come from?"        │
│     Visual flow: Source → ADF → Lake → Report│
│                                             │
│  📊 Data Quality                            │
│     "Is this data reliable?"                │
│                                             │
└─────────────────────────────────────────────┘
```

### 13.4 Row-Level Security & Column-Level Security

```sql
-- ROW-LEVEL SECURITY (RLS):
-- "Rahul (Mumbai manager) should only see Mumbai data"
-- "Priya (Delhi manager) should only see Delhi data"

-- Like: Each teacher can only see their OWN class students' marks

CREATE FUNCTION dbo.fn_SecurityPredicate(@city AS VARCHAR(50))
RETURNS TABLE
WITH SCHEMABINDING
AS
RETURN SELECT 1 AS result
WHERE @city = USER_NAME()  -- User sees only their city's data

-- COLUMN-LEVEL SECURITY:
-- "Finance team can see salary column, HR team cannot"

GRANT SELECT ON Employees(name, department, join_date) TO HR_Role;
-- HR_Role cannot see 'salary' column!

-- DATA MASKING:
-- "Show last 4 digits of phone number: ****6789"
ALTER TABLE Customers
ALTER COLUMN phone ADD MASKED WITH (FUNCTION = 'partial(0,"****",4)');
```

---

## Module 14: Performance Optimization (Day 44-47)

### 14.1 Spark Optimization

```
TOP PERFORMANCE KILLERS & SOLUTIONS:

1. DATA SKEW 🎯
   Problem: One partition has 90% of data, others have 10%
   Like: 1 cashier has 100 customers, 9 cashiers have 1 each
   
   Detection:
   df.groupBy("city").count().show()
   # If Delhi has 10M rows and others have 100K → SKEWED!
   
   Solutions:
   # Salting: Add random prefix to skewed key
   df = df.withColumn("salted_key", 
       concat(col("city"), lit("_"), (rand() * 10).cast("int")))
   
   # Broadcast join for small tables
   from pyspark.sql.functions import broadcast
   result = big_df.join(broadcast(small_df), "key")
   
   # Repartition
   df = df.repartition(200, "city")

2. TOO MANY SMALL FILES 📁
   Problem: 10,000 files of 1KB each (instead of 10 files of 1MB)
   Like: Sending 10,000 individual WhatsApp messages instead of 1 document
   
   Solution:
   # Coalesce (reduce partitions without shuffle)
   df.coalesce(10).write.parquet("/output/")
   
   # Repartition (redistribute with shuffle)
   df.repartition(10).write.parquet("/output/")
   
   # Delta OPTIMIZE
   spark.sql("OPTIMIZE delta_table")

3. UNNECESSARY DATA SCANNING 🔍
   Problem: Reading all data when you need only one month
   
   Solution:
   # Partition your data by date
   df.write.partitionBy("year", "month").parquet("/output/")
   
   # Now querying one month only reads that folder!
   spark.read.parquet("/output/year=2025/month=01/")
   
   # Z-ORDER in Delta
   spark.sql("OPTIMIZE table ZORDER BY (city)")

4. NOT CACHING REUSED DATA 💾
   Problem: Same DataFrame computed multiple times
   
   Solution:
   df.cache()  # Store in memory
   df.count()  # Materialize cache
   # Now all subsequent operations on df use cached version
   df.unpersist()  # Free memory when done

5. SHUFFLES (Data Movement) 🔄
   Problem: GroupBy, Join, Distinct cause data to move between nodes
   
   Solution:
   - Use broadcast joins for small tables
   - Pre-partition data by join key
   - Use map-side aggregations when possible
```

### 14.2 Partitioning Strategy

```
PHYSICAL PARTITIONING (How data is stored on disk):

WITHOUT Partitioning:
/sales/
  └── huge_file.parquet (500 GB - must scan everything!)

WITH Partitioning:
/sales/
  ├── year=2024/
  │   ├── month=01/ (5 GB)
  │   ├── month=02/ (5 GB)
  │   └── ...
  └── year=2025/
      ├── month=01/ (5 GB)
      └── month=02/ (5 GB)

Query: WHERE year = 2025 AND month = 01
Without partitioning: Scan 500 GB ❌
With partitioning: Scan only 5 GB ✅ (100x faster!)

PARTITION BY rules:
✅ Low cardinality columns (year, month, country, status)
❌ High cardinality columns (customer_id, email) → too many tiny files!

🍕 Analogy:
Partitioning = Organizing your closet by season
Without: Search through ALL clothes to find winter jacket
With: Go directly to "Winter" section → find jacket instantly
```

---

# 📅 PHASE 7: COMPLETE PROJECT + INTERVIEW PREP (Week 7-8)

---

## Module 15: End-to-End Project (Day 48-52)

### Project: E-Commerce Sales Analytics Platform

```
SCENARIO:
An e-commerce company (like Flipkart) needs a data platform that:
1. Ingests data from multiple sources (SQL DB, APIs, flat files)
2. Stores in a data lake with proper layers
3. Transforms and cleans the data
4. Creates business-ready datasets
5. Powers real-time and batch dashboards

ARCHITECTURE:
┌───────────────┐    ┌──────────┐    ┌──────────────────────────────┐
│ SOURCES        │    │   ADF    │    │    ADLS Gen2 (Data Lake)     │
│                │    │(Pipeline)│    │                              │
│ SQL Server  ───┼───▶│ Copy     │──▶ │ 🥉 Bronze: raw/sales/*.csv  │
│ REST API    ───┼───▶│ Copy     │──▶ │ 🥉 Bronze: raw/products/*.json│
│ CSV files   ───┼───▶│ Copy     │──▶ │ 🥉 Bronze: raw/customers/*.csv│
└───────────────┘    └──────┬───┘    └──────────┬───────────────────┘
                            │                    │
                            │         ┌──────────▼───────────────────┐
                            │         │   DATABRICKS                 │
                            │         │                              │
                            ├────────▶│ Notebook 1: Bronze → Silver  │
                            │         │ - Schema enforcement         │
                            │         │ - Deduplication              │
                            │         │ - Null handling              │
                            │         │ - Type casting               │
                            │         │                              │
                            │         │ Notebook 2: Silver → Gold    │
                            │         │ - Business aggregations      │
                            │         │ - Star schema tables         │
                            │         │ - KPI calculations           │
                            │         └──────────┬───────────────────┘
                            │                    │
                            │         ┌──────────▼───────────────────┐
                            │         │    GOLD LAYER (Delta Tables) │
                            │         │ - fact_sales                 │
                            │         │ - dim_customer               │
                            │         │ - dim_product                │
                            │         │ - agg_daily_revenue          │
                            │         └──────────┬───────────────────┘
                            │                    │
                            │         ┌──────────▼───────────────────┐
                            │         │ SYNAPSE (Serverless SQL Pool)│
                            │         │ Views on top of Gold Delta   │
                            │         │ for Power BI DirectQuery     │
                            │         └──────────┬───────────────────┘
                            │                    │
                            │         ┌──────────▼───────────────────┐
                            │         │       POWER BI               │
                            │         │ Sales Dashboard              │
                            │         │ Customer Analytics            │
                            │         │ Revenue Trends               │
                            │         └──────────────────────────────┘
```

### Project Code Walkthrough

```python
# ═══════════════════════════════════════════════════════
# NOTEBOOK 1: BRONZE → SILVER (Data Cleansing)
# ═══════════════════════════════════════════════════════

# --- Read Bronze Layer ---
sales_raw = spark.read \
    .option("header", "true") \
    .option("inferSchema", "true") \
    .csv("/mnt/bronze/raw/sales/")

print(f"Raw record count: {sales_raw.count()}")
sales_raw.printSchema()

# --- Data Quality Checks ---
# Check for nulls
from pyspark.sql.functions import *

null_report = sales_raw.select(
    [count(when(col(c).isNull(), c)).alias(c) for c in sales_raw.columns]
)
null_report.show()

# Check for duplicates
total = sales_raw.count()
distinct = sales_raw.dropDuplicates(["order_id"]).count()
print(f"Duplicates: {total - distinct}")

# --- SILVER Transformations ---
sales_silver = sales_raw \
    .dropDuplicates(["order_id"]) \
    .filter(col("order_id").isNotNull()) \
    .filter(col("amount") > 0) \
    .withColumn("amount", col("amount").cast("double")) \
    .withColumn("order_date", to_date(col("order_date"), "yyyy-MM-dd")) \
    .withColumn("year", year(col("order_date"))) \
    .withColumn("month", month(col("order_date"))) \
    .withColumn("customer_name", initcap(trim(col("customer_name")))) \
    .withColumn("city", upper(trim(col("city")))) \
    .withColumn("ingestion_timestamp", current_timestamp()) \
    .fillna({"discount": 0, "shipping_fee": 0})

print(f"Silver record count: {sales_silver.count()}")

# --- Write to Silver (Delta) ---
sales_silver.write \
    .format("delta") \
    .mode("overwrite") \
    .partitionBy("year", "month") \
    .save("/mnt/silver/sales/")

# ═══════════════════════════════════════════════════════
# NOTEBOOK 2: SILVER → GOLD (Business Aggregations)
# ═══════════════════════════════════════════════════════

# --- Read Silver ---
sales = spark.read.format("delta").load("/mnt/silver/sales/")
customers = spark.read.format("delta").load("/mnt/silver/customers/")
products = spark.read.format("delta").load("/mnt/silver/products/")

# --- Create Star Schema ---

# FACT TABLE: fact_sales
fact_sales = sales.select(
    col("order_id"),
    col("customer_id"),
    col("product_id"),
    col("order_date"),
    col("quantity"),
    col("amount"),
    col("discount"),
    col("amount") - col("discount") + col("shipping_fee")).alias("net_amount")
)

# DIMENSION: dim_customer
dim_customer = customers.select(
    col("customer_id"),
    col("customer_name"),
    col("email"),
    col("city"),
    col("state"),
    col("segment")
).dropDuplicates(["customer_id"])

# DIMENSION: dim_product  
dim_product = products.select(
    col("product_id"),
    col("product_name"),
    col("category"),
    col("sub_category"),
    col("brand"),
    col("unit_price")
).dropDuplicates(["product_id"])

# GOLD AGGREGATE: Daily Revenue by City
daily_revenue = fact_sales \
    .join(dim_customer, "customer_id") \
    .groupBy("order_date", "city", "state") \
    .agg(
        count("order_id").alias("total_orders"),
        sum("net_amount").alias("total_revenue"),
        avg("net_amount").alias("avg_order_value"),
        countDistinct("customer_id").alias("unique_customers")
    )

# GOLD AGGREGATE: Customer Lifetime Value
customer_ltv = fact_sales \
    .groupBy("customer_id") \
    .agg(
        count("order_id").alias("total_orders"),
        sum("net_amount").alias("lifetime_value"),
        min("order_date").alias("first_order"),
        max("order_date").alias("last_order"),
        datediff(max("order_date"), min("order_date")).alias("customer_age_days")
    ) \
    .withColumn("ltv_tier",
        when(col("lifetime_value") > 50000, "Platinum")
        .when(col("lifetime_value") > 20000, "Gold")
        .when(col("lifetime_value") > 5000, "Silver")
        .otherwise("Bronze")
    )

# --- Write Gold Tables ---
fact_sales.write.format("delta").mode("overwrite") \
    .save("/mnt/gold/fact_sales/")
dim_customer.write.format("delta").mode("overwrite") \
    .save("/mnt/gold/dim_customer/")
dim_product.write.format("delta").mode("overwrite") \
    .save("/mnt/gold/dim_product/")
daily_revenue.write.format("delta").mode("overwrite") \
    .save("/mnt/gold/agg_daily_revenue/")
customer_ltv.write.format("delta").mode("overwrite") \
    .save("/mnt/gold/agg_customer_ltv/")

print("✅ Gold layer created successfully!")
```

### ADF Pipeline Configuration for this Project

```json
// Pipeline: MasterPipeline_DailyLoad
{
  "activities": [
    {
      "name": "Get_Table_List",
      "type": "Lookup",
      "description": "Get list of tables to copy from config"
    },
    {
      "name": "ForEach_Table",
      "type": "ForEach",
      "items": "@activity('Get_Table_List').output.value",
      "activities": [
        {
          "name": "Copy_To_Bronze",
          "type": "Copy",
          "source": {
            "type": "SqlServerSource",
            "query": "SELECT * FROM @{item().schema}.@{item().table_name} WHERE modified_date >= @{pipeline().parameters.watermark}"
          },
          "sink": {
            "type": "ParquetSink",
            "path": "raw/@{item().table_name}/@{formatDateTime(utcnow(),'yyyy/MM/dd')}/"
          }
        }
      ]
    },
    {
      "name": "Run_Bronze_to_Silver",
      "type": "DatabricksNotebook",
      "notebook_path": "/Pipelines/Bronze_to_Silver",
      "dependsOn": ["ForEach_Table"]
    },
    {
      "name": "Run_Silver_to_Gold",
      "type": "DatabricksNotebook",
      "notebook_path": "/Pipelines/Silver_to_Gold",
      "dependsOn": ["Run_Bronze_to_Silver"]
    },
    {
      "name": "Send_Success_Email",
      "type": "WebActivity",
      "dependsOn": ["Run_Silver_to_Gold"],
      "dependencyConditions": ["Succeeded"]
    },
    {
      "name": "Send_Failure_Alert",
      "type": "WebActivity",
      "dependsOn": ["Run_Silver_to_Gold"],
      "dependencyConditions": ["Failed"]
    }
  ],
  "trigger": {
    "type": "ScheduleTrigger",
    "recurrence": {
      "frequency": "Day",
      "interval": 1,
      "startTime": "2025-01-01T02:00:00Z"
    }
  }
}
```

---

## Module 16: Interview Preparation (Day 53-60)

### 16.1 Top 30 Interview Questions with Answers

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
CATEGORY 1: CONCEPTUAL QUESTIONS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

**Q1: What is the difference between a Data Lake and a Data Warehouse?**

```
"A Data Lake stores raw, unprocessed data in its native format — 
structured, semi-structured, and unstructured — using cheap storage 
like ADLS Gen2. It's schema-on-read, meaning structure is applied 
when you read the data. Think of it as a massive lake where you dump 
everything.

A Data Warehouse stores processed, structured data optimized for 
analytical queries. It's schema-on-write — data must conform to a 
schema before being loaded. Think of it as a well-organized library.

In my projects, I use ADLS Gen2 as the data lake for Bronze and Silver 
layers, and either Delta tables in the Gold layer or Synapse Dedicated 
SQL Pool as the data warehouse for reporting."
```

**Q2: Explain the Medallion Architecture you've implemented.**

```
"I implement a three-layer Medallion architecture:

Bronze: Raw data ingested as-is from sources. No transformations. 
Serves as the immutable audit layer. I use ADF Copy Activity to land 
data here in Parquet format, partitioned by ingestion date.

Silver: Cleaned and standardized data. I use Databricks notebooks 
to deduplicate, handle nulls, enforce data types, and apply business 
validation rules. Written as Delta tables for ACID reliability.

Gold: Business-level aggregates and curated datasets. Star schema 
fact and dimension tables, pre-calculated KPIs. Optimized for Power BI 
consumption. This is where I implement the star schema with fact_sales, 
dim_customer, dim_product, and dim_date tables."
```

**Q3: How do you handle incremental data loading?**

```
"I use two main approaches depending on the source:

1. Watermark-based: I track the last loaded timestamp or ID in a 
   control table. Each pipeline run queries only records WHERE 
   modified_date > last_watermark. In ADF, I use a Lookup activity 
   to get the watermark, pass it as a parameter to the Copy activity, 
   and update the watermark after successful load.

2. Change Data Capture (CDC): For real-time or near-real-time needs, 
   I enable CDC on the source SQL database. This captures INSERT, 
   UPDATE, DELETE operations. I then use these change records to 
   perform Delta MERGE (upsert) operations in the Silver layer — 
   matching on the primary key, updating if matched, inserting if 
   not matched."
```

**Q4: What is data skew and how do you handle it in Spark?**

```
"Data skew occurs when data is unevenly distributed across partitions. 
For example, if I'm joining on city and 80% of records are from Delhi, 
one executor gets overwhelmed while others sit idle.

I handle it by:
1. Salting: Adding a random suffix to the skewed key to distribute 
   data more evenly, then aggregating back after processing.
2. Broadcast join: If one side of the join is small (< 500MB), I 
   broadcast it to all nodes, eliminating the shuffle entirely.
3. Adaptive Query Execution (AQE): In Spark 3.x, I enable 
   spark.sql.adaptive.enabled=true, which automatically handles 
   skewed partitions at runtime."
```

**Q5: Explain the difference between Synapse Dedicated Pool and Serverless Pool.**

```
"Dedicated SQL Pool is a pre-provisioned compute resource measured 
in DWUs. Data is stored within the pool. It's ideal for predictable, 
heavy workloads — like powering daily dashboards that thousands of 
users hit. But you pay even when it's idle.

Serverless SQL Pool has no infrastructure to provision. It queries 
data directly in the data lake using OPENROWSET or external tables. 
You pay only per TB scanned. It's perfect for ad-hoc exploration, 
data discovery, and creating logical views on top of lake data 
for Power BI.

In my architecture, I typically use Serverless to create SQL views 
on Gold Delta tables, which Power BI then consumes via DirectQuery."
```

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
CATEGORY 2: SCENARIO-BASED QUESTIONS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

**Q6: Your daily pipeline failed at 3 AM. How do you troubleshoot?**

```
"First, I check ADF Monitor to identify which activity failed and 
read the error message and error code. Common issues I check:

1. Source issues: Database down, API rate limited, credentials expired
   → Check linked service connectivity, Key Vault secret expiry

2. Data issues: Schema changed at source, unexpected nulls, 
   data type mismatches
   → Check Data Flow debug output, add schema validation steps

3. Compute issues: Databricks cluster failed to start, out of memory
   → Check cluster logs in Databricks, consider increasing node 
   count or switching to memory-optimized instances

4. Sink issues: Storage permission denied, disk full
   → Verify RBAC/ACL permissions, check storage capacity

I also implement proactive monitoring — alerts via Azure Monitor, 
retry policies on activities (3 retries with 10-min intervals), 
and idempotent pipeline design so re-running doesn't create duplicates."
```

**Q7: You need to process 10 TB of data daily. How would you design the architecture?**

```
"For 10 TB daily:

Ingestion: ADF with parallel copy activities, each handling a subset. 
I'd use partitioned copy (by date range or ID range) for large tables 
and set DIU (Data Integration Units) to 256 for maximum throughput.

Processing: Databricks with auto-scaling clusters (8 to 64 workers). 
I'd partition data by date, use Delta Lake for incremental MERGE 
operations rather than full reloads, and apply Z-ORDER on frequently 
filtered columns.

Storage: ADLS Gen2 with lifecycle management — hot tier for recent 
data, cool tier for 30-90 day old data, archive for older data.

Optimization: Partition pruning (only read today's partition), 
Parquet/Delta format for columnar efficiency, broadcast joins for 
dimension tables, and OPTIMIZE with ZORDER on the Gold Delta tables.

Monitoring: Azure Monitor alerts for pipeline duration, 
auto-scaling policies based on queue depth."
```

**Q8: How do you implement SCD Type 2 using Delta Lake?**

```python
# "I use Delta MERGE with a flag-based approach"

from delta.tables import DeltaTable

# Read incoming (new/changed) customer data
updates = spark.read.parquet("/mnt/bronze/customers_daily/")
target = DeltaTable.forPath(spark, "/mnt/silver/dim_customer/")

# Step 1: Identify changed records
changes = updates.alias("s").join(
    target.toDF().filter("is_current = true").alias("t"),
    "customer_id"
).filter(
    (col("s.city") != col("t.city")) | 
    (col("s.name") != col("t.name"))
).select("s.*")

# Step 2: Expire old records
target.alias("t").merge(
    changes.alias("s"),
    "t.customer_id = s.customer_id AND t.is_current = true"
).whenMatchedUpdate(set={
    "is_current": "false",
    "end_date": "current_date()"
}).execute()

# Step 3: Insert new current records
new_records = changes.withColumn("is_current", lit(True)) \
    .withColumn("start_date", current_date()) \
    .withColumn("end_date", lit("9999-12-31").cast("date"))

new_records.write.format("delta").mode("append") \
    .save("/mnt/silver/dim_customer/")

# Step 4: Insert brand new customers (not existing at all)
target.alias("t").merge(
    updates.alias("s"),
    "t.customer_id = s.customer_id"
).whenNotMatchedInsert(values={
    "customer_id": "s.customer_id",
    "name": "s.name",
    "city": "s.city",
    "is_current": "true",
    "start_date": "current_date()",
    "end_date": "'9999-12-31'"
}).execute()
```

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
CATEGORY 3: QUICK-FIRE QUESTIONS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

```
Q9:  What is ACID?
A: Atomicity (all or nothing), Consistency (valid state), 
   Isolation (concurrent safety), Durability (survives crash).
   Delta Lake provides ACID on data lakes.

Q10: Parquet vs CSV?
A: Parquet is columnar, compressed, type-safe, and 10-100x faster 
   for analytics. CSV is row-based, human-readable, but slow and large.

Q11: What is a partition in Spark?
A: A logical chunk of data processed by one task. More partitions = 
   more parallelism. Default 200 for shuffles. Too few = some nodes 
   overloaded. Too many = overhead of managing tiny tasks.

Q12: Cache vs Persist?
A: cache() = persist(StorageLevel.MEMORY_ONLY). persist() lets you 
   choose storage level — MEMORY_ONLY, MEMORY_AND_DISK, DISK_ONLY, etc.

Q13: Narrow vs Wide transformation?
A: Narrow: Each input partition maps to one output partition (map, filter).
   No shuffle needed.
   Wide: Input partitions map to many output partitions (groupBy, join).
   Requires shuffle (expensive!).

Q14: What is a Spark DAG?
A: Directed Acyclic Graph — Spark's execution plan showing all 
   transformations and their dependencies before running.

Q15: What is AQE in Spark 3?
A: Adaptive Query Execution — Spark re-optimizes the query plan 
   at runtime based on actual data statistics, handling skew and 
   choosing better join strategies dynamically.

Q16: Self-hosted vs Azure Integration Runtime?
A: Self-hosted IR runs on your on-premises machine to access 
   on-prem data sources. Azure IR runs in Azure cloud for 
   cloud-to-cloud data movement.

Q17: What is Unity Catalog in Databricks?
A: Unified governance solution for all data assets across all 
   workspaces — provides access control, auditing, lineage, 
   and data discovery at table/column level.

Q18: How does Z-ORDER work?
A: Z-ORDER co-locates related data in the same set of files based 
   on specified columns. When you filter on those columns, Spark 
   uses data skipping metadata to read far fewer files.

Q19: What is data lineage?
A: Tracking data flow from source to destination — where it came 
   from, how it was transformed, and where it went. Microsoft 
   Purview provides this automatically for ADF pipelines.

Q20: Explain idempotency in pipelines.
A: Running the same pipeline multiple times produces the same result. 
   I achieve this using MERGE (upsert) instead of INSERT, overwrite 
   mode for full loads, and watermark-based incremental logic.
```

---

### 16.2 Resume Keywords & Skills Section

```
TECHNICAL SKILLS (for Resume):

Cloud Platform: Microsoft Azure (ADLS Gen2, ADF, Databricks, 
                Synapse Analytics, Event Hubs, Key Vault, Purview)

Big Data: Apache Spark, PySpark, Spark SQL, Delta Lake, 
          Structured Streaming

Databases: SQL Server, Azure SQL, PostgreSQL, Cosmos DB

Languages: Python, SQL, PySpark

Data Modeling: Star Schema, Snowflake Schema, SCD Type 1/2, 
              Medallion Architecture (Bronze/Silver/Gold)

ETL/ELT: Azure Data Factory, Databricks Workflows, 
         Incremental Loading, CDC, Data Quality Framework

DevOps: Azure DevOps, Git, CI/CD for ADF & Databricks

Formats: Parquet, Delta, JSON, CSV, Avro

Governance: Microsoft Purview, Unity Catalog, RBAC, ACL, RLS

Visualization: Power BI (DirectQuery, Import mode)

Certification: DP-203 (Azure Data Engineer Associate)
```

---

### 16.3 Study Schedule Summary

```
WEEK 1-2:   Data fundamentals, SQL, Database concepts
            Star Schema, SCD, ETL/ELT
            ⏰ 3-4 hours/day
            
WEEK 2-3:   Azure basics, ADLS Gen2, Medallion Architecture
            Azure Data Factory (deep dive)
            ⏰ 4-5 hours/day

WEEK 3-5:   PySpark, Databricks, Delta Lake
            Transformations, Window functions
            ⏰ 5-6 hours/day (lots of hands-on coding!)

WEEK 5-6:   Synapse Analytics (Dedicated + Serverless)
            Streaming basics (Event Hubs, Structured Streaming)
            ⏰ 4-5 hours/day

WEEK 6-7:   Security, Governance, Optimization
            Performance tuning, monitoring
            ⏰ 3-4 hours/day

WEEK 7-8:   End-to-end project build
            Interview preparation
            Mock interviews, resume optimization
            ⏰ 5-6 hours/day
```

### 16.4 Certification Path

```
📜 DP-203: Azure Data Engineer Associate
   - Primary certification for this role
   - Covers everything in this roadmap
   - 40-60 questions, 120 minutes
   - Passing: 700/1000
   
Study resources:
1. Microsoft Learn (free official modules)
2. Practice tests on MeasureUp
3. This roadmap covers 90% of exam topics

Exam Topics Weightage:
┌────────────────────────────────────┬──────────┐
│ Topic                              │ Weight   │
├────────────────────────────────────┼──────────┤
│ Design & implement data storage    │ 15-20%   │
│ Develop data processing            │ 40-45%   │
│ Secure, monitor, optimize          │ 30-35%   │
│ Design & implement data pipelines  │ 10-15%   │ (overlap)
└────────────────────────────────────┴──────────┘
```

---

## 🎯 Final Cheat Sheet: What to Say in Every Interview

```
┌─────────────────────────────────────────────────────────────┐
│               INTERVIEW GOLDEN FRAMEWORK                     │
│                                                             │
│  For ANY technical question, structure your answer as:       │
│                                                             │
│  1. WHAT it is (1 sentence definition)                      │
│  2. WHY it matters (business value)                         │
│  3. HOW you've used it (your project example)               │
│  4. WHEN to use vs alternatives (tradeoffs)                 │
│                                                             │
│  Example:                                                    │
│  Q: "Tell me about Delta Lake"                              │
│                                                             │
│  WHAT: "Delta Lake is an open-source storage layer that     │
│         brings ACID transactions to data lakes."            │
│                                                             │
│  WHY: "Without it, partial writes can corrupt data, and     │
│        you can't do updates or deletes on lake files."      │
│                                                             │
│  HOW: "In my project, I used Delta MERGE for incremental    │
│        upserts in the Silver layer and Time Travel for      │
│        debugging data issues in production."                │
│                                                             │
│  WHEN: "I use Delta over plain Parquet whenever I need      │
│         updates, ACID guarantees, or schema enforcement.    │
│         Plain Parquet is fine for one-time batch writes      │
│         where I always overwrite."                          │
└─────────────────────────────────────────────────────────────┘
```

---

> **🏆 You now have a complete roadmap from absolute zero to interview-ready Azure Data Engineer. Every concept is explained with real-life analogies so you can explain them naturally. Build the project, practice the SQL and PySpark code daily, and you'll be ready. Good luck! 🚀**
