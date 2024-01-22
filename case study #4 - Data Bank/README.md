# Case Study #4 - Data Bank
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
SELECT cn.region_id, r.region_name, COUNT(DISTINCT cn.customer_id) AS total_customers
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
SELECT cn.region_id, r.region_name, COUNT(DISTINCT cn.customer_id) AS total_customers
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
SELECT a.region_id, ROUND(a.total_relocate*0.5,0) AS '50th', ROUND(a.total_relocate*0.85,0) AS '85th', ROUND(a.total_relocate*0.9,0) AS '90th'
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
SELECT p.region_id, '50th' as percent, p.50th as ranking from percentile p
UNION ALL
SELECT p.region_id, '85th', p.85th FROM percentile p
UNION ALL
SELECT p.region_id, '90th', p.90th FROM percentile p;
```

<img width="147" alt="Screen Shot 2024-01-22 at 16 13 59" src="https://github.com/chile2706/8-week-sql/assets/147631781/af827db4-f2ac-42a7-b316-bfb395e8980a">


- Use `ROW_NUMBER()` to assign the order to date diff to match with the relocation days for each percentile later.
```MYSQL
SELECT cn.region_id, DATEDIFF(cn.end_date, cn.start_date) AS days, ROW_NUMBER() OVER(PARTITION BY cn.region_id ORDER BY DATEDIFF(cn.end_date, cn.start_date)) AS ranking
FROM customer_nodes cn
WHERE cn.end_date != '9999-12-31'
```
<img width="136" alt="Screen Shot 2024-01-22 at 16 10 28" src="https://github.com/chile2706/8-week-sql/assets/147631781/ec0c9140-031d-4c77-91e4-5390e4789259">

- Use `INNER JOIN` to join the latest two queries we just made `ON` the same `region_id` and `ranking` to get  the relocations day for each percentile
```mysql
SELECT a.region_id, a.percent, b.ranking, b.days
FROM
(SELECT p.region_id, '50th' as percent, p.50th as ranking from percentile p
UNION ALL
SELECT p.region_id, '85th', p.85th FROM percentile p
UNION ALL
select p.region_id, '90th', p.90th FROM percentile p) a
INNER JOIN
(SELECT cn.region_id, DATEDIFF(cn.end_date, cn.start_date) AS days, ROW_NUMBER() OVER(PARTITION BY cn.region_id ORDER BY DATEDIFF(cn.end_date, cn.start_date)) AS ranking
FROM customer_nodes cn
WHERE cn.end_date != '9999-12-31') b
ON a.ranking = b.ranking AND a.region_id = b.region_id
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
<img width="182" alt="Screen Shot 2024-01-22 at 16 19 18" src="https://github.com/chile2706/8-week-sql/assets/147631781/18514feb-7ef8-465b-bd89-55301b7a84ae">



### B. Customer Transactions
### C. Data Allocation Challenge
