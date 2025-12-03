# Database Systems — Final Exam Master Guide

---

## 0) Map of Topics
DB/DBMS/DMS/PDIP • OLTP/OLAP/Data Warehouse/Metadata/FMS • Abstraction levels (Physical/Logical/View; DIKW; Entity/Information) •  
“What is data in S=(DB,Pr)” + Proper data vs Metadata + Dictionary/Catalog + IMS • Data models & sublanguages (DDL/DML/TCL/DCL/SDL) •  
Procedural vs Declarative (RA/TRC/DRC/SQL) • Relational model (relation/tuple/attribute/domain/degree/cardinality/semantics/1NF) •  
Keys (superkey/PK/FK) • Integrity (entity/referential) • Associative relations • RA operations (σ,π,×,⋈,∪,−,∩,ρ; joinability; natural vs cross) •  
Outer joins • CTE • Aggregation vs Window; Ranking; DISTINCT notes; `OVER` no partition • Transactions & ACID (atomic statements vs multi‑statement; TCL) •  
CAP theorem • Triggers (`NEW`/`OLD`; BEFORE/AFTER) • Visual refs (FMS vs RDBMS, Graph vs Relational, Views, Query processing) •  
**Plus deep expansions:** Normalization (2NF/3NF/BCNF), Functional Dependencies, Indexes & Execution Plans, Join Algorithms, Concurrency (locks vs MVCC), Recovery (WAL), Distributed DB (replication/sharding; PACELC), Window frames, Advanced SQL idioms, Optimization heuristics.

## 1) DB, DBMS, Datastore, DMS, PDIP
### Notebook Core
- **Database**: organized related data.  
- **DBMS**: software to **store** and **manage** the database via a language (SQL).  
- **Datastore**: any data holding place (may be non‑relational).  
- **DMS**: umbrella term for FMS/DBMS.  
- **PDIP**: changing physical storage must not force program changes (**DBMS: yes; FMS: no**).
- **Performance = knowledge + attitude** (tech skills + discipline).

### Deep Dive
- **Logical vs Physical independence:**  
  - *Physical*: change files/indexes/placement without changing queries.  
  - *Logical (extra)*: change schema (split a table into two with a view) without app breakage.  
- **Why DBMS beats files:** integrity rules, concurrency control, recovery, declarative queries, security, optimization.

### Exam Patterns
- “Independent from disk layout” → PDIP.  
- “Manage data efficiently for many users” → DBMS.  
- “Flat files + scripts” → FMS (no PDIP).

## 2) Application Types: OLTP vs OLAP (Data Warehouse)
### Notebook Core
- **OLTP**: read/write, simple queries, fast response, concurrency.  
- **OLAP**: mostly read, complex queries, decision support.  
- **Warehouse**: OLAP store for historical analysis.  
- **Metadata** helps interpretation.

### Deep Dive
- **Schema shapes:** OLTP → normalized; OLAP → star/snowflake (denormalized).  
- **Typical metrics:** OLTP latency < 50ms; OLAP scans millions of rows.  
- **Workload mistakes:** reporting on OLTP causes locks/slowdowns; use replicas or warehouse ETL.

### Exam Patterns & Templates
- “per month totals” → `GROUP BY month` (OLAP).  
- “book a seat/payment” → transaction with ACID (OLTP).

## 3) Abstraction Levels (Physical, Logical, View) + DIKW
### Notebook Core
- **Physical** how; **Logical** what; **View** user perspective.  
- **DIKW**: Data → Information → Knowledge.  
- **Entity**: thing we store.

### Deep Dive
- **Security via views** (students can’t see salary, instructors can).  
- **Updatable views** (limited) vs **materialized views** (precomputed; refresh) — useful in OLAP.  

### Exam Patterns
- “same data seen differently” → **views**.  
- “hide sensitive column” → **CREATE VIEW** projecting columns.

## 4) What is Data in S=(DB,Pr): Proper Data vs Metadata; Dictionary; IMS
### Notebook Core
- “Data in S” = anything representable/storable.  
- **Proper data** (time‑dependent) vs **metadata** (schema; time‑independent).  
- **Data dictionary/system catalog** stores metadata.  
- **Information about entity e** = data about e paired with metadata per data model.  
- **IMS** = info store + programs (DBMS is an IMS).

### Deep Dive
- **Catalog contents:** tables, columns, types, constraints, indexes, statistics (for planner).  
- **Statistics (extra)**: cardinality & histograms drive query plans; stale stats → bad plans.

## 5) Data Models & DB Languages
### Notebook Core
- **Data model** describes data/relationships/semantics/constraints.  
- **Sub‑languages:** DDL, DML, TCL, DCL, SDL.  
- **Styles:** Procedural (RA) vs Declarative (SQL, TRC/DRC).

### Deep Dive
- **Conceptual (ER) modeling** → tables (attributes/types) → constraints (PK/FK/UNIQUE/CHECK).  
- **Security model**: GRANT/REVOKE; role‑based access.  
- **Physical design** (SDL): partitioning, clustering, compression.

### Templates
- DDL quick start:
```sql
CREATE TABLE course(
  id INT PRIMARY KEY,
  title TEXT NOT NULL,
  dept  TEXT
);
```

## 6) Relational Model: Domains, Degree, 1NF, Keys
### Notebook Core
- **R ⊆ S₁×…×Sₙ**; tuple’s i‑th value from domain Sᵢ.  
- **Tuple/Attribute/Domain**; **Degree/Cardinality**; **Domain semantics**.  
- **1NF**: domains **simple/atomic**.  
- **Superkey**, **Primary key (minimal)**; **Foreign key**.  
- **Associative** vs **purely associative** relations.

### Deep Dive
- **Functional Dependencies (FDs):** X→Y means X determines Y.  
- **Normalization**:  
  - **2NF** (no partial dependency on part of a composite key),  
  - **3NF** (no transitive dependency on key),  
  - **BCNF** (for every FD X→Y, X is a superkey).  
  Benefits: update anomalies removed; costs: more joins.  
- **Surrogate vs natural keys:** integer `id` vs real‑world identifier (email).

### Exam Patterns
- “single attribute domain” → 1NF.  
- “duplicate group of columns” → normalize.  
- “junction table” → many‑to‑many.

## 7) Integrity: Entity + Referential
### Notebook Core
- **Entity integrity**: PK unique & non‑NULL.  
- **Referential integrity**: FK matches referenced PK (or NULL).

### Deep Dive
- **Actions:** `ON DELETE/UPDATE RESTRICT | CASCADE | SET NULL | SET DEFAULT`.  
- **Deferrable constraints (extra)**: check at commit; helpful for cyclic FKs.

### SQL Template
```sql
ALTER TABLE takes
ADD CONSTRAINT takes_fk
FOREIGN KEY (course_id) REFERENCES course(id) ON DELETE CASCADE;
```

## 8) Relational Operations (σ, π, ×, ⋈, ∪, −, ∩, ρ)
### Notebook Core
- Selection, Projection, Product, Join, Union, Difference, Intersection, Rename.  
- **Joinability**, **Natural join**, **Cross join** (no entity meaning).

### Deep Dive
- **Algebra laws:** selection/projection pushdown; join associativity; rewriting product+selection into join.  
- **Set vs Bag:** SQL keeps duplicates → use `DISTINCT`.  
- **Anti‑join & semi‑join patterns:** `EXISTS`, `NOT EXISTS`, `IN/NOT IN`.

### Exam Patterns
- “only rows meeting …” → σ/`WHERE`.  
- “only these columns” → π/`SELECT cols`.

## 9) Joins (Inner, Natural, Outer) + Predicates
### Notebook Core
- Inner join; Natural join (same‑named attrs); Outer joins (LEFT/RIGHT/FULL) add NULLs.  
- Cross join = product.

### Deep Dive
- **Join algorithms**: nested‑loop, hash join, merge join; index nested‑loop for selective joins.  
- **Predicate placement:** `LEFT JOIN ... ON ... WHERE right.col IS NULL` → anti‑join.  
- **Null logic:** comparisons with NULL are UNKNOWN → use `IS NULL`/`IS NOT NULL`.

### SQL Templates
```sql
-- Natural/Using
SELECT * FROM student NATURAL JOIN takes;
SELECT * FROM student JOIN takes USING (student_id);

-- Anti-semi join (who took no courses)
SELECT s.* FROM student s
LEFT JOIN takes t ON t.student_id = s.id
WHERE t.student_id IS NULL;
```

## 10) CTE (Common Table Expression)
### Notebook Core
- `WITH name AS (...) SELECT ... FROM name`

### Deep Dive
- **Why use:** readability, reuse, layering logic.  
- **Recursive CTE (extra)**: trees/graphs, e.g., org chart.

### Template
```sql
WITH totals AS (
  SELECT dept, COUNT(*) AS n FROM student GROUP BY dept
)
SELECT * FROM totals WHERE n > 50;
```

