# 03 — PySpark & SQL Mastery

---

## PART A: PySpark

## 1. DataFrame API, Transformations vs Actions, Lazy Evaluation, DAG

### Quick Theory
A **DataFrame** is Spark's distributed table abstraction — data split across partitions on different executors, with a schema. **Transformations** (e.g., `.filter()`, `.select()`, `.groupBy()`) are *lazy* — they just build up a logical plan, nothing runs yet. **Actions** (e.g., `.show()`, `.count()`, `.write()`) trigger actual execution. Spark converts the chain of transformations into a **DAG (Directed Acyclic Graph)** of stages, optimizes it (via Catalyst optimizer), and only then executes.

- **Why lazy evaluation**: lets Spark see the *whole* chain of operations before running anything, so it can optimize (e.g., push filters down, skip unnecessary columns) instead of executing each line naively.
- **Limitation**: debugging is less intuitive for beginners since errors in transformations may only surface when an action finally triggers execution.

### Real-Life Analogy
Ordering at a restaurant with a **tasting menu you build course-by-course** (transformations = adding courses to your order slip), but the kitchen doesn't start cooking anything until you say "send it to the kitchen now" (the action). The head chef (Catalyst optimizer) looks at your whole order first and might resequence steps (e.g., prep the sauce that takes longest first) for efficiency before any pan touches the stove.

### Technical Explanation
```
df = spark.read.parquet(...)          # lazy
   .filter(col("status") == "FILLED") # lazy (transformation)
   .select("order_id", "amount")      # lazy (transformation)
df.count()                            # ACTION — triggers execution
```
```
Logical Plan → Catalyst Optimizer → Physical Plan → DAG of Stages → Tasks on Executors
```
- Each **wide transformation** (e.g., `groupBy`, `join`, `repartition`) creates a new **stage boundary** because it requires a shuffle (data movement across the network between executors). **Narrow transformations** (e.g., `filter`, `select`, `map`) stay within the same stage — no shuffle needed.

### Interview Q&A
1. *What's the difference between a transformation and an action?* — Transformations define a new DataFrame lazily (nothing computed yet), like `.filter()` or `.withColumn()`; actions trigger actual computation and return a result or write output, like `.count()`, `.collect()`, or `.write.save()`.
2. *Why does Spark use lazy evaluation instead of executing each line immediately?* — It lets the Catalyst optimizer see the entire chain of operations and rewrite/optimize the whole plan (predicate pushdown, column pruning, join reordering) before running anything, which is far more efficient than executing line-by-line.
3. *What is a DAG in Spark and why is it "acyclic"?* — It's the graph of stages/tasks Spark builds from your transformations, showing dependencies between them; it's acyclic because data only flows forward through transformations — there are no loops back to earlier stages.
4. *What's the difference between a narrow and wide transformation?* — Narrow transformations (filter, map, select) process each partition independently with no data movement; wide transformations (groupBy, join, distinct) require shuffling data across the network between executors, and create a new stage boundary.
5. *If lazy evaluation means nothing runs until an action, why does `df.show()` sometimes seem slow the first time but fast if you call it again?* — Without caching, each action re-triggers the entire lazy chain from source; if you cache/persist the DataFrame after the first action, subsequent actions reuse the cached result instead of recomputing from scratch.

### One-Line Revision
**Transformations plan the meal, actions send it to the kitchen, and the DAG is the optimized recipe Spark cooks from.**

---

## 2. Partitioning, Repartition vs Coalesce

### Quick Theory
Data in a Spark DataFrame is split into **partitions** distributed across executors — this is the unit of parallelism. `repartition(n)` does a **full shuffle** to redistribute data into exactly `n` partitions (can increase or decrease, and balances evenly). `coalesce(n)` **avoids a full shuffle** by merging existing partitions together — only reduces partition count, and can result in uneven partition sizes.

### Real-Life Analogy
Imagine boxes on a warehouse floor. `repartition` is calling everyone in to **completely re-sort and re-box everything** evenly across N new stacks — thorough but labor-intensive (shuffle). `coalesce` is just **pushing a few existing stacks together** without re-sorting individual boxes — fast, but stacks might end up uneven if some were already very full.

