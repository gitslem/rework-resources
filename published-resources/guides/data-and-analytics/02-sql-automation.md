# SQL for Automation Engineers: Essential Queries

## Overview
SQL (Structured Query Language) is essential for automation engineers working with databases. This guide covers the most common queries and patterns you'll use in automation workflows.

**Key Principle:** SQL lets you extract, transform, and load data at scale without writing Python code.

---

## Part 1: SQL Basics

### SELECT
Retrieve data from a table.

```sql
-- Get all columns
SELECT * FROM customers;

-- Get specific columns
SELECT first_name, email FROM customers;

-- Limit results
SELECT * FROM customers LIMIT 10;

-- Rename columns in output
SELECT first_name AS 'First Name', email AS 'Email' FROM customers;
```

### WHERE
Filter results based on conditions.

```sql
-- Simple condition
SELECT * FROM orders WHERE status = 'pending';

-- Multiple conditions
SELECT * FROM orders 
WHERE status = 'pending' AND amount > 100;

-- OR condition
SELECT * FROM customers 
WHERE country = 'USA' OR country = 'Canada';

-- NOT condition
SELECT * FROM users WHERE NOT deleted = true;

-- IN clause
SELECT * FROM orders WHERE status IN ('pending', 'processing', 'shipped');

-- BETWEEN
SELECT * FROM orders WHERE order_date BETWEEN '2024-01-01' AND '2024-01-31';

-- LIKE (pattern matching)
SELECT * FROM customers WHERE email LIKE '%@gmail.com';

-- NULL check
SELECT * FROM users WHERE phone IS NULL;
```

### ORDER BY
Sort results.

```sql
-- Ascending (default)
SELECT * FROM customers ORDER BY created_at;

-- Descending
SELECT * FROM customers ORDER BY created_at DESC;

-- Multiple columns
SELECT * FROM customers ORDER BY country, name;

-- Reverse order
SELECT * FROM customers ORDER BY created_at DESC LIMIT 10;
```

---

## Part 2: Aggregation Functions

### COUNT
Count rows matching condition.

```sql
-- Total customers
SELECT COUNT(*) FROM customers;

-- Customers in USA
SELECT COUNT(*) FROM customers WHERE country = 'USA';

-- Count non-null values
SELECT COUNT(phone) FROM customers;

-- Named result
SELECT COUNT(*) AS total_customers FROM customers;
```

### SUM, AVG, MIN, MAX
Calculate statistics.

```sql
-- Total revenue
SELECT SUM(amount) FROM orders;

-- Average order value
SELECT AVG(amount) FROM orders;

-- Highest and lowest
SELECT MAX(amount) AS highest, MIN(amount) AS lowest FROM orders;

-- Multiple aggregates
SELECT 
  COUNT(*) AS total_orders,
  SUM(amount) AS total_revenue,
  AVG(amount) AS avg_order
FROM orders;
```

### GROUP BY
Aggregate by category.

```sql
-- Total orders per customer
SELECT customer_id, COUNT(*) AS order_count 
FROM orders 
GROUP BY customer_id;

-- Revenue by month
SELECT DATE(order_date) AS month, SUM(amount) AS revenue
FROM orders
GROUP BY DATE(order_date);

-- Multiple grouping
SELECT status, country, COUNT(*) AS order_count
FROM orders
GROUP BY status, country;

-- Having (filter after grouping)
SELECT customer_id, COUNT(*) AS order_count
FROM orders
GROUP BY customer_id
HAVING COUNT(*) > 5;  -- Only customers with 5+ orders
```

---

## Part 3: JOINs

### INNER JOIN
Get rows with matches in both tables.

```sql
SELECT customers.name, orders.amount
FROM customers
INNER JOIN orders ON customers.id = orders.customer_id;
```

### LEFT JOIN
Keep all rows from left table, even if no match in right.

```sql
-- All customers and their orders (NULL if no orders)
SELECT customers.name, COUNT(orders.id) AS order_count
FROM customers
LEFT JOIN orders ON customers.id = orders.customer_id
GROUP BY customers.id;
```

### RIGHT JOIN
Keep all rows from right table.

```sql
-- All orders even if customer was deleted
SELECT customers.name, orders.amount
FROM customers
RIGHT JOIN orders ON customers.id = orders.customer_id;
```

### FULL JOIN
Keep all rows from both tables.

```sql
-- All customers and all orders
SELECT customers.name, orders.amount
FROM customers
FULL JOIN orders ON customers.id = orders.customer_id;
```

---

## Part 4: Common Automation Queries

### Find Duplicates
```sql
SELECT email, COUNT(*) AS occurrences
FROM customers
GROUP BY email
HAVING COUNT(*) > 1;
```

### Find Missing Data
```sql
SELECT * FROM customers WHERE email IS NULL;

SELECT * FROM orders WHERE customer_id IS NULL;
```

### Find Recent Changes
```sql
-- Orders from last 7 days
SELECT * FROM orders 
WHERE order_date >= NOW() - INTERVAL 7 DAY;

-- Customers created today
SELECT * FROM customers
WHERE DATE(created_at) = CURDATE();
```