## 11) Aggregation vs Window (Partition); Ranking; Frames
### Notebook Core
- Aggregates collapse rows by `GROUP BY`.  
- Windows compute per row with `OVER (PARTITION BY ...)` and **do not collapse**.  
- `DISTINCT` may slow; `OVER` without partition = whole table.  
- Ranking: `ROW_NUMBER`, `RANK`, `DENSE_RANK`.

### Deep Dive
- **Window frames**: `ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW` → running totals; `RANGE` vs `ROWS`.  
- **Top‑N per group**: `ROW_NUMBER() OVER (PARTITION BY dept ORDER BY score DESC) = 1`.  
- **Group sets (extra)**: `ROLLUP`, `CUBE`, `GROUPING SETS` (if supported).  
- **FILTER (WHERE …) on aggregates (extra, Postgres)** to count conditional subsets.

### Templates
```sql
-- Running total per student
SELECT user_id, ts, points,
SUM(points) OVER (PARTITION BY user_id ORDER BY ts
                  ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS run_total
FROM events;

-- Top scorer per department
SELECT * FROM (
  SELECT s.*, ROW_NUMBER() OVER (PARTITION BY dept ORDER BY score DESC) AS rn
  FROM scores s
) x WHERE rn = 1;
```

## 12) Transactions & ACID; Isolation & Concurrency
### Notebook Core
- Transaction = unit of work; **ACID**; each DDL/DML is atomic but sequences need explicit transaction.  
- `START TRANSACTION`, `COMMIT`, `ROLLBACK`.

### Deep Dive
- **Isolation models:** locks vs **MVCC** (multi‑version concurrency control).  
- **Anomalies:** dirty read, non‑repeatable read, phantom.  
- **Levels:** Read Committed, Repeatable Read, Serializable.  
- **Deadlocks:** detection & victim rollback; keep a consistent lock order to avoid.  
- **Durability via WAL (write‑ahead logging)** and checkpoints.

### Templates
```sql
START TRANSACTION;
  UPDATE accounts SET balance = balance - 100 WHERE id=1;
  UPDATE accounts SET balance = balance + 100 WHERE id=2;
COMMIT;
```

## 13) CAP (and PACELC for intuition)
### Notebook Core
- Choose 2 of **Consistency**, **Availability**, **Partition tolerance**.

### Deep Dive
- **PACELC (extra):** if **P**artition, choose **A** or **C**; **E**lse (no partition), choose **L**atency or **C**onsistency trade‑off.  
- **Replication modes:** leader‑follower (stronger C), multi‑leader/AP (higher A).  
- **Client guarantees (extra):** read‑your‑writes, monotonic reads.

## 14) Triggers
### Notebook Core
- Fire automatically BEFORE/AFTER INSERT/UPDATE/DELETE; use `NEW`/`OLD`.

### Deep Dive
- **Row vs Statement level**; **transition tables** (Postgres extra).  
- Use for auditing, derived values, RI gaps; avoid heavy logic that hides side effects.

### Template
```sql
CREATE TRIGGER audit_orders
AFTER UPDATE ON orders
FOR EACH ROW
INSERT INTO orders_log(order_id, old_status, new_status, ts)
VALUES(OLD.id, OLD.status, NEW.status, now());
```

## 15) Visual References
- **FMS vs RDBMS**; **Graph vs Relational**; **Views** for role‑based visibility; Query flow (parse → plan → execute).

## 16) SQL Keyword Playbook — MAX Depth (Exam + Practical)
> A superset of your earlier cheat‑sheet; you won’t need every item, but you’ll never be stuck.

### Retrieval & Filtering
`SELECT`, `FROM`, `WHERE`, `DISTINCT`, `ORDER BY`, `LIMIT/OFFSET`, `FETCH FIRST`, `NULLS FIRST/LAST (extra)`,  
`BETWEEN`, `LIKE`, `ILIKE (extra)`, `IN/NOT IN`, `EXISTS/NOT EXISTS`, `IS [NOT] NULL`, `CASE WHEN`, `COALESCE`, `GREATEST/LEAST (extra)`.

### Sets & Subqueries
`UNION/ALL`, `INTERSECT`, `EXCEPT/MINUS`, subquery operators `=, >, <, >=, <=`, `ALL`, `ANY/SOME`.

### Joins
`JOIN ... ON`, `NATURAL JOIN`, `USING`, `LEFT/RIGHT/FULL [OUTER]`, `CROSS JOIN`, `LATERAL / CROSS APPLY (extra)`.

### Grouping & Windows
`GROUP BY`, `HAVING`, aggregates `COUNT/SUM/AVG/MIN/MAX`, `GROUPING SETS / ROLLUP / CUBE (extra)`;  
`WINDOW` clause (extra), `OVER (PARTITION BY ... ORDER BY ...)`, `ROWS/RANGE frame`, `ROW_NUMBER/RANK/DENSE_RANK`, `NTILE (extra)`, `FIRST_VALUE/LAST_VALUE/LAG/LEAD (extra)`.

### Data Change & Schema
`INSERT`, `UPDATE`, `DELETE`, `MERGE (extra, if supported)`,  
`CREATE/ALTER/DROP TABLE`, `CREATE VIEW/MATERIALIZED VIEW (extra)`, `CREATE INDEX`, `UNIQUE`, `CHECK`, `DEFAULT`, `GENERATED ALWAYS AS (extra)`, `FOREIGN KEY REFERENCES`, `ON DELETE|UPDATE`, `DEFERRABLE (extra)`.

### Security & Transactions
`GRANT`, `REVOKE`, `START TRANSACTION`, `COMMIT`, `ROLLBACK`, `SAVEPOINT`, `SET TRANSACTION ISOLATION LEVEL`.

### Storage & Partitioning (SDL‑flavored)
`CREATE TABLE ... PARTITION BY (extra)`, `CLUSTER/REINDEX (extra)`, `ANALYZE` (update stats).

**English → SQL cues:** kept from previous guide; now with more:  
- “for each X” → `GROUP BY X` or `PARTITION BY X`.  
- “include even those without match” → left/right/full outer join.  
- “in both lists” → `INTERSECT`; “in A not B” → `EXCEPT/MINUS` or `NOT EXISTS`.  
- “top n per group” → window + `ROW_NUMBER()=1`.  
- “running total / moving average” → window with frame.  
- “at least one exists” → `EXISTS`; “exactly none” → `NOT EXISTS`.  
- “if ... then ... else” → `CASE WHEN`.

## 17) Indexes & Execution Plans (Big Practical Win)
### Deep Dive
- **Index types:** B+‑tree (range scans), Hash (equality), Bitmap (warehouses).  
- **Clustered vs non‑clustered**; **composite (multi‑column)**; **covering index** (all columns in index).  
- **Sargability:** write predicates the optimizer can use (`col >= 5` not `FUNCTION(col) >= 5`).  
- **Read plans:** scan → filter → join → aggregate; prefer index scans over full scans when selective.  
- **Cardinality estimation & statistics**: keep ANALYZE up‑to‑date.

### Templates
```sql
CREATE INDEX idx_orders_cust_date ON orders(customer_id, order_date);
-- Now: WHERE customer_id=? AND order_date BETWEEN ... is sargable and covering.
```

## 18) Join Algorithms & Cost Intuition
- **Nested‑loop**: good with tiny outer or indexed inner.  
- **Hash join**: equi‑joins on large sets; needs memory.  
- **Merge join**: both sides sorted on join key.  
**Heuristic:** push filters early, project needed columns, choose join order by selectivity.

## 19) Recovery & Reliability (WAL/Checkpoints)
- **Write‑Ahead Logging**: log changes before data pages; allows **ROLLBACK** and crash recovery.  
- **Checkpoint**: flush dirty pages to shorten recovery time.

## 20) Distributed Data: Replication & Sharding
- **Replication**: leader‑follower (read scaling, strong C on leader), multi‑leader (AP).  
- **Sharding**: split data by key across nodes; requires re‑thinking joins/transactions (often avoid cross‑shard).

## 21) Relational Algebra ↔ SQL Mini‑Dictionary
- σθ(R) ↔ `SELECT * FROM R WHERE θ`  
- πA(R) ↔ `SELECT A FROM R`  
- R × S ↔ `FROM R CROSS JOIN S`  
- R ⋈θ S ↔ `FROM R JOIN S ON θ`  
- R ⋈ S (natural) ↔ `NATURAL JOIN` or `JOIN ... USING (...)`  
- R ∪ S ↔ `UNION` (schema‑compatible)  
- R − S ↔ `EXCEPT/MINUS`  
- R ∩ S ↔ `INTERSECT`  
- ρnew(R) ↔ `SELECT ... FROM R AS new`