### Interview Q&A
1. *When would you use coalesce instead of repartition?* — When reducing partition count before writing output (e.g., avoiding hundreds of tiny output files) and you don't need perfectly even partition sizes — coalesce avoids the cost of a full shuffle that repartition requires.
2. *Why might repartition still be worth its shuffle cost sometimes?* — When data is heavily skewed across current partitions (some huge, some tiny) and you need even distribution for balanced downstream processing (e.g., before a join), the shuffle cost is worth it to avoid a slow "straggler" task.
3. *What happens if you call coalesce(1) on a huge DataFrame before writing?* — It forces everything into a single partition/output file, which can create a severe memory/performance bottleneck on one executor and produce one huge unsplittable file — usually a mistake unless the data is genuinely small.
4. *How does the number of partitions relate to the number of Spark tasks?* — Roughly one task is created per partition per stage, so partition count directly controls the degree of parallelism — too few partitions underutilizes the cluster, too many creates excessive scheduling overhead.

### One-Line Revision
**Repartition reshuffles everything evenly (costly but balanced); coalesce just merges existing pieces (cheap but can be lopsided).**

---

## 3. Joins — Broadcast Join & Shuffle Join

### Quick Theory
A **shuffle (sort-merge) join** redistributes both DataFrames across the network so matching keys land on the same executor — expensive for large-large joins. A **broadcast join** sends the *entire smaller* DataFrame to every executor (no shuffle of the big table needed) — very fast when one side is small enough to fit in memory (default threshold ~10MB, configurable, and Spark's Adaptive Query Execution can auto-broadcast at runtime).

### Real-Life Analogy
Joining a huge customer transaction table with a tiny "country code → country name" lookup table is like every cashier in every store (executor) **keeping a small printed reference card** (the broadcasted small table) in their pocket, instead of shipping every single transaction to one central warehouse to look up the country name.

### Interview Q&A
1. *When does Spark automatically choose a broadcast join?* — When one side of the join is smaller than the broadcast threshold (`spark.sql.autoBroadcastJoinThreshold`, default 10MB) — or you can force it explicitly with `broadcast(df)` hint regardless of size estimate.
2. *What's the risk of forcing a broadcast join on a table that turns out to be large?* — Out-of-memory errors on executors, since the entire "small" table gets copied to every executor's memory — if its actual size was underestimated, this can crash the job.
3. *How does Adaptive Query Execution (AQE) help with join strategy?* — AQE re-optimizes the physical plan at runtime using actual observed data sizes (not just static statistics), so it can switch a planned shuffle join to a broadcast join if the actual data turns out small enough, and can also fix skewed partitions mid-query.
4. *A join is taking forever and one executor task is stuck for a long time. What's likely happening and how do you fix it?* — Likely data skew — one join key has disproportionately more rows than others, so one partition/task does far more work; fix with salting the skewed key, enabling AQE skew join optimization, or broadcasting the smaller side if applicable.

### One-Line Revision
**Broadcast the small table to every worker's pocket; shuffle-join only when both sides are genuinely big.**

---

## 4. Window Functions (PySpark)

### Quick Theory
Window functions compute a value **across a set of rows related to the current row** (a "window") without collapsing rows like `groupBy` does — e.g., running totals, rankings, previous/next row comparisons, all while keeping every original row.

### Real-Life Analogy
Like a **leaderboard at a race** — every runner (row) can see their own rank *and* still exists individually in the results list, unlike a summary that only shows "1st place: John" and discards everyone else.

### Technical Explanation
```python
from pyspark.sql.window import Window
from pyspark.sql.functions import row_number, rank, lag, sum as _sum

window_spec = Window.partitionBy("account_id").orderBy("trade_date")

df.withColumn("running_total", _sum("amount").over(window_spec)) \
  .withColumn("rank", rank().over(window_spec)) \
  .withColumn("prev_amount", lag("amount", 1).over(window_spec))
```
- `partitionBy` = groups rows into independent windows (like a mini-groupBy) without collapsing them.
- `orderBy` inside the window = defines the sequence for ranking/running calculations.
- `lag`/`lead` = look at previous/next row's value within the window — extremely common for detecting "changed since last record" in SCD Type 2 or CDC logic.

### Interview Q&A
1. *How is a window function different from groupBy?* — `groupBy` collapses multiple rows into one summary row per group; window functions compute a value across a related set of rows (partition) but **keep every original row** in the output.
2. *How would you find the latest record per account_id in a Silver table using a window function?* — Use `row_number().over(Window.partitionBy("account_id").orderBy(col("updated_at").desc()))` and filter where `row_number = 1` — a very common dedup/latest-record pattern.
3. *What's the difference between `rank()` and `dense_rank()`?* — `rank()` leaves gaps after ties (1,1,3), `dense_rank()` doesn't (1,1,2) — matters when generating tie-aware leaderboards or rankings.
4. *How would you detect a value change from the previous row (e.g., account status changed) using window functions?* — Use `lag("status").over(window_spec)` and compare it to the current row's status; a mismatch flags a "changed" record — the backbone of building SCD Type 2 change detection logic.

### One-Line Revision
**Window functions let each row "see" its neighbors without losing its own seat at the table.**

---

## 5. Caching & Persist

### Quick Theory
`.cache()` (shorthand for `.persist(MEMORY_AND_DISK)`) tells Spark to keep a DataFrame's computed result in memory/disk after the first action, so subsequent actions on it don't recompute the whole lazy chain from source. Useful when a DataFrame is reused multiple times downstream (e.g., referenced by 3 different aggregations).

### Interview Q&A
1. *When should you cache a DataFrame?* — When it's expensive to compute (e.g., involves a big join/shuffle) and will be reused multiple times in the same session — caching avoids recomputing the whole lineage each time.
2. *What's the difference between `.cache()` and `.persist()`?* — `.cache()` is shorthand for `.persist(StorageLevel.MEMORY_AND_DISK)`; `.persist()` lets you explicitly choose a storage level (memory-only, disk-only, memory-and-disk, with/without serialization or replication) to match memory constraints.
3. *What happens if you cache a DataFrame but never reuse it?* — It wastes cluster memory/disk for no benefit — caching only pays off with repeated use; caching something used just once adds overhead without gain.
4. *How do you remove a DataFrame from cache when you're done with it?* — Call `.unpersist()` to free the memory/disk space explicitly rather than waiting for Spark's LRU eviction.

### One-Line Revision
**Cache what you'll reuse repeatedly; don't cache what you'll touch only once.**

---

## 6. UDFs (User-Defined Functions) & Performance

### Quick Theory
A UDF lets you run custom Python logic row-by-row when built-in Spark SQL functions can't express what you need. **Downside**: standard Python UDFs serialize data out of the JVM into a Python process row-by-row, which is much slower than native Spark/Photon operations — a major performance anti-pattern if overused. **Pandas UDFs (vectorized UDFs)** batch rows into Pandas Series, dramatically reducing serialization overhead versus row-at-a-time UDFs.

### Real-Life Analogy
A native Spark function is like the kitchen's **built-in industrial dishwasher** (fast, integrated). A Python UDF is like **pulling each dish out, washing it by hand in a side sink, then putting it back** — works, but painfully slow at scale. A Pandas UDF is like washing a **whole tray of dishes at once** in that side sink instead of one at a time — better, but still not as fast as the built-in machine.

### Interview Q&A
1. *Why are Python UDFs slower than built-in Spark SQL functions?* — Built-in functions run natively within the JVM/Catalyst-optimized execution (and can use Photon); Python UDFs require serializing data out of the JVM to a separate Python process row-by-row, executing, and serializing results back — that cross-process overhead dominates at scale.
2. *What's a Pandas UDF and why is it better than a regular UDF?* — It operates on batches of rows as Pandas Series/DataFrames instead of one row at a time, using Apache Arrow for efficient serialization — cutting overhead significantly versus row-at-a-time Python UDFs while still allowing custom Python logic.
3. *When is it actually justified to write a UDF instead of finding a built-in function?* — When the logic genuinely can't be expressed with Spark SQL built-ins (e.g., a complex custom parsing/business rule, or calling an external Python library not available as a Spark function) — always check for a built-in alternative first.
4. *How would you convince a teammate to stop using a row-at-a-time UDF for a simple string transformation?* — Show that Spark already has a built-in function (e.g., `upper()`, `regexp_replace()`) that achieves the same result natively without the serialization overhead — most "custom" logic beginners UDF-ify already exists as a built-in.

### One-Line Revision
**Prefer built-ins > Pandas UDFs > row-at-a-time UDFs, in that order, for performance.**

---

## PART B: SQL

## 7. Joins (SQL)

### Quick Theory
INNER, LEFT/RIGHT OUTER, FULL OUTER, CROSS, and SEMI/ANTI joins — the fundamental way to combine rows from multiple tables based on a condition.

### Interview Q&A
1. *Difference between LEFT JOIN and LEFT SEMI JOIN?* — LEFT JOIN returns all left rows plus matched right columns (nulls if no match); LEFT SEMI JOIN returns only left rows that *have* a match, without including any columns from the right table at all — useful for existence checks/filtering without duplicating rows on a one-to-many match.
2. *What's a LEFT ANTI JOIN used for?* — Returns rows from the left table that have **no match** in the right table — the classic way to find "records in Bronze not yet processed into Silver," or "customers with no orders."
3. *Why can a JOIN unexpectedly multiply row counts?* — If the join key isn't unique on one side (a one-to-many or many-to-many relationship), each match produces a separate output row, so counts can balloon — always check key cardinality before joining, especially before aggregating.

### One-Line Revision
**Pick the join type by what you want kept: INNER = only matches, LEFT = keep left no matter what, ANTI = only the unmatched.**

---

## 8. Window Functions (SQL) & Ranking

### Quick Theory
Same concept as PySpark's window functions, expressed in SQL: `ROW_NUMBER()`, `RANK()`, `DENSE_RANK()`, `LAG()`/`LEAD()`, `SUM() OVER (...)`.

### Interview Q&A
1. *Write SQL to get the latest record per customer from a changelog table.*
```sql
SELECT * FROM (
  SELECT *, ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY updated_at DESC) AS rn
  FROM customer_changelog
) WHERE rn = 1;
```
2. *How would you compute a 7-day rolling average of daily trade volume in SQL?*
```sql
SELECT trade_date, account_id,
       AVG(volume) OVER (PARTITION BY account_id ORDER BY trade_date
                          ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) AS rolling_7d_avg
FROM daily_trades;
```
3. *Difference between window functions and GROUP BY, again for SQL specifically?* — GROUP BY reduces the result set to one row per group; window functions (`OVER`) keep every row and just compute an aggregate/ranking value relative to a partition, letting you see both detail and aggregate side by side.

### One-Line Revision
**`OVER (PARTITION BY ... ORDER BY ...)` is SQL's way of doing group math without losing your rows.**

---

## 9. CTE (Common Table Expressions)

### Quick Theory
A `WITH` clause that defines a named, temporary result set usable within a single query — improves readability and lets you break complex logic into named, sequential steps (and supports recursive CTEs for hierarchical data).

### Interview Q&A
1. *Why use a CTE instead of a nested subquery?* — Readability and reusability within the query — a CTE gives a name to an intermediate result so a long query reads like a sequence of logical steps instead of deeply nested parentheses; multiple CTEs can also reference earlier ones.
2. *Does a CTE get materialized/cached automatically?* — Not necessarily — most engines (including Spark SQL) treat a CTE as a logical alias that can be inlined and re-evaluated wherever referenced, unless the engine specifically optimizes/caches it; if a CTE is expensive and used multiple times, consider actually caching the underlying DataFrame if performance matters.
3. *What's a recursive CTE and when would you need one?* — A CTE that references itself to walk hierarchical/graph-like data (e.g., an org chart, or a chain of trade amendments referencing a parent trade) — not extremely common in typical ETL but occasionally needed for hierarchical reference data.

### One-Line Revision
**CTEs are named checkpoints in a query's logic — readability tool, not automatically a performance tool.**

---

## 10. Views

### Quick Theory
A **view** is a saved query definition that acts like a virtual table — recomputed each time it's queried (unless it's a **materialized view**, which physically stores results and refreshes on a schedule/trigger).

