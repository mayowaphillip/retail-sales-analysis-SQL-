# retail-sales-sql-analysis
This project is a Retail Sales Analysis using PostgreSQL, designed to answer 10 key business questions from transactional sales data, covering revenue by category, top customers, gender purchasing patterns, shift-level orders, and best selling months.
# 🗄️ Retail Sales Analysis — SQL (PostgreSQL)

<div align="center">

![PostgreSQL](https://img.shields.io/badge/PostgreSQL-4169E1?style=for-the-badge&logo=postgresql&logoColor=white)
![SQL](https://img.shields.io/badge/SQL-Business%20Analytics-orange?style=for-the-badge)
![Questions](https://img.shields.io/badge/Business%20Questions-10-green?style=for-the-badge)

**Answering 10 real-world retail business questions using production-ready SQL on a PostgreSQL retail transactions database.**

[View Queries](#-sql-queries--business-questions) · [Schema](#-database-schema) · [Key Findings](#-key-findings) · [Author](#-author)

</div>

---

## 📌 Project Overview

Retail businesses generate thousands of transactions daily. Without structured querying, the patterns buried in that data — peak selling periods, top customers, category performance, and gender-based purchasing differences — remain invisible to decision-makers.

This project demonstrates end-to-end SQL proficiency by building a retail sales database in **PostgreSQL** and writing 10 targeted queries that answer real business questions — from basic filtering to aggregation, window functions, conditional logic, and time-based analysis.

---

## 🎯 Objective

To demonstrate practical SQL skills by:
- Designing and populating a retail transactions table in PostgreSQL
- Writing clean, well-commented SQL queries that answer specific business questions
- Covering a range of SQL techniques: filtering, aggregation, grouping, date functions, CASE expressions, CTEs, and ranking
- Delivering query results that are directly actionable for retail business stakeholders

---

## ❓ Problem Statement

A retail business has accumulated transactional sales data across multiple product categories, customer demographics, and time periods. The business needs answers to key operational and strategic questions:

1. Which transactions occurred on a specific date?
2. Which clothing transactions had high volume in a given month?
3. Which categories generate the most revenue?
4. What is the typical age of beauty product buyers?
5. Which transactions are high-value?
6. How do male and female customers differ in purchasing behaviour across categories?
7. Which month in each year drove the most sales?
8. Who are the top 5 highest-spending customers?
9. How many unique customers shop in each category?
10. How are orders distributed across morning, afternoon, and evening shifts?

---

## 🏆 Goals

- Build a clean, normalised retail transactions table in PostgreSQL
- Write 10 production-ready SQL queries — each answering one business question
- Use a range of SQL techniques progressing from basic to advanced
- Document each query with its business context, logic, and result interpretation
- Structure the repo for easy review by technical hiring managers and international clients

---

## 📐 Database Schema

```sql
CREATE TABLE retail_sales (
    transactions_id  INT PRIMARY KEY,
    sale_date        DATE,
    sale_time        TIME,
    customer_id      INT,
    gender           VARCHAR(10),
    age              INT,
    category         VARCHAR(15),
    quantity         INT,
    price_per_unit   FLOAT,
    cogs             FLOAT,
    total_sale       FLOAT
);
```

### Column Definitions

| Column | Type | Description |
|---|---|---|
| `transactions_id` | INT (PK) | Unique identifier for each transaction |
| `sale_date` | DATE | Date the sale occurred |
| `sale_time` | TIME | Time the sale occurred |
| `customer_id` | INT | Unique customer identifier |
| `gender` | VARCHAR(10) | Customer gender (Male / Female) |
| `age` | INT | Customer age in years |
| `category` | VARCHAR(15) | Product category (e.g. Clothing, Beauty, Electronics) |
| `quantity` | INT | Number of units sold |
| `price_per_unit` | FLOAT | Selling price per unit |
| `cogs` | FLOAT | Cost of goods sold |
| `total_sale` | FLOAT | Total sale value (quantity × price_per_unit) |

---

## 🔍 SQL Queries & Business Questions

### Q1 — Sales on a Specific Date
**Business Question:** Retrieve all transactions that occurred on 5th November 2022.

```sql
SELECT *
FROM retail_sales
WHERE sale_date = '2022-11-05';
```
**Purpose:** Enables same-day sales auditing, reconciliation, and daily performance reviews.

---

### Q2 — High-Volume Clothing Sales in November 2022
**Business Question:** Find all transactions in the Clothing category where quantity sold was 4 or more during November 2022.

```sql
SELECT *
FROM retail_sales
WHERE category = 'Clothing'
  AND quantity >= 4
  AND TO_CHAR(sale_date, 'YYYY-MM') = '2022-11';
```
**Purpose:** Identifies bulk clothing purchases in a specific month — useful for inventory planning and promotional campaign evaluation.

---

### Q3 — Total Sales by Category
**Business Question:** Calculate the total revenue generated by each product category.

```sql
SELECT
    category,
    SUM(total_sale)  AS total_revenue,
    COUNT(*)         AS total_transactions
FROM retail_sales
GROUP BY category
ORDER BY total_revenue DESC;
```
**Purpose:** Reveals which categories drive the most revenue — critical for stock allocation, marketing spend, and category management decisions.

---

### Q4 — Average Age of Beauty Category Customers
**Business Question:** What is the average age of customers who purchased items from the Beauty category?

```sql
SELECT
    ROUND(AVG(age), 2) AS avg_customer_age
FROM retail_sales
WHERE category = 'Beauty';
```
**Purpose:** Enables targeted marketing — understanding the typical age of beauty buyers helps tailor campaigns, product range, and in-store experience.

---

### Q5 — High-Value Transactions
**Business Question:** Retrieve all transactions where the total sale value exceeds $1,000.

```sql
SELECT *
FROM retail_sales
WHERE total_sale > 1000
ORDER BY total_sale DESC;
```
**Purpose:** Identifies premium transactions for VIP customer recognition, fraud detection, and high-value order analysis.

---

### Q6 — Transaction Count by Gender and Category
**Business Question:** How many transactions has each gender made within each product category?

```sql
SELECT
    category,
    gender,
    COUNT(transactions_id) AS transaction_count
FROM retail_sales
GROUP BY category, gender
ORDER BY category, gender;
```
**Purpose:** Reveals gender-based purchasing patterns by category — enabling gender-targeted marketing strategies and product range decisions.

---

### Q7 — Best-Selling Month per Year
**Business Question:** Calculate the average sale value for each month, and identify the single best-selling month in each year.

```sql
WITH monthly_avg AS (
    SELECT
        EXTRACT(YEAR  FROM sale_date) AS sale_year,
        EXTRACT(MONTH FROM sale_date) AS sale_month,
        AVG(total_sale)               AS avg_sale,
        RANK() OVER (
            PARTITION BY EXTRACT(YEAR FROM sale_date)
            ORDER BY AVG(total_sale) DESC
        ) AS sale_rank
    FROM retail_sales
    GROUP BY sale_year, sale_month
)
SELECT
    sale_year,
    sale_month,
    ROUND(avg_sale::NUMERIC, 2) AS avg_sale
FROM monthly_avg
WHERE sale_rank = 1
ORDER BY sale_year;
```
**Purpose:** Identifies seasonality and peak revenue months per year — essential for inventory planning, staffing, and promotional scheduling.

---

### Q8 — Top 5 Customers by Total Sales
**Business Question:** Who are the five customers with the highest cumulative spend?

```sql
SELECT
    customer_id,
    SUM(total_sale) AS total_spend
FROM retail_sales
GROUP BY customer_id
ORDER BY total_spend DESC
LIMIT 5;
```
**Purpose:** Identifies the most valuable customers for loyalty programme targeting, personalised offers, and VIP relationship management.

---

### Q9 — Unique Customers per Category
**Business Question:** How many distinct customers have purchased from each product category?

```sql
SELECT
    category,
    COUNT(DISTINCT customer_id) AS unique_customers
FROM retail_sales
GROUP BY category
ORDER BY unique_customers DESC;
```
**Purpose:** Measures category reach — a category with high sales but low unique customers may be driven by repeat buyers, while a category with many unique customers signals broad appeal.

---

### Q10 — Orders by Time-of-Day Shift
**Business Question:** Classify all transactions into Morning (before 12:00), Afternoon (12:00–17:00), and Evening (after 17:00) shifts and count the orders in each shift.

```sql
WITH shift_classification AS (
    SELECT
        transactions_id,
        sale_time,
        CASE
            WHEN EXTRACT(HOUR FROM sale_time) < 12  THEN 'Morning'
            WHEN EXTRACT(HOUR FROM sale_time) <= 17 THEN 'Afternoon'
            ELSE 'Evening'
        END AS shift
    FROM retail_sales
)
SELECT
    shift,
    COUNT(transactions_id) AS total_orders
FROM shift_classification
GROUP BY shift
ORDER BY total_orders DESC;
```
**Purpose:** Identifies when customers shop the most — enabling optimised staffing schedules, time-targeted promotions, and operational planning around peak hours.

---

## 💡 Key Findings

| # | Finding |
|---|---|
| 1 | **Category revenue ranking** reveals which lines drive disproportionate value — enabling smarter shelf space and marketing allocation |
| 2 | **Gender × Category breakdown** shows meaningful differences in purchasing behaviour that can inform targeted campaigns |
| 3 | **Monthly sales ranking** surfaces seasonal peaks — the best-selling months consistently differ by year, highlighting the need for flexible planning |
| 4 | **Top 5 customers** account for a disproportionate share of revenue — a small number of buyers drive outsized business value |
| 5 | **Shift analysis** reveals whether the business is morning, afternoon, or evening dominant — with direct implications for staffing and time-sensitive promotions |
| 6 | **Unique customer counts** show which categories attract the widest audience vs. which are driven by repeat buyers |

---

## 🛠️ Tools & Technologies

| Tool | Purpose |
|---|---|
| **PostgreSQL** | Database engine and query execution |
| **SQL** | Data extraction, transformation, and analysis |
| **pgAdmin / psql** | Query interface and database management |
| **CTEs (Common Table Expressions)** | Used in Q7 and Q10 for readable, modular query structure |
| **Window Functions** | `RANK() OVER()` used in Q7 for year-on-year best month ranking |
| **CASE Expressions** | Used in Q10 for shift classification logic |
| **Date Functions** | `EXTRACT()`, `TO_CHAR()` used for time-based filtering and grouping |

---

## 📁 Project Structure

```
retail-sales-sql-analysis/
│
├── README.md
├── sql/
│   ├── 01_create_table.sql          # Table creation script
│   ├── 02_data_exploration.sql      # Row counts, nulls, distinct values
│   ├── 03_business_questions.sql    # All 10 business question queries
│   └── 04_data_cleaning.sql         # Optional: null handling, type checks
│
├── data/
│   └── retail_sales_sample.csv      # Sample dataset (first 100 rows)
│
└── docs/
    └── query_outputs/
        ├── q1_result.png            # Screenshot of each query result
        ├── q2_result.png
        └── ...
```

---

## 🚀 How to Use

1. Clone or download this repository
2. Open **pgAdmin** or connect via **psql** to your PostgreSQL instance
3. Run `sql/01_create_table.sql` to create the `retail_sales` table
4. Import your data using the PostgreSQL `COPY` command or pgAdmin's import tool:
   ```sql
   COPY retail_sales FROM '/path/to/retail_sales.csv' DELIMITER ',' CSV HEADER;
   ```
5. Run `sql/02_data_exploration.sql` to verify the data loaded correctly
6. Run `sql/03_business_questions.sql` to execute all 10 queries — each is clearly commented with its business question

---

## 📊 SQL Techniques Demonstrated

| Technique | Used In |
|---|---|
| Basic SELECT with WHERE filtering | Q1, Q2, Q5 |
| Aggregate functions (SUM, COUNT, AVG) | Q3, Q4, Q6, Q8, Q9 |
| GROUP BY and ORDER BY | Q3, Q6, Q8, Q9 |
| Date formatting with TO_CHAR and EXTRACT | Q2, Q7, Q10 |
| Common Table Expressions (CTEs) | Q7, Q10 |
| Window functions — RANK() OVER() | Q7 |
| CASE expressions | Q10 |
| DISTINCT for unique counts | Q9 |
| LIMIT for top-N results | Q8 |

---

## 👤 Author

<div align="center">

**Mayowa Phillip**
*Data Analyst · Microsoft Certified Power BI Data Analyst Associate (PL-300)*

[![Email](https://img.shields.io/badge/Email-mayowaphillip%40yahoo.com-D14836?style=flat&logo=gmail&logoColor=white)](mailto:mayowaphillip@yahoo.com)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-mayowaphillipelnuk-0077B5?style=flat&logo=linkedin&logoColor=white)](https://linkedin.com/in/mayowaphillipelnuk)
[![GitHub](https://img.shields.io/badge/GitHub-mayowaphillip-181717?style=flat&logo=github&logoColor=white)](https://github.com/mayowaphillip)

</div>

---

<div align="center">

*Built with PostgreSQL · SQL · pgAdmin*

⭐ If this project was useful to you, consider starring the repository.

</div>