## 22) Exam Sprint — 15 Patterns to Memorize
1. **Selection:** `WHERE` with comparisons, `IN`, `BETWEEN`, `LIKE`, `IS NULL`.
2. **Projection:** list columns; use `AS` to rename.
3. **Inner join:** join two+ tables, ON keys; add filters.
4. **Outer join:** keep all rows from one/both sides; be careful with `WHERE` turning it inner.
5. **Self join:** join a table to itself (manager/employee).
6. **Anti‑join:** `LEFT JOIN ... WHERE right.id IS NULL` or `NOT EXISTS`.
7. **Group & HAVING:** count per X; filter groups.
8. **Top‑N overall:** `ORDER BY ... DESC FETCH FIRST N ROWS ONLY`.
9. **Top‑N per group:** window + `ROW_NUMBER()=1`.
10. **Running total:** window + frame.
11. **CASE:** conditional columns; e.g., bucketize scores.
12. **CTE:** multi‑step query without temp tables.
13. **Subqueries with ALL/ANY:** comparisons against sets.
14. **INSERT/UPDATE/DELETE:** with `WHERE` to target rows.
15. **Transactions:** wrap multi‑step logic; `COMMIT/ROLLBACK`.

## 23) Coverage Ledger (Notebook preserved)
All original notebook items are present (see Sections 1–15) with clarifications. Sections 16–22 add depth without removing or contradicting notebook content.

### Final Advice
1. **Translate English → operators/keywords** using Playbook cues.  
2. **Sketch joins** (draw tables/keys) *before* writing SQL.  
3. **Check NULL logic** in outer joins.  
4. **For performance**: sargable predicates, right indexes, avoid unnecessary `DISTINCT`/`SELECT *` in big joins.  
5. **For correctness**: enforce PK/FK; wrap multi‑statement operations in transactions.

Good luck — you’re exam‑proof and interview‑ready.

> **Why this doc?** You asked for *no information loss* plus a bit more depth so you can answer “why/how” questions.
> Everything from is preserved and clarified. Where a tiny amount of extra context helps, it’s marked as **(extra)**.

**Sections:** DB/DBMS/DMS • OLTP/OLAP • Abstraction levels • Data vs Metadata • Data models & languages •
Procedural vs Declarative • Relational model (domains, keys, 1NF) • Integrity • Relational operations & algebraic laws •
Joins (inner/natural/outer) • CTE • Aggregation vs Window (frames & ranking) • Transactions & ACID (isolation) •
CAP • Triggers • Visual refs (FMS vs RDBMS, Graph vs Relational, Views) • SQL Keyword Playbook (with English cues).

## 1) Database, DBMS, Datastore, DMS (and PDIP)
- **Database**: organized **related data**.
- **DBMS**: software that **stores** and **manages** the database.
- **Datastore**: any place data is kept (may be files, key–value store, etc.).
- **DMS**: umbrella term — can be a **DBMS** or an **FMS**.
- **Goal of DBMS**: store via DB, manage via a language (SQL).  
- **PDIP** (Physical Data Independence Property): **changing disks/files/layout must not require app changes**. Achieved by **DBMS**, not by plain **FMS**.

**Exam cues:** “manage data efficiently”, “independent from storage”, “software to operate on data” → **DBMS**.

## 2) Types of Applications — OLTP vs OLAP (and Warehouses)
- **OLTP**: read/write; short, simple queries; **fast response**; **high concurrency**; keeps current state.
- **OLAP**: read‑mostly; complex aggregations; **decision support**; can scan lots of data.
- **Data Warehouse**: OLAP store that keeps **historical** data; feeds dashboards.
- **Metadata**: schema about tables/columns/constraints — lets different users interpret the same data correctly.

**Pitfalls:** Using OLAP patterns (huge scans) on OLTP tables can slow transactions; vice versa denormalized OLAP schemas aren’t ideal for frequent updates.

## 3) Levels of Abstraction — Physical • Logical • View
- **Physical**: files, pages, indexes, disks.
- **Logical**: tables, columns, relationships.
- **View (External)**: **what a specific user sees** (subset/rename/hide).  
  *Example:* student view of `Instructor(ID, Name)`; instructor view includes `Salary`.

**DIKW pyramid:** Data → Information (data + context) → Knowledge (rules you infer).  
**Entity**: a “thing” we store (Student, Course).

**Exam cues:** “same data, two users see differently” → **views/metadata**.

## 4) What Counts as Data Inside S = (DB, Pr)
- “**Data in S**” = anything **representable & storable** in the system.
- Two parts: **(1) Proper data** (time‑dependent) and **(2) Metadata** (schema; time‑independent).
- **Data Dictionary/System Catalog**: where metadata lives.
- **Information about e** in S = data about e **paired with metadata** under S’s data model.
- **IMS** = information store + management programs; DBMSs are IMSs.

## 5) Data Models & Languages
**Data Model describes:** data • relationships • semantics • constraints.

**Sublanguages (by purpose):**
- **DDL**: `CREATE`, `ALTER`, `DROP` (define schema).
- **DML**: `SELECT`, `INSERT`, `UPDATE`, `DELETE` (work with rows).
- **TCL**: `START TRANSACTION`, `COMMIT`, `ROLLBACK`.
- **DCL**: `GRANT`, `REVOKE` (permissions).
- **SDL**: storage‑tuning part of DDL.

**Styles:**
- **Procedural** (how): **Relational Algebra**.
- **Declarative** (what): **SQL, TRC, DRC**.  
*(SQL → optimizer → many possible RA plans → one chosen.)*

