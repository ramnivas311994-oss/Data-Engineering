
# 🎓 Data Engineering Bootcamp — 15-Day Teaching Guide
### Insurance Domain | For Frontend Developer Transitioning to Data Warehouse
**Trainer:** Ram (Senior Data Engineer, Cognizant)
**Student:** Frontend Developer (No prior data engineering experience)

---

## 📌 Before You Start 

### Tell Her This on Day 0 (Before Anything)

> *"You've been building the UI that users see — buttons, forms, dashboards. Now we're going to build the ENGINE that powers those dashboards. Every chart a business analyst sees in Power BI? We built the entire pipeline behind it. That pipeline is what data engineering is."*

### The Master Flow Map

```
SOURCES                     PIPELINE                   WAREHOUSE              REPORTING
--------                    --------                   ---------              ---------

Policy System    ──►                                   STAGE Layer
Claims System    ──►     ETL / Python / IICS   ──►    CLEANSE Layer   ──►   Power BI
Payment System   ──►     (Extract, Transform,          CORE Layer             Dashboard
Customer Portal  ──►      Load)                        AUDIT Layer
CSV / Excel      ──►
```

### Connect to Frontend Knowledge

| She Already Knows | Data Engineering Equivalent |
|---|---|
| REST API (request/response) | Data pipeline (source → destination) |
| JSON object structure | Table row / record structure |
| React component state | Staging table (temporary holding) |
| Build tool (Webpack) | DBT (build tool for SQL) |
| Git / GitHub | Same — version control for pipelines |
| API response → UI | Snowflake → Power BI |
| Component hierarchy | Star Schema (fact/dimension hierarchy) |
| Dev → Staging → Prod | Dev → QA → Pre-Prod → Production |

---

## 🏥 Insurance Domain Context — Use This Throughout

### Real Insurance Business Story

> *"An insurance company has thousands of customers. Every day:*
> - *New policies are created*
> - *Claims are filed*
> - *Payments are processed*
> - *Customers update their details*
>
> *Our job: Take all this data, clean it, connect it, store it in Snowflake, and make it available for the business to analyze in Power BI."*

### Insurance Tables You Will Build Together

| Layer | Table Name | What It Contains |
|---|---|---|
| STAGE | `STG_POLICY_RAW` | Raw policy data exactly as received |
| STAGE | `STG_CLAIMS_RAW` | Raw claims data from source system |
| STAGE | `STG_CUSTOMER_RAW` | Raw customer records |
| CLEANSE | `CLN_POLICY` | Cleaned policies — nulls handled, dates formatted |
| CLEANSE | `CLN_CLAIMS` | Cleaned claims — duplicates removed |
| CLEANSE | `CLN_CUSTOMER` | Clean customer records |
| CORE | `FCT_CLAIMS` | Fact table — one row per claim with all keys |
| CORE | `DIM_CUSTOMER` | Dimension — customer details |
| CORE | `DIM_POLICY_TYPE` | Dimension — policy categories (Health, Life, Auto) |
| CORE | `DIM_DATE` | Date dimension — year, month, quarter, week |
| AUDIT | `AUDIT_LOG` | Pipeline run log — rows loaded, timestamps, status |

### Star Schema Diagram

```
                    DIM_DATE
                       │
                       │
DIM_CUSTOMER ────── FCT_CLAIMS ────── DIM_POLICY_TYPE
                       │
                       │
                  DIM_AGENT
```

---

## 📅 WEEK 1 — Foundations (Days 1–5)

---

### 📗 DAY 1 — What Is Data Engineering?

**Goal:** She understands what we do and why it matters.

#### Talking Points

1. **What is data?**
   - Every action in a business creates data
   - Customer buys a policy → row in policy table
   - Customer files a claim → row in claims table
   - Payment processed → row in payments table

2. **What is a database vs a data warehouse?**
   - Database = operational (fast writes, small reads) — like the insurance policy system
   - Data Warehouse = analytical (slow writes, massive reads) — like Snowflake

3. **What is a data pipeline?**
   - A process that moves data from source → destination
   - Think of it like a water pipe — data flows from one place to another

4. **What do data engineers do?**
   - Build the pipes
   - Clean the water (data)
   - Make sure the water flows reliably every day

#### Whiteboard Exercise

Draw this together:

```
Insurance App (Frontend) → Database (MySQL/Oracle) → ETL Pipeline → Snowflake → Power BI
        ↑                                                                            ↓
  Customer logs in                                                          Manager sees report
  files a claim
```

#### Day 1 Homework

- Watch: "What is a Data Warehouse?" (10 min YouTube)
- Read: Difference between OLTP and OLAP (give her a 1-page doc)
- Think about: What data does an insurance company collect?

---

### 📗 DAY 2 — OLTP vs OLAP | Databases vs Data Warehouses

**Goal:** She understands WHY we need a separate warehouse.

#### Talking Points

