
# 🛍️ Fashion E-Commerce Consumer Behavior Analysis (SQL Project)

## 📘 Overview
This project analyzes **Indian fashion e-commerce consumer behavior** using **PostgreSQL**.  
It explores insights like customer patterns, sales growth, product performance, and website engagement.

The dataset simulates **10,000+ transactions** across five relational tables: customers, products, orders, order items, and website activity.  
The project focuses on **business intelligence**, **marketing analytics**, and **revenue optimization** using advanced SQL.

## 🧱 Database Schema

### 1️⃣ Customers
| Column | Type | Description |
|---------|------|-------------|
| customer_id | VARCHAR(10) | Unique customer identifier |
| name | VARCHAR(100) | Customer name |
| gender | CHAR(1) | Gender (M/F) |
| age | INT | Age of customer |
| city | VARCHAR(50) | City name |
| country | VARCHAR(50) | Country name |
| signup_date | DATE | Date of registration |

### 2️⃣ Products
| Column | Type | Description |
|---------|------|-------------|
| product_id | VARCHAR(10) | Unique product identifier |
| product_name | VARCHAR(100) | Product name |
| category | VARCHAR(50) | Product category |
| sub_category | VARCHAR(50) | Product subcategory |
| brand | VARCHAR(50) | Brand name |
| price | NUMERIC(10,2) | Selling price |
| cost | NUMERIC(10,2) | Cost price |

### 3️⃣ Orders
| Column | Type | Description |
|---------|------|-------------|
| order_id | VARCHAR(10) | Unique order ID |
| customer_id | VARCHAR(10) | Linked to customers table |
| order_date | DATE | Date of purchase |
| order_status | VARCHAR(20) | Status (completed, cancelled, returned) |
| payment_mode | VARCHAR(20) | Payment type (Credit, COD, etc.) |

### 4️⃣ Order Items
| Column | Type | Description |
|---------|------|-------------|
| order_item_id | SERIAL | Unique item ID |
| order_id | VARCHAR(10) | Linked to orders |
| product_id | VARCHAR(10) | Linked to products |
| quantity | INT | Quantity ordered |
| discount | NUMERIC(10,2) | Discount amount |
| final_price | NUMERIC(10,2) | Final sale price after discount |

### 5️⃣ Website Activity
| Column | Type | Description |
|---------|------|-------------|
| session_id | VARCHAR(10) | Unique web session ID |
| customer_id | VARCHAR(10) | Linked to customers |
| device_type | VARCHAR(20) | Device used (Mobile, Desktop, Tablet) |
| traffic_source | VARCHAR(20) | Source (Google, Direct, Email, etc.) |
| session_start | TIMESTAMP | Session start time |
| pages_viewed | INT | Pages viewed during session |
| time_spent | INT | Duration in seconds |
| purchase_made | VARCHAR(5) | 'Yes' if purchase made |

## 🧮 SQL Analysis and Insights

### 1️⃣ Sales by Category
```sql
SELECT p.category, SUM(oi.final_price) AS total_sales
FROM order_items oi
JOIN products p ON oi.product_id = p.product_id
JOIN orders o ON oi.order_id = o.order_id
WHERE o.order_status = 'completed'
GROUP BY p.category
ORDER BY total_sales DESC;
```
**Insight:** Identifies top-performing fashion categories by revenue.

### 2️⃣ Monthly Sales Trend
```sql
SELECT DATE_TRUNC('month', order_date) AS month,
       SUM(total_amount) AS total_sales
FROM (
    SELECT o.order_id, o.order_date, SUM(oi.final_price) AS total_amount
    FROM orders o
    JOIN order_items oi ON o.order_id = oi.order_id
    WHERE o.order_status = 'completed'
    GROUP BY o.order_id, o.order_date
) t
GROUP BY month
ORDER BY month;
```
**Insight:** Detects seasonal peaks and business cycles.

### 3️⃣ Top 10 Products
```sql
SELECT p.product_name, SUM(oi.quantity) AS total_units_sold, SUM(oi.final_price) AS revenue
FROM order_items oi
JOIN products p ON oi.product_id = p.product_id
JOIN orders o ON oi.order_id = o.order_id
WHERE o.order_status = 'completed'
GROUP BY p.product_name
ORDER BY revenue DESC
LIMIT 10;
```
**Insight:** Shows which products dominate sales.

### 4️⃣ New vs Repeat Customers
```sql
SELECT 
    CASE 
        WHEN order_count = 1 THEN 'New Customer'
        ELSE 'Repeat Customer'
    END AS customer_type,
    COUNT(*) AS num_customers
FROM (
    SELECT customer_id, COUNT(order_id) AS order_count
    FROM orders
    GROUP BY customer_id
) t
GROUP BY customer_type;
```
**Insight:** Measures retention and repeat purchase behavior.