## 6) Relational Model — Domains, Degree, 1NF, Keys
- **Relation R ⊆ S₁×S₂×…×Sₙ**: set of tuples; i‑th component from domain **Sᵢ**.
- **Tuple** (row), **Attribute** (column), **Domain** (allowed values), **Degree** (#attributes), **Cardinality** (#rows).
- **Simple (atomic) domain**: indivisible value. **1NF** requires simple domains.
- **Superkey**: attributes uniquely identifying rows.  
  **Primary key (PK)**: minimal superkey.  
  **Candidate keys**: all minimal superkeys; one chosen as **PK**. *(extra, but helps)*
- **Foreign key (FK)**: attribute(s) referencing another table’s PK.

**Associative relations:** junction tables for many‑to‑many (e.g., `Takes(StudentID, CourseID)`).

**Exam cues:** “degree”, “domain”, “atomic values”, “name of relation”, “superkey vs PK”.

## 7) Integrity — Entity & Referential
- **Entity integrity**: every row must have a non‑NULL, unique **PK**.
- **Referential integrity**: **FK** must match an existing **PK** (or be NULL if allowed).  
  Typical actions **(extra)**: `ON DELETE/UPDATE CASCADE | SET NULL | RESTRICT`.

**Common mistakes:** FK points to the wrong column; names mismatched when using `USING`/`NATURAL JOIN`.

## 8) Relational Operations — With Laws and Rewrites
**Retrieval operators** : σ (select rows), π (project cols), × (product), ⋈ (join), ∪, −, ∩, ρ.

**Algebraic laws (handy for reasoning/optimization)** *(extra but small)*:
- **σ distributes over ∪**: σₚ(R ∪ S) = σₚ(R) ∪ σₚ(S)
- **σ on product becomes join**: σₚ(R × S) = R ⋈ₚ S
- **Projection pushdown**: πₐ(σₚ(R)) = σₚ(π_{attrs needed}(R)) (when safe)
- **Join associativity/commutativity**: (R ⋈ S) ⋈ T = R ⋈ (S ⋈ T)

**Set vs Bag** *(extra but tiny)*: RA treats relations as **sets** (no duplicates). SQL is **bag** by default (duplicates kept) — use `DISTINCT` to force set behavior.

## 9) Joins — Inner, Natural, Outer (with NULL logic)
- **Inner join**: keep only matching pairs. Syntax: `JOIN ... ON`, or `NATURAL JOIN`, or `USING(col)`.
- **Natural join**: automatically matches equal **same‑named** columns.
- **Outer joins**: like inner, **plus unmatched rows** with **NULLs**. Types: **LEFT**, **RIGHT**, **FULL**.
- **Cross join**: product (no condition); rarely meaningful.

**Predicates placement pitfall (important):**
- `LEFT JOIN ... ON ... WHERE <filter on right table>` can **turn into inner join**.  
  Safe pattern: put right‑table filter in the **ON** clause when you want to keep unmatched left rows.

## 10) CTE — Common Table Expression
- Temporary, named subquery for clarity/reuse within a single statement:
```sql
WITH A AS (SELECT ...)
SELECT ... FROM A;
```
- Good to **structure multi‑step logic** without creating a real table.  
- **Recursive CTEs** exist **(extra)**.

## 11) Aggregation vs Window (Partition) — With Frames & Ranking
- **Aggregates** collapse rows per group using `GROUP BY` → one row per group.
- **Windows** compute per row over a **partition** using `OVER (PARTITION BY ...)` → **rows preserved**.
- **DISTINCT** inside aggregates/windows can be slow (dedup step).

**Frames (extra but tiny):** window functions can use frames for running totals:
```sql
SUM(amount) OVER (PARTITION BY account ORDER BY ts
                  ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW)
```
- If you omit the frame, many DBs use a sensible default; knowing this helps for rankings and moving averages.

**Ranking functions:**
- `ROW_NUMBER()` → 1,2,3,4
- `RANK()` → 1,2,2,4 (gaps)
- `DENSE_RANK()` → 1,2,2,3 (no gaps)

## 12) Transactions (TCL) & ACID — With Isolation (extra)
- **Transaction**: group of statements that must succeed/fail **together** (bank transfer example).
- **Atomic statements**: each DDL/DML statement is atomic; multiple statements need an explicit transaction to be atomic.
- **ACID**: Atomicity, Consistency, Isolation, Durability.
- **Commands**: `START TRANSACTION; ... COMMIT;` or `ROLLBACK;`

**Isolation levels (extra but common):**
- **Read Committed**: no dirty reads.
- **Repeatable Read**: also prevents non‑repeatable reads.
- **Serializable**: behaves as if transactions run one by one (strongest).

**Anomalies (extra names):** dirty read, non‑repeatable read, phantom read.

**Autocommit note (extra)**: many systems auto‑commit each statement; turn it off when you need multi‑statement atomicity.

## 13) CAP Theorem (Distributed DBMS)
- You can only have **two**: **Consistency (C)**, **Availability (A)**, **Partition tolerance (P)**.
- During a network split, a system must choose **CP** (consistent but may refuse requests) or **AP** (available but eventually consistent).

## 14) Triggers — Details & Caveats
- Fire **BEFORE/AFTER** `INSERT`/`UPDATE`/`DELETE`.
- Use **NEW** / **OLD** to access row values.
- **Row‑level** (`FOR EACH ROW`) vs **statement‑level** (**extra**, supported in some DBs).  
- Avoid heavy work and long transactions in triggers; they run implicitly on every affected row.

Skeleton (MySQL‑style):
```sql
DELIMITER $$
CREATE TRIGGER trig_name
BEFORE INSERT ON tbl
FOR EACH ROW
BEGIN
  -- use NEW.col / OLD.col
END $$
DELIMITER ;
```

## 15) Visual Reference
- **FMS vs RDBMS**: files + manual programs **vs** tables + SQL + integrity.
- **Graph DB vs Relational**: choose model that **fits** data shape (networked relations vs tabular facts).
- **Views**: different users → different visible columns (privacy/role).  
- **Query processing**: query producer → optimizer/executor.

## 16) SQL Keyword Playbook (with English cues) — Expanded
> This covers the ~80% most used SQL pieces you’ll write in exams. A few clearly marked items are **(extra)** to help when the task wording hints at them.

### Core Retrieval
| Keyword | Purpose | English cues |
|---|---|---|
| `SELECT` | choose columns | list, display, show, find |
| `FROM` | choose table(s) | from, using data of |
| `WHERE` | filter rows | whose, with, such that, that have |
| `DISTINCT` | deduplicate | unique, without repeats |
| `ORDER BY [ASC|DESC]` | sort | ascending/descending, highest/lowest, alphabetically |
| `LIMIT` / `FETCH FIRST` | limit count | top N, first N |
| `OFFSET` | skip | after first N, page |

### Sets & Subqueries
| Keyword | Purpose | Cues |
|---|---|---|
| `UNION` / `UNION ALL` | combine results | combine, together |
| `INTERSECT` | common rows | in both |
| `EXCEPT` / `MINUS` | difference | in A but not in B |
| `IN` / `NOT IN` | membership | in list, exclude list |
| `EXISTS` / `NOT EXISTS` | (anti)semi‑join pattern | if there exists / none exists |
| `ALL` | compare to all | greater than all |
| `ANY` / `SOME` | compare to at least one | greater than some |

### Joins
| Keyword | Purpose | Cues |
|---|---|---|
| `INNER JOIN ... ON` | matching rows only | with their matching, pair each with |
| `NATURAL JOIN` | auto match same‑named cols | natural, same name |
| `USING (c1,...)` | shorthand same names | using column(s) |
| `LEFT/RIGHT/FULL [OUTER] JOIN` | keep unmatched side(s) | include even those without a match |
| `CROSS JOIN` | all pairs | every combination |

**Join tips:** Put right‑side filters in **ON** for left joins when you must keep unmatched left rows.

### Grouping & Windows
| Keyword/Func | Purpose | Cues |
|---|---|---|
| `GROUP BY` | form groups | per department, for each user |
| `HAVING` | filter groups | groups having more than, departments with count > |
| `COUNT/SUM/AVG/MIN/MAX` | totals | number of, total/average/highest/lowest |
| `OVER (PARTITION BY ...)` | window | per X but keep all rows |
| `ORDER BY` inside `OVER` | ranking window order | top within each X |
| `ROW_NUMBER/RANK/DENSE_RANK` | ranking | enumerate, rank |
| `CASE WHEN ... THEN ... ELSE ... END` | conditional | if... then... else |
| `COALESCE(x, y)` | replace NULL | if missing then use |

### Data Changes & Schema
| Keyword | Purpose | Cues |
|---|---|---|
| `INSERT INTO ... VALUES/SELECT` | add rows | add/record/register |
| `UPDATE ... SET ... WHERE` | modify | change/increase/reduce |
| `DELETE FROM ... WHERE` | remove | delete/remove |
| `CREATE TABLE` | define | design a table |
| `ALTER TABLE ... ADD/MODIFY/DROP` | change structure | add column/change type |
| `DROP TABLE` | delete table | remove table |
| `CREATE VIEW ... AS` | virtual table | show limited columns/hide salary |
| `CREATE INDEX` | speed lookup | optimize/faster search **(extra)** |
| `CAST(expr AS type)` | change type | convert to, treat as **(extra)** |

### Integrity, Permissions, Transactions
| Keyword | Purpose | Cues |
|---|---|---|
| `PRIMARY KEY` | unique id | unique identifier |
| `FOREIGN KEY ... REFERENCES` | link tables | references/belongs to |
| `CHECK (...)` | enforce rule | must be between, must satisfy |
| `DEFAULT` | auto value | by default |
| `GRANT/REVOKE` | permissions | allow/deny user |
| `WITH` (CTE) | name subquery | reuse temporary result |
| `START TRANSACTION / COMMIT / ROLLBACK` | unit of work | make steps one action / save / undo |

**Null & pattern helpers (extra):** `IS NULL`, `IS NOT NULL`, `LIKE '%x%'`, `_` (single‑char wildcard).

## 17) Mini “Why/How” Answers (Fast Oral‑Exam Readiness)
- **Why DBMS over files?** Independence (PDIP), integrity rules, concurrency control, query optimization, recovery.
- **Why 1NF?** Simple domains guarantee predictable operators; no nested tables inside cells.
- **Why FK constraints?** To prevent orphan rows and keep cross‑table consistency.
- **Why window functions?** To calculate per‑row metrics (like “each user’s total posts”) without collapsing rows.
- **Why transactions?** Business actions need all‑or‑nothing behavior (transfer, booking).

## Coverage Audit (nothing lost)
This deep guide keeps every notion from (DB/DBMS/FMS/DMS/PDIP • OLTP/OLAP • levels • data vs metadata •
dictionary/catalog • IMS • data model & sublanguages • RA/TRC/DRC • relation/domains/degree/1NF • keys (superkey, PK, FK) •
associative relations • RA operators and joinability • natural vs cross join • outer joins • CTE • aggregates vs windows
(with DISTINCT note) • ranking • transactions + ACID • CAP • triggers with NEW/OLD • visual refs). Additions are minimal
and flagged as **(extra)** to help you reason in exam situations.

### Last tip
When reading an exam question, **circle the English cues** (e.g., “for each”, “even without”, “common to both”, “greater than all”),
then map them directly to the **Playbook** section above to choose operators/keywords.

> **Goal:** Everything below is rewritten in simple, exam-friendly English.  
> **Policy:** Only concepts present in are included; duplicates removed; wording clarified.  
> **Quick map of sections:** DBMS basics • Application types (OLTP/OLAP) • Data abstraction levels • Data/metadata •
> Data models & query languages • RA/Calculus • Relational model • Keys & integrity • Relational operations • Joins •
> Outer joins • CTE • Aggregation vs Window functions • Ranking • Transactions & ACID • CAP • Triggers • Visual refs •
> SQL keyword cheat‑sheet (with English cues).

## 1) DB, DBMS, Datastore, Performance
- **Database**: an organized collection of **related data**.
- **DBMS** (database management system): **software + operations** used to **store and manage** that data.
- **Datastore**: a broader word for any place data is kept (can be non‑relational or unstructured).
- **Goal of DBMS**: **store** data (via a database) and **manage** it (via a programming/query language).
- **Performance = Knowledge + Attitude**  
  - Knowledge → techniques/hard skills (SQL, design).  
  - Attitude → tactics/soft skills (logic, discipline). Together they determine effectiveness.

## 2) Types of DBMS Applications
- **OLTP (Online Transaction Processing)**: day‑to‑day operations. *Read + write, simple queries, very fast, many users at once (concurrency).*
- **OLAP (Online Analytical Processing)**: analysis/reporting. *Mostly read, complex queries, decision support.*
- **Data warehouse**: an **OLAP** system for historical data.
- **Metadata**: **data about data** (schemas, types, constraints). It helps us interpret stored values.
- **FMS (File Management System)**: a set of **files + programs** to handle those files.
- **DMS (Data Management System)**: either an **FMS** or a **DBMS**.
- **PDIP (Physical Data Independence Property)**: changes in **physical storage** **do not force** changes in programs that use the data. **Satisfied by DBMS**, **not** by FMS.

## 3) Levels of Data Abstraction
There are **three levels** where data is understood:
1. **Physical** — how data is stored on disk.
2. **Logical** — what data is stored and how pieces relate.
3. **View (external)** — how a **user** sees the data (different users can see different subsets).

**View** = differences in interpretation. **Metadata** enables correct understanding.  
**Data → Information → Knowledge** (DIKW pyramid):
- **Data**: raw facts (“70”).
- **Information**: data in context (“70°C of CPU”).
- **Knowledge**: understanding that guides action.

**Entity**: any “thing” represented (Student, Course).

## 4) What counts as “data” inside a DBMS?
Let a system be **S = (DB, Pr)** where **DB** = data store, **Pr** = programs.
- “**Data in S**” is **anything representable and storable** in S.
- Two groups inside S:  
  1) **Proper data** (actual records). *(Time‑dependent → changes.)*  
  2) **Metadata** (definitions/schema). *(Time‑independent → structure.)*
