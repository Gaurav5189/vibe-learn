# Data Mining & Data Warehousing — Exam Preparation Notes
### Topics: DBMS Schemas · Data Warehouse & Architecture · OLAP · KDD · KNN · Decision Tree

---

## TABLE OF CONTENTS
1. [DBMS Schemas for Decision Support](#1-dbms-schemas-for-decision-support)
2. [Data Warehouse — Concepts & Architecture](#2-data-warehouse--concepts--architecture)
3. [OLAP — Online Analytical Processing](#3-olap--online-analytical-processing)
4. [KDD — Knowledge Discovery in Databases](#4-kdd--knowledge-discovery-in-databases)
5. [KNN — K-Nearest Neighbors](#5-knn--k-nearest-neighbors)
6. [Decision Tree Induction](#6-decision-tree-induction)
7. [Quick Revision Cheatsheet](#7-quick-revision-cheatsheet)

---

## 1. DBMS Schemas for Decision Support

### What is a Schema?
A **schema** is the logical structure/blueprint of a database — it defines how data is organized into tables and how tables are related. In decision support systems (like data warehouses), specific schema designs are used to optimize **read-heavy analytical queries** rather than transactional updates.

---

### 1.1 Star Schema

The **Star Schema** is the simplest and most common schema used in data warehouses.

**Structure:**
- One central **Fact Table** (stores measurable, quantitative data — called *measures*)
- Multiple **Dimension Tables** surrounding it (store descriptive attributes)
- The shape resembles a star ⭐

**Fact Table** contains:
- Foreign keys pointing to each dimension table
- Numerical measures (e.g., sales amount, quantity sold, profit)

**Dimension Tables** contain:
- Descriptive attributes (e.g., product name, customer city, date)
- Are **not normalized** (data is deliberately denormalized for query speed)

#### Example — Retail Sales Star Schema

```
         [Time_Dim]
              |
[Product_Dim]--[SALES_FACT]--[Customer_Dim]
              |
         [Store_Dim]
```

**SALES_FACT table:**
| time_id | product_id | customer_id | store_id | sales_amount | quantity |
|---------|------------|-------------|----------|--------------|----------|
| T001    | P01        | C100        | S05      | 5000         | 10       |

**Product_Dim table:**
| product_id | product_name | category   | brand   |
|------------|--------------|------------|---------|
| P01        | Laptop       | Electronics| Dell    |

**Advantages:**
- Simple to understand and design
- Faster query performance (fewer joins needed)
- Easy for business users to navigate

**Disadvantages:**
- Data redundancy in dimension tables (denormalized)
- Harder to maintain when dimensions change frequently

---

### 1.2 Snowflake Schema

The **Snowflake Schema** is an extension of the Star Schema where dimension tables are **normalized** — split into multiple related tables.

**Structure:**
- One central Fact Table
- Dimension tables are further broken down into sub-dimension tables
- The shape resembles a snowflake ❄️

#### Example

In a Star Schema, Product_Dim might look like:

| product_id | product_name | category   | category_manager |
|------------|--------------|------------|------------------|
| P01        | Laptop       | Electronics| John             |

In a **Snowflake Schema**, this becomes:

**Product_Dim:**
| product_id | product_name | category_id |
|------------|--------------|-------------|
| P01        | Laptop       | C01         |

**Category_Dim:**
| category_id | category   | category_manager |
|-------------|------------|------------------|
| C01         | Electronics| John             |

**Advantages:**
- Reduces data redundancy (normalized)
- Saves storage space

**Disadvantages:**
- More complex queries (more joins required)
- Slower query performance compared to Star Schema

---

### 1.3 Galaxy Schema (Fact Constellation Schema)

The **Galaxy Schema** (also called Fact Constellation) uses **multiple Fact Tables** sharing some dimension tables.

**When to use:** When multiple business processes need to be analyzed (e.g., Sales AND Purchases in the same warehouse).

```
[Time_Dim]   [Product_Dim]
     \          /     \
  [SALES_FACT]   [PURCHASE_FACT]
     /                  \
[Customer_Dim]       [Supplier_Dim]
```

**Advantages:**
- Models complex business environments
- Shared dimensions ensure consistent reporting

**Disadvantages:**
- Complex to design and maintain
- Harder for end-users to understand

---

### 1.4 Comparison Table (Exam Favourite!)

| Feature           | Star Schema     | Snowflake Schema | Galaxy Schema       |
|-------------------|-----------------|------------------|---------------------|
| Normalization     | Denormalized    | Normalized       | Mixed               |
| No. of Fact Tables| One             | One              | Multiple            |
| Query Performance | Fast            | Slower           | Complex             |
| Storage           | More space      | Less space       | Moderate            |
| Complexity        | Simple          | Moderate         | High                |
| Redundancy        | High            | Low              | Low to Moderate     |

> **Exam Tip:** "A Star Schema is preferred when query speed matters. A Snowflake Schema is preferred when storage efficiency matters."

---

## 2. Data Warehouse — Concepts & Architecture

### 2.1 What is a Data Warehouse?

A **Data Warehouse (DW)** is a large, centralized repository of integrated data collected from multiple heterogeneous sources, designed specifically for **analytical reporting and decision support** — NOT for day-to-day transactions.

**Bill Inmon's Definition:** *"A subject-oriented, integrated, time-variant, and non-volatile collection of data in support of management's decision-making process."*

---

### 2.2 Key Characteristics (the 4 pillars — very commonly asked!)

| Characteristic | Meaning | Example |
|----------------|---------|---------|
| **Subject-Oriented** | Organized around major subjects of the business | Customer, Sales, Product (not around applications like Order Entry) |
| **Integrated** | Data from multiple sources merged consistently | "Male/Female" from one system + "M/F" from another → unified as "M/F" |
| **Time-Variant** | Data is stored with time stamps; historical data is kept | Sales data for the last 10 years, not just current records |
| **Non-Volatile** | Once loaded, data is not deleted or changed (read-only) | Old sales records are never overwritten; new records are appended |

---

### 2.3 Data Warehouse vs. OLTP

| Feature              | OLTP (Operational DB)       | Data Warehouse (OLAP)        |
|----------------------|-----------------------------|------------------------------|
| Purpose              | Daily transactions           | Decision support / Analysis  |
| Data                 | Current, up-to-date          | Historical                   |
| Operations           | INSERT, UPDATE, DELETE       | SELECT (read-mostly)         |
| Query type           | Simple, frequent             | Complex, ad-hoc              |
| Users                | Clerks, customers            | Analysts, managers           |
| Size                 | MB to GB                     | GB to TB                     |
| Data model           | ER / Relational              | Star / Snowflake             |
| Normalization        | Highly normalized            | Denormalized                 |
| Response time        | Milliseconds                 | Seconds to minutes           |

---

### 2.4 Data Warehouse Architecture

A standard DW architecture has the following layers:

```
[Source Systems]
      |
      ↓
[Data Staging Area / ETL]
      |
      ↓
[Data Warehouse (Central Repository)]
      |
   ┌──┴──┐
   ↓     ↓
[Data Marts]  [Metadata Repository]
      |
      ↓
[Reporting / OLAP / Mining Tools]
      |
      ↓
[End Users / Decision Makers]
```

**Layer-by-layer explanation:**

**1. Source Systems (Bottom Layer)**
- Operational databases (OLTP), flat files, external data, spreadsheets
- Examples: ERP systems, CRM systems, sales databases, web logs

**2. ETL Layer (Extract, Transform, Load)**
- **Extract:** Pull data from all source systems
- **Transform:** Clean, standardize, and integrate the data
  - Handle missing values, resolve naming conflicts, apply business rules
  - Example: Convert all dates to a standard format (DD-MM-YYYY)
- **Load:** Push transformed data into the data warehouse

**3. Data Warehouse (Central Repository)**
- The cleaned, integrated, historical data store
- Organized using star/snowflake schemas
- Optimized for complex analytical queries

**4. Data Marts**
- **Subsets** of the data warehouse focused on a specific business area
- Example: A Sales Data Mart contains only sales-related data; a Finance Data Mart contains only financial data
- Faster query response for specific departments

**5. Metadata Repository**
- Data about the data (descriptions, lineage, transformation rules)
- Helps users understand what data exists and where it came from

**6. Front-End Tools**
- Reporting tools, OLAP tools, data mining tools, query tools
- Used by analysts and executives to make decisions

---

### 2.5 Three-Tier Architecture (Most Commonly Tested!)

```
         TIER 3: Client / Presentation
         (OLAP tools, dashboards, reports)
                     ↑
         TIER 2: Application / OLAP Server
         (ROLAP, MOLAP, HOLAP engines)
                     ↑
         TIER 1: Data Warehouse Server
         (Relational DBMS with star/snowflake schema)
```

- **Tier 1 (Bottom):** The actual data warehouse database server
- **Tier 2 (Middle):** The OLAP server that processes analytical queries
- **Tier 3 (Top):** Client-side tools for visualization and reporting

---

### 2.6 Metadata

**Metadata** = "Data about Data"

Types of metadata in a data warehouse:

| Type | Description | Example |
|------|-------------|---------|
| **Technical Metadata** | Describes structure and storage | Table names, column types, indexes |
| **Business Metadata** | Describes meaning in business terms | "Revenue = Gross Sales - Returns" |
| **Operational Metadata** | Describes ETL processes | When data was last loaded, source of each record |

> **Exam Tip:** Metadata is the "road map" of the data warehouse — without it, users wouldn't know what data exists or how to use it.

---

## 3. OLAP — Online Analytical Processing

### 3.1 What is OLAP?

**OLAP (Online Analytical Processing)** is a category of software tools that enable users to analyze multidimensional data **interactively** from multiple perspectives. It is designed for **complex queries, data aggregation, and trend analysis** — the analytical counterpart to OLTP.

The core concept of OLAP is the **Data Cube**.

---

### 3.2 The Data Cube

A **Data Cube** is a multidimensional representation of data. Each **dimension** represents an attribute (e.g., Time, Product, Location), and the **cells** contain numerical measures (e.g., sales amount).

**Example — 3D Data Cube:**

Imagine tracking sales across:
- **Dimension 1:** Time (Q1, Q2, Q3, Q4)
- **Dimension 2:** Product (Laptop, Phone, Tablet)
- **Dimension 3:** Location (Delhi, Mumbai, Patna)

Each cell in the cube holds the total sales for a specific (Time, Product, Location) combination.

```
           Q1     Q2     Q3     Q4
          /      /      /      /
Laptop  [5000] [6000] [5500] [7000]   ← Delhi
Phone   [3000] [3500] [4000] [3200]
Tablet  [2000] [2500] [1800] [2200]
```

---

### 3.3 OLAP Operations (The Core — Always in Exams!)

#### 1. Roll-Up (Drill-Up)
- **Moves from detailed data → summary data** by climbing up a concept hierarchy or reducing dimensions
- Example: From **Monthly Sales → Quarterly Sales → Annual Sales**
- Or: From **City-level → State-level → Country-level**

#### 2. Drill-Down (Roll-Down)
- **Opposite of Roll-Up** — moves from summary data → detailed data
- Example: From **Annual Sales → Quarterly Sales → Monthly Sales → Daily Sales**

#### 3. Slice
- **Selects one dimension** and fixes its value, creating a 2D cross-section (a "slice") of the cube
- Example: Fixing Time = "Q1" gives you a table of [Product × Location] sales for Q1 only

#### 4. Dice
- **Selects a sub-cube** by specifying conditions on two or more dimensions
- Example: Where Time ∈ {Q1, Q2} AND Product ∈ {Laptop, Phone}

#### 5. Pivot (Rotate)
- **Rotates the data cube** to provide an alternate presentation/view
- Example: Swapping rows and columns — viewing Products on the X-axis instead of the Y-axis

> **Memory Trick:**
> - **Roll-Up** = Zoom Out (less detail)
> - **Drill-Down** = Zoom In (more detail)
> - **Slice** = Cut (one value for one dimension)
> - **Dice** = Sub-cube (range of values across dimensions)
> - **Pivot** = Rotate view

---

### 3.4 Types of OLAP

| Type | Full Form | Storage | Speed | Scalability |
|------|-----------|---------|-------|-------------|
| **MOLAP** | Multidimensional OLAP | Multidimensional arrays (proprietary) | Fastest | Limited |
| **ROLAP** | Relational OLAP | Relational DB (star/snowflake) | Slower | High |
| **HOLAP** | Hybrid OLAP | Both (detail in RDBMS, aggregates in MD arrays) | Balanced | Balanced |

**MOLAP:** Pre-computes and stores aggregations in multidimensional arrays. Very fast but storage-hungry.

**ROLAP:** Generates SQL dynamically against a relational database. Slower but scales to large data volumes.

**HOLAP:** Best of both worlds — summary/aggregate data stored in multidimensional format (fast access), raw detail stored relationally.

---

### 3.5 Data Generalization by Attribute-Oriented Induction (AOI)

**AOI** is a technique to generalize (summarize) data by replacing low-level concepts with higher-level ones using a **concept hierarchy**.

**How it works:**
1. Start with a relation (query result)
2. Apply a concept hierarchy to each attribute
3. Remove attributes with too many distinct values
4. Count/aggregate tuples at each generalization level

**Concept Hierarchy Example:**

```
Street → City → State → Country   (Location hierarchy)
Day → Week → Month → Quarter → Year   (Time hierarchy)
Laptop → Computer → Electronics   (Product hierarchy)
```

**Example:**

Original data:
| Customer | City    | Product | Sales |
|----------|---------|---------|-------|
| Alice    | Patna   | Laptop  | 5000  |
| Bob      | Mumbai  | Phone   | 3000  |
| Carol    | Patna   | Tablet  | 2000  |

After AOI (generalizing City → State):
| State | Product  | Total_Sales |
|-------|----------|-------------|
| Bihar | Computer | 7000        |
| MH    | Phone    | 3000        |

The system automatically generalizes from specific instances to more general descriptions.

---

## 4. KDD — Knowledge Discovery in Databases

### 4.1 What is KDD?

**KDD (Knowledge Discovery in Databases)** is the **overall process** of discovering useful, valid, novel, and understandable patterns from large databases.

> **Key distinction:** Data Mining is just **one step** in the KDD process — KDD is the complete pipeline.

---

### 4.2 The KDD Process (Step-by-Step)

```
[Raw Data]
    ↓
1. DATA SELECTION
    ↓
2. DATA PREPROCESSING (Cleaning)
    ↓
3. DATA TRANSFORMATION
    ↓
4. DATA MINING ← (This is what most people call "data mining")
    ↓
5. INTERPRETATION & EVALUATION
    ↓
[Knowledge]
```

**Step 1 — Data Selection:**
- Choose the relevant subset of data from the database that is relevant to the analysis task
- Example: From a hospital database with 50 tables, select only Patient, Diagnosis, and Medication tables for disease prediction

**Step 2 — Data Preprocessing (Cleaning):**
- Handle noisy, missing, or inconsistent data
- Techniques:
  - **Missing values:** Fill with mean/median, or remove the record
  - **Noisy data:** Binning, regression, clustering to smooth values
  - **Inconsistencies:** Resolve conflicts between sources

**Step 3 — Data Transformation:**
- Convert data into forms appropriate for mining
- Techniques:
  - **Normalization:** Scale values to [0,1] or [-1,1] — important for distance-based algorithms like KNN
  - **Aggregation:** Summarize data (daily → monthly)
  - **Discretization:** Convert continuous to categorical (Age → Young/Middle/Old)
  - **Attribute construction:** Create new attributes (BMI = weight/height²)

**Step 4 — Data Mining:**
- Apply algorithms to extract patterns
- Tasks include:
  - **Classification** (Decision Tree, KNN, Naive Bayes) — predicting class labels
  - **Clustering** (K-Means) — grouping similar records
  - **Association Rule Mining** (Apriori, FP-Growth) — finding co-occurrence patterns
  - **Regression** — predicting numerical values
  - **Anomaly Detection** — finding outliers

**Step 5 — Interpretation & Evaluation:**
- Evaluate the discovered patterns for:
  - **Validity:** Is the pattern statistically significant?
  - **Novelty:** Is it new/interesting?
  - **Usefulness:** Can it support decisions?
  - **Understandability:** Can humans interpret it?
- Patterns that don't meet criteria are discarded; the process may loop back

---

### 4.3 Application Areas of Data Mining

| Domain | Application |
|--------|-------------|
| **Retail / E-commerce** | Market basket analysis ("customers who bought X also bought Y") |
| **Banking & Finance** | Credit card fraud detection, loan risk assessment |
| **Healthcare** | Disease prediction, drug effectiveness analysis |
| **Telecommunications** | Customer churn prediction |
| **Marketing** | Customer segmentation, targeted advertising |
| **Web** | Web usage mining, recommendation systems |

---

## 5. KNN — K-Nearest Neighbors

### 5.1 What is KNN?

**K-Nearest Neighbors (KNN)** is a **simple, instance-based, lazy learning** classification algorithm. It classifies a new data point based on the **majority class among its K nearest neighbors** in the feature space.

- **Lazy learner:** No explicit model is built during training — all computation is deferred to query time
- **Instance-based:** Stores all training instances and compares new points against them
- **Non-parametric:** Makes no assumptions about the data distribution

---

### 5.2 How KNN Works — Step by Step

1. **Choose K** (the number of neighbors to consider)
2. **Calculate distance** from the new point to all training points
3. **Sort** training points by distance (ascending)
4. **Select the K nearest** training points
5. **Majority vote:** Assign the class that appears most among the K neighbors

---

### 5.3 Distance Metrics

**Euclidean Distance (most common):**
$$d(x, y) = \sqrt{\sum_{i=1}^{n} (x_i - y_i)^2}$$

**Manhattan Distance:**
$$d(x, y) = \sum_{i=1}^{n} |x_i - y_i|$$

---

### 5.4 Worked Example (Exam Style)

**Training Data — Classify whether a patient has Diabetes (Yes/No):**

| ID | Age | BMI  | Class    |
|----|-----|------|----------|
| 1  | 25  | 22.0 | No       |
| 2  | 45  | 30.0 | Yes      |
| 3  | 35  | 25.0 | No       |
| 4  | 50  | 35.0 | Yes      |
| 5  | 40  | 28.0 | Yes      |

**New Patient:** Age = 38, BMI = 27.0, K = 3

**Step 1 — Calculate Euclidean Distance from new patient to each training point:**

| ID | Distance = √((Age_diff)² + (BMI_diff)²)             | Distance |
|----|------------------------------------------------------|----------|
| 1  | √((38-25)² + (27-22)²) = √(169 + 25) = √194        | 13.93    |
| 2  | √((38-45)² + (27-30)²) = √(49 + 9) = √58           | 7.62     |
| 3  | √((38-35)² + (27-25)²) = √(9 + 4) = √13            | 3.61     |
| 4  | √((38-50)² + (27-35)²) = √(144 + 64) = √208        | 14.42    |
| 5  | √((38-40)² + (27-28)²) = √(4 + 1) = √5             | 2.24     |

**Step 2 — Sort by distance and pick K=3 nearest:**

| Rank | ID | Distance | Class |
|------|----|----------|-------|
| 1    | 5  | 2.24     | Yes   |
| 2    | 3  | 3.61     | No    |
| 3    | 2  | 7.62     | Yes   |

**Step 3 — Majority vote:**
- Yes: 2 votes
- No: 1 vote

**Result → New patient is classified as: Diabetes = Yes**

---

### 5.5 Choosing the Right K

| K value | Effect |
|---------|--------|
| **K = 1** | Very sensitive to noise; overfitting |
| **Large K** | Smoother decision boundary; may underfit |
| **K = even number** | Can result in ties (avoid for binary classification) |
| **Best practice** | Use **cross-validation** to find optimal K |

> **Rule of thumb:** Start with K = √N where N is the number of training samples

---

### 5.6 Normalization — Why It Matters for KNN

Without normalization, features with large ranges dominate the distance calculation.

**Example:**
- Age: 25 to 80 (range = 55)
- BMI: 18 to 40 (range = 22)

Without normalization, Age differences will dominate distances, even if BMI is equally important.

**Min-Max Normalization:**
$$x' = \frac{x - x_{min}}{x_{max} - x_{min}}$$

---

### 5.7 Advantages and Disadvantages

| Advantages | Disadvantages |
|------------|---------------|
| Simple, easy to implement | Slow at query time (computes distance to all points) |
| No training phase | High memory usage (stores all data) |
| Naturally handles multi-class problems | Sensitive to irrelevant features |
| Adapts to new training data instantly | Sensitive to scale (requires normalization) |
| Non-parametric (no distribution assumption) | Poor performance on high-dimensional data ("Curse of Dimensionality") |

---

## 6. Decision Tree Induction

### 6.1 What is a Decision Tree?

A **Decision Tree** is a tree-structured classification model where:
- **Internal nodes** represent tests on attributes (features)
- **Branches** represent outcomes of the test
- **Leaf nodes** represent class labels (the decision/prediction)

It works like a flowchart — you start at the root and follow branches based on attribute values until you reach a leaf (the class prediction).

---

### 6.2 Key Concepts

**Attribute Selection Measures** — used to decide which attribute to split on at each node:

#### Information Gain (ID3 Algorithm)

Based on the concept of **entropy** — a measure of impurity/disorder in the data.

**Entropy formula:**
$$Entropy(S) = -\sum_{i=1}^{c} p_i \log_2(p_i)$$

Where $p_i$ is the proportion of class $i$ in set $S$.

- **Entropy = 0:** All records belong to the same class (pure node — perfect!)
- **Entropy = 1:** Records are equally split between classes (maximum impurity)

**Information Gain:**
$$Gain(S, A) = Entropy(S) - \sum_{v \in values(A)} \frac{|S_v|}{|S|} \cdot Entropy(S_v)$$

The attribute with the **highest Information Gain** is selected as the splitting attribute.

---

#### Gini Index (CART Algorithm)

$$Gini(S) = 1 - \sum_{i=1}^{c} p_i^2$$

- **Gini = 0:** Pure node (all same class)
- **Gini = 0.5:** Maximum impurity (for 2 classes)

The attribute that gives the **lowest Gini Index** after splitting is selected.

---

### 6.3 Worked Example — Building a Decision Tree

**Dataset — Will a person buy a computer? (Classic example)**

| ID | Age    | Income | Student | Credit  | Buy Computer |
|----|--------|--------|---------|---------|--------------|
| 1  | ≤30    | High   | No      | Fair    | No           |
| 2  | ≤30    | High   | No      | Excellent | No         |
| 3  | 31–40  | High   | No      | Fair    | Yes          |
| 4  | >40    | Medium | No      | Fair    | Yes          |
| 5  | >40    | Low    | Yes     | Fair    | Yes          |
| 6  | >40    | Low    | Yes     | Excellent | No         |
| 7  | 31–40  | Low    | Yes     | Excellent | Yes        |
| 8  | ≤30    | Medium | No      | Fair    | No           |
| 9  | ≤30    | Low    | Yes     | Fair    | Yes          |
| 10 | >40    | Medium | Yes     | Fair    | Yes          |
| 11 | ≤30    | Medium | Yes     | Excellent | Yes        |
| 12 | 31–40  | Medium | No      | Excellent | Yes        |
| 13 | 31–40  | High   | Yes     | Fair    | Yes          |
| 14 | >40    | Medium | No      | Excellent | No         |

Total: 14 samples → 9 Yes, 5 No

**Step 1 — Calculate Entropy of the entire dataset:**

$$Entropy(S) = -\frac{9}{14}\log_2\frac{9}{14} - \frac{5}{14}\log_2\frac{5}{14}$$
$$= -0.643 \times (-0.637) - 0.357 \times (-1.485)$$
$$= 0.410 + 0.530 = 0.940$$

**Step 2 — Calculate Information Gain for attribute "Age":**

Split by Age:
- Age ≤ 30: IDs {1,2,8,9,11} → 2 Yes, 3 No
- Age 31–40: IDs {3,7,12,13} → 4 Yes, 0 No
- Age >40: IDs {4,5,6,10,14} → 3 Yes, 2 No

$$Entropy(Age \leq 30) = -\frac{2}{5}\log_2\frac{2}{5} - \frac{3}{5}\log_2\frac{3}{5} = 0.971$$

$$Entropy(Age\ 31{-}40) = -\frac{4}{4}\log_2\frac{4}{4} - 0 = 0\ \ (\text{pure!})$$

$$Entropy(Age > 40) = -\frac{3}{5}\log_2\frac{3}{5} - \frac{2}{5}\log_2\frac{2}{5} = 0.971$$

$$Gain(S, Age) = 0.940 - \left[\frac{5}{14}(0.971) + \frac{4}{14}(0) + \frac{5}{14}(0.971)\right]$$
$$= 0.940 - [0.347 + 0 + 0.347] = 0.940 - 0.694 = 0.246$$

Similarly compute Gain for Income, Student, Credit — the attribute with the **highest gain** becomes the root node.

**Resulting Tree Structure (simplified):**

```
                    [Age?]
                  /    |    \
              ≤30   31-40   >40
               |      |      |
           [Student?] YES  [Credit?]
            /    \         /      \
          No     Yes   Excellent  Fair
           |      |       |         |
          NO     YES      NO       YES
```

---

### 6.4 Overfitting and Pruning

**Overfitting:** The tree becomes too complex, fitting noise in training data, and performs poorly on new data.

**Signs of overfitting:**
- Tree has too many branches
- Very high training accuracy but low test accuracy

**Solutions:**

**1. Pre-Pruning (Early Stopping):**
- Stop building the tree early when:
  - The number of samples at a node falls below a threshold
  - Information gain is below a minimum threshold
  - Tree depth exceeds a maximum limit

**2. Post-Pruning:**
- Build the full tree, then **prune back** branches that do not improve accuracy on a validation set
- Replace subtrees with leaf nodes

---

### 6.5 ID3 vs C4.5 vs CART

| Feature         | ID3           | C4.5              | CART                   |
|-----------------|---------------|-------------------|------------------------|
| Split measure   | Information Gain | Gain Ratio     | Gini Index             |
| Tree type       | Multiway      | Multiway          | Binary (always 2-way)  |
| Missing values  | Not handled   | Handled           | Handled                |
| Continuous attr | Not handled   | Handled           | Handled                |
| Pruning         | No            | Yes               | Yes                    |
| Output          | Classification| Classification    | Classification/Regression |

**Why Gain Ratio (C4.5)?**
Information Gain is biased toward attributes with **many distinct values** (e.g., an ID attribute would have gain = 1 since each split is pure but is meaningless for prediction). **Gain Ratio** corrects this by penalizing attributes with many branches:

$$GainRatio(S, A) = \frac{Gain(S, A)}{SplitInfo(S, A)}$$

$$SplitInfo(S, A) = -\sum_{v} \frac{|S_v|}{|S|}\log_2\frac{|S_v|}{|S|}$$

---

### 6.6 Advantages and Disadvantages of Decision Trees

| Advantages | Disadvantages |
|------------|---------------|
| Easy to understand and visualize | Prone to overfitting |
| Handles both numerical and categorical data | Small data changes can cause drastically different trees |
| Requires little data preprocessing | Biased toward attributes with more levels (ID3) |
| Non-parametric | Not ideal for continuous output (use regression trees) |
| Fast at prediction time | Can create overly complex trees |

---

## 7. Quick Revision Cheatsheet

### DBMS Schemas
| Schema | Key Feature |
|--------|-------------|
| Star | 1 Fact + denormalized Dimensions; fast queries |
| Snowflake | Normalized dimensions; saves storage |
| Galaxy | Multiple Fact Tables; complex environments |

### Data Warehouse
- **4 Properties:** Subject-Oriented, Integrated, Time-Variant, Non-Volatile
- **ETL:** Extract → Transform → Load
- **Architecture:** 3-tier (Data Layer → OLAP Server → Client)

### OLAP Operations
- **Roll-Up** → aggregate (more summary)
- **Drill-Down** → detail (more granular)
- **Slice** → fix one dimension
- **Dice** → sub-cube (multiple dimensions)
- **Pivot** → rotate view

### KDD Steps
**Selection → Preprocessing → Transformation → Mining → Interpretation**

### KNN
- Find K nearest neighbors by distance (usually Euclidean)
- Majority vote determines class
- Requires normalization; K is a hyperparameter
- Lazy learner — no training phase

### Decision Tree
- **Entropy** measures impurity; **lower entropy = purer node**
- **Information Gain** = parent entropy − weighted child entropy
- Split on attribute with **highest Information Gain**
- **Pruning** prevents overfitting
- **ID3:** Info Gain | **C4.5:** Gain Ratio | **CART:** Gini Index

---

> **Final Exam Tips:**
> 1. Always remember: KDD is the **process**; Data Mining is just **one step** in it
> 2. Star Schema = speed; Snowflake = storage efficiency
> 3. For KNN: always normalize data and justify your choice of K
> 4. For Decision Trees: show entropy calculations step by step — examiners love the math!
> 5. OLAP operations are frequently asked as MCQs — memorize definitions with examples

---
*Notes compiled for exam preparation | Data Mining & Data Warehousing*