### Interview Q&A
1. *View vs Materialized View — when to use each?* — A regular view is best for logic reuse/security (e.g., masking PII) where freshness matters more than compute cost; a materialized view is best when the underlying query is expensive and queried very frequently, trading storage/staleness for read speed.
2. *How are views used for security in Unity Catalog?* — Dynamic views can filter rows or mask columns based on the querying user's group/role, letting you grant broad "SELECT on view" access while the base table stays locked down.

### One-Line Revision
**A view is a saved lens over data; a materialized view is a snapshot photo taken through that lens.**

---

## 11. Indexes (conceptual — Spark/Delta context)

### Quick Theory
Traditional B-tree indexes (as in SQL Server/Oracle) don't really exist in Spark/Delta Lake the same way. Instead, Delta Lake achieves similar speedups via **data skipping** (file-level min/max statistics automatically tracked in the transaction log), **Z-ORDER clustering**, and **partitioning** — these are the "indexing" tools of the Lakehouse world.

### Interview Q&A
1. *Does Delta Lake have indexes like a traditional database?* — Not B-tree indexes in the traditional RDBMS sense; instead it relies on automatically-collected file-level column statistics (min/max) for data skipping, plus optional Z-ordering to cluster related data together — both let queries skip irrelevant files rather than using a row-level index lookup.
2. *How does partitioning act like a coarse index?* — Partitioning by a column (e.g., `trade_date`) physically separates data into different folders, so a query filtering on that column can skip entire folders without reading any data inside them — similar in spirit to an index narrowing a search.
3. *What's the risk of over-partitioning a Delta table (e.g., partitioning by a high-cardinality column like customer_id)?* — Creates the small-file problem — too many tiny partitions/files, each with overhead, making both storage and query planning inefficient; better to partition on lower-cardinality columns and use Z-ORDER for higher-cardinality filtering needs.