- **Data dictionary / system catalog**: part of S where metadata is kept.
- **Information about an entity in S** = data about that entity **paired with metadata** under the system’s data model.
- **IMS (Information Management System)**: stored information + programs to manage it. DBMSs are IMSs.

## 5) Data Models & Query Languages
**Data model** = concepts to describe:  
1) **Data**, 2) **Relationships**, 3) **Semantics** (meaning), 4) **Constraints** (rules).

**Query/programming languages in a DBMS** (sublanguages by purpose):  
- **DDL** — *Data Definition*: define schema/metadata (`CREATE`, `ALTER`, `DROP`).  
- **DML** — *Data Manipulation*: work with proper data (`SELECT`, `INSERT`, `UPDATE`, `DELETE`).  
- **TCL** — *Transaction Control*: `START TRANSACTION`, `COMMIT`, `ROLLBACK`, `SAVEPOINT`.  
- **DCL** — *Data Control*: permissions (`GRANT`, `REVOKE`).  
- **SDL** — *Storage Definition* (part of DDL): physical storage details for performance.

**Styles of query languages**:  
- **Procedural (imperative)** — say **how** to compute. *Relational Algebra (RA)*.  
- **Declarative (non‑procedural)** — say **what** you want. *SQL; Relational Calculus (TRC/DRC).*  
*Optimizers translate SQL into (possibly many) RA expressions and pick a good one.*

## 6) Relational Data Model
- **Relation** = **table** = a **set of n‑tuples**; the i‑th value of each tuple comes from domain **Sᵢ**.
- **Tuple** = row. **Attribute** = column. **Domain** = allowed values for a column.  
- **Degree** = number of attributes. **Cardinality** = number of rows.
- **Domain semantics** = what a domain’s values mean in the real world.
- **Simple/atomic domain** = indivisible value (e.g., integer).  
  **Non‑simple** = composite (e.g., address with parts).  
- **Normal form (here, 1NF idea)**: relation whose **domains are simple** (no nested relations).  
- **Superkey**: attribute(s) that uniquely identify rows.  
  **Primary key**: a *minimal* superkey (no extra attributes).  
  **Foreign key**: attribute(s) in one relation that point to a primary key in another.

**Associative relations**:  
- **Purely associative**: all domains simple → ordinary table.  
- **Associative** (relationship table): some domains are themselves relations → used to model many‑to‑many (junction table).

## 7) Keys & Integrity
- **Entity integrity**: every table has a valid **primary key** (no NULLs).
- **Referential integrity**: a **foreign key** must match an existing value in the referenced primary key (or be NULL if allowed).  
  *Foreign key mistakes:* wrong column name or value not present → error.

## 8) Relational Operations (Retrieval Only)
We consider retrieval operations (not `CREATE/INSERT/UPDATE/DELETE` here).

| Operation | Symbol | Effect (plain words) |
|---|---|---|
| **Selection** | σ | Keep rows that meet a condition. |
| **Projection** | π | Keep only certain columns. |
| **Cartesian product** | × | Pair every row of R with every row of S. *(No entity meaning; no PK context.)* |
| **Join (theta/natural)** | ⋈ | Combine rows from R and S that match on a rule or on same‑named attributes. |
| **Union** | ∪ | Rows from R or S (duplicates removed; schemas must match). |
| **Difference** | − | Rows in R that are not in S (schemas must match). |
| **Intersection** | ∩ | Rows common to both (schemas must match). |
| **Rename** | ρ | Change relation/attribute names (handy in multi‑table ops). |

**Joinability**: R and S are joinable if there is a relation U whose projections give back R and S.  
**Natural join**: join on attributes with the same names.  
**Cross join**: product without matching; keeps no relation meaning and loses PK/relationship context.

## 9) Outer Joins
- Like inner join **plus unmatched rows** filled with **NULLs**.
- Types: **LEFT**, **RIGHT**, **FULL**.  
  (Keyword `OUTER` is optional in many dialects.)

## 10) CTE (Common Table Expression)
Temporary, named query used **inside one statement**:
```sql
WITH name AS (SELECT ...)
SELECT ... FROM name;
```

## 11) Aggregation vs Window (Partition) Functions
- **Aggregates** (`COUNT`, `SUM`, `AVG`, `MIN`, `MAX`) **collapse rows into groups** using `GROUP BY`.
- **Window functions** compute **per row** over a **partition** using `OVER (PARTITION BY …)`. They **do not collapse** rows.
- **Grouping is a special case of partition.**
- **DISTINCT** inside aggregates/windows may slow queries (extra work).  
- `OVER` **without** `PARTITION BY` means: the **whole table** is the window.
- **Ranking**: `RANK()`, `DENSE_RANK()`, `ROW_NUMBER()` (use with `ORDER BY` in `OVER`).

## 12) Transactions (TCL) & ACID
- **Transaction**: one or more SQL statements treated as a **single unit of work** (according to business logic).  
- **ACID** properties:  
  - **Atomicity** — all or nothing.  
  - **Consistency** — rules are preserved.  
  - **Isolation** — no interference between transactions.  
  - **Durability** — committed results stay after crashes.
- **Atomic statements**: each **DDL/DML statement** is atomic by itself. A **sequence** becomes atomic **only** when wrapped in an explicit transaction.
- **Commands**: `START TRANSACTION`, `COMMIT`, `ROLLBACK`.

## 13) CAP Theorem (Distributed DBMS)
A distributed system can satisfy only **two** of these **three** at the same time:
- **Consistency** (every node sees the same data),
- **Availability** (every request gets a response),
- **Partition tolerance** (keeps working despite network breaks).

## 14) Triggers
- A **trigger** runs **automatically** when a row is inserted, updated, or deleted.
- **Timing**: `BEFORE` or `AFTER` the event.  
- **Inside triggers**:  
  - **NEW** → the incoming/current values,  
  - **OLD** → the previous values.
- Example skeleton (MySQL‑style):
```sql
DELIMITER $$
CREATE TRIGGER trig_name
BEFORE INSERT ON table_name
FOR EACH ROW
BEGIN
  -- SQL statements
END; $$
DELIMITER ;
```

## 15) Visual Reference Notes
- **FMS vs RDBMS**: files vs tables (RDBMS adds relations + SQL).  
- **Graph DB vs Relational**: nodes+edges vs tables. Use the model that **fits your data shape**.  
- **Views**: different users see different columns (e.g., student cannot see instructor salaries, instructor can).  
- **Query processing**: producer → processor (parse, optimize, execute).  
- **Relational diagrams**: selection/projection/join drawn as boxes and arrows help verify keys and joins.

## 16) SQL Keyword Cheat‑Sheet (with Plain Purposes & English Cues)

> **Use these to map exam English → SQL quickly.**

### 16.1 Core retrieval
| Keyword | What it does | English cues |
|---|---|---|
| `SELECT` | choose columns | list, display, show, find |
| `FROM` | choose tables | from, using data of |
| `WHERE` | filter rows | whose, with, that have, such that |
| `DISTINCT` | remove duplicates | unique, different, no repetition |
| `ORDER BY` | sort results | ascending/descending, top/best/highest/lowest, alphabetically |
| `LIMIT` / `FETCH FIRST` | restrict row count | top N, first N rows, only N results |
| `OFFSET` | skip rows | after the first N, page K |