**OLTP (Online Transaction Processing)**
- The insurance company's core system
- Fast inserts and updates
- One record at a time
- Example: Customer files a claim → INSERT INTO claims_table
- Optimized for WRITING

**OLAP (Online Analytical Processing)**
- The analytics layer (Snowflake)
- Reads millions of rows at once
- Used for reports and analysis
- Example: How many claims were filed in Nova Scotia in Q1?
- Optimized for READING

| Feature | OLTP (Insurance System) | OLAP (Snowflake) |
|---|---|---|
| Purpose | Daily operations | Analytics |
| Data | Current | Historical |
| Queries | Simple, fast | Complex, slow |
| Users | Agents, staff | Analysts, managers |
| Example | File a claim | How many claims this year? |

#### Real Insurance Example

```sql
-- OLTP Query (runs on insurance core system)
SELECT * FROM claims WHERE claim_id = 'CLM001';

-- OLAP Query (runs on Snowflake)
SELECT
    policy_type,
    COUNT(*) as total_claims,
    SUM(claim_amount) as total_payout
FROM FCT_CLAIMS
GROUP BY policy_type
ORDER BY total_payout DESC;
```

#### Day 2 Homework

- List 5 questions a business analyst at an insurance company might ask
- These questions = what we build pipelines to answer

---

### 📗 DAY 3 — SQL Basics

**Goal:** She can write basic SQL queries on insurance data.

#### Why SQL First?

> *"SQL is the language of data. Every tool — Snowflake, Power BI, DBT — uses SQL underneath. Master this and everything else becomes easy."*

#### Core SQL Commands

```sql
-- SELECT: Get data
SELECT customer_name, policy_type, premium_amount
FROM CLN_POLICY;

-- WHERE: Filter data
SELECT *
FROM CLN_CLAIMS
WHERE claim_status = 'APPROVED';

-- ORDER BY: Sort results
SELECT customer_name, claim_amount
FROM FCT_CLAIMS
ORDER BY claim_amount DESC;

-- LIMIT: Get top N rows
SELECT TOP 10 * FROM FCT_CLAIMS;

-- COUNT: Count rows
SELECT COUNT(*) as total_claims
FROM FCT_CLAIMS;

-- SUM, AVG, MIN, MAX
SELECT
    SUM(claim_amount) as total_payout,
    AVG(claim_amount) as avg_claim,
    MAX(claim_amount) as largest_claim
FROM FCT_CLAIMS
WHERE claim_year = 2024;
```

#### Hands-On Exercise

Use this sample data (create in Snowflake or use Excel):

```
CUSTOMER TABLE
--------------
customer_id | customer_name   | city      | age
C001        | Priya Sharma    | Toronto   | 45
C002        | John Smith      | Halifax   | 62
C003        | Maria Garcia    | Vancouver | 38

CLAIMS TABLE
--------------
claim_id | customer_id | claim_amount | claim_status | claim_date
CLM001   | C001        | 5000         | APPROVED     | 2024-01-15
CLM002   | C002        | 12000        | PENDING      | 2024-02-20
CLM003   | C001        | 800          | REJECTED     | 2024-03-01
```

**Tasks:**
1. Find all approved claims
2. Find the total amount paid out for approved claims
3. Find claims above $5000
4. Count how many claims each customer has filed

---

### 📗 DAY 4 — SQL Intermediate (JOINs and CTEs)

**Goal:** She can join multiple tables and write complex queries.

#### JOIN Types

```sql
-- INNER JOIN: Only matching records in BOTH tables
SELECT
    c.customer_name,
    cl.claim_amount,
    cl.claim_status
FROM DIM_CUSTOMER c
INNER JOIN FCT_CLAIMS cl ON c.customer_id = cl.customer_id;

-- LEFT JOIN: All records from left table, matching from right
SELECT
    c.customer_name,
    cl.claim_amount
FROM DIM_CUSTOMER c
LEFT JOIN FCT_CLAIMS cl ON c.customer_id = cl.customer_id;
-- Shows ALL customers, even those with no claims (NULL for claim_amount)

-- GROUP BY with JOIN
SELECT
    c.customer_name,
    COUNT(cl.claim_id) as total_claims,
    SUM(cl.claim_amount) as total_amount
FROM DIM_CUSTOMER c
LEFT JOIN FCT_CLAIMS cl ON c.customer_id = cl.customer_id
GROUP BY c.customer_name
ORDER BY total_amount DESC;
```

#### CTEs (Common Table Expressions)

> *"A CTE is like a temporary named result — like a variable in JavaScript but for SQL."*

```sql
-- CTE Example: Find high-value customers
WITH HighValueClaims AS (
    SELECT
        customer_id,
        SUM(claim_amount) as total_claimed
    FROM FCT_CLAIMS
    WHERE claim_status = 'APPROVED'
    GROUP BY customer_id
    HAVING SUM(claim_amount) > 10000
)
SELECT
    c.customer_name,
    hvc.total_claimed
FROM DIM_CUSTOMER c
INNER JOIN HighValueClaims hvc ON c.customer_id = hvc.customer_id;
```

