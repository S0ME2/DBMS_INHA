# Relational Algebra (RA) and Extended Relational Algebra (ERA) — Complete Study Guide

This document prepares you for a quiz on **Relational Algebra (RA)** and **Extended Relational Algebra (ERA)**. It covers all core operations, notation, examples, SQL-to-RA/ERA translation strategies, and common pitfalls.

---

## 1. Foundations of Relational Algebra

### 1.1 What Is Relational Algebra?
Relational Algebra is a **formal, procedural query language** used to manipulate and retrieve data stored in relational databases. Each operation takes one or more **relations** (tables) as input and produces another relation.

---

## 2. Basic RA Operations

### 2.1 Selection (σ)
Filters rows based on a condition.
- **Notation:** `σ_condition(R)`
- **Example:** `σ(A > 5 ∧ B = 'X')(R)`

### 2.2 Projection (π)
Selects specific columns.
- **Notation:** `π(A, B)(R)`
- **Example:** Keep only A and B from relation R.

### 2.3 Union (∪)
Combines tuples from two relations with identical schemas.
- **Notation:** `R ∪ S`

### 2.4 Set Difference (−)
Keeps tuples from the first relation that are **not** in the second.
- **Notation:** `R − S`

### 2.5 Cartesian Product (×)
Returns all combinations of tuples from two relations.
- **Notation:** `R × S`
- Leads to attribute-name conflicts → often followed by renaming.

### 2.6 Rename (ρ or \ro)
Used in ERA to rename relations or attributes.
- **Notation:** `ρ_{S(C, D)}{R(A, B)}`
- Renames relation **R(A, B)** to **S(C, D)**.

---

## 3. Additional RA Operations (Derived)

### 3.1 Intersection
Defined using difference:
- `R ∩ S = R − (R − S)`

### 3.2 Natural Join (⋈)
Matches tuples with equal values for shared attribute names.
- **Notation:** `R ⋈ S`

### 3.3 Theta Join (⋈_condition)
Join with any general condition.
- **Notation:** `R ⋈_{A > B}(S)`

### 3.4 Division (÷)
Used for "for all" queries.
- Example use: find students who took **all** courses.

### 3.5 Joins: USING(), Natural Join, and Outer Joins
This section clarifies **joins in the style of SQL USING()** and their representation in relational algebra.

SQL's `USING()` clause corresponds to **natural joins** or joins where the join attributes have the same name in both relations.

---

## Natural Join (⋈)
A natural join automatically combines tuples from R and S that share **attributes with the same name**.

- **Notation:** `R ⋈ S`
- **Equivalent to:**
  1. A theta join on all attributes with matching names.
  2. Followed by projection to remove duplicate attributes.

---

## USING() Joins
SQL:
```
SELECT *
FROM R JOIN S USING (C);
```
ERA equivalent:
```
R ⋈ S
```
(as long as both relations contain attribute `C` with the same name)

If attributes differ in name, a **theta join** followed by projection is used:
```
R ⋈_{R.C = S.D} S
```
Then project to remove duplicates if needed.

---

## Outer Joins
These extend natural or theta joins by preserving unmatched tuples.

### Left Outer Join (⟕)
- Returns all tuples from **R**, matching tuples from **S**, and NULLs for S attributes if there is no match.
- **Notation:** `R ⟕ S`

### Right Outer Join (⟖)
- Returns all tuples from **S**, matching tuples from **R**, and NULLs for R attributes if there is no match.
- **Notation:** `R ⟖ S`

### Full Outer Join (⟗)
- Returns all tuples from both R and S; unmatched sides padded with NULLs.
- **Notation:** `R ⟗ S`

### USING() with Outer Joins
SQL:
```
SELECT *
FROM R LEFT JOIN S USING (C);
```
ERA:
```
R ⟕ S
```
(as long as both have attribute C)

For differently named attributes:
```
R ⟕_{R.C = S.D} S
```

---


## 4. Extended Relational Algebra (ERA)
ERA is more expressive, enabling:
- **Subqueries**
- **Quantifiers (∀, ∃)**
- **Aggregates (COUNT, AVG, etc.)**
- **Grouping (γ)**
- **More advanced renaming**

---

## 5. Subqueries in ERA

### 5.1 Using ← for Subquery Assignment
You **must** assign subquery results to a variable.

Example:
```
S ← (σ(D = 'X')(S))
π(A)(σ(B ∈ S)(R))
```