### 16.2 Sets & subqueries
| Keyword | What it does | English cues |
|---|---|---|
| `UNION` / `UNION ALL` | combine results | combine, together |
| `INTERSECT` | common rows | in both, common to both |
| `EXCEPT` / `MINUS` | rows in A not in B | in A but not in B |
| `IN` / `NOT IN` | membership test | in the list, among, exclude these |
| `EXISTS` | check if any row found | if there exists, at least one |
| `ALL` | compare to every value | greater than all, less than all |
| `ANY` / `SOME` | compare to one value | greater than at least one |

### 16.3 Joins
| Keyword | What it does | English cues |
|---|---|---|
| `JOIN ... ON` | match with a condition | with their, paired with, matching |
| `NATURAL JOIN` | match on same‑named columns | same name columns, natural |
| `USING (col)` | match on listed same‑named columns | using column X |
| `LEFT JOIN` | keep all left rows | include even those without a match |
| `RIGHT JOIN` | keep all right rows | include every right‑side row |
| `FULL JOIN` | keep all rows of both sides | keep everything, even unmatched |
| `CROSS JOIN` | all combinations | combine every row with every row |

### 16.4 Grouping & windows
| Keyword / Func | What it does | English cues |
|---|---|---|
| `GROUP BY` | make groups | per department, for each user |
| `HAVING` | filter groups | groups having, departments with count > ... |
| `COUNT / SUM / AVG / MIN / MAX` | group totals | number of, total, average, highest, lowest |
| `OVER (PARTITION BY ...)` | window over groups | per X but keep all rows |
| `RANK / DENSE_RANK / ROW_NUMBER` | ranking | rank, enumerate, sequence |
| `CASE WHEN ... THEN ... END` | conditional value | if … then … else |
| `COALESCE(a, b)` | replace NULL with fallback | if missing then use, otherwise |

### 16.5 Data changes & schema
| Keyword | What it does | English cues |
|---|---|---|
| `INSERT INTO` | add rows | add, record, register |
| `UPDATE ... SET ... WHERE` | modify rows | change, increase, reduce, set |
| `DELETE FROM ... WHERE` | remove rows | delete, remove |
| `CREATE TABLE` | new table | design a table |
| `ALTER TABLE` | change structure | add column, change type |
| `DROP TABLE` | remove table | delete the table |
| `CREATE VIEW` | virtual table | show limited columns, hide sensitive data |
| `CREATE INDEX` | speed up lookups | optimize, faster search |

### 16.6 Constraints, permissions, transactions
| Keyword | What it does | English cues |
|---|---|---|
| `PRIMARY KEY` | unique identifier | unique id |
| `FOREIGN KEY ... REFERENCES` | relationship rule | belongs to, references |
| `CHECK` | allowed values rule | must be between, must satisfy |
| `DEFAULT` | auto value | by default |
| `GRANT / REVOKE` | permissions | allow, deny |
| `WITH` (CTE) | name a subquery | reuse the same result |
| `START TRANSACTION / COMMIT / ROLLBACK` | control a unit of work | treat steps as one, save, undo |

**Cautions**:  
- Natural/`USING` joins require same column names; mismatched names or missing referenced values cause errors.  
- **DISTINCT** can slow queries (extra de‑dup).  
- `OVER` without `PARTITION BY` = whole table window.  
- **Cross join** has no entity meaning; PK context is lost.

## 17) Final Coverage Checklist (no info lost)
- ✅ DB vs DBMS vs Datastore; goal; performance = knowledge + attitude.  
- ✅ OLTP vs OLAP; data warehouse; metadata; FMS vs DBMS; DMS; **PDIP (DBMS yes, FMS no)**.  
- ✅ Three abstraction levels; view meaning; DIKW; data vs information; entity.  
- ✅ S = (DB, Pr); “data in S” definition; proper data vs metadata; instance vs schema; **data dictionary/system catalog**; info = data + metadata; **IMS**.  
- ✅ Data model (data/relationships/semantics/constraints); QL sublanguages (**DDL/DML/TCL/DCL/SDL**); procedural vs declarative; SQL, RA, TRC/DRC.  
- ✅ Relational model: relation as set of tuples from domains; name; domain; **degree**; **semantics**; **atomic domain**; **1NF** link; **superkey**.  
- ✅ Keys: **primary key**, **foreign key**; referencing/referenced; **referential integrity**.  
- ✅ Associative & purely associative relations.  
- ✅ Retrieval operations: σ, π, ×, ⋈, ∪, −, ∩, ρ; **joinability**, **natural join**, **cross join** comment.  
- ✅ Outer joins (LEFT/RIGHT/FULL); note: `OUTER` optional.  
- ✅ **CTE** syntax and idea.  
- ✅ Aggregation vs partition; **DISTINCT slowdown**; **OVER without partition = whole table**; **RANK/DENSE_RANK/ROW_NUMBER**.  
- ✅ Transactions & **ACID**; **atomic statements vs sequence requiring transaction**; TCL commands.  
- ✅ **CAP theorem** (C, A, P — choose 2).  
- ✅ **Triggers**; BEFORE/AFTER; `NEW`/`OLD`; delimiter block.  
- ✅ Visual references: model fit; graph vs relational; FMS vs RDBMS; views (privacy); record type; query processing; relational diagrams.  
- ✅ SQL keyword cheat‑sheet with English cues (covers ~80% of practical exam needs).

### End of Guide
Focus on: joins, grouping vs window, FK/PK integrity, ACID/CAP, and translating problem English into SQL using the cheat‑sheet.

> Built from your Section 16. Each item below tells you **what it does**, **how to use it**, **pitfalls**, and gives **copy‑pasteable examples** using a small sample schema similar to your `User` / `Post` / `Reaction` setup.

## 0) Mini Demo Schema (so examples run anywhere)

```sql
-- Users who write posts and react to posts
CREATE TABLE "User" (
  userno       INTEGER PRIMARY KEY,
  username     VARCHAR(40) UNIQUE NOT NULL,
  joined_at    TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  status       VARCHAR(20) DEFAULT 'active'
);

CREATE TABLE "Post" (
  postno       INTEGER PRIMARY KEY,
  U_userno     INTEGER NOT NULL REFERENCES "User"(userno),
  contents     TEXT NOT NULL,
  reputation   INTEGER NOT NULL DEFAULT 0,
  created_at   TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE "Reaction" (
  reactionno   INTEGER PRIMARY KEY,
  U_userno     INTEGER NOT NULL REFERENCES "User"(userno),  -- who reacted
  P_postno     INTEGER NOT NULL REFERENCES "Post"(postno),  -- which post
  reaction_type VARCHAR(16) CHECK (reaction_type IN ('like','love','funny','sad','angry')),
  created_at   TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

-- Extra tables for joins/sets/windows demos
CREATE TABLE Department(
  dept_id   INTEGER PRIMARY KEY,
  dept_name VARCHAR(60) NOT NULL
);

CREATE TABLE Employee(
  emp_id    INTEGER PRIMARY KEY,
  full_name VARCHAR(80) NOT NULL,
  salary    NUMERIC(12,2) NOT NULL,
  dept_id   INTEGER REFERENCES Department(dept_id),
  hired_at  DATE NOT NULL
);
```

> Tip: Keep this schema handy; you can paste each snippet below and it will work in PostgreSQL (and nearly identical in other SQL dialects with tiny tweaks).

## 1) Core Retrieval

### `SELECT` — choose columns
**What:** Project columns; may compute expressions/aliases.  
**Pitfalls:** Unqualified `*` can hide bugs; prefer explicit column lists in exams.

```sql
-- List post ids and texts
SELECT p.postno, p.contents
FROM "Post" AS p;
```

### `FROM` — choose tables (and row sources like CTEs/subqueries)
**What:** Sets the row source(s) to read from.  
**Example:**

```sql
-- Pull all rows out of Post (explicit FROM even if trivial)
SELECT * FROM "Post";
```

### `WHERE` — filter rows
**What:** Row-level filtering **before** grouping.  
**Pitfalls:** `NULL` comparisons need `IS NULL` / `IS NOT NULL` (not `=`).

```sql
-- Only positive-reputation posts
SELECT postno, contents
FROM "Post"
WHERE reputation > 0;
```

### `DISTINCT` — remove duplicates
**What:** De-duplicates the selected columns.  
**Cost:** Extra sort/hash; use only if you truly need it.

```sql
-- All distinct reaction types present in the system
SELECT DISTINCT reaction_type
FROM "Reaction";
```

### `ORDER BY` — sort results
**What:** Sort output rows.  
**Pitfalls:** When mixing `NULL`s, define order with `NULLS FIRST/LAST` (PostgreSQL) or CASE.

```sql
SELECT postno, reputation
FROM "Post"
ORDER BY reputation DESC, postno ASC;
```

### `LIMIT` / `FETCH FIRST` and `OFFSET` — pagination
**What:** Return a subset of rows; `OFFSET` skips.  
**Pitfalls:** `OFFSET` alone is slow for big pages; keyset pagination is better.

