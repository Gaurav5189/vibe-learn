# 📚 Database Management Systems — Unit 1
### Complete Exam Preparation Notes

---

## Table of Contents
1. [File Systems vs Database Systems](#1-file-systems-vs-database-systems)
2. [History of DBMS](#2-history-of-dbms)
3. [Advantages and Disadvantages of DBMS](#3-advantages-and-disadvantages-of-dbms)
4. [Data Views (Three-Schema Architecture)](#4-data-views-three-schema-architecture)
5. [Database Languages](#5-database-languages)
6. [Database Administrator (DBA)](#6-database-administrator-dba)
7. [Database Architecture (ANSI/SPARC)](#7-database-architecture-ansisparc)
8. [Data Models](#8-data-models)
9. [Attribute Types](#9-attribute-types)
10. [Keys in DBMS](#10-keys-in-dbms)
11. [ER Diagram (Entity-Relationship Diagram)](#11-er-diagram)
12. [EER Diagram (Enhanced ER Diagram)](#12-eer-diagram)
13. [Network and Hierarchical Models](#13-overview-of-network-and-hierarchical-models)
14. [SQL — Data Definition Language (DDL)](#14-sql--data-definition-language-ddl)
15. [SQL — Basic Queries](#15-sql--basic-queries)
16. [SQL — Complex Queries](#16-sql--complex-queries)
17. [Insert, Delete, Update in SQL](#17-insert-delete-and-update-in-sql)
18. [Views in SQL](#18-views-in-sql)
19. [Constraints and Assertions](#19-constraints-and-assertions)

---

## 1. File Systems vs Database Systems

### What is a File System?
A **file system** is the traditional approach of storing data in separate, flat files managed by the OS. Each application has its own files. There is no centralized control.

**Example:** A university stores student records in `students.txt`, fee records in `fees.txt`, and exam results in `results.txt` — all managed separately by different programs.

### Problems with File Systems (Why DBMS was Needed)

| Problem | Explanation | Example |
|---|---|---|
| **Data Redundancy** | Same data stored in multiple files | Student name stored in both `students.txt` and `fees.txt` |
| **Data Inconsistency** | Updates in one file not reflected in others | Name changed in students file but not in fees file |
| **Difficulty in Accessing Data** | Requires writing new programs for every new query | New report needed? Write a new C program. |
| **Data Isolation** | Data in different files, different formats — hard to combine | Joining student + result data requires custom code |
| **Integrity Problems** | Hard to enforce rules (e.g., age > 18) | No built-in constraint mechanism |
| **Atomicity Problems** | Partial updates possible (no rollback) | Money deducted from one account but not credited to another |
| **Concurrent Access Anomalies** | Multiple users reading/writing simultaneously causes errors | Two clerks update the same seat booking at once |
| **Security Problems** | Hard to restrict access at field level | Can't easily hide salary field from some users |

### What is a Database System?
A **Database Management System (DBMS)** is software that manages a collection of interrelated data (the **database**) and provides a set of programs to access and manipulate that data.

**Key Players:**
- **Data** — the actual stored information
- **DBMS software** — e.g., MySQL, Oracle, PostgreSQL
- **Application Programs** — interact with DBMS
- **Users** — interact via application or directly

---

## 2. History of DBMS

| Era | Development |
|---|---|
| **1960s** | File-based systems; IDS (Integrated Data Store) by GE — first network DBMS |
| **Late 1960s** | IBM's IMS (Information Management System) — first hierarchical DBMS (used in Apollo moon mission) |
| **1970** | E.F. Codd proposes the **Relational Model** (IBM) — landmark paper |
| **1970s** | INGRES and System R — first relational database prototypes |
| **1980s** | Commercial relational DBMSs: Oracle, DB2, SQL Server. SQL standardized (SQL-86, SQL-89) |
| **1990s** | Object-oriented databases; distributed databases; SQL-92, SQL-99 |
| **2000s** | XML databases, data warehouses, NoSQL movement begins |
| **2010s** | Big Data (Hadoop, Spark), Cloud databases (Amazon RDS, Google Spanner), NewSQL |
| **Today** | Multi-model databases, AI-integrated DBMS, time-series DBs |

> **Exam Tip:** E.F. Codd's 1970 paper "A Relational Model of Data for Large Shared Data Banks" is the foundation of modern DBMS. Know this!

---

## 3. Advantages and Disadvantages of DBMS

### ✅ Advantages

1. **Reduced Data Redundancy** — Centralized storage avoids duplication.
   - *Example:* Student address stored once, used by all departments.

2. **Data Consistency** — Any update is reflected everywhere automatically.

3. **Data Sharing** — Multiple users/applications access the same data simultaneously.
   - *Example:* Both the Accounts dept and Library can access student records.

4. **Data Integrity** — Constraints enforce rules on data.
   - *Example:* `age > 0`, `salary BETWEEN 5000 AND 100000`

5. **Data Security** — Role-based access control.
   - *Example:* Clerk can view but not delete records; Admin has full access.

6. **Atomicity** — Transactions are all-or-nothing (no partial updates).
   - *Example:* Bank transfer either completes fully or rolls back.

7. **Concurrent Access** — DBMS handles simultaneous access without anomalies.

8. **Backup and Recovery** — DBMS provides automated backup and crash recovery.

9. **Data Independence** — Application programs are insulated from changes in storage structure.

10. **Query Language** — Powerful SQL eliminates need for custom programs.

### ❌ Disadvantages

1. **High Cost** — Software, hardware, and training are expensive.
2. **Complexity** — Complex to design, install, and maintain.
3. **Large Size** — DBMS software itself requires significant storage.
4. **Performance Overhead** — For simple applications, file systems may be faster.
5. **Single Point of Failure** — If DBMS crashes, all applications go down.
6. **Skilled Personnel Required** — Need DBAs, SQL experts.

---

## 4. Data Views (Three-Schema Architecture)

DBMS provides **data abstraction** — users see only what is relevant to them. There are **three levels of abstraction**:

```
        ┌────────────────────────────────┐
        │       EXTERNAL LEVEL           │  ← User View (what users see)
        │   View 1 | View 2 | View 3     │
        └────────────────────────────────┘
                        │
        ┌────────────────────────────────┐
        │      CONCEPTUAL LEVEL          │  ← Logical structure of entire DB
        │    (Logical / Community View)  │
        └────────────────────────────────┘
                        │
        ┌────────────────────────────────┐
        │       INTERNAL LEVEL           │  ← Physical storage details
        │   (Physical / Storage Level)   │
        └────────────────────────────────┘
```

### Level 1: External Level (View Level)
- What the **end user** sees.
- Different users see different **views** (subsets) of the database.
- *Example:* A student sees only their own marks and fee status. An accounts clerk sees fee data but not marks.
- Also called **subschema**.

### Level 2: Conceptual Level (Logical Level)
- Describes **what data** is stored and **relationships** among data.
- Hides physical storage details; shows entities, attributes, relationships.
- *Example:* Student(RollNo, Name, Branch, Age), Course(CID, CourseName, Credits)
- Designed by the **DBA**.
- Also called **schema**.

### Level 3: Internal Level (Physical Level)
- Describes **how** data is physically stored on disk.
- Covers storage structures: B-trees, hash indexes, file organization, etc.
- *Example:* Student records stored as a B+ tree indexed on RollNo, with 512-byte blocks.

### Data Independence
- **Logical Data Independence:** Ability to change the conceptual schema without changing external views.
  - *Example:* Adding a new column to a table without breaking existing applications.
- **Physical Data Independence:** Ability to change internal schema (storage) without changing the conceptual schema.
  - *Example:* Switching from sequential file to B-tree without affecting queries.

---

## 5. Database Languages

### 1. Data Definition Language (DDL)
Used to **define the structure** (schema) of the database.

```sql
CREATE TABLE Student (RollNo INT PRIMARY KEY, Name VARCHAR(50), Age INT);
ALTER TABLE Student ADD COLUMN Branch VARCHAR(30);
DROP TABLE Student;
TRUNCATE TABLE Student;
```

DDL commands compile into a **data dictionary** (also called system catalog), which stores metadata.

### 2. Data Manipulation Language (DML)
Used to **manipulate data** — insert, update, delete, retrieve.

- **Procedural DML:** User specifies *what* data AND *how* to get it. (e.g., relational algebra)
- **Non-Procedural DML:** User specifies *what* data, DBMS decides *how*. (e.g., SQL SELECT)

```sql
SELECT Name FROM Student WHERE Age > 20;   -- Retrieval
INSERT INTO Student VALUES (101, 'Ananya', 21);  -- Insertion
UPDATE Student SET Age = 22 WHERE RollNo = 101;  -- Update
DELETE FROM Student WHERE RollNo = 101;          -- Deletion
```

### 3. Data Control Language (DCL)
Controls **access and permissions**.

```sql
GRANT SELECT ON Student TO clerk_user;
REVOKE SELECT ON Student FROM clerk_user;
```

### 4. Transaction Control Language (TCL)
Manages **transactions** and their atomicity.

```sql
COMMIT;      -- Permanently save changes
ROLLBACK;    -- Undo changes since last commit
SAVEPOINT sp1;  -- Set a rollback point
```

### 5. Query Language
A high-level language to retrieve data. SQL is the most common. Others: QBE (Query By Example), Quel (INGRES).

---

## 6. Database Administrator (DBA)

The **DBA** is the person (or team) responsible for managing the database system.

### Responsibilities of a DBA:

| Responsibility | Details |
|---|---|
| **Schema Definition** | Creates the original DB schema using DDL |
| **Storage Structure** | Defines internal schema; decides storage and access methods |
| **Schema Modification** | Alters schema as requirements change |
| **Granting Authorization** | Controls who can access what data |
| **Routine Maintenance** | Backups, monitoring performance, recovery |
| **Security** | Protects DB from unauthorized access |
| **Integrity Constraint Definition** | Defines rules for data validity |
| **Performance Tuning** | Optimizes queries and indexes |

---

## 7. Database Architecture (ANSI/SPARC)

### Centralized Architecture
- Single machine (server) hosts the entire DBMS and all data.
- All clients connect to this central server.
- *Example:* A single Oracle server on which a company's HR database runs.

### Client-Server Architecture
- **Two-Tier:** Client talks directly to DBMS server.
  - Client (application) ↔ DBMS Server (data)
- **Three-Tier:** Application logic is separated into a middle tier.
  - Client (browser) ↔ Application Server (business logic) ↔ DBMS Server (data)
  - *Example:* Web browser → Apache Tomcat → MySQL

### Parallel Architecture
- Multiple processors work in parallel to speed up query execution.
- Types: Shared Memory, Shared Disk, Shared Nothing.
- *Example:* Google Spanner — queries run on thousands of machines in parallel.

### Distributed Architecture
- Data is distributed across multiple geographically separated machines.
- Each site may have its own DBMS (homogeneous or heterogeneous).
- *Example:* A bank with branches in Mumbai, Delhi, Chennai — each with its own DB, but appearing as one unified system.

---

## 8. Data Models

A **data model** is a collection of conceptual tools for describing data, relationships, data semantics, and consistency constraints.

### 1. Relational Model
- Data represented as **tables (relations)**.
- Each table has rows (**tuples**) and columns (**attributes**).
- Based on set theory and first-order predicate logic.
- *Example:*

| RollNo | Name | Branch |
|---|---|---|
| 101 | Priya | CSE |
| 102 | Arjun | ECE |

### 2. Entity-Relationship (ER) Model
- Conceptual model using entities, attributes, and relationships.
- Used in the **design phase** — not directly implemented.
- Represented as **ER diagrams**.

### 3. Object-Based Data Models
- **Object-Oriented Model:** Objects have attributes and methods; supports inheritance and encapsulation.
- **Object-Relational Model:** Extension of relational model with object-like features.

### 4. Semi-Structured Data Model
- Data that doesn't conform to a rigid schema.
- *Example:* XML, JSON — different records can have different attributes.

### 5. Network Model
- Data represented as a **graph** (records and sets).
- Allows many-to-many relationships directly.
- *Example:* CODASYL model — a Student can enroll in many Courses, a Course has many Students.

### 6. Hierarchical Model
- Data organized as a **tree** (parent-child relationships).
- Each child has exactly one parent.
- *Example:* IBM's IMS — a Department has many Employees; Employees belong to only one Department.

### Comparison Table

| Feature | Relational | Hierarchical | Network |
|---|---|---|---|
| Structure | Table | Tree | Graph |
| Relationship | Foreign Key | Parent-Child | Sets |
| Many-to-Many | Via join table | Not supported | Directly supported |
| Query Language | SQL | DL/1 | IDMS |
| Redundancy | Low | High | Medium |

---

## 9. Attribute Types

Attributes describe properties of an entity. There are several types:

### 1. Simple (Atomic) Attribute
- Cannot be divided further.
- *Example:* `RollNo`, `Age`, `PhoneNumber`

### 2. Composite Attribute
- Can be divided into sub-attributes.
- *Example:* `Name` → (`FirstName`, `MiddleName`, `LastName`)
- *Example:* `Address` → (`Street`, `City`, `PinCode`, `State`)

### 3. Single-Valued Attribute
- Holds exactly one value for each entity.
- *Example:* `DateOfBirth`, `AadhaarNumber`

### 4. Multi-Valued Attribute
- Holds multiple values for an entity.
- Represented with **double ellipse** in ER diagram.
- *Example:* `PhoneNumbers` (a person may have home, office, mobile)
- *Example:* `Skills` of an employee (Java, Python, C++)

### 5. Derived Attribute
- Value can be computed from other stored attributes.
- Represented with **dashed ellipse** in ER diagram.
- *Example:* `Age` can be derived from `DateOfBirth` and today's date.
- *Example:* `TotalSalary` = `BasicSalary` + `HRA` + `DA`

### 6. Null Attribute
- Attribute that may not have a value for some entities.
- *Example:* `MiddleName` (not everyone has one), `AlternatePhone`

### 7. Complex Attribute
- Combination of composite and multi-valued.
- *Example:* `Addresses` → multiple addresses, each with (`Street`, `City`, `Zip`)

### 8. Key Attribute
- Uniquely identifies each entity in the entity set.
- Underlined in ER diagrams.
- *Example:* `RollNo` for Student, `EmployeeID` for Employee.

---

## 10. Keys in DBMS

Keys are used to **uniquely identify tuples** in a relation and to **establish relationships** between tables.

### 1. Super Key
- Any set of attributes that can **uniquely identify** a tuple.
- May contain extra (unnecessary) attributes.
- *Example:* For Student(RollNo, Name, Age, Email):
  - `{RollNo}`, `{Email}`, `{RollNo, Name}`, `{RollNo, Age}`, `{RollNo, Name, Age, Email}` — all super keys.

### 2. Candidate Key
- A **minimal super key** — no attribute can be removed while still being unique.
- A relation can have multiple candidate keys.
- *Example:* `{RollNo}` and `{Email}` are candidate keys (both uniquely identify students).

### 3. Primary Key
- The **chosen candidate key** used to uniquely identify tuples.
- Cannot be NULL. Cannot be duplicate.
- *Example:* `RollNo` chosen as primary key for Student table.

### 4. Alternate Key
- Candidate keys that were **not chosen** as the primary key.
- *Example:* `Email` is an alternate key if `RollNo` is the primary key.

### 5. Foreign Key
- An attribute (or set) in one table that **refers to the primary key** of another table.
- Enforces **referential integrity**.
- *Example:*
```sql
Student(RollNo, Name, DeptID)   -- DeptID is FK referencing Department
Department(DeptID, DeptName)     -- DeptID is PK
```

### 6. Composite Key
- A primary key made up of **more than one attribute**.
- *Example:* `Enrollment(StudentID, CourseID)` — neither alone is unique, but together they are.

### 7. Compound Key
- Similar to composite key; a key made of multiple columns.

### 8. Surrogate Key
- A system-generated artificial key, usually an auto-incremented integer.
- *Example:* `OrderID INT AUTO_INCREMENT` in MySQL.

### 9. Natural Key
- A key based on real-world data that naturally exists.
- *Example:* `AadhaarNumber`, `PAN` number.

### Quick Comparison

```
Super Key ⊇ Candidate Key ⊇ Primary Key (one chosen)
                           ⊇ Alternate Keys (rest)
```

---

## 11. ER Diagram

### What is an ER Diagram?
The **Entity-Relationship (ER) Diagram** is a graphical representation of the logical structure of a database. Proposed by **Peter Chen in 1976**.

### Components of ER Diagram

| Component | Shape | Description |
|---|---|---|
| **Entity** | Rectangle | Real-world object (e.g., Student, Course) |
| **Weak Entity** | Double Rectangle | Depends on another entity for identification |
| **Attribute** | Ellipse | Property of an entity |
| **Key Attribute** | Underlined Ellipse | Uniquely identifies an entity |
| **Multivalued Attribute** | Double Ellipse | Multiple values possible |
| **Derived Attribute** | Dashed Ellipse | Computed from other attributes |
| **Composite Attribute** | Ellipse with sub-ellipses | Divisible attribute |
| **Relationship** | Diamond | Association between entities |
| **Weak Relationship** | Double Diamond | Relationship connecting weak entity |
| **Participation** | Lines | Single line = partial, Double line = total |

### Strong vs Weak Entity
- **Strong Entity:** Has its own primary key. *Example:* `Student`, `Department`
- **Weak Entity:** Cannot be uniquely identified without a parent (owner) entity. Has a **partial key** (discriminator).
  - *Example:* `Dependent` of an Employee — a dependent named "Rahul" only makes sense in context of a specific employee.
  - Identified by: **(Owner Entity's PK) + Discriminator**

### Cardinality Constraints (Mapping Cardinalities)

Defines how many instances of one entity are associated with instances of another.

| Type | Description | Example |
|---|---|---|
| **1:1 (One-to-One)** | One entity in A relates to one in B | Employee `manages` Department |
| **1:N (One-to-Many)** | One entity in A relates to many in B | Department `has` many Employees |
| **M:1 (Many-to-One)** | Many in A relate to one in B | Many Employees `work in` one Department |
| **M:N (Many-to-Many)** | Many in A relate to many in B | Students `enroll in` many Courses |

### Participation Constraints
- **Total Participation (double line):** Every entity MUST participate in the relationship.
  - *Example:* Every `Loan` MUST have a `Customer` (Loan has total participation in `Loan-Customer`).
- **Partial Participation (single line):** Some entities may not participate.
  - *Example:* Not every `Customer` has a `Loan`.

### Example: University ER Diagram (Description)

**Entities:** Student, Course, Instructor, Department  
**Relationships:**
- Student `Enrolls` Course — M:N (one student takes many courses, one course has many students)
- Instructor `Teaches` Course — 1:N (one instructor teaches many courses)
- Department `Offers` Course — 1:N
- Student `Belongs to` Department — M:1

**Attributes of Student:** `RollNo` (key, underlined), `Name` (composite: First, Last), `Age` (derived from DOB), `PhoneNumbers` (multivalued)

### ER to Relational Mapping Rules

| ER Construct | Relational Mapping |
|---|---|
| Strong Entity | One table; key attribute becomes primary key |
| Weak Entity | Table with PK of owner + partial key as composite PK |
| 1:1 Relationship | Add FK in either table (prefer total-participation side) |
| 1:N Relationship | Add FK in the "many" side table |
| M:N Relationship | Create a new **junction/bridge table** with both PKs as composite PK |
| Multi-valued Attribute | Create separate table with entity PK + attribute value |
| Composite Attribute | Include individual sub-attributes in the table |
| Derived Attribute | Usually not stored; computed on query |

---

## 12. EER Diagram

### What is EER?
**Enhanced Entity-Relationship (EER) Diagram** extends the ER model with advanced concepts like **generalization, specialization, aggregation, and categorization** to model more complex real-world situations.

### 1. Specialization
- **Top-down** approach: Start with a general entity and define specialized sub-entities.
- A superclass is divided into subclasses based on distinguishing characteristics.
- *Example:* `Employee` is specialized into `Manager`, `Engineer`, `Technician`.
- Sub-classes **inherit** attributes of the superclass.
- Sub-classes can have **additional attributes**.

```
           Employee
          /    |    \
       Manager Eng  Technician
       (Dept)  (Specialization) (ToolType)
```

**Constraints on Specialization:**
- **Disjointness:**
  - *Disjoint (d):* An entity belongs to AT MOST ONE subclass. *Example:* A vehicle is either a Car or a Truck, not both.
  - *Overlapping (o):* An entity can belong to MULTIPLE subclasses. *Example:* A person can be both a Student and an Employee.
- **Completeness:**
  - *Total:* Every superclass entity MUST belong to at least one subclass. (Double line)
  - *Partial:* Some superclass entities may not belong to any subclass. (Single line)

### 2. Generalization
- **Bottom-up** approach: Multiple entities are combined into a more general entity (superclass).
- *Example:* `Car` and `Truck` are generalized into `Vehicle`.
- Reverse of specialization.

### 3. Aggregation
- Used when a **relationship itself** needs to participate in another relationship.
- ER model cannot directly represent this — EER allows treating a relationship as an entity.
- *Example:* An `Employee` `Works_On` a `Project` with a `Manager`. The relationship `Works_On` itself is associated with a `Manager`.

```
[Employee] --- <Works_On> --- [Project]
                    |
               [Manager]
```
The entire `Works_On` relationship box is treated as an entity (surrounded by a rectangle) and linked to `Manager`.

### 4. Union / Category
- A subclass represents a **union of two or more distinct superclasses**.
- *Example:* `Vehicle_Owner` can be either a `Person` OR a `Company`.

---

## 13. Overview of Network and Hierarchical Models

### Hierarchical Model

**Structure:** Data organized as a **tree** — each node has exactly one parent (except root).

```
        University (Root)
       /         \
   Science      Arts
   /    \        |
 CS     ECE    English
 |
 Students
```

**Key Concepts:**
- **Root:** Top-level node (no parent). *Example:* University
- **Segment:** Record type (like a table). Each segment can have child segments.
- **Parent-Child Relationship (PCR):** The fundamental link.
- **Access:** Must traverse from the root; cannot directly jump to a child.

**Advantages:**
- Simple tree structure; easy to understand.
- Fast access for hierarchical data (e.g., org charts, file systems).

**Disadvantages:**
- Cannot represent **many-to-many** relationships naturally.
- **Data redundancy** — if a student is in two departments, store twice.
- Rigid structure — hard to reorganize.
- Deletion anomalies — deleting a parent deletes all children.

**Example DBMS:** IBM's IMS (Information Management System)

---

### Network Model

**Structure:** Data organized as a **graph** — records connected by links (sets). A child can have **multiple parents**.

**Key Concepts:**
- **Record Type:** Like a table (entity type).
- **Set Type:** Named relationship between owner (parent) and member (child). A set has one owner and can have many members.
- **CODASYL Model:** The standard network model specification.
- *Example:* Student can belong to multiple courses, and a course has multiple students — directly modeled.

```
Student ←—— Enrollment ——→ Course
               ↑
           (Set Type)
```

**Advantages:**
- Can directly model **many-to-many** relationships.
- Better performance than hierarchical for complex relationships.
- Less redundancy than hierarchical.

**Disadvantages:**
- **Complex structure** — difficult to design and navigate.
- Application programmer must know the physical data structure (not data-independent).
- Changes to schema require changes to all programs.
- No standard query language (unlike SQL for relational).

**Example DBMS:** IDMS (Integrated Database Management System), IDS

---

### Comparison: Hierarchical vs Network vs Relational

| Feature | Hierarchical | Network | Relational |
|---|---|---|---|
| Structure | Tree | Graph | Table |
| M:N Relationship | ❌ Not directly | ✅ Yes | ✅ Via join table |
| Data Independence | Low | Low | High |
| Query Language | Navigational | Navigational | SQL (declarative) |
| Redundancy | High | Medium | Low |
| Complexity | Medium | High | Low (for users) |
| Example | IMS | IDMS, IDS | Oracle, MySQL |

---

## 14. SQL — Data Definition Language (DDL)

SQL (**Structured Query Language**) is the standard language for relational databases. DDL commands define and modify the structure of database objects.

### CREATE TABLE

```sql
CREATE TABLE Department (
    DeptID    INT           PRIMARY KEY,
    DeptName  VARCHAR(50)   NOT NULL,
    Location  VARCHAR(100)
);

CREATE TABLE Employee (
    EmpID     INT           PRIMARY KEY,
    Name      VARCHAR(100)  NOT NULL,
    Salary    DECIMAL(10,2) DEFAULT 30000.00,
    Age       INT           CHECK (Age >= 18 AND Age <= 65),
    DeptID    INT,
    FOREIGN KEY (DeptID) REFERENCES Department(DeptID)
        ON DELETE SET NULL
        ON UPDATE CASCADE
);
```

### Data Types

| Category | Types | Example |
|---|---|---|
| **Numeric** | INT, BIGINT, SMALLINT, DECIMAL(p,s), FLOAT, REAL | `Salary DECIMAL(10,2)` |
| **Character** | CHAR(n), VARCHAR(n), TEXT | `Name VARCHAR(100)` |
| **Date/Time** | DATE, TIME, DATETIME, TIMESTAMP | `DOB DATE` |
| **Boolean** | BOOLEAN | `IsActive BOOLEAN` |
| **Binary** | BLOB, BINARY | `Photo BLOB` |

### ALTER TABLE

```sql
-- Add a new column
ALTER TABLE Employee ADD COLUMN Email VARCHAR(100);

-- Modify column type
ALTER TABLE Employee MODIFY COLUMN Name VARCHAR(150);

-- Rename column
ALTER TABLE Employee RENAME COLUMN Age TO EmployeeAge;

-- Drop a column
ALTER TABLE Employee DROP COLUMN Email;

-- Add a constraint
ALTER TABLE Employee ADD CONSTRAINT chk_salary CHECK (Salary > 0);

-- Drop a constraint
ALTER TABLE Employee DROP CONSTRAINT chk_salary;
```

### DROP and TRUNCATE

```sql
DROP TABLE Employee;       -- Deletes table structure AND all data permanently
TRUNCATE TABLE Employee;   -- Deletes all rows but keeps table structure; faster than DELETE
```

> **Exam Tip — DROP vs TRUNCATE vs DELETE:**
> | | DROP | TRUNCATE | DELETE |
> |---|---|---|---|
> | Structure removed? | Yes | No | No |
> | Data removed? | Yes | Yes | Yes (selectively) |
> | Can rollback? | No | No (usually) | Yes |
> | WHERE clause? | N/A | No | Yes |
> | Trigger fired? | No | No | Yes |

### CREATE INDEX

```sql
-- Speed up queries on frequently searched columns
CREATE INDEX idx_emp_name ON Employee(Name);

-- Unique index (also enforces uniqueness)
CREATE UNIQUE INDEX idx_emp_email ON Employee(Email);

-- Drop index
DROP INDEX idx_emp_name;
```

---

## 15. SQL — Basic Queries

### SELECT Statement Syntax

```sql
SELECT [DISTINCT] column_list
FROM   table_name
[WHERE condition]
[GROUP BY column_list]
[HAVING condition]
[ORDER BY column_list [ASC|DESC]]
[LIMIT n];
```

### Basic SELECT Examples

**Setup (Tables used throughout):**
```sql
Employee(EmpID, Name, Salary, DeptID, Age, City)
Department(DeptID, DeptName, ManagerID)
Works_On(EmpID, ProjectID, Hours)
Project(ProjectID, ProjName, Budget)
```

**Example 1: Select all employees**
```sql
SELECT * FROM Employee;
```

**Example 2: Select specific columns**
```sql
SELECT Name, Salary FROM Employee;
```

**Example 3: DISTINCT — remove duplicates**
```sql
SELECT DISTINCT DeptID FROM Employee;  -- List all unique departments employees work in
```

**Example 4: Aliases**
```sql
SELECT Name AS EmployeeName, Salary * 12 AS AnnualSalary FROM Employee;
```

### WHERE Clause — Conditions

```sql
-- Comparison operators
SELECT * FROM Employee WHERE Salary > 50000;
SELECT * FROM Employee WHERE Age BETWEEN 25 AND 35;  -- Inclusive
SELECT * FROM Employee WHERE DeptID IN (1, 3, 5);
SELECT * FROM Employee WHERE City NOT IN ('Delhi', 'Mumbai');

-- NULL checks (NEVER use = NULL, always IS NULL)
SELECT * FROM Employee WHERE DeptID IS NULL;
SELECT * FROM Employee WHERE DeptID IS NOT NULL;

-- LIKE pattern matching
SELECT * FROM Employee WHERE Name LIKE 'A%';       -- Starts with A
SELECT * FROM Employee WHERE Name LIKE '%kumar';   -- Ends with kumar
SELECT * FROM Employee WHERE Name LIKE '_a%';      -- Second char is 'a'
SELECT * FROM Employee WHERE Name LIKE '%ra%';     -- Contains 'ra'
-- % = any sequence of characters, _ = exactly one character
```

### ORDER BY

```sql
SELECT Name, Salary FROM Employee ORDER BY Salary DESC;  -- Highest first
SELECT Name, Age FROM Employee ORDER BY Age ASC, Name DESC;  -- Multi-column sort
```

### Aggregate Functions

| Function | Description | Example |
|---|---|---|
| `COUNT(*)` | Count all rows | `SELECT COUNT(*) FROM Employee;` |
| `COUNT(col)` | Count non-NULL values | `SELECT COUNT(DeptID) FROM Employee;` |
| `SUM(col)` | Total | `SELECT SUM(Salary) FROM Employee;` |
| `AVG(col)` | Average | `SELECT AVG(Salary) FROM Employee;` |
| `MAX(col)` | Maximum | `SELECT MAX(Salary) FROM Employee;` |
| `MIN(col)` | Minimum | `SELECT MIN(Age) FROM Employee;` |

### GROUP BY and HAVING

```sql
-- Total salary per department
SELECT DeptID, SUM(Salary) AS TotalSalary
FROM Employee
GROUP BY DeptID;

-- Departments with more than 5 employees
SELECT DeptID, COUNT(*) AS EmpCount
FROM Employee
GROUP BY DeptID
HAVING COUNT(*) > 5;

-- Average salary per city, only cities where avg salary > 40000
SELECT City, AVG(Salary) AS AvgSalary
FROM Employee
GROUP BY City
HAVING AVG(Salary) > 40000
ORDER BY AvgSalary DESC;
```

> **Exam Tip — WHERE vs HAVING:**
> - `WHERE` filters **rows before** grouping (cannot use aggregate functions).
> - `HAVING` filters **groups after** GROUP BY (can use aggregate functions).
> - *Example:* `WHERE Salary > 30000` first filters employees, then `HAVING COUNT(*) > 3` filters groups.

---

## 16. SQL — Complex Queries

### JOINS

Joins combine rows from two or more tables based on a related column.

#### INNER JOIN (Most common)
Returns rows where there is a match in BOTH tables.

```sql
SELECT E.Name, D.DeptName
FROM Employee E
INNER JOIN Department D ON E.DeptID = D.DeptID;
```
*Returns only employees who are assigned to an existing department.*

#### LEFT OUTER JOIN
Returns ALL rows from left table + matching rows from right (NULLs for non-matches).

```sql
SELECT E.Name, D.DeptName
FROM Employee E
LEFT JOIN Department D ON E.DeptID = D.DeptID;
```
*Returns ALL employees, even those without a department (DeptName will be NULL).*

#### RIGHT OUTER JOIN
Returns ALL rows from right table + matching from left.

```sql
SELECT E.Name, D.DeptName
FROM Employee E
RIGHT JOIN Department D ON E.DeptID = D.DeptID;
```
*Returns ALL departments, even those with no employees.*

#### FULL OUTER JOIN
Returns ALL rows from BOTH tables (NULLs where no match).

```sql
SELECT E.Name, D.DeptName
FROM Employee E
FULL OUTER JOIN Department D ON E.DeptID = D.DeptID;
```

#### CROSS JOIN (Cartesian Product)
Every row of left combined with every row of right. Result has `m × n` rows.

```sql
SELECT E.Name, P.ProjName
FROM Employee E CROSS JOIN Project P;
```
*If 10 employees and 5 projects: 50 rows returned.*

#### SELF JOIN
Joining a table with itself. Useful for hierarchical data.

```sql
-- Find employee and their manager (both in Employee table)
SELECT E.Name AS Employee, M.Name AS Manager
FROM Employee E
JOIN Employee M ON E.ManagerID = M.EmpID;
```

### Subqueries (Nested Queries)

A query **inside another query**. The inner query runs first.

#### Simple Subquery (Single Value)

```sql
-- Employees earning more than the average salary
SELECT Name, Salary
FROM Employee
WHERE Salary > (SELECT AVG(Salary) FROM Employee);
```

#### Subquery with IN

```sql
-- Employees who work on project 'Apollo'
SELECT Name FROM Employee
WHERE EmpID IN (
    SELECT EmpID FROM Works_On
    WHERE ProjectID = (SELECT ProjectID FROM Project WHERE ProjName = 'Apollo')
);
```

#### Correlated Subquery
The inner query references a column from the outer query (runs once per outer row — slower).

```sql
-- Employees earning more than the average salary of their OWN department
SELECT E1.Name, E1.Salary, E1.DeptID
FROM Employee E1
WHERE E1.Salary > (
    SELECT AVG(E2.Salary)
    FROM Employee E2
    WHERE E2.DeptID = E1.DeptID  -- Correlated: uses E1.DeptID
);
```

#### EXISTS / NOT EXISTS
Returns true if the subquery returns at least one row.

```sql
-- Employees who work on at least one project
SELECT Name FROM Employee E
WHERE EXISTS (
    SELECT 1 FROM Works_On W WHERE W.EmpID = E.EmpID
);

-- Employees with NO project assigned
SELECT Name FROM Employee E
WHERE NOT EXISTS (
    SELECT 1 FROM Works_On W WHERE W.EmpID = E.EmpID
);
```

#### ALL / ANY (SOME)

```sql
-- Employees earning more than ALL employees in DeptID = 3
SELECT Name FROM Employee
WHERE Salary > ALL (SELECT Salary FROM Employee WHERE DeptID = 3);

-- Employees earning more than at least ONE employee in DeptID = 3
SELECT Name FROM Employee
WHERE Salary > ANY (SELECT Salary FROM Employee WHERE DeptID = 3);
```

### Set Operations

```sql
-- UNION: All rows from both (removes duplicates)
SELECT Name FROM Employee WHERE DeptID = 1
UNION
SELECT Name FROM Employee WHERE Salary > 60000;

-- UNION ALL: All rows including duplicates
SELECT DeptID FROM Employee
UNION ALL
SELECT DeptID FROM Department;

-- INTERSECT: Only rows in BOTH results
SELECT EmpID FROM Works_On WHERE ProjectID = 1
INTERSECT
SELECT EmpID FROM Works_On WHERE ProjectID = 2;

-- EXCEPT / MINUS: Rows in first but NOT in second
SELECT EmpID FROM Employee
EXCEPT
SELECT EmpID FROM Works_On;  -- Employees with no project
```

> **Exam Tip:** All set operations require the same number of columns with compatible data types in both SELECT statements.

### String Functions

```sql
SELECT UPPER(Name), LOWER(City), LENGTH(Name) FROM Employee;
SELECT CONCAT(Name, ' - ', City) AS NameCity FROM Employee;
SELECT SUBSTRING(Name, 1, 3) FROM Employee;  -- First 3 chars
SELECT TRIM('  Hello  ');  -- Removes leading/trailing spaces
SELECT REPLACE(City, 'Delhi', 'New Delhi') FROM Employee;
```

### Date Functions

```sql
SELECT NOW(), CURDATE(), YEAR(DOB), MONTH(DOB), DAY(DOB) FROM Employee;
SELECT DATEDIFF(NOW(), DOB) / 365 AS Age FROM Employee;
```

---

## 17. Insert, Delete, and Update in SQL

### INSERT

```sql
-- Insert with all columns (values must match column order)
INSERT INTO Employee VALUES (101, 'Ananya Sharma', 55000.00, 2, 28, 'Bangalore');

-- Insert with specific columns (others get DEFAULT or NULL)
INSERT INTO Employee (EmpID, Name, Salary, DeptID)
VALUES (102, 'Rohan Mehta', 48000.00, 1);

-- Insert multiple rows at once
INSERT INTO Employee (EmpID, Name, Salary, DeptID) VALUES
(103, 'Priya Joshi', 62000.00, 3),
(104, 'Kartik Nair', 41000.00, 2),
(105, 'Sneha Rao', 75000.00, 1);

-- Insert from another table (INSERT INTO ... SELECT)
INSERT INTO HighEarners (EmpID, Name, Salary)
SELECT EmpID, Name, Salary
FROM Employee
WHERE Salary > 70000;
```

### UPDATE

```sql
-- Update salary of a specific employee
UPDATE Employee SET Salary = 60000 WHERE EmpID = 101;

-- Update multiple columns
UPDATE Employee
SET Salary = Salary * 1.10, City = 'Hyderabad'
WHERE DeptID = 2;

-- Update with subquery
UPDATE Employee
SET Salary = Salary * 1.15
WHERE DeptID = (SELECT DeptID FROM Department WHERE DeptName = 'Research');

-- ⚠️ UPDATE without WHERE updates ALL rows — use with caution!
UPDATE Employee SET City = 'Unknown';  -- Updates EVERY employee's city
```

### DELETE

```sql
-- Delete a specific record
DELETE FROM Employee WHERE EmpID = 104;

-- Delete based on condition
DELETE FROM Employee WHERE Salary < 25000;

-- Delete with subquery
DELETE FROM Employee
WHERE DeptID IN (SELECT DeptID FROM Department WHERE Location = 'Closed');

-- ⚠️ DELETE without WHERE deletes ALL rows (but keeps table structure)
DELETE FROM Employee;

-- Use TRUNCATE instead for clearing entire table (faster, no logs)
TRUNCATE TABLE Employee;
```

> **Referential Integrity with DELETE:**
> If `Employee.DeptID` references `Department.DeptID`:
> - `ON DELETE CASCADE` — deleting a department deletes all its employees.
> - `ON DELETE SET NULL` — deletes department, sets DeptID to NULL in Employee.
> - `ON DELETE RESTRICT` — prevents deletion if employees exist in that department.

---

## 18. Views in SQL

### What is a View?
A **View** is a **virtual table** defined by a SQL query. It does not store data physically — it stores the query definition. When you query a view, the underlying query is executed.

### Creating a View

```sql
-- Simple view: employees in department 1
CREATE VIEW Dept1Employees AS
SELECT EmpID, Name, Salary
FROM Employee
WHERE DeptID = 1;

-- View with JOIN
CREATE VIEW EmpDeptView AS
SELECT E.EmpID, E.Name, E.Salary, D.DeptName
FROM Employee E
JOIN Department D ON E.DeptID = D.DeptID;

-- View with aggregation
CREATE VIEW DeptSalarySummary AS
SELECT DeptID, COUNT(*) AS EmpCount, AVG(Salary) AS AvgSalary
FROM Employee
GROUP BY DeptID;
```

### Using a View (Just like a table)

```sql
SELECT * FROM Dept1Employees;
SELECT * FROM EmpDeptView WHERE Salary > 50000;
SELECT DeptID, AvgSalary FROM DeptSalarySummary WHERE AvgSalary > 45000;
```

### Modifying and Dropping Views

```sql
-- Modify view definition
CREATE OR REPLACE VIEW Dept1Employees AS
SELECT EmpID, Name, Salary, City
FROM Employee
WHERE DeptID = 1 AND Salary > 40000;

-- Drop view
DROP VIEW Dept1Employees;
```

### Updatable Views
A view is **updatable** (INSERT/UPDATE/DELETE allowed) only if:
- Based on a single table.
- Does not contain DISTINCT, GROUP BY, HAVING, aggregate functions, or UNION.
- Does not contain subqueries in the SELECT list.

```sql
-- This view is updatable
CREATE VIEW SimpleEmpView AS
SELECT EmpID, Name, Salary FROM Employee;

UPDATE SimpleEmpView SET Salary = 70000 WHERE EmpID = 101;  -- ✅ Works

-- This view is NOT updatable (has aggregate)
CREATE VIEW DeptAvg AS SELECT DeptID, AVG(Salary) FROM Employee GROUP BY DeptID;
UPDATE DeptAvg SET AVG(Salary) = 60000;  -- ❌ Error
```

### WITH CHECK OPTION

```sql
-- Prevents inserting/updating rows that don't satisfy the view's WHERE clause
CREATE VIEW HighSalaryEmp AS
SELECT * FROM Employee WHERE Salary > 60000
WITH CHECK OPTION;

INSERT INTO HighSalaryEmp VALUES (110, 'Test', 40000, 1, 25, 'Delhi');
-- ❌ Error! 40000 < 60000, violates WITH CHECK OPTION
```

### Advantages of Views

1. **Security:** Hide sensitive columns. *Example:* Create a view that excludes the `Salary` column for non-HR users.
2. **Simplicity:** Complex joins/calculations stored in view — users run simple `SELECT * FROM viewname`.
3. **Logical Data Independence:** Applications using the view are unaffected by table changes (as long as view is updated).
4. **Consistency:** Same logic reused across applications.

---

## 19. Constraints and Assertions

### Types of Constraints in SQL

#### 1. NOT NULL
```sql
CREATE TABLE Student (
    RollNo INT NOT NULL,
    Name   VARCHAR(100) NOT NULL,
    Email  VARCHAR(100)  -- Email can be NULL
);
```
Ensures a column **cannot have NULL values**.

#### 2. UNIQUE
```sql
CREATE TABLE Student (
    RollNo INT PRIMARY KEY,
    Email  VARCHAR(100) UNIQUE,  -- No two students can share an email
    AadhaarNo CHAR(12) UNIQUE
);
```
Ensures all values in a column are distinct. (Unlike PRIMARY KEY, UNIQUE can have one NULL.)

#### 3. PRIMARY KEY
```sql
-- Single column PK
CREATE TABLE Student (RollNo INT PRIMARY KEY, Name VARCHAR(100));

-- Composite PK
CREATE TABLE Enrollment (
    StudentID INT,
    CourseID  INT,
    Grade     CHAR(2),
    PRIMARY KEY (StudentID, CourseID)
);
```

#### 4. FOREIGN KEY
```sql
CREATE TABLE Employee (
    EmpID   INT PRIMARY KEY,
    DeptID  INT,
    FOREIGN KEY (DeptID) REFERENCES Department(DeptID)
        ON DELETE CASCADE    -- Delete employee if dept is deleted
        ON UPDATE CASCADE    -- Update DeptID if it changes in Department
);
```

**Referential Actions:**
- `CASCADE` — propagate the change.
- `SET NULL` — set FK to NULL.
- `SET DEFAULT` — set FK to its default value.
- `RESTRICT / NO ACTION` — prevent the change if violation would occur.

#### 5. CHECK
```sql
CREATE TABLE Employee (
    EmpID  INT PRIMARY KEY,
    Age    INT CHECK (Age >= 18 AND Age <= 65),
    Gender CHAR(1) CHECK (Gender IN ('M', 'F', 'O')),
    Salary DECIMAL(10,2) CHECK (Salary > 0)
);
```

#### 6. DEFAULT
```sql
CREATE TABLE Orders (
    OrderID   INT PRIMARY KEY,
    OrderDate DATE DEFAULT CURRENT_DATE,
    Status    VARCHAR(20) DEFAULT 'Pending',
    Quantity  INT DEFAULT 1
);
```

### Assertions

An **assertion** is a general constraint that can involve multiple tables. Unlike CHECK (limited to one table), assertions are database-wide constraints.

```sql
-- Assertion: No department should have more than 50 employees
CREATE ASSERTION max_emp_per_dept
CHECK (NOT EXISTS (
    SELECT DeptID
    FROM Employee
    GROUP BY DeptID
    HAVING COUNT(*) > 50
));

-- Assertion: Total budget of all projects must not exceed 10 million
CREATE ASSERTION total_budget_limit
CHECK (
    (SELECT SUM(Budget) FROM Project) <= 10000000
);
```

> **Note:** Assertions are defined in SQL standard but many DBMS (MySQL, PostgreSQL partially) do not implement them. They are tested on every INSERT/UPDATE/DELETE and can be expensive. In practice, **triggers** are used to implement assertion-like behavior.

### Triggers (Bonus — Closely Related)

A **trigger** is an automatic action executed in response to INSERT, UPDATE, or DELETE.

```sql
-- Trigger to log salary changes
CREATE TRIGGER log_salary_change
AFTER UPDATE OF Salary ON Employee
FOR EACH ROW
BEGIN
    INSERT INTO SalaryLog(EmpID, OldSalary, NewSalary, ChangeDate)
    VALUES (OLD.EmpID, OLD.Salary, NEW.Salary, NOW());
END;
```

---

## 🔑 Quick Revision — Exam Cheat Sheet

### Must-Know Definitions (1-2 marks)
- **DBMS:** Software for managing databases; provides data storage, retrieval, and manipulation.
- **Schema:** Overall logical structure/design of the database.
- **Instance:** The actual data stored at a particular moment (snapshot).
- **Metadata:** Data about data; stored in the data dictionary/catalog.
- **Data Independence:** Ability to modify schema without affecting other levels.
- **Tuple:** A row in a relational table.
- **Attribute:** A column in a relational table.
- **Relation:** A table in a relational model.
- **Degree:** Number of attributes (columns) in a relation.
- **Cardinality:** Number of tuples (rows) in a relation.

### SQL Clause Execution Order (Important!)
```
FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT
```

### Key Comparisons (Frequently Examined)
- **DDL vs DML:** DDL defines structure (CREATE, ALTER, DROP); DML manipulates data (SELECT, INSERT, UPDATE, DELETE).
- **WHERE vs HAVING:** WHERE filters rows; HAVING filters groups.
- **CHAR vs VARCHAR:** CHAR is fixed-length (pads with spaces); VARCHAR is variable-length (no padding).
- **DELETE vs TRUNCATE vs DROP:** DELETE removes rows (can rollback, fires triggers); TRUNCATE removes all rows fast (no rollback usually); DROP removes table entirely.
- **View vs Table:** View is virtual (no storage); Table is physical (stores data).
- **Primary Key vs Unique Key:** PK — one per table, no NULL; UNIQUE — multiple per table, one NULL allowed.
- **ER vs EER:** ER has entities, attributes, relationships; EER adds generalization, specialization, aggregation.
- **Specialization vs Generalization:** Specialization is top-down (split); Generalization is bottom-up (merge).

---

*Notes compiled for DBMS Unit 1 — Good luck with your exam! 🎯*