### 5.2 Membership Operators
- `∈` corresponds to SQL `IN`
- `/∈` corresponds to SQL `NOT IN`

---

## 6. Quantifiers in ERA

### 6.1 Existential Quantifier (∃)
Means "there exists at least one".
- **Example:** ∃x ∈ S (A = x)

### 6.2 Universal Quantifier (∀)
Means "for all".
- SQL's `ALL` keyword.
- **Example:** A < ALL(SELECT C FROM S) → `∀x ∈ S (A < x)`

### 6.3 Using Quantifiers for Joins or Conditions
Example:
```
σ(∀x ∈ S (A < x))(R)
```

---

## 7. Aggregation and Grouping (γ)

### 7.1 Syntax
```
γ_{grouping attributes; aggregate functions}(Relation)
```

### 7.2 Allowed Aggregate Functions
- `count-distinct(X)`
- `count(x)`
- `sum(X)`
- `avg(X)`
- `min(X)`
- `max(X)`

### 7.3 Example
SQL:
```
SELECT A, COUNT(DISTINCT B)
FROM R
WHERE C > 10
GROUP BY A;
```
ERA:
```
γ_{A; count-distinct(B)}(σ(C > 10)(R))
```

---

## 8. SQL → ERA Translation Rules

### 8.1 WHERE Clause
Translate boolean expressions directly to ERA conditions using:
- ∧ (AND)
- ∨ (OR)
- ¬ (NOT)

### 8.2 IN / NOT IN
Always use a subquery assignment.

Example:
SQL:
```
SELECT A
FROM R
WHERE A > 5 AND B IN (SELECT C FROM S WHERE D = 'X');
```
ERA:
```
S1 ← (σ(D = 'X')(S))
π(A)(σ(A > 5 ∧ B ∈ S1)(R))
```

### 8.3 EXISTS / NOT EXISTS
Translate using ∃.

Example:
- `WHERE EXISTS (SELECT ...)` → `∃x ∈ Subquery(...)`
- `WHERE NOT EXISTS ...` → `¬∃x ...`

### 8.4 ALL / SOME
SQL:
- `A < ALL (SELECT ...)` → `∀x (A < x)`
- `A = SOME (...)` → `∃x (A = x)`

### 8.5 NULL Handling
- `IS NULL` → `= NULL`
- `IS NOT NULL` → `≠ NULL`

---

## 9. Common Mistakes Students Make

### 9.1 Forgetting the Subquery Assignment (←)
❌ `π(A)(σ(B ∈ (σ(D='X')(S)))(R))`  
✔️ Must assign: `S1 ← (σ(D='X')(S))`

### 9.2 Using SQL Instead of Relational Algebra
RA/ERA never uses SQL keywords like `WHERE`, `FROM`, `SELECT`.

### 9.3 Incorrect Renaming
Do **not** rename like SQL.
Use:
```
ρ_{NewName(newA, newB)}{OldName(A, B)}
```

### 9.4 Aggregation Without γ
If grouping is involved, γ must be used.

### 9.5 Missing Parentheses
Always write:
```
σ(condition)(R)
```
Not:
```
σ condition R
```

---

## 10. Practice Problems

### Problem 1
SQL:
```
SELECT DISTINCT A
FROM R
WHERE B NOT IN (SELECT C FROM S WHERE D > 10);
```

### Problem 2
SQL:
```
SELECT A, AVG(B)
FROM R
GROUP BY A;
```

### Problem 3
Find all tuples from R that have a matching tuple in S with equal C values.

### Problem 4
Find all rows in R where A is greater than **every** C in S.

---

## 11. Tips for Getting 100% on the Quiz
- Master the notation - most errors come from syntax.
- Always assign subqueries.
- Always use γ for aggregates.
- Use quantifiers correctly for ALL/SOME.
- Practice translating SQL to ERA until it becomes automatic.
- Memorize renaming syntax - it is the most commonly graded detail.

---

## 12. Summary Table

| SQL Feature | ERA Equivalent |
|-------------|----------------|
| WHERE | σ |
| SELECT columns | π |
| GROUP BY | γ |
| IN | ∈ |
| NOT IN | /∈ |
| ALL | ∀ |
| SOME | ∃ |
| EXISTS | ∃ |
| Renaming | ρ |
| Aggregates | γ with functions |

---

## 13. Technical Note --- Simplified, Professor-Level Explanation

