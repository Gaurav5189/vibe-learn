# 📚 Database Management Systems — Unit 2
### Complete Exam Preparation Notes

---

## Table of Contents
1. [The Relational Data Model](#1-the-relational-data-model)
2. [Relational Constraints](#2-relational-constraints)
3. [Relational Algebra](#3-relational-algebra)
4. [Relational Calculus](#4-relational-calculus)
5. [Domain Relational Calculus (DRC)](#5-domain-relational-calculus-drc)
6. [ER and EER to Relational Mapping](#6-er-and-eer-to-relational-mapping)
7. [Database Design Phases](#7-database-design-phases)
8. [Pitfalls in Database Design](#8-pitfalls-in-database-design)
9. [Functional Dependency](#9-functional-dependency)
10. [Armstrong's Axioms](#10-armstrongs-axioms)
11. [Closure, Cover, and Minimal Cover](#11-closure-cover-and-minimal-cover)
12. [Normalization — 1NF, 2NF, 3NF](#12-normalization--1nf-2nf-3nf)
13. [BCNF (Boyce-Codd Normal Form)](#13-bcnf-boyce-codd-normal-form)
14. [4NF and 5NF](#14-4nf-and-5nf)
15. [Lossless Join Decomposition](#15-lossless-join-decomposition)
16. [Dependency Preservation](#16-dependency-preservation)

---

## 1. The Relational Data Model

### Formal Definitions

The **Relational Model** was proposed by **E.F. Codd (1970)**. It represents data as a collection of **relations** (tables), based on **set theory** and **first-order predicate logic**.

| Term | Formal Name | Informal Name |
|---|---|---|
| Relation | Relation | Table |
| Tuple | Tuple | Row / Record |
| Attribute | Attribute | Column / Field |
| Domain | Domain | Set of valid values |
| Degree | Degree | Number of columns |
| Cardinality | Cardinality | Number of rows |

### Domain
A **domain** is the set of **atomic (indivisible) values** permitted for an attribute.

- *Example:* Domain of `Age` = integers from 1 to 120
- *Example:* Domain of `Gender` = {'M', 'F', 'O'}
- *Example:* Domain of `PhoneNumber` = 10-digit numeric strings

### Relation Schema vs Relation Instance
- **Relation Schema** `R(A1, A2, ..., An)` — the blueprint/structure of a table.
  - *Example:* `Employee(EmpID, Name, Salary, DeptID)`
- **Relation Instance** — the actual set of tuples (rows) stored at a given time.
  - The instance changes as data is inserted/deleted/updated.
  - The schema is relatively stable.

### Properties of a Relation
1. **No duplicate tuples** — Every row must be unique (ensured by primary key).
2. **Tuples are unordered** — No inherent row ordering; order of rows doesn't matter.
3. **Attributes are unordered** — Column order in a schema doesn't matter logically.
4. **All attribute values are atomic** — No multi-valued or composite values in a single cell. This is the **first normal form (1NF)** condition.
5. **Each attribute has a distinct name.**
6. **NULL values represent unknown or missing data** — NULL ≠ 0 ≠ empty string.

### NULL in Relational Model
- NULL means: value is **unknown**, **not applicable**, or **missing**.
- Any arithmetic operation with NULL = NULL: `5 + NULL = NULL`
- Any comparison with NULL = **UNKNOWN** (not TRUE or FALSE): `NULL = NULL` is UNKNOWN.
- This leads to **three-valued logic**: TRUE, FALSE, UNKNOWN.

---

## 2. Relational Constraints

Constraints are rules enforced on relations to ensure **data integrity** — correctness and consistency of data.

### 2.1 Domain Constraints
Every attribute value must belong to its **defined domain** and be **atomic**.

- *Example:* `Age INT CHECK (Age > 0 AND Age < 150)` — age must be a positive integer.
- *Example:* `Gender CHAR(1) CHECK (Gender IN ('M','F','O'))` — only these three values allowed.

### 2.2 Key Constraints

**Super Key:** Any set of attributes that uniquely identifies each tuple.

**Candidate Key:** Minimal super key — no proper subset is also a super key.

**Primary Key:** The chosen candidate key. Must be:
- **Unique** — no two tuples have the same PK value.
- **Not NULL** — PK attributes cannot be NULL (**Entity Integrity**).

*Example:* In `Employee(EmpID, Name, Email, DeptID)`:
- Super keys: `{EmpID}`, `{Email}`, `{EmpID, Name}`, `{EmpID, Email}`, ...
- Candidate keys: `{EmpID}`, `{Email}`
- Primary key: `{EmpID}` (chosen)
- Alternate key: `{Email}`

### 2.3 Entity Integrity Constraint
**Primary key attributes cannot be NULL.**

Rationale: If PK is NULL, we can't uniquely identify a tuple — it defeats the purpose of the PK.

*Example:* `INSERT INTO Employee VALUES (NULL, 'Arjun', 50000, 2)` — REJECTED if EmpID is PK.

### 2.4 Referential Integrity Constraint
A **foreign key** value in one relation must either:
- Match a **primary key value** in the referenced relation, OR
- Be **NULL** (if the FK is not part of a PK).

*Example:*
```
Employee(EmpID, Name, DeptID)    DeptID is FK → Department(DeptID)
Department(DeptID, DeptName)
```
- If `Employee.DeptID = 5`, then `DeptID = 5` MUST exist in `Department`.
- Inserting an employee with `DeptID = 99` (which doesn't exist) is REJECTED.
- Deleting `Department` where `DeptID = 5` is REJECTED if employees reference it (unless CASCADE/SET NULL).

**Referencing relation** = Employee (has the FK)
**Referenced relation** = Department (has the PK)

### 2.5 Semantic (General) Integrity Constraints
Business rules that go beyond structure.

*Examples:*
- An employee's salary must be greater than their manager's salary minus 20%.
- A student cannot enroll in more than 6 courses per semester.
- These are enforced via `CHECK`, `ASSERTION`, or `TRIGGER`.

### Summary Table

| Constraint | What It Enforces | Violation Example |
|---|---|---|
| Domain | Values must be from valid domain | Inserting 'abc' into an INT column |
| Key | No duplicate tuples | Two employees with same EmpID |
| Entity Integrity | PK not NULL | EmpID = NULL |
| Referential Integrity | FK references valid PK | DeptID = 99 doesn't exist in Dept |

---

## 3. Relational Algebra

**Relational Algebra** is a **procedural query language** — you specify **what data to retrieve AND how** (step-by-step operations on relations). Each operation takes one or two relations as input and produces a new relation as output. Basis for SQL optimization.

**Fundamental Operations:**
1. SELECT (σ)
2. PROJECT (π)
3. RENAME (ρ)
4. UNION (∪)
5. SET DIFFERENCE (−)
6. CARTESIAN PRODUCT (×)

**Derived Operations (can be expressed using fundamental ones):**
- INTERSECTION (∩)
- JOIN (⋈)
- DIVISION (÷)

---

### Tables Used in Examples Throughout Section 3:

```
Employee (EmpID, Name, Salary, DeptID, City)
101  Ananya   55000  1  Bangalore
102  Rohan    42000  2  Delhi
103  Priya    70000  1  Bangalore
104  Kartik   38000  3  Mumbai
105  Sneha    85000  2  Delhi

Department (DeptID, DeptName, Location)
1  Research   Bangalore
2  Sales      Delhi
3  HR         Mumbai
4  Finance    Chennai

Works_On (EmpID, ProjectID, Hours)
101  P1  40
101  P2  20
103  P1  30
105  P3  50

Project (ProjectID, ProjName, Budget)
P1  Apollo   500000
P2  Zeus     300000
P3  Hermes   700000
```

---

### 3.1 SELECT — σ (Sigma)
Filters **rows** (tuples) satisfying a condition. Like SQL WHERE.

**Notation:** σ_condition(Relation)

```
σ_Salary > 50000 (Employee)
→ Returns: rows where salary > 50000

Result:
101  Ananya   55000  1  Bangalore
103  Priya    70000  1  Bangalore
105  Sneha    85000  2  Delhi
```

```
σ_DeptID = 1 AND City = 'Bangalore' (Employee)
→ Returns: employees in dept 1 who are in Bangalore

Result:
101  Ananya   55000  1  Bangalore
103  Priya    70000  1  Bangalore
```

- SELECT is **commutative:** σ_c1(σ_c2(R)) = σ_c2(σ_c1(R))
- SELECT can be **combined:** σ_c1 AND c2(R) = σ_c1(σ_c2(R))

---

### 3.2 PROJECT — π (Pi)
Selects specific **columns** (attributes). Like SQL SELECT column_list. Automatically removes duplicates.

**Notation:** π_attribute_list(Relation)

```
π_Name, Salary (Employee)
→ Returns only Name and Salary columns

Result:
Ananya   55000
Rohan    42000
Priya    70000
Kartik   38000
Sneha    85000
```

```
π_DeptID (Employee)
→ Returns unique DeptIDs (duplicates removed!)

Result:
1
2
3
```

- PROJECT is **not commutative** in general.
- `π_A(π_AB(R)) = π_A(R)` — outer projection subsumes inner.

---

### 3.3 RENAME — ρ (Rho)
Renames a relation or its attributes.

**Notation:** ρ_new_name(Relation)  or  ρ_new_name(A1, A2, ...)(Relation)

```
ρ_Emp(Employee)                        -- Renames relation to Emp
ρ_Emp(ID, FullName, Pay, Dept, Town)(Employee)  -- Renames all attributes
```

Useful in **self-joins** to differentiate the two copies of the same table.

---

### 3.4 UNION — ∪
Combines tuples from two relations. **Removes duplicates.**

**Compatibility Rule:** Both relations must be **union-compatible**:
- Same number of attributes (same **degree**).
- Corresponding attributes must have compatible domains.

**Notation:** R ∪ S

```
π_EmpID (σ_DeptID=1 (Employee)) ∪ π_EmpID (σ_DeptID=2 (Employee))
→ EmpIDs of employees in Dept 1 OR Dept 2

Result: {101, 102, 103, 105}
```

---

### 3.5 SET DIFFERENCE — −
Tuples in R but NOT in S.

**Notation:** R − S

```
π_EmpID(Employee) − π_EmpID(Works_On)
→ Employees NOT assigned to any project

Result: {102, 104}   (Rohan and Kartik have no projects)
```

- Set difference is **NOT commutative:** R − S ≠ S − R

---

### 3.6 INTERSECTION — ∩
Tuples that appear in **both** R and S. Can be derived: R ∩ S = R − (R − S)

```
π_City(σ_DeptID=1(Employee)) ∩ π_City(σ_DeptID=2(Employee))
→ Cities that have employees from both Dept 1 and Dept 2

Dept1 cities: {Bangalore}
Dept2 cities: {Delhi}
Result: {} (empty — no city is in both)
```

---

### 3.7 CARTESIAN PRODUCT — × (Cross Product)
Combines every tuple of R with every tuple of S. Result has |R| × |S| rows.

**Notation:** R × S

```
Employee × Department
→ 5 employees × 4 departments = 20 tuples
Each row: (EmpID, Name, Salary, DeptID, City, DeptID, DeptName, Location)
```

Cartesian product alone is rarely useful — it's used as the basis of JOIN.

---

### 3.8 JOIN Operations

#### Natural Join — ⋈
Combines Cartesian product + SELECT on equal common attributes + PROJECT to remove duplicate columns.

```
Employee ⋈ Department
→ Joins on common attribute DeptID (matches Employee.DeptID = Department.DeptID)

Result (5 matching tuples, DeptID appears once):
EmpID  Name    Salary  DeptID  City        DeptName  Location
101    Ananya  55000   1       Bangalore   Research  Bangalore
102    Rohan   42000   2       Delhi       Sales     Delhi
103    Priya   70000   1       Bangalore   Research  Bangalore
104    Kartik  38000   3       Mumbai      HR        Mumbai
105    Sneha   85000   2       Delhi       Sales     Delhi
```

Employee with DeptID=4 doesn't exist here; Dept Finance (DeptID=4) has no employees → not shown (like INNER JOIN).

#### Theta Join — ⋈_θ
Join with an arbitrary condition θ (can be any comparison: =, <, >, ≠, ≤, ≥).

**Notation:** R ⋈_condition S = σ_condition(R × S)

```
Employee ⋈_(Employee.DeptID = Department.DeptID) Department
→ Same as equijoin (most common case of theta join)
```

#### Equijoin
Theta join where condition uses only **=**. Duplicate join attributes are kept (unlike natural join).

#### Outer Joins
Preserve tuples that don't match:

- **Left Outer Join** (⟕): All tuples from LEFT relation; NULLs for non-matching right.
- **Right Outer Join** (⟖): All tuples from RIGHT relation; NULLs for non-matching left.
- **Full Outer Join** (⟗): All tuples from BOTH relations; NULLs where no match.

```
Employee ⟕ Department   -- Includes employees with no department
Department ⟖ Employee   -- Includes departments with no employees (e.g., Finance)
```

---

### 3.9 DIVISION — ÷
"Find all X that are related to ALL Y."

**Notation:** R ÷ S

- R has attributes (A, B), S has attribute (B) → Result has attribute (A).
- Result contains all values of A in R such that for every value of B in S, the pair (A, B) exists in R.

**Example:**
```
Works_On (EmpID, ProjectID)
AllProjects = π_ProjectID(Project) = {P1, P2, P3}

Works_On ÷ π_ProjectID(Works_On WHERE ProjectID IN {P1, P2})
→ "Employees who work on BOTH P1 and P2"

Works_On tuples:
101-P1, 101-P2, 103-P1, 105-P3

Employees working on P1: 101, 103
Employees working on P2: 101
Employees working on BOTH P1 AND P2: 101 (only EmpID 101 has both)

Result: {101}
```

**Derivation:** R ÷ S = π_A(R) − π_A((π_A(R) × S) − R)

---

### 3.10 Extended Relational Algebra Operations

#### Generalized Projection
Allows arithmetic expressions in PROJECT.

```
π_EmpID, Name, Salary*12 AS AnnualSalary (Employee)
```

#### Aggregate Functions
`G` symbol for grouping with aggregate operations.

```
DeptID G COUNT(EmpID), AVG(Salary) (Employee)
→ Count and average salary per department
```

---

### Relational Algebra Expression Examples (Exam-style)

**Q1: Find names of employees in the Research department.**
```
π_Name (σ_DeptName='Research' (Employee ⋈ Department))
```

**Q2: Find employees who work on project 'Apollo'.**
```
π_Name (Employee ⋈ (σ_ProjName='Apollo' (Works_On ⋈ Project)))
```

**Q3: Find employees who work on ALL projects.**
```
π_Name (Employee ⋈ (Works_On ÷ π_ProjectID(Project)))
```

**Q4: Find employees NOT working on any project.**
```
π_Name (Employee) − π_Name (Employee ⋈ Works_On)
```

**Q5: Find pairs of employees in the same department.**
```
π_E1.Name, E2.Name (
  σ_E1.DeptID = E2.DeptID AND E1.EmpID < E2.EmpID (
    ρ_E1(Employee) × ρ_E2(Employee)
  )
)
```

---

## 4. Relational Calculus

**Relational Calculus** is a **non-procedural (declarative) query language** — you specify *what* data you want, not *how* to get it. Based on **mathematical logic (predicate calculus)**.

Two types:
1. **Tuple Relational Calculus (TRC)** — variables represent tuples.
2. **Domain Relational Calculus (DRC)** — variables represent attribute values (domains).

Both are **equivalent in expressive power** to relational algebra (**Codd's theorem**).

### 4.1 Tuple Relational Calculus (TRC)

**General Form:** `{t | P(t)}`

Meaning: "The set of all tuples `t` such that predicate `P(t)` is true."

- `t` is a **tuple variable** (represents a row in a relation).
- `t.A` refers to attribute A of tuple t.
- `P(t)` is a formula (condition) using logical connectives (∧ AND, ∨ OR, ¬ NOT) and quantifiers (∃ EXISTS, ∀ FORALL).

#### TRC Examples

**Q1: Find all employees with salary > 50000.**
```
{t | t ∈ Employee ∧ t.Salary > 50000}
```
*"All tuples t in Employee such that salary is greater than 50000."*

**Q2: Find names and salaries of employees in department 1.**
```
{t.Name, t.Salary | t ∈ Employee ∧ t.DeptID = 1}
```

**Q3: Find names of employees who work in the Research department.**
```
{t.Name | t ∈ Employee ∧
  ∃d (d ∈ Department ∧ d.DeptID = t.DeptID ∧ d.DeptName = 'Research')}
```
*"Tuples t in Employee where THERE EXISTS a tuple d in Department such that the DeptIDs match AND DeptName is Research."*

**Q4: Find employees who work on at least one project.**
```
{t.Name | t ∈ Employee ∧
  ∃w (w ∈ Works_On ∧ w.EmpID = t.EmpID)}
```

**Q5: Find employees who work on ALL projects (using FORALL).**
```
{t.Name | t ∈ Employee ∧
  ∀p (p ∈ Project →
    ∃w (w ∈ Works_On ∧ w.EmpID = t.EmpID ∧ w.ProjectID = p.ProjectID))}
```
*"Employees such that for EVERY project p, there EXISTS a Works_On record linking them."*

#### Safe Expression
A TRC expression is **safe** if it is guaranteed to produce a finite result. 

`{t | ¬(t ∈ Employee)}` is **unsafe** — could return infinite tuples from the entire universe.

In practice, we restrict variables to domains of existing relations.

---

## 5. Domain Relational Calculus (DRC)

In **DRC**, variables range over **domains (attribute values)**, not entire tuples.

**General Form:** `{<x1, x2, ..., xn> | P(x1, x2, ..., xn)}`

Where x1...xn are **domain variables** (each representing a value in some attribute's domain).

### DRC Examples

**Q1: Find names and salaries of employees earning > 50000.**
```
{<n, s> | ∃id ∃d ∃c (<id, n, s, d, c> ∈ Employee ∧ s > 50000)}
```
*n = Name, s = Salary, id = EmpID, d = DeptID, c = City*

**Q2: Find names of employees in department 1.**
```
{<n> | ∃id ∃s ∃c (<id, n, s, 1, c> ∈ Employee)}
```
*The DeptID is directly substituted as 1.*

**Q3: Find names of employees working in the Research department.**
```
{<n> | ∃id ∃s ∃d ∃c (
  <id, n, s, d, c> ∈ Employee ∧
  ∃loc (<d, 'Research', loc> ∈ Department)
)}
```

**Q4: Find employees NOT working on any project.**
```
{<n> | ∃id ∃s ∃d ∃c (
  <id, n, s, d, c> ∈ Employee ∧
  ¬∃pid ∃h (<id, pid, h> ∈ Works_On)
)}
```

### TRC vs DRC Comparison

| Feature | TRC | DRC |
|---|---|---|
| Variable represents | Entire tuple (row) | Single domain value (column value) |
| Syntax | `t.Attribute` | Positional variables `<x1, x2, ...>` |
| Quantifiers | ∃, ∀ | ∃, ∀ |
| Expressive power | Equal | Equal |
| Closeness to SQL | More similar to SQL | Less natural |
| Basis for | SQL | QBE (Query By Example) |

### Relational Algebra vs Relational Calculus

| Feature | Relational Algebra | Relational Calculus |
|---|---|---|
| Type | Procedural | Non-Procedural (Declarative) |
| Focus | How to retrieve | What to retrieve |
| Basis | Operations (σ, π, ⋈, ...) | Logical predicates (∃, ∀, ∧, ∨) |
| Equivalent power | Yes (Codd's Theorem) | Yes |
| Basis for | SQL query optimization | SQL query specification |

---

## 6. ER and EER to Relational Mapping

Converting an ER/EER conceptual design into actual relational tables. There are **7 standard steps**.

### Step 1: Map Strong Entity Types
For each **strong entity** E with attributes a1, a2, ..., an:
- Create a relation R with all attributes.
- Choose the **primary key** from candidate keys.
- Composite attributes → include individual sub-attributes (not the composite itself).
- Derived attributes → usually NOT stored.
- Multi-valued attributes → handled in Step 6.

**Example:**
```
Entity: Employee (EmpID, Name{First, Last}, DOB, Phones*, Age°)
         (* = multi-valued, ° = derived)

Relation: Employee(EmpID, FirstName, LastName, DOB)
          PK: EmpID
          (Age is derived → not stored; Phones → Step 6)
```

---

### Step 2: Map Weak Entity Types
For each **weak entity** W with partial key k, owned by strong entity E:
- Create a relation for W.
- Include all attributes of W.
- Include the **primary key of the owner entity** as a foreign key.
- **PK of W** = (Owner's PK) + (Partial Key of W).

**Example:**
```
Weak Entity: Dependent (DependentName, Relationship, DOB)
             owned by Employee (EmpID)

Relation: Dependent(EmpID, DependentName, Relationship, DOB)
          PK: (EmpID, DependentName)
          FK: EmpID → Employee(EmpID)
```

---

### Step 3: Map Binary 1:1 Relationship Types
Three approaches:

**Option A (Preferred) — Foreign Key Approach:** Add PK of one entity as FK in the other. Choose the entity with **total participation** as the one receiving the FK.

**Option B — Merged Relation:** Merge both entities into one relation (only if both have total participation and are closely related).

**Option C — Cross-Reference / Relationship Relation:** Create a separate relationship table (wasteful for 1:1, only use if both have partial participation).

**Example:**
```
Employee ——manages—— Department (1:1)
Each department has exactly one manager.
Department has total participation.

Option A (preferred):
Department(DeptID, DeptName, ManagerEmpID)
FK: ManagerEmpID → Employee(EmpID)
```

---

### Step 4: Map Binary 1:N Relationship Types
Add the **PK of the "1-side"** as a **FK in the "N-side" table**. Also include any relationship attributes.

**Example:**
```
Department (1) ——has—— Employee (N)
One department has many employees.

Employee(EmpID, Name, Salary, DeptID)   ← DeptID is FK
FK: DeptID → Department(DeptID)
```

**Another Example:**
```
Employee (1) ——supervises—— Employee (N)   (self-referential)

Employee(EmpID, Name, Salary, SupervisorEmpID)
FK: SupervisorEmpID → Employee(EmpID)
```

---

### Step 5: Map Binary M:N Relationship Types
Create a **new junction (bridge) table** S:
- Include PKs of both entities as FKs in S.
- PK of S = combination of both FKs (composite PK).
- Include any attributes of the relationship itself.

**Example:**
```
Student (M) ——enrolls—— Course (N)
Relationship attribute: Grade

Junction table: Enrollment(StudentID, CourseID, Grade)
PK: (StudentID, CourseID)
FK1: StudentID → Student(StudentID)
FK2: CourseID → Course(CourseID)
```

**Another Example:**
```
Employee (M) ——works_on—— Project (N)
Relationship attribute: Hours

Works_On(EmpID, ProjectID, Hours)
PK: (EmpID, ProjectID)
FK1: EmpID → Employee(EmpID)
FK2: ProjectID → Project(ProjectID)
```

---

### Step 6: Map Multi-Valued Attributes
Create a **new table** for each multi-valued attribute:
- Include the PK of the original entity as FK.
- Include the multi-valued attribute.
- PK = (Entity's PK + Multi-valued attribute).

**Example:**
```
Employee has multi-valued attribute: Phones

New table: EmployeePhone(EmpID, PhoneNumber)
PK: (EmpID, PhoneNumber)
FK: EmpID → Employee(EmpID)
```

---

### Step 7: Map N-ary Relationship Types (N > 2)
Create a new relation S:
- Include PKs of ALL participating entities as FKs.
- PK of S = combination of all FKs (usually).
- Include any relationship attributes.

**Example:**
```
Employee ——supplies—— Project using Part (3-way / ternary)

Supply(EmpID, ProjectID, PartNo, Quantity)
PK: (EmpID, ProjectID, PartNo)
FK1: EmpID → Employee
FK2: ProjectID → Project
FK3: PartNo → Part
```

---

### EER to Relational Mapping (Additional Steps)

#### Step 8: Map Specialization / Generalization
Multiple options depending on constraints:

**Option A — Multiple Relations with Superclass:**
- Create a relation for the **superclass** with all shared attributes.
- Create a relation for **each subclass** with its specific attributes + superclass PK as FK and PK.
- Works for both disjoint and overlapping.

```
Superclass: Employee(EmpID, Name, Salary)
Subclass Engineer: Engineer(EmpID, Specialization)  FK: EmpID → Employee
Subclass Manager: Manager(EmpID, Dept)              FK: EmpID → Employee
```

**Option B — Multiple Relations without Superclass (Total Disjoint only):**
- No superclass table; each subclass includes ALL attributes (inherited + own).
- Only valid for **total, disjoint** specialization.

```
Engineer(EmpID, Name, Salary, Specialization)
Manager(EmpID, Name, Salary, Dept)
```
Disadvantage: Redundancy if an entity moves between subclasses.

**Option C — Single Relation with Type Attribute:**
- One table for everything + a type discriminator column.
- Best for overlapping or when subclasses have few extra attributes.

```
Employee(EmpID, Name, Salary, Type, Specialization, Dept)
Type = 'Engineer' | 'Manager' | 'Technician'
Non-applicable attributes are NULL.
```
Disadvantage: Many NULLs.

**Option D — Single Relation with Multiple Type Flags (Overlapping):**
- One table + boolean flags for each subclass membership.

```
Employee(EmpID, Name, Salary, IsEngineer, IsManager, Specialization, Dept)
```

#### Step 9: Map Union Types (Categories)
- Create a surrogate key as PK.
- Each superclass relation gets the surrogate key as FK.

```
VehicleOwner is a union of Person and Company.

VehicleOwner(OwnerID, ...)   ← Surrogate PK
Person(PersonID, ..., OwnerID)   FK: OwnerID → VehicleOwner
Company(CompanyID, ..., OwnerID) FK: OwnerID → VehicleOwner
```

#### Step 10: Map Aggregation
The aggregated relationship is treated as an entity — its PK (combination of participating entities' PKs) becomes a FK in the relationship involving the aggregation.

```
Employee ——works_on—— Project [aggregation] ——managed_by—— Manager

Works_On(EmpID, ProjectID, Hours)
Managed_By(EmpID, ProjectID, ManagerEmpID)
FK1,FK2: (EmpID, ProjectID) → Works_On
FK3: ManagerEmpID → Employee
```

---

## 7. Database Design Phases

### Phase 1: Requirements Collection and Analysis
- Interview users, study existing systems, document requirements.
- Output: **Functional requirements** (what the system should do) + **Data requirements** (what data to store).

### Phase 2: Conceptual Database Design
- Build a **high-level conceptual schema** independent of any DBMS.
- Tool: **ER / EER diagram**.
- Focus: entities, attributes, relationships, constraints.
- No implementation details (no data types, no storage).

### Phase 3: Logical Database Design (Data Model Mapping)
- Map conceptual schema to the **data model of the target DBMS** (usually relational).
- Tool: ER-to-Relational mapping (Steps 1–10 above).
- Output: **Relational schema** (set of tables with attributes and constraints).
- Apply **normalization** to refine the schema.

### Phase 4: Physical Database Design
- Decide how to physically implement the logical schema.
- Choices: file organizations, indexes, clustering, partitioning.
- Goal: maximize **performance** (minimize response time, maximize throughput).
- Highly dependent on the specific DBMS.

### Phase 5: Implementation and Fine-Tuning
- Create the actual database using DDL.
- Load data, run applications, monitor performance.
- Fine-tune indexes, queries, and schema based on actual usage patterns.

---

## 8. Pitfalls in Database Design

When designing a relational schema, poor choices lead to problems called **update anomalies**.

### The Problem Schema (Example)

Consider this poorly designed table:

```
EmpProject(EmpID, EmpName, DeptID, DeptName, ProjectID, ProjName, Hours)

101  Ananya  1  Research  P1  Apollo   40
101  Ananya  1  Research  P2  Zeus     20
102  Rohan   2  Sales     P3  Hermes   50
102  Rohan   2  Sales     P1  Apollo   10
```

### Anomaly 1: Insertion Anomaly
Cannot insert data for one entity without having data for another.

*Example:* Cannot add a new Department that has no employees yet (DeptName would have no EmpID to associate with).

### Anomaly 2: Deletion Anomaly
Deleting one entity's data accidentally deletes another entity's data.

*Example:* If Rohan (102) leaves and we delete both his rows, we also lose information that Project P3 (Hermes) exists — there's no other record of it!

### Anomaly 3: Update Anomaly
Changing a single fact requires updating multiple rows; if any row is missed, the database becomes inconsistent.

*Example:* If Department 1 changes its name from "Research" to "R&D", we must update TWO rows for Ananya. If we miss one, we have contradictory data.

### Root Cause: Redundancy
All three anomalies stem from **data redundancy** — the same fact is stored multiple times.

Solution: **Normalization** — decompose the relation into smaller, well-structured relations.

### Types of Design Pitfalls
1. **Redundancy** — same data stored multiple times.
2. **Incompleteness** — hard to represent certain aspects of the enterprise.
3. **Spurious tuples** — bad decomposition leads to JOINS producing incorrect extra rows.

---

## 9. Functional Dependency

### Definition
A **Functional Dependency (FD)** is a constraint between two sets of attributes in a relation.

**X → Y** (X functionally determines Y) means:
- For any two tuples t1, t2 in relation R,
- If t1[X] = t2[X] (same X values), THEN t1[Y] = t2[Y] (same Y values must follow).
- Knowing the value of X uniquely determines the value of Y.

*In other words:* X → Y means Y is a function of X. One value of X maps to exactly one value of Y.

**Example:**
```
Employee(EmpID, Name, DeptID, DeptName, Salary)

EmpID → Name         ✅ (each EmpID has one Name)
EmpID → DeptID       ✅ (each employee belongs to one department)
DeptID → DeptName    ✅ (each DeptID has one DeptName)
EmpID → DeptName     ✅ (transitively, via DeptID)
Name → EmpID         ❌ (two employees can have the same name)
DeptName → DeptID    ✅ (assuming DeptName is also unique)
EmpID → Salary       ✅
```

### Types of Functional Dependencies

#### Trivial FD
X → Y is **trivial** if Y ⊆ X (Y is a subset of X). Always true, carries no useful information.

*Example:* `{EmpID, Name} → EmpID` — trivially true (EmpID is in the left side).

#### Non-trivial FD
Y ⊄ X. Carries actual information.

*Example:* `EmpID → Name` — non-trivial (Name is not part of EmpID).

#### Full Functional Dependency
X → Y is a **full FD** if removing ANY attribute from X destroys the dependency — i.e., no proper subset of X can determine Y.

*Example:* In `Enrollment(StudentID, CourseID, Grade)`:
- `{StudentID, CourseID} → Grade` ← FULL FD
- Because `StudentID → Grade` is false (student can get different grades in different courses)
- And `CourseID → Grade` is false (different students get different grades)

#### Partial Functional Dependency
X → Y is a **partial FD** if some proper subset of X can also determine Y.

*Example:* In a badly designed relation `EmpProject(EmpID, ProjectID, EmpName, Hours)`:
- `{EmpID, ProjectID} → Hours` — FULL FD ✅
- `{EmpID, ProjectID} → EmpName` — PARTIAL FD ❌ (because `EmpID → EmpName` alone)

#### Transitive Dependency
Given X → Y and Y → Z (where Y ↛ X and Z is not a subset of X), then X → Z is a **transitive dependency**.

*Example:*
```
EmpID → DeptID      (direct)
DeptID → DeptName   (direct)
∴ EmpID → DeptName  (TRANSITIVE — DeptName depends on DeptID, not directly on EmpID)
```

### Equivalence of FD Sets
Two sets of FDs F and G are **equivalent** if F⁺ = G⁺ (they have the same closure — same set of all derivable FDs).

---

## 10. Armstrong's Axioms

**Armstrong's Axioms** are a complete and sound set of inference rules for deriving all functional dependencies from a given set F.

**Complete:** Every FD derivable from F can be derived using these axioms.
**Sound:** Every FD derived using these axioms is valid (actually holds).

### Primary Rules (Axioms)

#### Rule 1: Reflexivity (Trivial FDs)
If Y ⊆ X, then X → Y.

*Example:* `{EmpID, Name} → EmpID` ✅ (EmpID ⊆ {EmpID, Name})
*Example:* `{A, B, C} → {A, B}` ✅

#### Rule 2: Augmentation
If X → Y, then XZ → YZ (for any set of attributes Z).

*Example:* If `EmpID → DeptID`, then `{EmpID, Name} → {DeptID, Name}`.
*Example:* If `A → B`, then `AC → BC`.

#### Rule 3: Transitivity
If X → Y and Y → Z, then X → Z.

*Example:* `EmpID → DeptID` and `DeptID → DeptName` → `EmpID → DeptName`.

### Derived Rules (from the three axioms)

#### Rule 4: Union
If X → Y and X → Z, then X → YZ.

*Example:* `EmpID → Name` and `EmpID → Salary` → `EmpID → {Name, Salary}`.

#### Rule 5: Decomposition (Splitting Rule)
If X → YZ, then X → Y and X → Z.

*Example:* `EmpID → {Name, Salary}` → `EmpID → Name` AND `EmpID → Salary`.

#### Rule 6: Pseudotransitivity
If X → Y and WY → Z, then WX → Z.

*Example:* `EmpID → DeptID` and `{DeptID, City} → Budget` → `{EmpID, City} → Budget`.

> **Exam Tip:** Armstrong's Axioms are **sound and complete** — these three axioms are sufficient to derive ALL valid FDs. Know the proofs by example, not just definitions.

---

## 11. Closure, Cover, and Minimal Cover

### 11.1 Closure of a Set of Attributes (X⁺)

The **closure of X under F** (written X⁺) = the set of ALL attributes that can be functionally determined by X using the FDs in F.

**Algorithm to compute X⁺:**
```
X⁺ = X   (initialize with X itself)
REPEAT:
  For each FD (Y → Z) in F:
    If Y ⊆ X⁺:
      X⁺ = X⁺ ∪ Z
UNTIL no change in X⁺
```

**Example:**
```
R = (A, B, C, D, E, F)
F = {A → BC, BC → DE, D → F}

Find A⁺:
Start: A⁺ = {A}
A → BC: A ⊆ {A} → A⁺ = {A, B, C}
BC → DE: {B,C} ⊆ {A,B,C} → A⁺ = {A, B, C, D, E}
D → F: D ⊆ {A,B,C,D,E} → A⁺ = {A, B, C, D, E, F}
No more changes.

A⁺ = {A, B, C, D, E, F} = R
∴ A is a candidate key (it determines all attributes)!
```

**Uses of Closure:**
1. **Testing if X is a superkey:** If X⁺ = all attributes of R, then X is a superkey.
2. **Testing if X → Y holds:** If Y ⊆ X⁺, then X → Y is derivable from F.
3. **Finding candidate keys:** Find minimal X such that X⁺ = R.

### 11.2 Closure of a Set of FDs (F⁺)
F⁺ = the set of all FDs that can be derived from F using Armstrong's Axioms.
Computing F⁺ directly is expensive (can be exponential). We use attribute closure instead.

### 11.3 Cover (Equivalent Set of FDs)
G is a **cover** (or **equivalent set**) of F if F⁺ = G⁺.
Multiple covers can represent the same set of dependencies.

### 11.4 Canonical (Minimal) Cover — Fc

A **minimal cover** Fc of F is an equivalent set of FDs that is as small/simple as possible with no redundancy.

**Properties of Minimal Cover Fc:**
1. **Right-hand side is a single attribute** — each FD has form X → A.
2. **No redundant FD** — removing any FD from Fc changes F⁺.
3. **No redundant attribute on the left** — no attribute in the left side of any FD can be removed without changing F⁺.

**Algorithm to find Minimal Cover:**

**Step 1:** Decompose RHS — replace each X → {A1, A2, ..., An} with X → A1, X → A2, ..., X → An.

**Step 2:** Remove redundant LHS attributes — for each FD X → A, check if any attribute in X is extraneous:
- For attribute B in X: check if (X − {B}) → A is derivable. If yes, replace X with (X − {B}).

**Step 3:** Remove redundant FDs — for each FD X → A, check if X → A is derivable from the remaining FDs. If yes, remove it.

**Example:**
```
F = {A → BC, B → C, A → B, AB → C}

Step 1 (Decompose RHS):
F = {A → B, A → C, B → C, A → B, AB → C}
Remove duplicate: F = {A → B, A → C, B → C, AB → C}

Step 2 (Remove redundant LHS attributes):
Check AB → C: Can we remove A? Check B⁺ under F−{AB→C} = {A→B, A→C, B→C}
B⁺ = {B, C} (B→C applies). C ∈ B⁺? Yes! So AB → C becomes B → C.
But B → C already exists → AB → C is redundant.

Step 3 (Remove redundant FDs):
Check A → C: Is it derivable from {A→B, B→C}?
A → B (use A→B), B → C (use B→C) → A → C by transitivity. YES → remove A→C.

Check A → B: Is it derivable from {B→C}? A⁺ = {A}. B ∉ A⁺. NO → keep.
Check B → C: Is it derivable from {A→B}? B⁺ = {B}. C ∉ B⁺. NO → keep.

Minimal Cover: Fc = {A → B, B → C}
```

---

## 12. Normalization — 1NF, 2NF, 3NF

**Normalization** is the process of organizing a relational schema to:
- Reduce **data redundancy**.
- Eliminate **update anomalies** (insertion, deletion, modification).
- Ensure **data integrity**.

It is a stepwise process — each normal form is stronger than the previous.

```
Unnormalized → 1NF → 2NF → 3NF → BCNF → 4NF → 5NF
          (increasingly stricter conditions)
```

---

### 12.1 First Normal Form (1NF)

**Condition:** Every attribute value must be **atomic (indivisible)** — no multi-valued or composite attributes. All entries in a column must be of the same type.

**Violation Example:**
```
Student(RollNo, Name, Subjects)
101  Ananya  {Math, Physics, CS}    ← MULTI-VALUED: violates 1NF
102  Rohan   {Math, Chemistry}
```

**Converting to 1NF — Option A (Separate rows):**
```
Student(RollNo, Name, Subject)
101  Ananya  Math
101  Ananya  Physics
101  Ananya  CS
102  Rohan   Math
102  Rohan   Chemistry
PK: (RollNo, Subject)
```

**Converting to 1NF — Option B (Separate table for multi-valued):**
```
Student(RollNo, Name)
Student_Subjects(RollNo, Subject)
PK of Student_Subjects: (RollNo, Subject)
```

**Another Violation (Composite attribute stored as one):**
```
Employee(EmpID, Name, Address)
101  Ananya  "123 MG Road, Bangalore - 560001"  ← Composite: violates 1NF
```
Fix: Separate into `Street`, `City`, `PinCode`.

---

### 12.2 Second Normal Form (2NF)

**Condition:** Must be in 1NF AND every **non-prime attribute** must be **fully functionally dependent** on the **entire primary key** (no partial dependencies allowed).

- Applies only when PK is **composite** (multi-attribute).
- **Non-prime attribute:** An attribute NOT part of any candidate key.
- **Partial dependency:** Non-prime attribute depends on part of the PK, not the whole PK.

**Violation Example:**
```
EmpProject(EmpID, ProjectID, EmpName, ProjectName, Hours)
PK: (EmpID, ProjectID)

FDs:
{EmpID, ProjectID} → Hours     ← FULL dependency ✅
EmpID → EmpName                 ← PARTIAL dependency ❌ (EmpName depends only on EmpID)
ProjectID → ProjectName         ← PARTIAL dependency ❌ (ProjName depends only on ProjectID)
```

**2NF Decomposition:**
```
EmpProject relation suffers from:
- Update anomaly: Changing EmpName requires updating multiple rows.
- Insertion anomaly: Can't add a project with no employees.
- Deletion anomaly: Deleting last employee on a project loses project info.

Decompose into:
1. Employee(EmpID, EmpName)                 PK: EmpID
2. Project(ProjectID, ProjectName)          PK: ProjectID
3. Works_On(EmpID, ProjectID, Hours)        PK: (EmpID, ProjectID)
                                            FK: EmpID → Employee, ProjectID → Project
```

Now every non-prime attribute is fully dependent on its table's PK. ✅

**2NF Check Process:**
1. Identify the primary key (if composite, proceed; if single attribute, relation is automatically in 2NF).
2. List all non-prime attributes.
3. Check if any non-prime attribute depends on a proper subset of the PK.
4. If yes → partial dependency → violates 2NF → decompose.

---

### 12.3 Third Normal Form (3NF)

**Condition:** Must be in 2NF AND there is **no transitive dependency** of a non-prime attribute on the primary key.

- **Transitive dependency:** A → B → C, where A is the PK, B is a non-prime attribute, and C is another non-prime attribute. C depends on A only indirectly (through B).

**Violation Example:**
```
Employee(EmpID, Name, DeptID, DeptName, DeptLocation)
PK: EmpID

FDs:
EmpID → DeptID          ← Direct ✅
DeptID → DeptName       ← Direct (but DeptID is a non-prime attribute!)
DeptID → DeptLocation
∴ EmpID → DeptName      ← TRANSITIVE (via DeptID) ❌
∴ EmpID → DeptLocation  ← TRANSITIVE (via DeptID) ❌
```

**3NF Decomposition:**
```
Remove transitive dependencies by creating separate relations:

1. Employee(EmpID, Name, DeptID)             PK: EmpID
                                              FK: DeptID → Department
2. Department(DeptID, DeptName, DeptLocation) PK: DeptID
```

Now:
- No partial dependencies (single-attribute PK in each table) ✅
- No transitive dependencies ✅

**Another Example:**
```
STJ(Student, Teacher, Subject)
FDs: Student → Teacher, Teacher → Subject
PK: Student (assume each student has one teacher and one subject)

Teacher → Subject is a transitive dependency (Student → Teacher → Subject)

Decompose:
ST(Student, Teacher)   PK: Student
TJ(Teacher, Subject)   PK: Teacher
```

### 3NF — Formal Definition
A relation R is in 3NF if for every non-trivial FD X → A in R⁺, at least one of these holds:
1. X is a **superkey** of R, OR
2. A is a **prime attribute** (member of some candidate key).

This definition allows some transitive dependencies IF the right-hand side is a prime attribute.

---

## 13. BCNF (Boyce-Codd Normal Form)

**BCNF** is a stronger version of 3NF, proposed by Boyce and Codd (1974).

### Condition
A relation R is in **BCNF** if for every non-trivial FD X → Y in F⁺:
**X must be a superkey of R.**

No exceptions — unlike 3NF which allows "A is prime attribute" as an exception.

### 3NF vs BCNF
- Every BCNF relation is in 3NF.
- Not every 3NF relation is in BCNF.
- 3NF is **weaker** — it allows some anomalies; BCNF is **stricter** — eliminates more anomalies.
- 3NF guarantees lossless join AND dependency preservation.
- BCNF guarantees lossless join but may **NOT preserve all dependencies**.

### When 3NF ≠ BCNF (The Classic Example)
```
Relation: CourseTeacher(Student, Course, Teacher)
Semantics:
  - Each teacher teaches only one course.
  - Each student can take multiple courses.
  - A student can have only one teacher per course.

FDs:
  {Student, Course} → Teacher   (a student in a course has one teacher)
  Teacher → Course               (a teacher teaches only one course)

Candidate Keys: {Student, Course} and {Student, Teacher}
Prime attributes: Student, Course, Teacher (ALL are prime!)

3NF check: Teacher → Course: Is Teacher a superkey? NO. Is Course prime? YES.
→ Satisfies 3NF condition 2 → IN 3NF ✅

BCNF check: Teacher → Course: Is Teacher a superkey? NO.
→ VIOLATES BCNF ❌

Anomaly: If we change which course a teacher teaches, multiple rows must be updated.
```

### BCNF Decomposition

**Algorithm:**
```
Given relation R and set of FDs F:
WHILE (some relation Ri is not in BCNF):
  Find a non-trivial FD X → Y in Ri that violates BCNF (X is not a superkey).
  Decompose Ri into:
    R1 = X ∪ Y               (attributes of the violating FD)
    R2 = Ri − Y              (original relation minus Y, keeping X)
  Replace Ri with R1 and R2.
```

**Example:**
```
Relation: R(A, B, C, D)
FDs: A → B, B → C

Candidate key: {A, D} (A determines B,C; D is needed to complete)
Check: B → C: B is not a superkey → BCNF violation!

Decompose on B → C:
R1(B, C)      PK: B    ← captures B → C
R2(A, B, D)   PK: (A, D) with FK B → R1

Check R2: FDs applicable = A → B. Is A a superkey of R2? A⁺ in R2 = {A, B} ≠ {A,B,D}.
A is NOT a superkey → still violates BCNF!

Decompose R2 on A → B:
R3(A, B)      PK: A
R4(A, D)      PK: (A, D)

Final BCNF decomposition: R1(B,C), R3(A,B), R4(A,D)

Note: FD A→C is LOST (not preserved) — this is the trade-off with BCNF.
```

### BCNF — Key Points
- BCNF always achieves **lossless join decomposition**.
- BCNF may **lose some FDs** (dependency preservation not always guaranteed).
- When dependency preservation is critical, use 3NF instead of BCNF.

---

## 14. 4NF and 5NF

### 14.1 Multivalued Dependency (MVD)

Before 4NF, we need to understand **Multi-Valued Dependencies (MVDs)**.

**X ↠ Y** (X multi-determines Y) means: Given a value of X, the set of Y values is independent of the other attributes Z = R − X − Y.

*Informal:* The values of Y associated with each X value do not depend on any other attribute.

**Example:**
```
Employee(EmpID, Skill, Hobby)

EmpID  Skill    Hobby
101    Java     Cricket
101    Java     Chess
101    Python   Cricket
101    Python   Chess

EmpID ↠ Skill   (for each employee, their skills are independent of hobbies)
EmpID ↠ Hobby   (for each employee, their hobbies are independent of skills)

Each combination of (EmpID, Skill) must appear with EVERY Hobby of that EmpID
→ This causes redundancy!
```

**Trivial MVD:** X ↠ Y is trivial if Y ⊆ X or X ∪ Y = R.

**Relationship between FD and MVD:** Every FD is also an MVD. If X → Y, then X ↠ Y (but NOT vice versa).

### 14.2 Fourth Normal Form (4NF)

**Condition:** A relation R is in 4NF if for every non-trivial MVD X ↠ Y in R:
**X must be a superkey of R.**

Eliminates redundancy caused by multi-valued dependencies.

**Violation Example:**
```
EmpSkillHobby(EmpID, Skill, Hobby)
PK: (EmpID, Skill, Hobby) — all three needed to uniquely identify a row

MVDs: EmpID ↠ Skill, EmpID ↠ Hobby

EmpID is not a superkey → VIOLATES 4NF!

EmpID  Skill    Hobby
101    Java     Cricket  ← Must repeat ALL combos
101    Java     Chess
101    Python   Cricket
101    Python   Chess

If EmpID 101 learns a new skill "C++":
Must add: (101, C++, Cricket) AND (101, C++, Chess) — easy to miss one!
```

**4NF Decomposition:**
```
Decompose into:
EmpSkill(EmpID, Skill)   PK: (EmpID, Skill)
EmpHobby(EmpID, Hobby)   PK: (EmpID, Hobby)

Now:
EmpID ↠ Skill in EmpSkill: EmpID is superkey? No, but Y = Skill, X ∪ Y = R → TRIVIAL ✅
EmpID ↠ Hobby in EmpHobby: Similarly trivial ✅
Both tables are in 4NF!
```

### 14.3 Join Dependency (JD)

A relation R has a **Join Dependency** `*(R1, R2, ..., Rn)` if R can always be reconstructed by joining R1, R2, ..., Rn without introducing spurious tuples.

Every MVD is a special case of JD with two components:
`*(XY, XZ) ≡ X ↠ Y | Z` (where R = XYZ)

### 14.4 Fifth Normal Form (5NF) / Project-Join Normal Form (PJNF)

**Condition:** A relation R is in 5NF if for every non-trivial join dependency `*(R1, R2, ..., Rn)` in R:
Every Ri contains a candidate key of R.

5NF eliminates redundancy due to **join dependencies** that are NOT implied by MVDs.

**Example (Classic Suppliers-Parts-Projects):**
```
SPJ(Supplier, Part, Project)

Data semantics: A supplier S supplies part P, and P is used in project J,
and S can supply to J → then S supplies P for J.

S1  P1  J1
S1  P1  J2
S1  P2  J1
S2  P1  J1

JD: *(SP, PJ, SJ) — can reconstruct SPJ from three binary projections

If the JD is non-trivial and no Ri contains a candidate key of SPJ,
then SPJ violates 5NF.

Decompose into:
SP(Supplier, Part)
PJ(Part, Project)
SJ(Supplier, Project)
```

5NF is rarely needed in practice. 3NF and BCNF are sufficient for most real-world databases.

### Normal Form Progression Summary

| Normal Form | Eliminates | Condition |
|---|---|---|
| 1NF | Non-atomic values | All attributes atomic |
| 2NF | Partial dependencies | No non-prime attr depends on part of PK |
| 3NF | Transitive dependencies | No non-prime attr depends on another non-prime attr |
| BCNF | All FD-based anomalies | Every determinant is a superkey |
| 4NF | Multi-valued dependencies | Every MVD determinant is a superkey |
| 5NF | Join dependencies | Every JD component contains a candidate key |

---

## 15. Lossless Join Decomposition

### What is Lossless Join?
When we decompose a relation R into R1, R2, ..., Rn:
- The decomposition is **lossless (non-additive)** if joining R1, R2, ..., Rn back together gives **exactly the original relation R** — no extra (spurious) tuples.
- The decomposition is **lossy** if joining produces extra spurious tuples not in the original relation.

**Lossless condition:** R1 ⋈ R2 ⋈ ... ⋈ Rn = R

### Why Lossless Decomposition is Critical
A lossy decomposition generates **spurious tuples** — tuples that appear valid but don't represent real data — making the database incorrect.

**Example of Lossy Decomposition:**
```
R(A, B, C)
r = {(1, 2, 3), (4, 5, 6), (7, 2, 8)}

Lossy decomposition: R1(A, B), R2(B, C)
R1: {(1,2), (4,5), (7,2)}
R2: {(2,3), (5,6), (2,8)}

R1 ⋈ R2 (join on B):
(1,2,3) ✅ Real
(1,2,8) ← SPURIOUS! Not in original R
(4,5,6) ✅ Real
(7,2,3) ← SPURIOUS! Not in original R
(7,2,8) ✅ Real

Result has 5 tuples but original had 3 → LOSSY DECOMPOSITION!
```

**Same example — Lossless if B → A or B → C held:**
If `B → C` holds: R2(B,C) has B as key. R1 ∩ R2 = {B} → B determines attributes in R2.
The test below would confirm losslessness.

### Test for Lossless Binary Decomposition
For decomposition of R into R1 and R2:

**Condition (sufficient and necessary for binary decomposition):**
The decomposition R → {R1, R2} is lossless if and only if:
- **(R1 ∩ R2) → R1** (the common attributes functionally determine all of R1), OR
- **(R1 ∩ R2) → R2** (the common attributes functionally determine all of R2)

In other words: The common attributes must form a **superkey** of at least one of R1 or R2.

**Example:**
```
R(A, B, C, D)
FDs: A → B, B → C, C → D

Decompose into R1(A, B, C) and R2(C, D)

R1 ∩ R2 = {C}
Does C → all of R1(A,B,C)? C⁺ = {C, D} (only). C ∉ {A,B} → NO
Does C → all of R2(C,D)?   C⁺ = {C, D} = {C, D} → YES! ✅

∴ Lossless decomposition!
```

**Another Example:**
```
R(A, B, C)
FDs: A → B

Decompose into R1(A, C) and R2(B, C)

R1 ∩ R2 = {C}
Does C → all of R1(A,C)? C⁺ = {C}. A ∉ C⁺ → NO
Does C → all of R2(B,C)? C⁺ = {C}. B ∉ C⁺ → NO

∴ LOSSY decomposition! ❌
```

### Chase Algorithm (General Test for Multiple Relations)
For decompositions into more than 2 relations, use the **Chase Algorithm**:

1. Create a tableau (table) where each row represents one Ri.
2. Initialize: For attribute Aj in Ri, use `aj` (distinguished symbol). Otherwise, use `bij` (non-distinguished symbol — unique per cell).
3. Repeatedly apply FDs to equate symbols.
4. If any row becomes all distinguished symbols `(a1, a2, ..., an)`, the decomposition is lossless.

**Example:**
```
R(A, B, C, D), FDs: A→B, B→C, CD→A
Decompose into: R1(A,D), R2(A,B), R3(B,C,D)

Initial tableau:
     A    B    C    D
R1:  a1   b12  b13  a4      ← R1 has A,D → a for A and D; b for B and C
R2:  a1   a2   b23  b24     ← R2 has A,B → a for A and B; b for C and D
R3:  b31  a2   a3   a4      ← R3 has B,C,D → a for B,C,D; b for A

Apply A→B: Rows with same A value must have same B value.
R1 and R2 have a1 for A → equate B: b12 = a2 (a takes priority)
Update R1: a1, a2, b13, a4

Apply B→C: Rows with same B value must have same C value.
R2, R3 have a2 for B → equate C: b23 = a3
Update R2: a1, a2, a3, b24

Apply CD→A: Rows with same C,D must have same A value.
R3 has a3, a4 for C,D; R1 has b13,a4 (C,D don't match yet)
After R2 updated: R2 has (a3, b24) for (C,D); R3 has (a3, a4) — still differ on D.
...continue...

If eventually a row becomes (a1, a2, a3, a4) → LOSSLESS
```

### BCNF Decomposition is Always Lossless
When decomposing using a BCNF-violating FD X → Y:
- R1 = XY, R2 = R − Y (keep X)
- R1 ∩ R2 = X
- X → R1 (since X → XY = R1) → **lossless** ✅

---

## 16. Dependency Preservation

### What is Dependency Preservation?
A decomposition of R into {R1, R2, ..., Rk} is **dependency-preserving** if:

**(F1 ∪ F2 ∪ ... ∪ Fk)⁺ = F⁺**

Where Fi is the set of FDs from F⁺ that involve only attributes of Ri (the **projection** of F onto Ri).

In simple terms: **All the original FDs can be checked using only the individual decomposed relations** — no need to JOIN them to verify a constraint.

### Why Dependency Preservation Matters
If a dependency is lost after decomposition, enforcing it requires joining relations back together every time data is modified — expensive and impractical.

**Example of Dependency Loss:**
```
Relation: CST(Course, Section, Teacher)
FDs: {Course, Section} → Teacher   (a section of a course has one teacher)
     Teacher → Course              (a teacher teaches one course)

Candidate Keys: {Course, Section} and {Section, Teacher}

BCNF violation: Teacher → Course (Teacher is not a superkey)

BCNF Decomposition:
R1(Teacher, Course)    captures Teacher → Course
R2(Teacher, Section)   captures Teacher

What happened to {Course, Section} → Teacher?
  - {Course, Section} spans BOTH R1 and R2.
  - We CANNOT check this constraint without joining R1 and R2.
  - FD IS LOST! ❌

The decomposition is lossless but NOT dependency-preserving.
```

### Testing Dependency Preservation

**Algorithm:** For each FD X → Y in F:
1. Set Result = X.
2. For each Ri in the decomposition:
   - Compute the closure of Result restricted to Ri's attributes using Ri's FDs.
   - Add any new attributes to Result.
3. Repeat step 2 until no change.
4. If Y ⊆ Result: the FD X → Y is preserved. Else: LOST.

**Example:**
```
R(A, B, C), F = {A→B, B→C, A→C}
Decompose into: R1(A,B), R2(B,C)

Test A→B: X = {A}
  R1(A,B): A→B applicable → Result = {A,B}. B ∈ Result? YES ✅

Test B→C: X = {B}
  R1(A,B): B is in R1 but B→C is not in R1's FDs → no change
  R2(B,C): B→C applicable → Result = {B,C}. C ∈ Result? YES ✅

Test A→C: X = {A}
  R1(A,B): A→B → Result = {A,B}
  R2(B,C): B→C → Result = {A,B,C}. C ∈ Result? YES ✅ (transitively preserved)

All FDs preserved → DEPENDENCY-PRESERVING ✅
```

### 3NF vs BCNF — The Trade-off

| Property | 3NF | BCNF |
|---|---|---|
| Lossless Join | ✅ Always | ✅ Always |
| Dependency Preservation | ✅ Always | ❌ Not always |
| Redundancy Elimination | Partial | Complete |
| Anomaly Elimination | Partial | Complete (for FD-based anomalies) |
| Practical Preference | When FD preservation needed | When complete normalization needed |

### 3NF Synthesis Algorithm (Guarantees both Lossless + Dependency Preserving)

```
Given R and F:
Step 1: Find the minimal cover Fc of F.
Step 2: For each FD X → A in Fc, create a relation Ri(X ∪ A).
Step 3: If no relation contains a candidate key of R, add a relation containing a candidate key.
Step 4: Eliminate any redundant relations (those whose attributes are a subset of another).
```

**Example:**
```
R(A, B, C, D, E)
F = {A→B, BC→D, D→E, E→B}

Step 1 (Minimal Cover):
After simplification (check for redundancy): Fc = {A→B, BC→D, D→E, E→B}

Step 2: Create relations:
R1(A, B)        from A→B
R2(B, C, D)     from BC→D
R3(D, E)        from D→E
R4(E, B)        from E→B

Step 3: Candidate key of R?
A⁺ = {A, B}. AC⁺ = {A,B,C,D,E} = R → AC is a candidate key!
No existing relation contains AC → Add R5(A, C) or R5(A, C) is the candidate key relation.
Actually check: R2(B,C,D) — does it contain a CK? BC→D. {BC}⁺={B,C,D,E} ≠ R.
Add R5(A, C).

Final 3NF schema:
R1(A,B), R2(B,C,D), R3(D,E), R4(E,B), R5(A,C)
```

---

## 🔑 Quick Revision — Exam Cheat Sheet (Unit 2)

### Relational Algebra Quick Reference

| Operation | Symbol | SQL Equivalent | Notes |
|---|---|---|---|
| Select | σ_c(R) | WHERE c | Filters rows |
| Project | π_A(R) | SELECT A | Filters columns, removes dups |
| Rename | ρ_S(R) | AS | Renames relation/attributes |
| Union | R ∪ S | UNION | Union-compatible required |
| Difference | R − S | EXCEPT | R but not S |
| Intersection | R ∩ S | INTERSECT | Both R and S |
| Cartesian Prod | R × S | CROSS JOIN | m×n rows |
| Natural Join | R ⋈ S | JOIN (on common attrs) | Auto-joins on equal attrs |
| Division | R ÷ S | (no direct SQL) | "For all" queries |

### Key Definitions (1-2 marks)
- **Functional Dependency X→Y:** Same X values always produce same Y values.
- **Full FD:** No proper subset of X can determine Y.
- **Partial FD:** Some proper subset of X determines Y (violates 2NF).
- **Transitive FD:** X→Y→Z where Y is non-prime (violates 3NF).
- **Candidate Key:** Minimal superkey (closure = all attributes).
- **Prime Attribute:** Member of at least one candidate key.
- **Non-prime Attribute:** Not part of any candidate key.
- **MVD X↠Y:** Y values for each X are independent of R−X−Y.
- **Lossless Join:** R = R1 ⋈ R2 (no spurious tuples).
- **Dependency Preserving:** All FDs checkable without join.
- **Minimal Cover:** Equivalent FD set with no redundancy, single-attribute RHS.

### Normal Forms — One-Line Conditions
- **1NF:** All values atomic.
- **2NF:** 1NF + No partial dependencies on PK.
- **3NF:** 2NF + No transitive dependencies (or: every determinant is superkey OR RHS is prime).
- **BCNF:** 3NF + Every determinant is a superkey (no exceptions).
- **4NF:** BCNF + No non-trivial MVDs unless determinant is superkey.
- **5NF:** 4NF + No non-trivial Join Dependencies unless components contain candidate key.

### Armstrong's Axioms — Memory Aid: "RAT"
- **R**eflexivity: Y ⊆ X → X→Y
- **A**ugmentation: X→Y → XZ→YZ
- **T**ransitivity: X→Y, Y→Z → X→Z

Derived: **Union** (X→Y, X→Z → X→YZ), **Decomposition** (X→YZ → X→Y, X→Z), **Pseudotransitivity**

### Lossless Test for Binary Decomposition
R → {R1, R2} is lossless iff: **(R1 ∩ R2) → R1** OR **(R1 ∩ R2) → R2**

### ER to Relational Mapping Steps (Memory Aid)
1. **Strong Entities** → direct table, choose PK
2. **Weak Entities** → table with owner's PK + partial key
3. **1:1 Relationship** → FK in total-participation side
4. **1:N Relationship** → FK in the N-side
5. **M:N Relationship** → new junction table
6. **Multi-valued Attributes** → new table with entity PK + attribute
7. **N-ary Relationships** → new table with all participating PKs

### Frequently Tested Comparisons
- **Partial FD vs Transitive FD:** Partial = non-prime depends on PART of PK; Transitive = non-prime depends on ANOTHER non-prime.
- **3NF vs BCNF:** 3NF preserves dependencies; BCNF may lose them. BCNF is stricter.
- **Lossless vs Dependency-Preserving:** BCNF always lossless, not always DP. 3NF synthesis gives both.
- **MVD vs FD:** FD is special case of MVD. MVD is weaker — X↠Y doesn't mean X→Y.
- **TRC vs DRC:** TRC variables = tuples; DRC variables = domain values. Both equally expressive.

---

*Notes compiled for DBMS Unit 2 — Master these concepts and anomalies won't stand a chance! 🎯*