### One-Line Revision
**Delta's "index" is smart file-skipping via stats, partitioning, and Z-order — not a classic B-tree.**

---

## 12. Aggregations & SQL Execution Order

### Quick Theory
SQL is *written* in one order (`SELECT ... FROM ... WHERE ... GROUP BY ... HAVING ... ORDER BY`) but *logically executed* in a different order — understanding this explains why you can't use a `SELECT` alias in `WHERE` but can in `ORDER BY`.

### Technical Explanation — Logical Execution Order
```
1. FROM / JOIN        — build the source rowset
2. WHERE              — filter individual rows
3. GROUP BY           — group remaining rows
4. HAVING             — filter groups (after aggregation)
5. SELECT             — compute output columns/aliases
6. ORDER BY           — sort final result
7. LIMIT              — cut down to requested row count
```

### Interview Q&A
1. *Why can't you use a column alias defined in SELECT inside a WHERE clause?* — Because WHERE is logically executed before SELECT, so the alias doesn't exist yet at that stage — you'd need to repeat the full expression in WHERE, or wrap the query in a subquery/CTE.
2. *What's the difference between WHERE and HAVING?* — WHERE filters individual rows before grouping/aggregation happens; HAVING filters groups after aggregation (e.g., `HAVING COUNT(*) > 10`) — HAVING can reference aggregate functions, WHERE cannot.
3. *Why does query optimization sometimes reorder operations even though there's a fixed logical execution order?* — The logical order defines correctness (what the query *means*), but the query optimizer (Catalyst in Spark SQL) can choose a different physical execution order/strategy (e.g., pushing a filter down before a join) as long as the final result is identical — performance optimization without changing semantics.

### One-Line Revision
**SQL is written top-down but executed FROM→WHERE→GROUP BY→HAVING→SELECT→ORDER BY — that's why aliases can't be used too early.**

---

## 13. Query Optimization (general)

### Interview Q&A
1. *What's predicate pushdown?* — Pushing filter conditions as close to the data source as possible (even into the file format/storage layer) so irrelevant data is never read into memory in the first place — Delta Lake's file-level stats enable this automatically.
2. *What's column pruning?* — Only reading the columns actually referenced by the query (critical benefit of columnar formats like Parquet/Delta) instead of scanning entire rows — drastically reduces I/O for wide tables.
3. *How would you troubleshoot a slow SQL query on a Delta table?* — Check the query plan (`EXPLAIN`) for full table scans vs partition/file pruning, verify statistics are up to date, check for data skew or missing Z-ordering on filter columns, and confirm join strategy (broadcast vs shuffle) is appropriate for table sizes.

### One-Line Revision
**Fast queries read as little data as possible — pushdown and pruning are how that happens automatically.**

---

Next file: `04_etl_patterns_and_architecture.md`
