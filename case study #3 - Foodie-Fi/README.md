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
I use the `COUNT()` function combined with `DISTINCT` to get the count of unique customers
```mysql
SELECT COUNT(DISTINCT s.customer_id) AS total_customer
FROM subscriptions s;
```
**Answer:**


#### 2. What is the monthly distribution of trial plan start_date values for our dataset - use the start of the month as the group by value
