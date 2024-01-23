<img width ="500" src ="https://github.com/chile2706/8-week-sql/assets/147631781/27227b2a-f876-49ff-9769-79d56008f377">

## Table of Contents
* [Case Study Introduction](#case-study-introduction)
* [Entity Relationship Diagram](#entity-relationship-diagram)
* [Case Study Questions & Solutions](#case-study-questions)
---
## Case Study Introduction
Danny decided to launch a new initiative - Data Bank!

Data Bank runs just like any other digital bank. Customers are allocated cloud data storage limits which are directly linked to how much money they have in their accounts. 
The management team at Data Bank wants to increase their total customer base - but also needs some help tracking just how much data storage their customers will need.

This case study is all about calculating metrics, growth and helping the business smartly analyse their data to better forecast and plan for their future developments!

## Entity Relationship Diagram
<img width="713" alt="Screen Shot 2024-01-18 at 16 02 49" src="https://github.com/chile2706/8-week-sql/assets/147631781/1119e016-d384-4bbe-be83-a62430f208f2">
  

**Table 1:** `regions`

Data Bank runs off a network of nodes where both money and data are stored across the globe. We can think of these nodes as bank branches or stores that exist around the world.

This `regions` table contains the `region_id` and their respective `region_name` values

<img width="170" alt="Screen Shot 2024-01-18 at 16 10 23" src="https://github.com/chile2706/8-week-sql/assets/147631781/96ee45cf-7007-40d6-87ec-c34d6090c378">



  
**Table 2:** `customer_nodes`

Customers are randomly distributed across the nodes according to their region - this also specifies exactly which node contains both their cash and data.

<img width="350" alt="Screen Shot 2024-01-18 at 16 10 47" src="https://github.com/chile2706/8-week-sql/assets/147631781/b6dec12c-a823-434f-a5e2-d7b2dd82ecd5">


**Table 3:** `customer_transactions`

This table stores all customer deposits, withdrawals and purchases made using their Data Bank debit card.

<img width="300" alt="Screen Shot 2024-01-18 at 16 11 08" src="https://github.com/chile2706/8-week-sql/assets/147631781/2749797c-9057-44fc-8785-2ab482fb7c89">

## Case Study Questions
### A. Customer Nodes Exploration
#### 1. How many unique nodes are there in the Data Bank system?
- Use `GROUP BY()` and `COUNT(DISTINCT)` to count unique nodes in each region
- Use `SUM()` to calculate the total nodes
```MySQL
SELECT SUM(a.count) AS total_nodes
FROM
(SELECT cn.region_id, COUNT(DISTINCT cn.node_id) AS count
FROM customer_nodes cn
GROUP BY cn.region_id) a;
```
**Answer:**

<img width="100" alt="Screen Shot 2024-01-18 at 16 28 15" src="https://github.com/chile2706/8-week-sql/assets/147631781/7a21e8bb-053c-4944-ae05-231aa2a44f04">

#### 2. What is the number of nodes per region?
- Use `GROUP BY()` and `COUNT(DISTINCT)` to count unique nodes in each region
```mysql
SELECT
  cn.region_id, r.region_name,
  COUNT(DISTINCT cn.customer_id) AS total_customers
FROM customer_nodes cn
NATURAL JOIN regions r
GROUP BY cn.region_id, r.region_name
ORDER BY cn.region_id;
```
**Answer:**

<img width="180" alt="Screen Shot 2024-01-18 at 16 31 46" src="https://github.com/chile2706/8-week-sql/assets/147631781/e79246fb-3e6d-4d54-851a-34a6676f12bf">

#### 3. How many customers are allocated to each region?
- Use `GROUP BY()` and `COUNT(DISTINCT)` to count unique customers in each region
```MYSQL
SELECT
  cn.region_id, r.region_name,
  COUNT(DISTINCT cn.customer_id) AS total_customers
FROM customer_nodes cn
NATURAL JOIN regions r
GROUP BY cn.region_id, r.region_name
ORDER BY cn.region_id;
```
**Answer:**

<img width="250" alt="Screen Shot 2024-01-18 at 16 30 17" src="https://github.com/chile2706/8-week-sql/assets/147631781/47c40470-c43e-4095-a8d4-2fcc0e22f64e">

#### 4. How many days on average are customers reallocated to a different node?
- Use `DATEDIFF()` to calculate the time duration of each customer on each node
- Use `AVG()` to caculate the average
```mysql
SELECT ROUND(AVG(DATEDIFF(cn.end_date, cn.start_date)), 0) AS avg_day
FROM customer_nodes cn
WHERE cn.end_date != '9999-12-31';
```
**Answer:**

<img width="80" alt="Screen Shot 2024-01-18 at 16 36 01" src="https://github.com/chile2706/8-week-sql/assets/147631781/a72ecc59-d7f7-4f32-a122-a2736bdb7475">

#### 5. What is the median, 80th, and 95th percentile for this same reallocation days metric for each region?
- Create view `percentile` to calculate the order of the relocation days for eadch percentile  
```mysql
CREATE VIEW percentile AS
SELECT a.region_id,
  ROUND(a.total_relocate*0.5,0) AS '50th',
  ROUND(a.total_relocate*0.85,0) AS '85th',
  ROUND(a.total_relocate*0.9,0) AS '90th'
FROM
  (SELECT cn.region_id, COUNT(cn.customer_id) AS total_relocate
  FROM customer_nodes cn
  WHERE cn.end_date != '9999-12-31'
  GROUP BY cn.region_id
  ORDER BY cn.region_id) a;
```
<img width="180" alt="Screen Shot 2024-01-18 at 16 44 52" src="https://github.com/chile2706/8-week-sql/assets/147631781/d163d210-5857-4f3f-a622-5b5164d4e8b0">

- Unpivot the `percentile` view
```mysql
SELECT p.region_id, '50th' as percent, p.50th AS ranking from percentile p
UNION ALL
SELECT p.region_id, '85th', p.85th FROM percentile p
UNION ALL
SELECT p.region_id, '90th', p.90th FROM percentile p;
```

<img width="147" alt="Screen Shot 2024-01-22 at 16 13 59" src="https://github.com/chile2706/8-week-sql/assets/147631781/af827db4-f2ac-42a7-b316-bfb395e8980a">


- Use `ROW_NUMBER()` to assign the order to date diff to match with the relocation days for each percentile later.
```MYSQL
SELECT cn.region_id,
  DATEDIFF(cn.end_date, cn.start_date) AS days,
  ROW_NUMBER() OVER(PARTITION BY cn.region_id ORDER BY DATEDIFF(cn.end_date, cn.start_date)) AS ranking
FROM customer_nodes cn
WHERE cn.end_date != '9999-12-31'
```
<img width="136" alt="Screen Shot 2024-01-22 at 16 10 28" src="https://github.com/chile2706/8-week-sql/assets/147631781/ec0c9140-031d-4c77-91e4-5390e4789259">

- Use `INNER JOIN` to join the latest two queries we just made `ON` the same `region_id` and `ranking` to get  the relocations day for each percentile
```mysql
SELECT a.region_id, a.percent, b.ranking, b.days
FROM
  (SELECT p.region_id, '50th' AS percent, p.50th AS ranking FROM percentile p
  UNION ALL
  SELECT p.region_id, '85th', p.85th FROM percentile p
  UNION ALL
  SELECT p.region_id, '90th', p.90th FROM percentile p) a
INNER JOIN
  (SELECT cn.region_id,
    DATEDIFF(cn.end_date, cn.start_date) AS days,
    ROW_NUMBER() OVER(PARTITION BY cn.region_id ORDER BY DATEDIFF(cn.end_date, cn.start_date)) AS ranking
  FROM customer_nodes cn
  WHERE cn.end_date != '9999-12-31') b
ON a.ranking = b.ranking
AND a.region_id = b.region_id
ORDER BY region_id;
```
<img width="182" alt="Screen Shot 2024-01-22 at 16 17 25" src="https://github.com/chile2706/8-week-sql/assets/147631781/5dfcdabc-a6da-4f95-a2c5-372ffa4cb9af">

- Pivot the query for better format

```mysql
SELECT c.region_id, 
  SUM(CASE WHEN c.percent = '50th' THEN c.days ELSE 0 END ) AS '50th',
  SUM(CASE WHEN c.percent = '85th' THEN c.days ELSE 0 END ) AS '85th',
  SUM(CASE WHEN c.percent = '90th' THEN c.days ELSE 0 END ) AS '90th'
FROM
(SELECT a.region_id, a.percent, b.ranking, b.days
FROM
(SELECT p.region_id, '50th' as percent, p.50th as ranking from percentile p
UNION ALL
SELECT p.region_id, '85th', p.85th FROM percentile p
UNION ALL
select p.region_id, '90th', p.90th from percentile p) a
INNER JOIN
(SELECT cn.region_id, DATEDIFF(cn.end_date, cn.start_date) AS days, ROW_NUMBER() OVER(PARTITION BY cn.region_id ORDER BY DATEDIFF(cn.end_date, cn.start_date)) AS ranking
FROM customer_nodes cn
WHERE cn.end_date != '9999-12-31') b
ON a.ranking = b.ranking AND a.region_id = b.region_id
ORDER BY region_id) c
GROUP BY c.region_id;
```
**Answers:**

<img width="182" alt="Screen Shot 2024-01-22 at 16 19 18" src="https://github.com/chile2706/8-week-sql/assets/147631781/18514feb-7ef8-465b-bd89-55301b7a84ae">



### B. Customer Transactions
#### 1. What is the unique count and total amount for each transaction type?
- Use `GROUP BY()` and aggregate functions `COUNT(DISTINCT)` and `SUM` to calculate unique count and total amount of each transaction type

```mysql
SELECT ct.txn_type, COUNT(ct.txn_type) AS count, FORMAT(SUM(txn_amount),0) AS total_amount
FROM customer_transactions ct
GROUP BY ct.txn_type;
```

**Answers:**

<img width="173" alt="Screen Shot 2024-01-22 at 16 24 19" src="https://github.com/chile2706/8-week-sql/assets/147631781/a2421565-2472-4d9d-8016-8e3ba0750b9c">

#### 2. What is the average total historical deposit counts and amounts for all customers?
- Use `COUNT(DISTINCT)` to determine the total customers that the bank has had overall

```mysql
SELECT COUNT(DISTINCT ct.customer_id) FROM customer_transactions ct;
```
<img width="166" alt="Screen Shot 2024-01-22 at 16 31 25" src="https://github.com/chile2706/8-week-sql/assets/147631781/6086f201-81d4-4ed2-8820-dd1a2b23ed24">

- Use `COUNT()`, `SUM()` with `WHERE` clause - only choose `deposit` transaction type
```mysql
SELECT ROUND(COUNT(ct.txn_type)/ (SELECT COUNT(DISTINCT ct.customer_id) FROM customer_transactions ct),0) AS avg_count, 
FORMAT(SUM(txn_amount)/ (SELECT COUNT(DISTINCT ct.customer_id) FROM customer_transactions ct),0) AS avg_total_amount
FROM customer_transactions ct
WHERE ct.txn_type = 'deposit';
```
**Answers:**

<img width="166" alt="Screen Shot 2024-01-22 at 16 26 28" src="https://github.com/chile2706/8-week-sql/assets/147631781/8e20622d-564b-4f6f-a6f6-196dbcd58bfb">

#### 3. For each month - how many Data Bank customers make more than 1 deposit and either 1 purchase or 1 withdrawal in a single month?

- Use `GROUP BY()` and `CASE WHEN` to count how many times a month did each customer make deposit, purchase or withdrawal
```mysql
SELECT MONTH(ct.txn_date) AS month_id, ct.customer_id,
    SUM(CASE WHEN ct.txn_type = 'deposit' THEN 1 ELSE 0 END) AS deposit,
    SUM(CASE WHEN ct.txn_type = 'withdrawal' THEN 1 ELSE 0 END) AS withdrawal,
    SUM(CASE WHEN ct.txn_type = 'purchase' THEN 1 ELSE 0 end) AS purchase
  FROM customer_transactions ct
  GROUP BY month_id, ct.customer_id;
```

<img width="283" alt="Screen Shot 2024-01-22 at 16 44 59" src="https://github.com/chile2706/8-week-sql/assets/147631781/c548b5f1-f614-49f2-a708-46f43eccf995">


- Use `GROUP BY`, `COUNT()` and `WHERE` clause to count 
```mysql
SELECT a.month_id, COUNT(a.customer_id) AS total_customer
FROM
  (SELECT MONTH(ct.txn_date) AS month_id, ct.customer_id,
    SUM(CASE WHEN ct.txn_type = 'deposit' THEN 1 ELSE 0 END) AS deposit,
    SUM(CASE WHEN ct.txn_type = 'withdrawal' THEN 1 ELSE 0 END) AS withdrawal,
    SUM(CASE WHEN ct.txn_type = 'purchase' THEN 1 ELSE 0 end) AS purchase
  FROM customer_transactions ct
  GROUP BY month_id, ct.customer_id;) a
WHERE a.deposit > 1 AND (a.withdrawal = 1 OR a.purchase = 1)
GROUP BY a.month_id
ORDER BY a.month_id;
```
<img width="142" alt="Screen Shot 2024-01-22 at 16 43 08" src="https://github.com/chile2706/8-week-sql/assets/147631781/9474eabd-e240-46e7-86d7-f35a4bd79a24">

#### 4. What is the closing balance for each customer at the end of the month?
- Use `GROUP BY`, `CASE WHEN` to combine `txn_type` and `txn_amount` into `-(txn_amount)` or `txn_amount`, depends on whether it is `deposit` or not
```MYSQL
SELECT ct.customer_id,
  MONTH(ct.txn_date) AS month_id, 
  SUM(CASE WHEN ct.txn_type = 'deposit' THEN ct.txn_amount ELSE -ct.txn_amount END) AS total_txn
FROM customer_transactions ct
GROUP BY ct.customer_id,month_id
ORDER BY ct.customer_id;
```
<img width="175" alt="Screen Shot 2024-01-22 at 17 03 15" src="https://github.com/chile2706/8-week-sql/assets/147631781/88a8e7ea-9094-4b26-b347-4e50f0fb9a3e">

- Use `CASE WHEN` and `SUM()` to calculate total transaction of each month
```MYSQL
SELECT a.customer_id, 
    SUM(CASE WHEN a.month_id = 1 THEN a.total_txn ELSE 0 END) AS jan_txn,
    SUM(CASE WHEN a.month_id = 2 THEN a.total_txn ELSE 0 END) AS feb_txn,
    SUM(CASE WHEN a.month_id = 3 THEN a.total_txn ELSE 0 END) AS mar_txn,
    SUM(CASE WHEN a.month_id = 4 THEN a.total_txn ELSE 0 END) AS apr_txn
  FROM 
    (SELECT ct.customer_id,
      MONTH(ct.txn_date) AS month_id, 
      SUM(CASE WHEN ct.txn_type = 'deposit' THEN ct.txn_amount ELSE -ct.txn_amount END) AS total_txn
    FROM customer_transactions ct
    GROUP BY ct.customer_id,month_id
    ORDER BY ct.customer_id) a
  GROUP BY a.customer_id;
```
<img width="255" alt="Screen Shot 2024-01-22 at 16 59 29" src="https://github.com/chile2706/8-week-sql/assets/147631781/18a3a513-7cbc-4955-aa6d-82c6565027e3">

- Use `GROUP BY` to calculate the closing balance for each customer
```mysql
SELECT b.customer_id,
  b.jan_txn AS jan_balance,
  b.jan_txn + b.feb_txn AS feb_balance,
  b.jan_txn + b.feb_txn + b.mar_txn AS mar_balance,
  b.jan_txn + b.feb_txn + b.mar_txn + b.apr_txn AS apr_balance
FROM
  (SELECT a.customer_id, 
    SUM(CASE WHEN a.month_id = 1 THEN a.total_txn ELSE 0 END) AS jan_txn,
    SUM(CASE WHEN a.month_id = 2 THEN a.total_txn ELSE 0 END) AS feb_txn,
    SUM(CASE WHEN a.month_id = 3 THEN a.total_txn ELSE 0 END) AS mar_txn,
    SUM(CASE WHEN a.month_id = 4 THEN a.total_txn ELSE 0 END) AS apr_txn
  FROM 
    (SELECT ct.customer_id,
      MONTH(ct.txn_date) AS month_id, 
      SUM(CASE WHEN ct.txn_type = 'deposit' THEN ct.txn_amount ELSE -ct.txn_amount END) AS total_txn
    FROM customer_transactions ct
    GROUP BY ct.customer_id,month_id
    ORDER BY ct.customer_id) a
  GROUP BY a.customer_id) b;
```
**Answers:**

<img width="349" alt="Screen Shot 2024-01-22 at 16 53 44" src="https://github.com/chile2706/8-week-sql/assets/147631781/18bbda4f-aca3-4fa0-a530-77c63a1bcd4d">