#### CASE Statements

```sql
-- CASE: Like if/else in programming
SELECT
    claim_id,
    claim_amount,
    CASE
        WHEN claim_amount < 1000 THEN 'Low'
        WHEN claim_amount BETWEEN 1000 AND 10000 THEN 'Medium'
        ELSE 'High'
    END as claim_category
FROM FCT_CLAIMS;
```

#### Day 4 Exercise

1. Join customer + claims + policy type tables
2. Find which policy type generates the most claims
3. Categorize customers as Low/Medium/High risk based on total claims

---

### 📗 DAY 5 — Data Modeling (Star Schema)

**Goal:** She understands how a data warehouse is structured.

#### What Is Data Modeling?

> *"In frontend, you think about how components relate to each other. In data warehousing, we think about how tables relate to each other. That's data modeling."*

#### Fact Tables vs Dimension Tables

**Fact Table (FCT_CLAIMS)**
- Contains the measurements / events
- One row per transaction/event
- Has foreign keys pointing to dimensions
- Contains numbers (amounts, counts, durations)

**Dimension Tables (DIM_*)**
- Contains descriptive information
- WHO, WHAT, WHEN, WHERE, HOW
- Changes slowly (customer address updates rarely)

#### Insurance Star Schema — Full Design

```sql
-- FACT TABLE: One row per claim
CREATE TABLE FCT_CLAIMS (
    claim_key       NUMBER PRIMARY KEY,      -- Surrogate key
    customer_key    NUMBER,                  -- FK → DIM_CUSTOMER
    policy_key      NUMBER,                  -- FK → DIM_POLICY_TYPE
    date_key        NUMBER,                  -- FK → DIM_DATE
    agent_key       NUMBER,                  -- FK → DIM_AGENT
    claim_id        VARCHAR(20),             -- Source system ID
    claim_amount    DECIMAL(15,2),           -- Measure
    approved_amount DECIMAL(15,2),           -- Measure
    processing_days NUMBER,                  -- Measure
    claim_status    VARCHAR(20)
);

-- DIMENSION: Customer
CREATE TABLE DIM_CUSTOMER (
    customer_key    NUMBER PRIMARY KEY,
    customer_id     VARCHAR(20),
    customer_name   VARCHAR(100),
    city            VARCHAR(50),
    province        VARCHAR(50),
    age_group       VARCHAR(20),            -- Derived: '18-30', '31-50', '50+'
    risk_category   VARCHAR(20)
);

-- DIMENSION: Policy Type
CREATE TABLE DIM_POLICY_TYPE (
    policy_key      NUMBER PRIMARY KEY,
    policy_code     VARCHAR(20),
    policy_name     VARCHAR(100),           -- Health, Life, Auto, Home
    policy_category VARCHAR(50),
    coverage_tier   VARCHAR(20)             -- Basic, Standard, Premium
);

-- DIMENSION: Date
CREATE TABLE DIM_DATE (
    date_key        NUMBER PRIMARY KEY,     -- Format: 20240115
    full_date       DATE,
    year            NUMBER,
    quarter         NUMBER,
    month           NUMBER,
    month_name      VARCHAR(20),
    week            NUMBER,
    day_of_week     VARCHAR(20),
    is_weekend      BOOLEAN
);
```

#### Day 5 Exercise

Draw the star schema for a new scenario:
- An insurance company wants to track POLICY RENEWALS
- What would the fact table contain?
- What dimensions would you need?

---

## 📅 WEEK 2 — Snowflake + Pipeline (Days 6–10)

---

### 📘 DAY 6 — Snowflake Introduction

**Goal:** She can navigate Snowflake and understand its architecture.

#### Snowflake Key Concepts

| Concept | What It Is | Frontend Analogy |
|---|---|---|
| Database | Container for everything | Project folder |
| Schema | Group of related tables | Feature module folder |
| Table | Stores data | Data model/interface |
| Warehouse | Compute engine that runs queries | Server/CPU |
| Role | Access control | User permissions |
| Stage | Temporary file storage area | Upload folder |

#### Snowflake Architecture for Insurance

```
INSURANCE_DB
├── STAGE_SCHEMA          ← Raw data landing zone
│   ├── STG_POLICY_RAW
│   ├── STG_CLAIMS_RAW
│   └── STG_CUSTOMER_RAW
│
├── CLEANSE_SCHEMA        ← Cleaned data
│   ├── CLN_POLICY
│   ├── CLN_CLAIMS
│   └── CLN_CUSTOMER
│
├── CORE_SCHEMA           ← Final analytical tables
│   ├── FCT_CLAIMS
│   ├── DIM_CUSTOMER
│   ├── DIM_POLICY_TYPE
│   └── DIM_DATE
│
└── AUDIT_SCHEMA          ← Pipeline logs
    └── AUDIT_LOG
```

#### Hands-On: Create the Insurance Database