### 5️⃣ Customers by City
```sql
SELECT c.city, COUNT(DISTINCT o.customer_id) AS num_customers
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id
GROUP BY c.city
ORDER BY num_customers DESC;
```
**Insight:** Highlights top-performing cities in India for e-commerce activity.

### 6️⃣ Traffic Source Conversion Rate
```sql
SELECT traffic_source,
       COUNT(*) FILTER (WHERE purchase_made='yes')*100.0/COUNT(*) AS conversion_rate
FROM website_activity
GROUP BY traffic_source;
```
**Insight:** Evaluates which marketing sources drive purchases most effectively.

### 7️⃣ Monthly Sales Growth (%)
```sql
WITH monthly_sales AS (
    SELECT 
        DATE_TRUNC('month', o.order_date) AS month,
        SUM(oi.final_price * oi.quantity) AS monthly_revenue
    FROM orders o
    JOIN order_items oi ON o.order_id = oi.order_id
    GROUP BY 1
)
SELECT 
    month,
    monthly_revenue,
    ROUND(
        (monthly_revenue - LAG(monthly_revenue) OVER (ORDER BY month)) 
        / NULLIF(LAG(monthly_revenue) OVER (ORDER BY month), 0) * 100,
        2
    ) AS growth_percent
FROM monthly_sales
ORDER BY month;
```
**Insight:** Measures month-over-month growth to track momentum.

### 8️⃣ Top 5 Customers by Lifetime Value
```sql
SELECT 
    c.customer_id,
    c.name AS customer_name,
    SUM(oi.final_price * oi.quantity) AS lifetime_value
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id
JOIN order_items oi ON o.order_id = oi.order_id
GROUP BY c.customer_id, c.name
ORDER BY lifetime_value DESC
LIMIT 5;
```
**Insight:** Identifies high-value customers for loyalty campaigns.

### 9️⃣ Category-wise Average Order Value (AOV)
```sql
SELECT 
    p.category,
    ROUND(SUM(oi.final_price * oi.quantity) / COUNT(DISTINCT o.order_id), 2) AS avg_order_value
FROM order_items oi
JOIN products p ON oi.product_id = p.product_id
JOIN orders o ON oi.order_id = o.order_id
GROUP BY p.category
ORDER BY avg_order_value DESC;
```
**Insight:** Determines which product categories drive premium sales.

### 🔟 Profit Margin by Category
```sql
SELECT 
    p.category,
    ROUND(SUM((oi.final_price - p.cost) * oi.quantity), 2) AS total_profit,
    ROUND(SUM((oi.final_price - p.cost) * oi.quantity) / SUM(oi.final_price * oi.quantity) * 100, 2) AS profit_margin_percent
FROM order_items oi
JOIN products p ON oi.product_id = p.product_id
GROUP BY p.category
ORDER BY profit_margin_percent DESC;
```
**Insight:** Analyzes profitability per category.

### 1️⃣1️⃣ RFM Segmentation
```sql
WITH customer_rfm AS (
    SELECT 
        c.customer_id,
        MAX(o.order_date) AS last_purchase_date,
        COUNT(DISTINCT o.order_id) AS frequency,
        SUM(oi.final_price * oi.quantity) AS monetary
    FROM customers c
    JOIN orders o ON c.customer_id = o.customer_id
    JOIN order_items oi ON o.order_id = oi.order_id
    GROUP BY c.customer_id
)
SELECT 
    customer_id,
    (CURRENT_DATE - last_purchase_date) AS recency,
    frequency,
    monetary,
    CASE
        WHEN monetary > 15000 THEN 'VIP'
        WHEN monetary BETWEEN 8000 AND 15000 THEN 'Loyal'
        ELSE 'Occasional'
    END AS customer_segment
FROM customer_rfm
ORDER BY monetary DESC;
```
**Insight:** Segments customers into VIP, Loyal, and Occasional groups.

### 1️⃣2️⃣ Category Performance Over Time
```sql
SELECT 
    DATE_TRUNC('month', o.order_date) AS month,
    p.category,
    SUM(oi.final_price * oi.quantity) AS monthly_sales,
    COUNT(DISTINCT o.order_id) AS total_orders
FROM orders o
JOIN order_items oi ON o.order_id = oi.order_id
JOIN products p ON oi.product_id = p.product_id
GROUP BY month, p.category
ORDER BY month, p.category;
```
**Insight:** Tracks monthly performance by category for Tableau dashboards.

## 🚀 Key Business Insights
- **Clothing & Accessories** dominate total sales.
- **Repeat customers** contribute to over 60% of revenue.
- **Mobile devices** have the highest purchase conversion.
- **Top 5 cities** generate 70% of total orders.
- **VIP customers** (RFM) spend 3x more than average.

## 🧠 Tools & Technologies
- **Database:** PostgreSQL  
- **Environment:** pgAdmin / DBeaver  
- **Language:** SQL  
- **Dataset Size:** 10,000+ records (India-focused)  

