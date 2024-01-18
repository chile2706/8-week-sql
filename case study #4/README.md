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