```sql
-- Step 1: Create database
CREATE DATABASE INSURANCE_DW;

-- Step 2: Create schemas
CREATE SCHEMA INSURANCE_DW.STAGE_SCHEMA;
CREATE SCHEMA INSURANCE_DW.CLEANSE_SCHEMA;
CREATE SCHEMA INSURANCE_DW.CORE_SCHEMA;
CREATE SCHEMA INSURANCE_DW.AUDIT_SCHEMA;

-- Step 3: Set context
USE DATABASE INSURANCE_DW;
USE SCHEMA STAGE_SCHEMA;

-- Step 4: Create first staging table
CREATE TABLE STG_CLAIMS_RAW (
    claim_id        VARCHAR(50),
    customer_id     VARCHAR(50),
    claim_date      VARCHAR(20),       -- VARCHAR first — we clean later
    claim_amount    VARCHAR(20),       -- VARCHAR first — might have $ signs
    claim_status    VARCHAR(50),
    policy_type     VARCHAR(50),
    loaded_at       TIMESTAMP DEFAULT CURRENT_TIMESTAMP()
);
```

---

### 📘 DAY 7 — Loading Data into Snowflake

**Goal:** She can load a CSV file into Snowflake and query it.

#### The Loading Process

```
CSV File → Internal Stage → COPY INTO → Table → Query
```

#### Step-by-Step: Load Claims CSV

```sql
-- Step 1: Create a stage (temporary storage)
CREATE STAGE INSURANCE_DW.STAGE_SCHEMA.CLAIMS_STAGE;

-- Step 2: Upload CSV using Snowflake UI (drag and drop)
-- Or using Python (Day 9)

-- Step 3: Preview the file in stage
LIST @CLAIMS_STAGE;
SELECT $1, $2, $3 FROM @CLAIMS_STAGE LIMIT 5;

-- Step 4: Load into table
COPY INTO STG_CLAIMS_RAW (claim_id, customer_id, claim_date, claim_amount, claim_status)
FROM @CLAIMS_STAGE
FILE_FORMAT = (TYPE = CSV FIELD_OPTIONALLY_ENCLOSED_BY = '"' SKIP_HEADER = 1);

-- Step 5: Verify
SELECT COUNT(*) FROM STG_CLAIMS_RAW;
SELECT * FROM STG_CLAIMS_RAW LIMIT 10;
```

#### STAGE → CLEANSE Transformation

```sql
-- Transform raw stage data into clean data
INSERT INTO INSURANCE_DW.CLEANSE_SCHEMA.CLN_CLAIMS
SELECT
    claim_id,
    customer_id,
    TRY_TO_DATE(claim_date, 'MM/DD/YYYY') as claim_date,   -- Fix date format
    TRY_TO_NUMBER(REPLACE(claim_amount, '$', '')) as claim_amount, -- Remove $
    UPPER(TRIM(claim_status)) as claim_status,              -- Standardize
    CURRENT_TIMESTAMP() as loaded_at
FROM INSURANCE_DW.STAGE_SCHEMA.STG_CLAIMS_RAW
WHERE claim_id IS NOT NULL                                  -- Remove nulls
  AND TRY_TO_DATE(claim_date, 'MM/DD/YYYY') IS NOT NULL;  -- Remove bad dates
```

---

### 📘 DAY 8 — ETL vs ELT | What Is a Pipeline?

**Goal:** She understands the pipeline concept end to end.

#### ETL vs ELT

| | ETL (Traditional) | ELT (Modern — What We Do) |
|---|---|---|
| Order | Extract → Transform → Load | Extract → Load → Transform |
| Where transform? | Outside the warehouse | INSIDE Snowflake using SQL |
| Tool example | Informatica IICS | DBT + Snowflake |
| Better for | Legacy systems | Cloud warehouses |

#### Full Pipeline Diagram

```
DAY 8 WHITEBOARD — Draw This Together

SOURCE                EXTRACT              LOAD              TRANSFORM
------                -------              ----              ---------

Insurance          Python script        STG_CLAIMS_RAW    CLN_CLAIMS
Policy System  →   reads the API    →   (raw data,    →   (cleaned,
                   or CSV file          exact copy)        formatted)
                                                              │
                                                              ▼
                                                         FCT_CLAIMS
                                                         (final fact
                                                          table for
                                                          Power BI)
```

#### Pipeline Layers Explained

**STAGE Layer**
- Exact copy of source data
- No transformation
- Think: "I just received this file — I'm storing it as-is"
- Why: Audit trail, re-processing if something goes wrong

**CLEANSE Layer**
- Fix data quality issues
- Standardize formats
- Remove duplicates
- Handle nulls
- Think: "I'm cleaning the raw data"

**CORE Layer**
- Final business-ready tables
- Star schema applied
- Joins done, calculations complete
- Think: "This is what Power BI connects to"

**AUDIT Layer**
- Did the pipeline run?
- How many rows loaded?
- Any errors?
- Think: "The pipeline's own diary"