### Bulk Updates
```sql
-- Update all pending orders to processing
UPDATE orders 
SET status = 'processing'
WHERE status = 'pending';

-- Set a value based on condition
UPDATE customers
SET tier = 'gold'
WHERE total_spent > 1000;
```

### Bulk Deletes
```sql
-- Delete old records
DELETE FROM logs
WHERE created_at < DATE_SUB(NOW(), INTERVAL 90 DAY);

-- Delete duplicates (keep newest)
DELETE FROM customers
WHERE id NOT IN (
  SELECT MAX(id)
  FROM customers
  GROUP BY email
);
```

---

## Part 5: Subqueries

### Subquery in WHERE
```sql
-- Orders from customers who spent > 1000
SELECT * FROM orders
WHERE customer_id IN (
  SELECT id FROM customers
  WHERE total_spent > 1000
);

-- Recent orders only
SELECT * FROM orders
WHERE order_date > (
  SELECT MAX(order_date) - INTERVAL 30 DAY
  FROM orders
);
```

### Subquery in FROM
```sql
-- Get top 10 customers by spending
SELECT * FROM (
  SELECT customer_id, SUM(amount) AS total_spent
  FROM orders
  GROUP BY customer_id
  ORDER BY total_spent DESC
  LIMIT 10
) top_customers;
```

---

## Part 6: SQL in Automation Tools

### Zapier SQL Queries
```sql
-- In Zapier, use SQL action to query database
SELECT customer_id, email, COUNT(*) AS order_count
FROM orders
GROUP BY customer_id
HAVING COUNT(*) > 5
```

### Make (Integromat) Database Module
```sql
-- Use "Query Records" module
SELECT * FROM customers 
WHERE created_at > NOW() - INTERVAL 1 DAY
```

### n8n Database Node
```sql
-- n8n "Database" node with custom SQL
SELECT 
  c.id,
  c.name,
  COUNT(o.id) AS order_count,
  SUM(o.amount) AS total_spent
FROM customers c
LEFT JOIN orders o ON c.id = o.customer_id
GROUP BY c.id
```

---

## Part 7: Performance Tips

### Use Indexes
```sql
-- Create index on frequently filtered columns
CREATE INDEX idx_customer_email ON customers(email);
CREATE INDEX idx_order_status ON orders(status);
```

### Limit Results
```sql
-- Always limit when testing
SELECT * FROM customers LIMIT 100;

-- Paginate large results
SELECT * FROM orders LIMIT 1000 OFFSET 0;
SELECT * FROM orders LIMIT 1000 OFFSET 1000;  -- page 2
```

### Select Only Needed Columns
```sql
-- Good: Get only what you need
SELECT id, email FROM customers;

-- Bad: Get everything
SELECT * FROM customers;
```

### Avoid Expensive Operations
```sql
-- Good: Filter before grouping
SELECT customer_id, COUNT(*) FROM orders
WHERE order_date > DATE_SUB(NOW(), INTERVAL 30 DAY)
GROUP BY customer_id;

-- Bad: Process everything then filter
SELECT customer_id, COUNT(*) FROM orders
GROUP BY customer_id
HAVING DATE(MAX(order_date)) > DATE_SUB(NOW(), INTERVAL 30 DAY);
```

---

## Part 8: Common Patterns

### Top N Records
```sql
-- Top 5 highest-value customers
SELECT customer_id, SUM(amount) AS total
FROM orders
GROUP BY customer_id
ORDER BY total DESC
LIMIT 5;
```

### Running Total
```sql
SELECT 
  order_id,
  amount,
  SUM(amount) OVER (ORDER BY created_at) AS running_total
FROM orders
ORDER BY created_at;
```

### Date Calculations
```sql
-- Orders from last quarter
SELECT * FROM orders
WHERE order_date >= DATE_SUB(NOW(), INTERVAL 3 MONTH);

-- Age in days
SELECT 
  id,
  name,
  DATEDIFF(NOW(), created_at) AS days_customer
FROM customers;
```

---

## Summary

**Essential SQL for Automation:**
- SELECT, WHERE, ORDER BY for data retrieval
- COUNT, SUM, AVG, GROUP BY for analysis
- JOINs to combine multiple tables
- Subqueries for complex filtering
- INSERT, UPDATE, DELETE for data changes

**In Automation Tools:**
- Zapier, Make, n8n all support SQL
- Use SQL to extract, filter, and transform data
- Combine with other actions for complete workflows

**Key Takeaway:**
SQL is essential for automation professionals. Master these patterns and you can automate any data workflow.

---

## Resources

- SQL Cheat Sheet: https://www.sqltutorial.org/
- PostgreSQL Docs: https://www.postgresql.org/docs/
- MySQL Docs: https://dev.mysql.com/doc/
- SQL Optimization: https://use-the-index-luke.com/

---

*This guide was created by **Rework Digital** - Resources Department for automation professionals.*

Questions? Reach out: resource@reworkdigital.io | Follow on GitHub: https://github.com/Reworkdigital-io