```sql
-- Top 5 highest-reputation posts
SELECT postno, contents, reputation
FROM "Post"
ORDER BY reputation DESC
FETCH FIRST 5 ROWS ONLY;  -- ANSI; PostgreSQL also supports LIMIT 5

-- Page 2 (rows 11..20) with LIMIT/OFFSET
SELECT postno, contents
FROM "Post"
ORDER BY postno
LIMIT 10 OFFSET 10;
```

## 2) Sets & Subqueries

### `UNION` / `UNION ALL`
**What:** Stack results vertically. `UNION` removes dups; `UNION ALL` keeps all.  
**Example:**

```sql
-- People who posted OR reacted (UNION ALL keeps duplicates = activity weight)
SELECT U_userno AS userno FROM "Post"
UNION ALL
SELECT U_userno      FROM "Reaction";
```

### `INTERSECT`
**What:** Rows common to both inputs.  
```sql
-- Users who both posted AND reacted
SELECT U_userno AS userno FROM "Post"
INTERSECT
SELECT U_userno           FROM "Reaction";
```

### `EXCEPT` / `MINUS` (vendor name varies)
**What:** Rows in left query not present in right.  
```sql
-- Users who posted but NEVER reacted
SELECT U_userno AS userno FROM "Post"
EXCEPT
SELECT U_userno           FROM "Reaction";
```

### `IN` / `NOT IN`
**What:** Membership test against a list/subquery.  
**Pitfall:** `NOT IN (subquery)` with `NULL` inside subquery returns **no rows**; prefer `NOT EXISTS`.

```sql
-- Posts with reactions from specific users
SELECT p.postno, p.contents
FROM "Post" p
WHERE p.U_userno IN (SELECT u.userno FROM "User" u WHERE u.status='active');
```

### `EXISTS` / `NOT EXISTS`
**What:** Boolean test: “Does the subquery return any row?” Fast when correlated with good indexes.  
```sql
-- Posts that have at least one non-funny reaction
SELECT p.postno, p.contents
FROM "Post" p
WHERE EXISTS (
  SELECT 1
  FROM "Reaction" r
  WHERE r.P_postno = p.postno AND r.reaction_type <> 'funny'
);
```

### `ALL` / `ANY` (`SOME`)
**What:** Compare a scalar to **every** (`ALL`) or **any** (`ANY`) value from a set.  
```sql
-- Employees with salary >= ALL peers in their department (i.e., top earner in dept)
SELECT e.*
FROM Employee e
WHERE e.salary >= ALL (
  SELECT e2.salary FROM Employee e2 WHERE e2.dept_id = e.dept_id
);
```

## 3) Joins

### `JOIN ... ON` (inner join)
**What:** Combine rows that match a condition.  
```sql
SELECT p.postno, u.username, p.contents
FROM "Post" p
JOIN "User" u ON u.userno = p.U_userno;
```

### `LEFT/RIGHT/FULL OUTER JOIN`
**What:** Keep unmatched rows, filling the other side with `NULL`s.  
```sql
-- Users with their latest post (or NULL if none)
SELECT u.userno, u.username, p.postno, p.created_at
FROM "User" u
LEFT JOIN "Post" p
  ON p.U_userno = u.userno
  AND p.created_at = (
    SELECT MAX(p2.created_at)
    FROM "Post" p2
    WHERE p2.U_userno = u.userno
  );
```

### `NATURAL JOIN` / `USING (col)`
**What:** Auto-match by same-named columns.  
**Pitfall:** Renames break it; exams prefer explicit `ON` to avoid surprises.

```sql
-- Safer than NATURAL JOIN:
SELECT ...
FROM A JOIN B USING (common_id);
```

### `CROSS JOIN`
**What:** Cartesian product (all combinations). Rarely useful alone.  
```sql
-- Pair every department with every reaction type (for a reporting scaffold)
SELECT d.dept_name, rt.reaction_type
FROM Department d
CROSS JOIN (VALUES ('like'),('love'),('funny'),('sad'),('angry')) AS rt(reaction_type);
```

## 4) Grouping & Window Functions

### `GROUP BY` + aggregates
**What:** Collapse rows into groups.  
```sql
-- Number of reactions per post
SELECT r.P_postno AS postno, COUNT(*) AS reactions
FROM "Reaction" r
GROUP BY r.P_postno
HAVING COUNT(*) >= 3;  -- keep only popular posts
```

### Window functions: `OVER (PARTITION BY ...)`
**What:** Compute per-row analytics **without collapsing** rows.  
```sql
-- Rank employees by salary within each department
SELECT
  e.emp_id, e.full_name, e.dept_id, e.salary,
  RANK() OVER (PARTITION BY e.dept_id ORDER BY e.salary DESC) AS dept_salary_rank
FROM Employee e;
```

**Common window patterns**

```sql
-- Running total of reactions per user over time
SELECT
  r.U_userno,
  r.created_at,
  COUNT(*) OVER (PARTITION BY r.U_userno ORDER BY r.created_at
                 ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS running_reacts
FROM "Reaction" r;

-- Percent of department total
SELECT
  e.emp_id, e.dept_id, e.salary,
  e.salary * 1.0 / SUM(e.salary) OVER (PARTITION BY e.dept_id) AS pct_of_dept
FROM Employee e;
```

### `CASE WHEN ... THEN ... ELSE ... END`
**What:** Inline conditional. Good in `SELECT`, `ORDER BY`, and `GROUP BY` filters.

```sql
-- Treat 'love' as +2, 'like' as +1, 'funny' as +0.5, others 0
SELECT
  r.P_postno,
  SUM(CASE r.reaction_type
        WHEN 'love'  THEN 2
        WHEN 'like'  THEN 1
        WHEN 'funny' THEN 0.5
        ELSE 0
      END) AS sentiment_score
FROM "Reaction" r
GROUP BY r.P_postno
ORDER BY sentiment_score DESC;
```

### `COALESCE(a, b, ...)`
**What:** First non-NULL value.  
```sql
-- Show username or '(unknown)' if NULL
SELECT COALESCE(u.username, '(unknown)') AS name
FROM "User" u;
```

## 5) Data Changes & Schema

### `INSERT INTO`
```sql
INSERT INTO "User"(userno, username) VALUES (1001, 'alice');
```

### `UPDATE ... SET ... WHERE`
```sql
UPDATE "Post"
SET reputation = reputation + 1
WHERE postno = 42;
```

### `DELETE FROM ... WHERE`
```sql
DELETE FROM "Reaction"
WHERE reactionno = 777;
```

### `CREATE TABLE` (core columns + constraints)
```sql
CREATE TABLE Tag (
  tag_id   SERIAL PRIMARY KEY,  -- PostgreSQL; use IDENTITY in ANSI/SQL Server, AUTO_INCREMENT in MySQL
  tag_name VARCHAR(40) NOT NULL UNIQUE
);
```

### `ALTER TABLE`
```sql
ALTER TABLE "Post" ADD COLUMN title VARCHAR(120);
ALTER TABLE "Post" ALTER COLUMN contents SET NOT NULL;   -- PostgreSQL syntax
```

### `DROP TABLE`
```sql
DROP TABLE IF EXISTS Tag;
```

### `CREATE VIEW`
```sql
CREATE VIEW TopPosts AS
SELECT postno, contents, reputation
FROM "Post"
WHERE reputation >= 10;
```

### `CREATE INDEX`
```sql
-- Speed up search by user on posts
CREATE INDEX idx_post_user ON "Post"(U_userno);
```

## 6) Constraints, Permissions, Transactions

### `PRIMARY KEY`, `FOREIGN KEY`, `CHECK`, `DEFAULT`
```sql
CREATE TABLE Score (
  userno     INTEGER PRIMARY KEY REFERENCES "User"(userno),
  exam_score NUMERIC(5,2) CHECK (exam_score BETWEEN 0 AND 100) DEFAULT 0
);
```

### `GRANT` / `REVOKE` (dialect differences exist)
```sql
GRANT SELECT ON "Post" TO analyst_role;
REVOKE INSERT ON "Post" FROM analyst_role;
```

### Transactions: `START TRANSACTION` / `COMMIT` / `ROLLBACK`
```sql
START TRANSACTION;
  UPDATE "User" SET status = 'inactive' WHERE userno = 1001;
  DELETE FROM "Post" WHERE U_userno = 1001;
COMMIT;  -- or ROLLBACK;
```

## 7) CTE (Common Table Expressions)

### Basic CTE
```sql
WITH popular AS (
  SELECT P_postno, COUNT(*) AS c
  FROM "Reaction"
  GROUP BY P_postno
  HAVING COUNT(*) >= 3
)
SELECT p.postno, p.contents, pop.c
FROM "Post" p
JOIN popular pop ON pop.P_postno = p.postno
ORDER BY pop.c DESC;
```

