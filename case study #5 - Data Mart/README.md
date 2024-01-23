# Case Study #5 - Data Mart
<img width = 500 src = "https://github.com/chile2706/8-week-sql/assets/147631781/2ce3440a-b45a-49c6-8e1d-bc2f8ff25cc2">

## Table of Contents
* [Case Study Introduction](#case-study-introduction)
* [Entity Relationship Diagram](#entity-relationship-diagram)
* [Data Cleansing](#data-cleansing)
* [Case Study Questions & Solutions](#case-study-questions)
---
## Case Study Introduction
Data Mart is Danny’s latest venture and after running international operations for his online supermarket that specialises in fresh produce

In June 2020 - large scale supply changes were made at Data Mart. All Data Mart products now use sustainable packaging methods in every single step from the farm all the way to the customer.

Danny needs your help to quantify the impact of this change on the sales performance for Data Mart and it’s separate business areas.

## Entity Relationship Diagram
There is only 1 table
  

**Table :** `weekly_sales`
- Data Mart has international operations using a multi-`region` strategy
- Data Mart has both, a retail and online `platform` in the form of a Shopify store front to serve their customers
- Customer `segment` and `customer_type` data relates to personal age and demographics information that is shared with Data Mart
- `transactions` is the count of unique purchases made through Data Mart and `sales` is the actual dollar amount of purchases
- Each record in the dataset is related to a specific aggregated slice of the underlying sales data rolled up into a `week_date` value which represents the start of the sales week

<img width="436" alt="Screen Shot 2024-01-23 at 13 24 46" src="https://github.com/chile2706/8-week-sql/assets/147631781/dfa41449-f3fc-462a-aa11-2a53692a811a">



## Data Cleansing
In a single query, perform the following operations and generate a new table in the data_mart schema named `clean_weekly_sales`:

- Convert the `week_date` to a DATE format
```mysql
STR_TO_DATE(ws.week_date,'%d/%m/%y') AS week_date
```
- Add a `week_number` as the second column for each `week_date` value, for example any value from the 1st of January to 7th of January will be 1, 8th to 14th will be 2 etc
```mysql
WEEK(STR_TO_DATE(ws.week_date,'%d/%m/%y')) AS week_number
```
- Add a `month_number` with the calendar month for each `week_date` value as the 3rd column
```mysql
MONTH(STR_TO_DATE(ws.week_date,'%d/%m/%y')) AS month_number
```
- Add a `calendar_year` column as the 4th column containing either 2018, 2019 or 2020 values
```mysql
YEAR(STR_TO_DATE(ws.week_date,'%d/%m/%y')) AS calendar_year
```
- Add a new column called `age_band` after the original `segment` column using the following mapping on the number inside the segment value

|segment|age_band|
|-------|--------|
|1|Young Adults|
|2|Middle Aged|
|3 or 4|Retirees|

```MYSQL
CASE
  WHEN ws.segment LIKE '_1' THEN 'Young Adults'
  WHEN ws.segment LIKE '_2' THEN 'Middle Aged'
  WHEN ws.segment LIKE '_3' THEN 'Retirees'
  WHEN ws.segment LIKE '_4' THEN 'Retirees'
  ELSE NULL
  END AS age_band
```
- Add a new `demographic` column using the following mapping for the first letter in the `segment` values:

|segment|demographic|
|-------|--------|
|C|Couples|
|F|Families|
```MYSQL
CASE
  WHEN ws.segment LIKE 'C_' THEN 'Couples'
  WHEN ws.segment LIKE 'F_' THEN 'Families'
  ELSE NULL
  END AS demographic,
```
- Ensure all `null` string values with an `"unknown"` string value in the original `segment` column as well as the new `age_band` and `demographic` columns
```mysql
CASE
  WHEN ws.segment IS NULL OR ws.segment = 'null' THEN NULL
  ELSE ws.segment
  END AS segment
```
- Generate a new `avg_transaction` column as the `sales` value divided by `transactions` rounded to 2 decimal places for each record
```MYSQL
ROUND(ws.sales/ws.transactions, 2) AS avg_transaction
```
**Answers:**

```mysql
CREATE TABLE clean_weekly_sales AS
SELECT
  STR_TO_DATE(ws.week_date,'%d/%m/%y') AS week_date,
  WEEK(STR_TO_DATE(ws.week_date,'%d/%m/%y')) AS week_number,
  MONTH(STR_TO_DATE(ws.week_date,'%d/%m/%y')) AS month_number,
  YEAR(STR_TO_DATE(ws.week_date,'%d/%m/%y')) AS calendar_year,
  ws.region, ws.platform,
  CASE
    WHEN ws.segment IS NULL OR ws.segment = 'null' THEN NULL
    ELSE ws.segment
    END AS segment,
  CASE
    WHEN ws.segment LIKE '_1' THEN 'Young Adults'
    WHEN ws.segment LIKE '_2' THEN 'Middle Aged'
    WHEN ws.segment LIKE '_3' THEN 'Retirees'
    WHEN ws.segment LIKE '_4' THEN 'Retirees'
    ELSE NULL
    END AS age_band,
  CASE
    WHEN ws.segment LIKE 'C_' THEN 'Couples'
    WHEN ws.segment LIKE 'F_' THEN 'Families'
    ELSE NULL
    END AS demographic,
  ws.customer_type, ws.transactions, ws.sales,
  ROUND(ws.sales/ws.transactions, 2) AS avg_transaction
FROM weekly_sales ws;
```

<img width="973" alt="Screen Shot 2024-01-23 at 13 44 25" src="https://github.com/chile2706/8-week-sql/assets/147631781/a28d0d1f-b83d-49c3-b52c-0b656d563b50">

## Case Study Questions

### A. Data Exploration
#### 1. What day of the week is used for each `week_date` value?
-

```mysql
```

**Answers:**

#### 2. What range of week numbers are missing from the dataset?
-

```mysql
```

**Answers:**


#### 3. How many total transactions were there for each year in the dataset?
-

```mysql
```

**Answers:**


#### 4. What is the total sales for each region for each month?
-

```mysql
```

**Answers:**


#### 5. What is the total count of transactions for each platform
-

```mysql
```

**Answers:**


#### 6. What is the percentage of sales for Retail vs Shopify for each month?
-

```mysql
```

**Answers:**


#### 7. What is the percentage of sales by demographic for each year in the dataset?
-

```mysql
```

**Answers:**


#### 8. Which `age_band` and `demographic` values contribute the most to Retail sales?
-

```mysql
```

**Answers:**


#### 9. Can we use the `avg_transaction` column to find the average transaction size for each year for Retail vs Shopify? If not - how would you calculate it instead?
-

```mysql
```

**Answers:**
