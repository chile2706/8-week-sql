# Case Study #3 - Foodie-Fi
<img width="400" src ="https://github.com/chile2706/8-week-sql/assets/147631781/e6299734-ff9c-4336-b896-a1304311c525">

## Table of Contents
* [Case Study Introduction](#case-study-introduction)
* [Entity Relationship Diagram](#entity-relationship-diagram)
* [Case Study Questions & Solutions](#case-study-questions)
---
## Case Study Introduction
Danny with his friends launched the startup Foodie-Fi, new streaming service that only had food related content - something like Netflix but with only cooking shows, in 2020 and started selling monthly and annual subscriptions, giving their customers unlimited on-demand access to exclusive food videos from around the world!

This case study focuses on using subscription style digital data to answer important business questions.
## Entity Relationship Diagram
![image](https://github.com/chile2706/8-week-sql/assets/147631781/a870364b-c228-4033-84a9-bb9913c86fac)

**Table 1:** `plan`

<img width="312" alt="Screen Shot 2024-01-09 at 10 37 51" src="https://github.com/chile2706/8-week-sql/assets/147631781/f8c35e0a-501f-452f-a96c-217c029da389">

There are 5 subscription plans:
* Trial plan - Customers can sign up to an initial 7 day free trial and will automatically continue with the pro monthly subscription plan unless they cancel, downgrade to basic or upgrade to an annual pro plan at any point during the trial.
* Basic plan - Customers have limited access and can only stream their videos and is only available monthly at $9.90
* Pro plan  - Customers have no watch time limits and can download videos for offline viewing. Pro plans start at $19.90 a month or $199 for an annual subscription.
* Churn plan - When customers cancel their Foodie-Fi service they will have a `churn` plan record with a `null` price but their plan will continue until the end of the billing period.

**Table 2:** `subscriptions`

<img width="368" alt="Screen Shot 2024-01-09 at 10 43 02" src="https://github.com/chile2706/8-week-sql/assets/147631781/15f8b8e3-6fc2-4279-aa97-1f692fba6807">

Customer subscriptions show the exact date where their specific `plan_id` starts.

If customers downgrade from a pro plan or cancel their subscription - the higher plan will remain in place until the period is over - the `start_date` in the `subscriptions` table will reflect the date that the actual plan changes.

When customers upgrade their account from a basic plan to a pro or annual pro plan - the higher plan will take effect straightaway.

When customers churn - they will keep their access until the end of their current billing period but the `start_date` will be technically the day they decided to cancel their service.

## Case Study Questions
### A. Data Analysis Questions
#### 1. How many customers has Foodie-Fi ever had?
- Use  `COUNT()` function combined with `DISTINCT` to get the count of unique customers
```mysql
SELECT COUNT(DISTINCT s.customer_id) AS total_customer
FROM subscriptions s;
```
**Answer:**

<img width="150" alt="Screen Shot 2024-01-10 at 09 24 15" src="https://github.com/chile2706/8-week-sql/assets/147631781/c9b4ff51-42c2-40ce-9980-034bdc324ed9">


#### 2. What is the monthly distribution of trial plan start_date values for our dataset - use the start of the month as the group by value
- Use `GROUP BY` to group the total users by month
- Filter the results so that the user's `plan_id` is 0 for each month
```mysql
SELECT MONTH(s.start_date) as ind, MONTHNAME(s.start_date) as month, COUNT(s.plan_id) as count 
FROM subscriptions s
WHERE s.plan_id = 0
GROUP BY ind,month 
ORDER BY ind ASC;
```
**Answer:**

<img width="185" alt="Screen Shot 2024-01-10 at 09 36 03" src="https://github.com/chile2706/8-week-sql/assets/147631781/b79fd36c-e52a-4240-9d24-4439d08de735">

#### 3. What plan start_date values occur after the year 2020 for our dataset? Show the breakdown by count of events for each plan_name
- determine the count of each plan by using `GROUP BY` and `COUNT(start_date)`
- filter the results with `start_date >= '2021-01-01'`
- `JOIN` plans and subscriptions to get `plan_name`
```mysql
SELECT
  p.plan_id, p.plan_name,
  COUNT(s.start_date) AS count
FROM subscriptions s
NATURAL JOIN plans p
WHERE s.start_date >= '2021-01-01'
GROUP BY p.plan_id, p.plan_name
ORDER BY p.plan_id;
```
**Answer:**

<img width="200" alt="Screen Shot 2024-01-10 at 09 44 12" src="https://github.com/chile2706/8-week-sql/assets/147631781/7a62bb67-0737-4dee-b454-7b4a3f086e74">

#### 4. What is the customer count and percentage of customers who have churned rounded to 1 decimal place?
-  Use `CASE` to filter out customers who churned
-  Use `ROUND` to roud the percentage to 1 decimal place
```mysql
SELECT
  SUM(CASE WHEN s.plan_id = 4 THEN 1 ELSE 0 END) AS churned_customers,
  ROUND(SUM(CASE WHEN s.plan_id = 4 THEN 1 ELSE 0 END)/COUNT(DISTINCT s.customer_id)*100,1) AS percentage
FROM subscriptions s;
```
**Answer:**

<img width="230" alt="Screen Shot 2024-01-10 at 09 50 18" src="https://github.com/chile2706/8-week-sql/assets/147631781/7ab49ca0-7088-4d79-9a8e-1149cdb48710">

#### 5. How many customers have churned straight after their initial free trial - what percentage is this rounded to the nearest whole number?
- If a customer churned right after their initial free trial, the count of their customer_id in the subscriptions table would be 2 (the trial plan and the churn plan)
- Thus, determine the number of customers who churned and only had 2 records in the subscriptions table
```mysql
SELECT
  c.count, round(c.count/1000*100,0) as percentage
FROM (
  SELECT count(a.customer_id) AS count
  FROM (
    SELECT s.customer_id, count(s.customer_id) AS count
    FROM subscriptions s
    GROUP BY s.customer_id
    HAVING count = 2) a
  INNER JOIN (
    SELECT distinct s.customer_id
    FROM subscriptions s
    WHERE s.plan_id = 4) b
  ON a.customer_id = b.customer_id) c;
```
**Answer:**

<img width="150" alt="Screen Shot 2024-01-10 at 10 40 24" src="https://github.com/chile2706/8-week-sql/assets/147631781/fb352fed-48a5-4950-8460-774c1f820c60">

#### 6. What is the number and percentage of customer plans after their initial free trial?
- Approach: rank the customer choice of plan in ascending order by `start_date`, then the first plan would be the trial plan and the `second` one is what you are looking for
- Use `RANK() OVER(PARTITION BY ORDER BY)` to rank the customer choice of plan and choose `plan_id` which its ranking is 2

```mysql
SELECT
  t.plan_id, p.plan_name,
  COUNT(t.plan_id) AS count,
  ROUNF(COUNT(t.plan_id)/(SELECT COUNT(DISTINCT s.customer_id) FROM subscriptions s)*100,1) AS percentage 
FROM
  (SELECT s.customer_id, s.plan_id, RANK() OVER(PARTITION BY s.customer_id ORDER BY start_date ASC)AS ranking
  FROM subscriptions s) t
NATURAL JOIN plans p
WHERE t.ranking = 2
GROUP BY t.plan_id, p.plan_name
ORDER BY t.plan_id;
```
**Answer:**

<img width="330" alt="Screen Shot 2024-01-10 at 10 40 41" src="https://github.com/chile2706/8-week-sql/assets/147631781/05c70070-0bc2-4942-be87-1230b4a9d99c">

#### 7. What is the customer count and percentage breakdown of all 5 plan_name values at 2020-12-31?
- Approach: filter out customers that were in the system before 2021, find the last plan of customers before 2021
- Use `SUM(IF (s.start_date <= '2020-12-31', 1, 0)) AS date_check` to find how many plans a customer switched to before 2021
- Use `RANK() OVER (PARTITION BY s.customer_id ORDER BY s.start_date ASC)` to rank the customer plan according to `start_date`
- Return the `plan_id` where its ranking = `date_check` of a customer: this is the `plan_id` of the customer at 2020-12-31
- 
```mysql
SELECT
  b.plan_id, p.plan_name, COUNT(b.plan_id) AS count,
  ROUND(COUNT(b.plan_id)/(SELECT COUNT(DISTINCT s.customer_id) FROM subscriptions s WHERE s.start_date <= '2020-12-31')*100,0) AS percent
FROM
  (SELECT
    s.customer_id, s.plan_id,
    RANK() OVER (PARTITION BY s.customer_id ORDER BY s.start_date ASC) AS ranking, a.date_check
  FROM subscriptions s 
  LEFT JOIN
    (SELECT
      s.customer_id, SUM(IF (s.start_date <= '2020-12-31', 1, 0)) AS date_check
    FROM subscriptions s
    GROUP BY s.customer_id
    HAVING MIN(s.start_date) <= '2020-12-31') a 
  ON a.customer_id = s.customer_id) b
NATURAL JOIN plans p
WHERE b.ranking = b.date_check
GROUP BY b.plan_id, p.plan_name;
```
**Answer:**

<img width="250" alt="Screen Shot 2024-01-10 at 10 55 18" src="https://github.com/chile2706/8-week-sql/assets/147631781/ba045b0e-31b7-41cb-9b2e-3e39904c3d74">

####  8. How many customers have upgraded to an annual plan in 2020?
- Approach: filter out customers whose record of `plan_name = 'pro annual'` and `start_date` in 2020 exists

```mysql
SELECT COUNT(DISTINCT s.customer_id) AS count
FROM subscriptions s
WHERE s.plan_id IN (SELECT p.plan_id FROM plans p WHERE p.plan_name = 'pro annual')
AND s.start_date BETWEEN '2020-01-01' AND '2020-12-31';
```
**Answers:**

<img width="50" alt="Screen Shot 2024-01-10 at 11 00 06" src="https://github.com/chile2706/8-week-sql/assets/147631781/5e83fc35-3308-4987-af84-fa5e3d128dea">

#### 9. How many days on average does it take for a customer to an annual plan from the day they join Foodie-Fi?
- Approach: filter out customers that did upgrade to `annual plan` and calcualte the difference between `start_date` and `upgrade_date`
- Use `INNER JOIN` to filter out customers had annual plan
- Use `DATEDIFF()` to find the difference between `start_date` and `upgrade_date`
```mysql
SELECT
  ROUND(SUM(DATEDIFF(b.upgrade_date, a.start_date))/COUNT(DISTINCT b.customer_id), 0) AS avg_day
FROM
  (SELECT s.customer_id, MIN(s.start_date) AS start_date
  FROM subscriptions s
  GROUP BY s.customer_id) a
INNER JOIN
  (SELECT s.customer_id, MIN(s.start_date) as upgrade_date
  FROM subscriptions s
  WHERE
    s.plan_id IN (SELECT p.plan_id FROM plans p WHERE p.plan_name = 'pro annual')
  GROUP BY s.customer_id) b 
ON a.customer_id = b.customer_id;
```
**Answers:**

<img width="70" alt="Screen Shot 2024-01-10 at 11 11 06" src="https://github.com/chile2706/8-week-sql/assets/147631781/2d9c5aab-1d24-4ce4-b1b7-d07d9dcf566b">

#### 10. Can you further breakdown this average value into 30 day periods (i.e. 0-30 days, 31-60 days etc)
- Approach: calculate the date difference like in Q9 and then categorize the difference into each period
- Use `CEIL(DATEDIFF(b.upgrade_date, a.start_date)/30)` and `GROUP BY` to categorize the date difference into each period
- Use `CONCAT()` to format the string `'XX - XX days'`

```mysql
SELECT
  CONCAT((days - 1)*30 + 1, '-', days*30, ' days') AS 30_day_period, c.count
FROM
  (SELECT 
    CEIL(DATEDIFF(b.upgrade_date, a.start_date)/30) AS days,
    COUNT( b.customer_id) AS count
  FROM
    (SELECT s.customer_id, MIN(s.start_date) AS start_date
    FROM subscriptions s
    GROUP BY s.customer_id) a
  INNER JOIN 
    (SELECT s.customer_id, MIN(s.start_date) AS upgrade_date
    FROM subscriptions s
    WHERE s.plan_id IN (SELECT p.plan_id FROM plans p WHERE p.plan_name = 'pro annual')
    GROUP BY s.customer_id) b 
  ON a.customer_id = b.customer_id
  GROUP BY days) c
ORDER BY c.days ASC;
```

**Answers:**

<img width="200" alt="Screen Shot 2024-01-10 at 11 23 10" src="https://github.com/chile2706/8-week-sql/assets/147631781/0d637821-99d4-4e45-845c-5dff54317705">

#### 11. How many customers downgraded from a pro monthly to a basic monthly plan in 2020?
- Approach: ranking records of each customer by start_date, determine the number of customers whose ranking of `pro_monthly` is smaller than `basic_monthly`
- Use  `RANK() OVER(PARTITION BY s.customer_id ORDER BY s.start_date ASC)` to rank
- Use `INNER JOIN` to filter out customers had at least once subscribed to pro_monthly

```mysql
SELECT count(b.customer_id) as count
FROM
  (SELECT a.customer_id, a.ranking AS pro_rank
  FROM
    (SELECT
      s.customer_id, s.plan_id, s.start_date,
      RANK() OVER(PARTITION BY s.customer_id ORDER BY s.start_date ASC) AS ranking
    FROM subscriptions s
    WHERE s.start_date BETWEEN '2020-01-01' AND '2020-12-31') a
  WHERE
    a.plan_id IN (SELECT p.plan_id FROM plans p WHERE p.plan_name = 'pro monthly')) b
  INNER JOIN
  (SELECT c.customer_id, c.ranking AS basic_rank
  FROM
    (SELECT
      s.customer_id, s.plan_id, s.start_date,
      RANK() OVER(PARTITION BY s.customer_id ORDER BY s.start_date ASC) AS ranking
    FROM subscriptions s
    WHERE s.start_date BETWEEN '2020-01-01' AND '2020-12-31') c
  WHERE c.plan_id IN (SELECT p.plan_id FROM plans p WHERE p.plan_name = 'basic monthly')) d
ON d.customer_id = b.customer_id
WHERE b.pro_rank < d.basic_rank;
```

**Answer:**

<img width="50" alt="Screen Shot 2024-01-10 at 12 49 08" src="https://github.com/chile2706/8-week-sql/assets/147631781/a640c5d0-2230-4eff-b20b-8ddef2bec761">