### Recursive CTE (hierarchies)
```sql
-- Example employee hierarchy (manager_id refers to another emp_id)
WITH RECURSIVE org AS (
  SELECT emp_id, full_name, dept_id, 0 AS depth
  FROM Employee
  WHERE emp_id = 1  -- root (e.g., CEO)

  UNION ALL

  SELECT e.emp_id, e.full_name, e.dept_id, o.depth + 1
  FROM Employee e
  JOIN org o ON e.dept_id = o.dept_id  -- placeholder relationship
)
SELECT * FROM org ORDER BY depth, emp_id;
```

## 8) **Data Types** (Differences & When to Use)
> Aim: cover ~80% of what you’ll meet on exams/interviews. Examples mention PostgreSQL, MySQL, SQL Server names.

### 8.1 Numeric Types
| Concept | ANSI / PostgreSQL | MySQL | SQL Server | Use When | Notes |
|---|---|---:|---:|---|---|
| Exact integer | `SMALLINT`, `INTEGER`, `BIGINT` | same | `SMALLINT`, `INT`, `BIGINT` | counts, ids | Range differs by size; use smallest safe size. |
| Exact decimal | `DECIMAL(p,s)` / `NUMERIC(p,s)` | same | `DECIMAL(p,s)`/`NUMERIC` | money, precise ratios | No rounding error; slower than float for heavy math. |
| Approx float | `REAL` (float4), `DOUBLE PRECISION` (float8) | `FLOAT`, `DOUBLE` | `FLOAT` | scientific, analytics | Rounding error; don’t use for money. |

**DECIMAL vs FLOAT:** DECIMAL keeps exact digits, FLOAT trades precision for speed/range.

### 8.2 Character Types
| Type | Meaning | Use When | Notes |
|---|---|---|---|
| `CHAR(n)` | Fixed-length | Codes of fixed size (ISO country code) | Pads with spaces; wastes space if variable. |
| `VARCHAR(n)` | Variable-length, with max | Most text columns | Good default. Set realistic `n`. |
| `TEXT` | Unbounded (vendor) | Large free text | No `n`; in some DBs fewer index options. |

**`CHAR` vs `VARCHAR` vs `TEXT`:** Prefer `VARCHAR(n)` unless you truly need fixed-size semantics. For very large text, use `TEXT`/CLOB.

### 8.3 Dates & Times
| Type | PostgreSQL | MySQL | SQL Server | Use When | Notes |
|---|---|---|---|---|---|
| Date only | `DATE` | `DATE` | `DATE` | birthdays, calendar dates | |
| Time only | `TIME [WITHOUT TIME ZONE]` | `TIME` | `TIME` | local times | Beware time zones. |
| Timestamp | `TIMESTAMP [WITHOUT TZ]` | `DATETIME` | `DATETIME2` | event moments | |
| Zoned timestamp | `TIMESTAMPTZ` | (none; use `TIMESTAMP` + app TZ) | `DATETIMEOFFSET` | auditing, multi‑TZ apps | Store in UTC; show in user TZ. |

**Tip:** Always store in UTC (`TIMESTAMPTZ`/`DATETIMEOFFSET`) and convert at presentation.

### 8.4 Boolean
| Type | PostgreSQL `BOOLEAN` (`TRUE/FALSE`) | MySQL `BOOLEAN` = `TINYINT(1)` | SQL Server `BIT` |
|---|---|---|---|
| Use When | feature flags, simple yes/no |
| Notes | Be careful when migrating across vendors due to MySQL semantics. |

### 8.5 Binary / Large Objects
| Type | PostgreSQL | MySQL | SQL Server | Use When |
|---|---|---|---|---|
| Binary | `BYTEA` | `VARBINARY(n)` | `VARBINARY(n)` | small binary blobs |
| Large | `OID`/`BYTEA` | `BLOB`, `LONGBLOB` | `VARBINARY(MAX)` | files/blobs (or better: store in object storage + URL) |

### 8.6 Identifiers / Special
| Type | PostgreSQL | MySQL | SQL Server | Use When | Notes |
|---|---|---|---|---|---|
| UUID | `UUID` | `CHAR(36)`/`BINARY(16)` | `UNIQUEIDENTIFIER` | distributed ids | Random UUIDs are not index-friendly; use v7/time-ordered when possible. |
| SERIAL / Identity | `SERIAL`/`BIGSERIAL` (legacy), `GENERATED ALWAYS AS IDENTITY` | `AUTO_INCREMENT` | `IDENTITY` | autoincrement PKs | Prefer standard `IDENTITY` in new schemas. |

### 8.7 JSON & Arrays & Enums
| Type | PostgreSQL | MySQL | SQL Server | Use When | Notes |
|---|---|---|---|---|---|
| JSON | `JSON`/`JSONB` | `JSON` | `NVARCHAR` + JSON funcs | semi‑structured | In PG, `JSONB` is binary with indexes (`GIN`)—usually best. |
| Array | `type[]` | (no real arrays) | (no real arrays) | few small lists | Arrays can complicate normalization; use junction tables if many-to-many. |
| Enum | `ENUM` | `ENUM` | use CHECK/domain | fixed set of values | `CHECK` constraints + reference tables are more portable. |

### 8.8 Money/Currency
| Type | PostgreSQL `MONEY` (locale-aware) or `DECIMAL(19,4)` | MySQL `DECIMAL(19,4)` | SQL Server `MONEY`/`DECIMAL` |
|---|---|---|
| Prefer `DECIMAL(19,4)` for portability and avoid float rounding. |

### 8.9 Geospatial (FYI)
| PostgreSQL + PostGIS | MySQL | SQL Server | Use |
|---|---|---|---|
| `GEOMETRY`, `GEOGRAPHY` | `GEOMETRY` | `GEOMETRY`/`GEOGRAPHY` | maps, coordinates |

> **Exam tip:** Know **why** you choose a type: *precision* (DECIMAL), *range* (BIGINT), *exact text length* (CHAR), *very large text* (TEXT), *time zone safety* (TIMESTAMPTZ).

## 9) Common “English → SQL” Patterns (quick macros)

- “**List users who never reacted**” → **`NOT EXISTS` anti-join**:

```sql
SELECT u.userno, u.username
FROM "User" u
WHERE NOT EXISTS (
  SELECT 1 FROM "Reaction" r WHERE r.U_userno = u.userno
);
```

- “**Top 3 posts per user**” → **window + QUALIFY (or subquery)**:

```sql
-- PostgreSQL (subquery filter on row_number)
SELECT * FROM (
  SELECT p.*,
         ROW_NUMBER() OVER (PARTITION BY p.U_userno ORDER BY p.reputation DESC) AS rn
  FROM "Post" p
) t
WHERE t.rn <= 3;
```

- “**Users with > 5 reactions in the last week**” → **GROUP BY + HAVING**:

```sql
SELECT r.U_userno, COUNT(*) AS c
FROM "Reaction" r
WHERE r.created_at >= CURRENT_DATE - INTERVAL '7 days'
GROUP BY r.U_userno
HAVING COUNT(*) > 5;
```

- “**Sort by status but treat 'active' first**” → **ORDER BY with CASE**:

```sql
SELECT u.*
FROM "User" u
ORDER BY
  CASE u.status WHEN 'active' THEN 0 WHEN 'pending' THEN 1 ELSE 2 END,
  u.username;
```

## 10) Nulls & Three-Valued Logic (frequent exam trap)

- `= NULL` is **always UNKNOWN** → use `IS NULL` / `IS NOT NULL`.
- `NOT IN (subquery)` returns **no rows** if subquery yields any `NULL`. Prefer `NOT EXISTS`.
- Aggregates ignore `NULL` except `COUNT(*)` which counts rows, `COUNT(col)` counts non‑NULLs.

```sql
-- Safe anti-join
SELECT p.*
FROM "Post" p
WHERE NOT EXISTS (
  SELECT 1 FROM "Reaction" r
  WHERE r.P_postno = p.postno AND r.reaction_type = 'funny'
);
```

## 11) Index Hints for Your Schemas (one-liners)

- Foreign keys → index child columns: `CREATE INDEX ON "Post"(U_userno);` `CREATE INDEX ON "Reaction"(P_postno);`
- Common filters → index those columns (e.g., `reaction_type`, `created_at` if queried by range).
- For text search → use full-text or trigram indices (`GIN`/`GiST` on PostgreSQL).

## 12) Tiny “What would this do?” — exam-style micro Q&A

1) **`SELECT DISTINCT reaction_type FROM "Reaction";`** → list each reaction kind once.  
2) **`COUNT(*) OVER (PARTITION BY U_userno)`** → reactions **per user on each row**, no collapse.  
3) **`LEFT JOIN` vs `JOIN`** → keeps left rows even without matches.  
4) **`DECIMAL(10,2)` vs `FLOAT`** → exact money vs approximate science.  
5) **`NOT IN` vs `NOT EXISTS`** → `NOT EXISTS` is NULL‑safe.

### End, Star my repo please
