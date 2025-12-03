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
- **Notation:** `\ro_{S(C, D)}{R(A, B)}`
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
\ro_{NewName(newA, newB)}{OldName(A, B)}
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
- Master the notation — most errors come from syntax.
- Always assign subqueries.
- Always use γ for aggregates.
- Use quantifiers correctly for ALL/SOME.
- Practice translating SQL to ERA until it becomes automatic.
- Memorize renaming syntax — it is the most commonly graded detail.

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
| Renaming | \ro |
| Aggregates | γ with functions |

---

You now have all theoretical content and rules required to score **100%** on a quiz about RA and ERA.