```sql
-- AUDIT LOG Entry
INSERT INTO AUDIT_SCHEMA.AUDIT_LOG VALUES (
    'CLN_CLAIMS',               -- Table loaded
    CURRENT_TIMESTAMP(),        -- When it ran
    (SELECT COUNT(*) FROM CLN_CLAIMS),  -- Rows loaded
    'SUCCESS',                  -- Status
    NULL                        -- Error message (null if success)
);
```

---

### 📘 DAY 9 — Python for Data Engineering

**Goal:** She can write Python to read data and load it to Snowflake.

#### Why Python?

> *"You know JavaScript. Python is similar — cleaner syntax, same logic. In data engineering, Python is our automation language."*

#### Python vs JavaScript Comparison

```python
# Python                          # JavaScript equivalent
x = 10                            # let x = 10;
name = "Priya"                    # const name = "Priya";
my_list = [1, 2, 3]              # const myList = [1, 2, 3];

# For loop
for item in my_list:             # for (const item of myList) {
    print(item)                  #     console.log(item);
                                 # }

# Function
def add(a, b):                   # function add(a, b) {
    return a + b                 #     return a + b;
                                 # }
```

#### Real Pipeline Script — CSV to Snowflake

```python
import pandas as pd
import snowflake.connector
from datetime import datetime

# ─────────────────────────────────────
# STEP 1: Read the source CSV file
# ─────────────────────────────────────
def extract_claims_data(file_path):
    """Read claims CSV file"""
    print(f"Reading file: {file_path}")
    df = pd.read_csv(file_path)
    print(f"Rows extracted: {len(df)}")
    return df

# ─────────────────────────────────────
# STEP 2: Clean the data
# ─────────────────────────────────────
def transform_claims_data(df):
    """Clean and transform claims data"""
    print("Transforming data...")

    # Remove rows with null claim_id
    df = df.dropna(subset=['claim_id'])

    # Fix claim_amount — remove $ and convert to number
    df['claim_amount'] = df['claim_amount'].str.replace('$', '').str.replace(',', '')
    df['claim_amount'] = pd.to_numeric(df['claim_amount'], errors='coerce')

    # Standardize claim_status to uppercase
    df['claim_status'] = df['claim_status'].str.upper().str.strip()

    # Fix date format
    df['claim_date'] = pd.to_datetime(df['claim_date'], errors='coerce')

    # Add audit column
    df['loaded_at'] = datetime.now()

    print(f"Rows after cleaning: {len(df)}")
    return df

# ─────────────────────────────────────
# STEP 3: Load to Snowflake
# ─────────────────────────────────────
def load_to_snowflake(df, table_name):
    """Load dataframe to Snowflake"""
    print(f"Loading to Snowflake: {table_name}")

    conn = snowflake.connector.connect(
        user='YOUR_USER',
        password='YOUR_PASSWORD',
        account='YOUR_ACCOUNT',
        warehouse='COMPUTE_WH',
        database='INSURANCE_DW',
        schema='STAGE_SCHEMA'
    )

    cursor = conn.cursor()

    rows_loaded = 0
    for _, row in df.iterrows():
        cursor.execute(f"""
            INSERT INTO {table_name}
            (claim_id, customer_id, claim_date, claim_amount, claim_status, loaded_at)
            VALUES (%s, %s, %s, %s, %s, %s)
        """, (
            row['claim_id'],
            row['customer_id'],
            str(row['claim_date']),
            float(row['claim_amount']) if pd.notna(row['claim_amount']) else None,
            row['claim_status'],
            str(row['loaded_at'])
        ))
        rows_loaded += 1

    conn.commit()
    cursor.close()
    conn.close()

    print(f"✅ Loaded {rows_loaded} rows to {table_name}")
    return rows_loaded

# ─────────────────────────────────────
# MAIN PIPELINE RUN
# ─────────────────────────────────────
if __name__ == "__main__":
    # Extract
    raw_data = extract_claims_data("claims_data.csv")

    # Transform
    clean_data = transform_claims_data(raw_data)

    # Load
    rows = load_to_snowflake(clean_data, "STG_CLAIMS_RAW")

    print(f"\n🎉 Pipeline complete! {rows} rows loaded.")
```

---

### 📘 DAY 10 — DBT Introduction

**Goal:** She understands what DBT does and how it fits in the pipeline.

#### What Is DBT?

> *"DBT is like Webpack for SQL. Just like Webpack takes your React components and builds them into optimized JS files, DBT takes your SQL models and builds them into optimized tables in Snowflake."*

#### DBT in the Insurance Pipeline

```
Raw CSV → Python loads to STAGE → DBT transforms:
                                    STAGE → CLEANSE (SQL model)
                                    CLEANSE → CORE (SQL model)
                                    Results verified → Power BI connects
```

#### DBT Model Example — Cleanse Layer