This section explains **how to translate certain SQL constructs into
Relational Algebra (RA) / Extended Relational Algebra (ERA)**.\
The official version is very technical; here you get a **clear,
intuitive, example-based** explanation of every symbol and rule.

------------------------------------------------------------------------

## 13.1 Overview

When translating SQL queries into RA/ERA:

-   Subqueries should be given a **variable name** using the assignment
    operator `←`.
-   SQL keywords like **IN, EXISTS, ALL, SOME**, and Boolean operators
    must be translated into **logical (mathematical) notation**.
-   Aggregate functions (COUNT, AVG, etc.) translate mostly **as they
    are**, with a special note about DISTINCT.

We now explain each case with simple examples.

------------------------------------------------------------------------

## 13.2 Translating `IN` and `NOT IN`

### Rule

If SQL has:

``` sql
    IN (subquery)
```

translate it using the **set membership** symbol:

        ∈ S

If SQL has:

``` sql
    NOT IN (subquery)
```

translate it as:

        /∈ S

### What is **S**?

-   **S** is the variable name you assign to the result of the subquery.
-   You define it using:

```{=html}
    S ← (translation of subquery)
```

### Example

**SQL**

``` sql
SELECT name
FROM Students
WHERE sid IN (SELECT sid FROM HonorRoll);
```

**RA**

    S ← π sid (HonorRoll)
    π name (Students ⋈_{Students.sid ∈ S} S)

Here, - `S` is the set of all sids from HonorRoll. - `Students.sid ∈ S`
means: keep students whose sid belongs to S.

------------------------------------------------------------------------

## 13.3 Translating `EXISTS` and `NOT EXISTS`

### Rule

If SQL has:

``` sql
EXISTS (Q)
```

translate it as:

    ∃x Q(x)

If SQL has:

``` sql
NOT EXISTS (Q)
```

translate it as:

    ¬∃x Q(x)

### What is **Q**? What is **x**?

-   **Q** is the **relation produced by the subquery**.
-   We assign it like:

```{=html}
    Q ← (translation of subquery)
```

-   **x** represents a **tuple** (a row) from Q. It does *not* matter
    what letter is used (x, t, r... all fine).

### Example

**SQL**

``` sql
SELECT name
FROM Students S
WHERE EXISTS (SELECT *
              FROM Enroll E
              WHERE E.sid = S.sid);
```

**RA**

    Q ← σ_{E.sid = S.sid} (Enroll)
    ∃x Q(x)
    π name (Students ⋈ Q)

------------------------------------------------------------------------

## 13.4 Translating `ALL` and `SOME`

These compare a value to **every** or **at least one** value from a
subquery result.

### Rule Summary

  SQL form         RA/Logic translation
  ---------------- ----------------------
  `A < ALL (S)`    `∀x ∈ S (A < x)`
  `A < SOME (S)`   `∃x ∈ S (A < x)`

### What is **x**? What is **S**?

-   **S** is the relation representing the subquery.
-   **x** is a **tuple** from S.
-   `x` may represent a single attribute if S projects only one column.

### Example

**SQL**

``` sql
SELECT name
FROM Students
WHERE age < ALL (SELECT age FROM Professors);
```

**RA**

    S ← π age (Professors)
    σ_{∀x ∈ S (Students.age < x)} (Students)
    π name (...)

------------------------------------------------------------------------

## 13.5 Boolean Operators

Translate:

  SQL   RA/Logic
  ----- ----------
  AND   `∧`
  OR    `∨`
  NOT   `¬`

Example:

SQL:

``` sql
WHERE major = 'CS' AND age > 20
```

RA:

    σ_{major='CS' ∧ age>20} (...)

------------------------------------------------------------------------

## 13.6 `IS NULL` and `IS NOT NULL`

Translate:

  SQL           RA
  ------------- ----------
  IS NULL       `= NULL`
  IS NOT NULL   `≠ NULL`

------------------------------------------------------------------------

## 13.7 SELECT vs SELECT DISTINCT

For translation purposes, treat:

    SELECT

and

    SELECT DISTINCT

as equivalent. RA's projection `π` is defined as removing duplicates
already.

------------------------------------------------------------------------

## 13.8 Aggregate Functions

-   `COUNT(DISTINCT A)` → `count-distinct(A)`
-   `COUNT(A)` → `count(A)`
-   Others (`MIN`, `MAX`, `AVG`, `SUM`) translate directly as in SQL.

You now have all theoretical content and rules required to score **100%** on a quiz about RA and ERA. At least I hope so.

