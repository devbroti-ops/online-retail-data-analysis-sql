# online-retail-data-analysis-sql
# Online Retail E-Commerce Data Analysis (SQL)

## 📌 Project Overview
This project focuses on analyzing a transactional dataset of a UK-based online retail store. The goal is to clean the raw transactional data, transform it into a normalized relational database schema, and extract actionable business insights regarding sales performance, customer behavior, and product trends using advanced SQL queries.

## 🛠️ Tech Stack & Skills Demonstrated
* **Database Engine:**  MySQL
* **SQL Concepts:** Data Cleaning, Database Normalization (1NF, 2NF, 3NF), Common Table Expressions (CTEs), Window Functions, Inner/Left Joins, Aggregations.

## 📂 Database Architecture & Normalization
The raw flat dataset was normalized into 4 relational tables to minimize data redundancy and improve query performance:
* **`customers`**: Contains unique customer IDs and their locations.
* **`products`**: Holds product codes, descriptions, and unit prices.
* **`orders`**: Tracks unique transactions and purchase dates.
* **`order_items`**: Maps products to specific invoices with purchase quantities.

## 🧼 Data Cleaning Highlights
Before running analytics, the data was scrubbed of anomalies:
* Removed rows with missing `CustomerID`.
* Excluded cancelled/returned transactions (negative quantities) into a separate `returned_orders` table for separate analysis.
* Handled duplicate records using `DISTINCT` logic during the ETL phase.


## 📊 Business Insights & Key Metrics

### 1. Customer Lifetime Value (CLV) Analysis
**Business Question:** Who are our most valuable customers, and how much have they spent?
**Strategic Value:** Identifying high-value customers allows the marketing team to create exclusive loyalty programs and personalized offers, maximizing retention.

#### SQL Query:
```sql
SELECT 
    c.customer_id,
    c.country,
    COUNT(DISTINCT o.invoice_no) AS total_orders,
    SUM(oi.quantity * p.unit_price) AS total_spent
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id
JOIN order_items oi ON o.invoice_no = oi.invoice_no
JOIN products p ON oi.stock_code = p.stock_code
GROUP BY c.customer_id, c.country
ORDER BY total_spent DESC
LIMIT 5;

WITH customer_orders AS (
    SELECT 
        customer_id,
        COUNT(invoice_no) AS order_count
    FROM orders
    GROUP BY customer_id
)
SELECT 
    COUNT(CASE WHEN order_count >= 2 THEN 1 END) AS repeat_customers,
    COUNT(customer_id) AS total_customers,
    ROUND((COUNT(CASE WHEN order_count >= 2 THEN 1 END) * 100.0 / COUNT(customer_id)), 2) AS repeat_rate_percentage
FROM customer_orders;

[██████████████████████░░░░░░░░░░] 65.57% Repeat Customers
[████████████░░░░░░░░░░░░░░░░░░░░] 34.43% One-Time Buyers


CustomerID | Total Spent (£)
--------------------------------------------------
14646      | ██████████████████████████████ 280.2K
18102      | ████████████████████████████ 259.6K
17450      | █████████████████████ 194.5K
16446      | ██████████████████ 168.4K
14911      | ███████████████ 143.8K