```sql
-- File: models/cleanse/cln_claims.sql

{{ config(
    materialized = 'table',
    schema = 'CLEANSE_SCHEMA'
) }}

SELECT
    claim_id,
    customer_id,
    TRY_TO_DATE(claim_date, 'MM/DD/YYYY')                    AS claim_date,
    TRY_TO_NUMBER(REPLACE(claim_amount, '$', ''))             AS claim_amount,
    UPPER(TRIM(claim_status))                                 AS claim_status,
    CURRENT_TIMESTAMP()                                       AS dbt_loaded_at
FROM {{ source('stage_schema', 'stg_claims_raw') }}
WHERE claim_id IS NOT NULL
  AND TRY_TO_DATE(claim_date, 'MM/DD/YYYY') IS NOT NULL
```

#### DBT Model Example — Core Layer

```sql
-- File: models/core/fct_claims.sql

{{ config(
    materialized = 'table',
    schema = 'CORE_SCHEMA'
) }}

SELECT
    ROW_NUMBER() OVER (ORDER BY cl.claim_id)     AS claim_key,
    c.customer_key,
    p.policy_key,
    d.date_key,
    cl.claim_id,
    cl.claim_amount,
    cl.claim_status,
    DATEDIFF('day', cl.claim_date, CURRENT_DATE()) AS days_since_claim
FROM {{ ref('cln_claims') }} cl
LEFT JOIN {{ ref('dim_customer') }} c    ON cl.customer_id = c.customer_id
LEFT JOIN {{ ref('dim_policy_type') }} p ON cl.policy_type = p.policy_code
LEFT JOIN {{ ref('dim_date') }} d        ON cl.claim_date = d.full_date
```

---

## 📅 WEEK 3 — Power BI + Full Picture (Days 11–15)

---

### 📙 DAY 11 — Power BI Introduction + Connect to Snowflake

**Goal:** She can connect Power BI to Snowflake and see data.

#### Power BI in the Stack

```
Snowflake CORE Layer → Power BI → Business Analyst → Decision Making
```

#### Connect Power BI to Snowflake

1. Open Power BI Desktop
2. Get Data → Snowflake
3. Enter: Server = `your-account.snowflakecomputing.com`
4. Database = `INSURANCE_DW`
5. Select: `CORE_SCHEMA` → `FCT_CLAIMS`, `DIM_CUSTOMER`, `DIM_POLICY_TYPE`
6. Load all three tables

#### Create Relationships (Like Foreign Keys)

In Power BI Model view:
- `FCT_CLAIMS[customer_key]` → `DIM_CUSTOMER[customer_key]`
- `FCT_CLAIMS[policy_key]` → `DIM_POLICY_TYPE[policy_key]`
- `FCT_CLAIMS[date_key]` → `DIM_DATE[date_key]`

> *"This is exactly the Star Schema we designed on Day 5. Power BI just draws it visually."*

---

### 📙 DAY 12 — Build the Insurance Claims Dashboard

**Goal:** She builds a real dashboard that a manager would use.

#### Dashboard: Insurance Claims Analytics

**Visual 1: Total Claims Card**
- Visual type: Card
- Value: COUNT of claim_id
- Title: "Total Claims Filed"

**Visual 2: Total Payout Card**
- Visual type: Card
- Value: SUM of claim_amount where status = APPROVED
- Title: "Total Amount Paid Out"

**Visual 3: Claims by Policy Type (Bar Chart)**
- Axis: DIM_POLICY_TYPE[policy_name]
- Value: COUNT(FCT_CLAIMS[claim_id])
- Title: "Claims by Policy Type"

**Visual 4: Claims Trend (Line Chart)**
- Axis: DIM_DATE[month_name]
- Value: COUNT(FCT_CLAIMS[claim_id])
- Title: "Monthly Claims Trend"

**Visual 5: Claims by Province (Map)**
- Location: DIM_CUSTOMER[province]
- Size: COUNT(FCT_CLAIMS[claim_id])

**Visual 6: Claims Status Breakdown (Donut)**
- Legend: FCT_CLAIMS[claim_status]
- Values: COUNT(FCT_CLAIMS[claim_id])

#### DAX Measures

```dax
-- Total Claims
Total Claims = COUNTROWS(FCT_CLAIMS)

-- Approved Claims Amount
Approved Payout = 
CALCULATE(
    SUM(FCT_CLAIMS[claim_amount]),
    FCT_CLAIMS[claim_status] = "APPROVED"
)

-- Approval Rate
Approval Rate = 
DIVIDE(
    COUNTROWS(FILTER(FCT_CLAIMS, FCT_CLAIMS[claim_status] = "APPROVED")),
    COUNTROWS(FCT_CLAIMS),
    0
)

-- Average Claim Amount
Avg Claim Amount = AVERAGE(FCT_CLAIMS[claim_amount])
```

---

### 📙 DAY 13 — CI/CD in Data Engineering

**Goal:** She understands how code gets from her laptop to production.

#### What Is CI/CD?

