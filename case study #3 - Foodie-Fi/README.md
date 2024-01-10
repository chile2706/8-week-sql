# Case Study #3 - Foodie-Fi
![image](https://github.com/chile2706/8-week-sql/assets/147631781/c71c6e7b-8480-4d95-a0d5-d345893a75bf)
## Table of Contents
* [Case Study Introduction](#case-study-introduction)
* [Entity Relationship Diagram](#entity-relationship-diagram)
* [Case Study Questions & Solutions](#case-study-questions-&-solutions)
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

## Case Study Questions & Solutions
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
-  use `CASE` to filter out customers who churned
-  use `ROUND` to roud the percentage to 1 decimal place
```mysql
SELECT
  SUM(CASE WHEN s.plan_id = 4 THEN 1 ELSE 0 END) AS churned_customers,
  ROUND(SUM(CASE WHEN s.plan_id = 4 THEN 1 ELSE 0 END)/COUNT(DISTINCT s.customer_id)*100,1) AS percentage
FROM subscriptions s;
```
**Answer:**

<img width="230" alt="Screen Shot 2024-01-10 at 09 50 18" src="https://github.com/chile2706/8-week-sql/assets/147631781/7ab49ca0-7088-4d79-9a8e-1149cdb48710">

#### 5. How many customers have churned straight after their initial free trial - what percentage is this rounded to the nearest whole number?
- if a customer churned right after their initial free trial, the count of their customer_id in the subscriptions table would be 2 (the trial plan and the churn plan)
- thus, determine the number of customers who churned and only had 2 records in the subscriptions table
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
#### 6. What is the number and percentage of customer plans after their initial free trial?