> *"When you push code to GitHub, tests run automatically, and if they pass, the code is deployed. We do the exact same thing for our data pipelines."*

#### The 4-Tier Environment

```
DEV          →       QA          →    PRE-PROD    →    PRODUCTION
----                 --               --------         ----------
You write            Testers          Final check      Business
code here            validate         before go-live   uses this
                     here
```

#### Git Workflow for Data Engineers

```bash
# Start new feature
git checkout -b feature/claims-pipeline

# Make your changes (Python scripts, SQL files)

# Stage and commit
git add .
git commit -m "Add claims ETL pipeline with audit logging"

# Push to remote
git push origin feature/claims-pipeline

# Create Pull Request → Code Review → Merge → Auto-deploy
```

#### What CI/CD Does for a Pipeline

```
Developer pushes SQL/Python code to GitHub
        ↓
GitHub Actions triggers automatically
        ↓
Runs tests: Does the SQL syntax work? Do row counts match?
        ↓
If tests pass: Deploy to QA Snowflake environment
        ↓
QA team validates
        ↓
Approve: Deploy to Production Snowflake
```

---

### 📙 DAY 14 — Full Pipeline Walkthrough

**Goal:** Connect every single dot from source to dashboard.

#### End-to-End Insurance Claims Pipeline

```
STEP 1: SOURCE
Insurance policy system exports claims CSV file daily at 6 AM

STEP 2: EXTRACT (Python)
Python script runs at 6:05 AM
Reads the CSV file
Loads raw data to STG_CLAIMS_RAW in Snowflake STAGE schema

STEP 3: CLEANSE (SQL/DBT)
DBT model runs at 6:30 AM
Reads STG_CLAIMS_RAW
Cleans: fixes dates, removes $, standardizes status
Writes to CLN_CLAIMS in CLEANSE schema

STEP 4: CORE (SQL/DBT)
DBT model runs at 7:00 AM
Joins: CLN_CLAIMS + DIM_CUSTOMER + DIM_POLICY_TYPE + DIM_DATE
Builds: FCT_CLAIMS in CORE schema

STEP 5: AUDIT
Row count: Source file had 500 rows → FCT_CLAIMS has 498 rows
2 rows rejected → logged in AUDIT_LOG with reason

STEP 6: POWER BI
Scheduled refresh at 8:00 AM
Power BI reads from FCT_CLAIMS
Dashboard updated
Manager opens Power BI at 9 AM → sees yesterday's claims
```

#### Full Audit SQL

```sql
-- Check: Did all data load correctly?
SELECT
    'STG_CLAIMS_RAW' as layer,
    COUNT(*) as row_count
FROM STAGE_SCHEMA.STG_CLAIMS_RAW
WHERE DATE(loaded_at) = CURRENT_DATE()

UNION ALL

SELECT
    'CLN_CLAIMS' as layer,
    COUNT(*) as row_count
FROM CLEANSE_SCHEMA.CLN_CLAIMS
WHERE DATE(loaded_at) = CURRENT_DATE()

UNION ALL

SELECT
    'FCT_CLAIMS' as layer,
    COUNT(*) as row_count
FROM CORE_SCHEMA.FCT_CLAIMS
WHERE DATE(loaded_at) = CURRENT_DATE();
```

---

### 📙 DAY 15 — Mini Project (Capstone)

**Goal:** She builds a complete pipeline from scratch — by herself.

#### The Assignment

> *"You are the data engineer at Nova Scotia Insurance Co. The business needs a dashboard showing agent performance. Here is your CSV file. Build the complete pipeline."*

#### Agent Performance CSV (Sample Data)

```
agent_id, agent_name, region, claim_id, customer_id, policy_type, premium_amount, claim_amount, resolution_days, status
AGT001, Sarah Chen, Halifax, CLM101, C001, Health, 1200, 5000, 3, APPROVED
AGT002, Mike Johnson, Sydney, CLM102, C002, Auto, 800, 2200, 7, APPROVED
AGT001, Sarah Chen, Halifax, CLM103, C003, Life, 2000, 0, 2, REJECTED
AGT003, Priya Patel, Dartmouth, CLM104, C004, Home, 1500, 8500, 12, APPROVED
AGT002, Mike Johnson, Sydney, CLM105, C005, Health, 900, 3300, 5, PENDING
```

#### Capstone Tasks

**Task 1 — Snowflake Setup**
- Create new schema: `AGENT_SCHEMA`
- Create staging table: `STG_AGENT_PERFORMANCE`
- Create core table: `FCT_AGENT_PERFORMANCE`
- Create dimension: `DIM_AGENT`

**Task 2 — Python Script**
- Read the CSV file
- Clean the data (handle nulls, fix amounts)
- Load to `STG_AGENT_PERFORMANCE`

**Task 3 — SQL Transformation**
- Write SQL to move STAGE → CORE
- Add surrogate key
- Calculate: average resolution days per agent

**Task 4 — Audit Check**
- Write SQL to verify row counts match

**Task 5 — Power BI Dashboard**
- Connect to Snowflake
- Build 4 visuals:
  1. Top agents by approved claims (bar chart)
  2. Average resolution days by agent (bar chart)
  3. Claims by region (map)
  4. Total premium vs total payout (KPI cards)

---

## 📊 Quick Reference Cheat Sheet

### SQL Cheat Sheet — Insurance Queries

```sql
-- Count claims by status
SELECT claim_status, COUNT(*) FROM FCT_CLAIMS GROUP BY claim_status;

-- Top 5 customers by claim amount
SELECT TOP 5 customer_name, SUM(claim_amount) as total
FROM FCT_CLAIMS f JOIN DIM_CUSTOMER c ON f.customer_key = c.customer_key
GROUP BY customer_name ORDER BY total DESC;

-- Monthly claims trend
SELECT month_name, COUNT(*) as claims
FROM FCT_CLAIMS f JOIN DIM_DATE d ON f.date_key = d.date_key
GROUP BY month_name, month ORDER BY month;

-- Claims approval rate by policy type
SELECT
    policy_name,
    COUNT(*) as total,
    SUM(CASE WHEN claim_status = 'APPROVED' THEN 1 ELSE 0 END) as approved,
    ROUND(approved * 100.0 / total, 1) as approval_pct
FROM FCT_CLAIMS f JOIN DIM_POLICY_TYPE p ON f.policy_key = p.policy_key
GROUP BY policy_name;
```

### Snowflake Commands Cheat Sheet

```sql
-- Switch context
USE DATABASE INSURANCE_DW;
USE SCHEMA CORE_SCHEMA;
USE WAREHOUSE COMPUTE_WH;

-- See all tables
SHOW TABLES;

-- Check table structure
DESCRIBE TABLE FCT_CLAIMS;

-- Sample data
SELECT * FROM FCT_CLAIMS SAMPLE (100 ROWS);

-- Check row count quickly
SELECT COUNT(*) FROM FCT_CLAIMS;

-- Query history (what queries ran?)
SELECT * FROM TABLE(INFORMATION_SCHEMA.QUERY_HISTORY()) LIMIT 20;
```

### Python Cheat Sheet — Pandas Basics

```python
import pandas as pd

# Read CSV
df = pd.read_csv('claims.csv')

# Check shape
print(df.shape)              # (rows, columns)

# See first rows
print(df.head())

# Check for nulls
print(df.isnull().sum())

# Filter rows
approved = df[df['claim_status'] == 'APPROVED']

# Select columns
subset = df[['claim_id', 'claim_amount', 'claim_status']]

# Group by
summary = df.groupby('policy_type')['claim_amount'].sum()

# Fix data types
df['claim_amount'] = pd.to_numeric(df['claim_amount'], errors='coerce')
df['claim_date'] = pd.to_datetime(df['claim_date'], errors='coerce')
```

---

## 🗺️ Technology Map — Where Each Tool Fits

```
TOOL            WHERE IT SITS          WHAT IT DOES
----            -------------          ------------
Python          Pipeline Layer         Reads files, transforms, loads to Snowflake
SQL             Everywhere             Queries, transforms, builds tables
Snowflake       Warehouse Layer        Stores all data — STAGE, CLEANSE, CORE, AUDIT
DBT             Transform Layer        Manages SQL transformations as code
Power BI        Presentation Layer     Dashboards for business users
GitHub          DevOps Layer           Version control, CI/CD for all code
IICS            Pipeline Layer         Enterprise ETL tool (Informatica Cloud)
```

---

## 🎯 Job Description Skills Map

Based on the job descriptions you reviewed, here is what each day builds toward:

| Skill Required in Jobs | Days That Cover This |
|---|---|
| Snowflake hands-on | Days 6, 7, 8, 14 |
| SQL (strong) | Days 3, 4, 7, 8, 14 |
| Python | Days 9, 15 |
| ETL/ELT pipelines | Days 8, 9, 10, 14 |
| Data modeling / Star Schema | Days 5, 6 |
| Power BI | Days 11, 12, 15 |
| CI/CD concepts | Day 13 |
| DBT | Day 10 |
| Monitoring / Audit | Days 8, 14 |
| Documentation | Every day — write it as you go |

---

## 📝 Notes for the Trainer (Ram)

- **Always draw before typing** — whiteboard the flow, then open the laptop
- **Use real data** — download a Kaggle insurance dataset for hands-on exercises
- **Don't rush Days 3 and 4** — SQL is the foundation of everything else
- **Power BI is her comfort zone** — use it to build confidence in Week 3
- **Your real experience = her best teacher** — share actual stories from your Cognizant projects
- **Connect everything to insurance** — never use abstract examples when an insurance example works
- **Each day should end with her doing something herself** — not just watching

---

*Document prepared by Ram | Cognizant Data Engineering | Nova Scotia, Canada*
*Based on real project experience: Snowflake, Python, ETL, Informatica IICS, Power BI*
