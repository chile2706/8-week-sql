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
**Table :** `clean_weekly_sales`

<img width="973" alt="Screen Shot 2024-01-23 at 13 44 25" src="https://github.com/chile2706/8-week-sql/assets/147631781/a28d0d1f-b83d-49c3-b52c-0b656d563b50">

## Case Study Questions

### A. Data Exploration
#### 1. What day of the week is used for each `week_date` value?
- Use `DAYNAME()` and keyword `DISTINCT` to get the day of the week used in this case

```mysql
SELECT DISTINCT DAYNAME(c.week_date) AS day_of_week
FROM clean_weekly_sales c;
```

**Answers:**

<img width="78" alt="Screen Shot 2024-01-23 at 13 49 08" src="https://github.com/chile2706/8-week-sql/assets/147631781/2011f0a1-ce23-4527-9515-50bcaf350e1e">

Thus, Monday is used for each `week_date` value

#### 2. What range of week numbers are missing from the dataset?
- Use keyword `DISTINCT` and `ORDER BY` to get the existing week numbers

```mysql
SELECT DISTINCT c.week_number
FROM clean_weekly_sales c
ORDER BY c.week_number;
```

**Answers:**

<img width="78" alt="Screen Shot 2024-01-23 at 13 52 13" src="https://github.com/chile2706/8-week-sql/assets/147631781/5f9bd2b7-e491-47b0-b277-80f9fa90914f">

Thus, we are missing week 1 to week 11 and week 36 to week 52


#### 3. How many total transactions were there for each year in the dataset?
- Use `GROUP BY` with `calendar_year` and `SUM()` to calculate total transactions for each YEAR
- Use `FORMAT()` for better format

```mysql
SELECT c.calendar_year AS year, FORMAT(SUM(c.transactions),0) AS total_transactions
FROM clean_weekly_sales c
GROUP BY c.calendar_year
ORDER BY c.calendar_year DESC;
```

**Answers:**

<img width="148" alt="Screen Shot 2024-01-23 at 13 53 13" src="https://github.com/chile2706/8-week-sql/assets/147631781/58bf88a1-ec31-4ec5-a8d2-85cf3d7d86a9">


#### 4. What is the total sales for each region for each month?
> Unpivot table (bad format)
- Use `GROUP BY` with `region`, `calendar_year`, `month_number` and `SUM()` to calculate total transactions for each REGION for each MONTH

```mysql
SELECT c.region, c.calendar_year, c.month_number, FORMAT(SUM(c.sales),0) AS total_sales
FROM clean_weekly_sales c
GROUP BBY c.region, c.month_number, c.calendar_year
ORDER BY c.region, c.calendar_year desc, c.month_number;
```

**Answers:**

<img width="342" alt="Screen Shot 2024-01-23 at 13 57 18" src="https://github.com/chile2706/8-week-sql/assets/147631781/1aa2f11d-4206-4bbc-ae67-5035bc14982c">
  
> Pivot table (good format)
- Use `GROUP BY` with `calendar_year`, `month_number` for ROW
- Use `CASE WHEN` with `regionn` and `SUM()` for COLUMN

```mysql
SELECT c.calendar_year, c.month_number, 
FORMAT(SUM(CASE WHEN c.region = 'AFRICA' THEN c.sales ELSE 0 END),0) AS AFRICA,
FORMAT(SUM(CASE WHEN c.region = 'ASIA' THEN c.sales ELSE 0 END),0) AS ASIA,
FORMAT(SUM(CASE WHEN c.region = 'EUROPE' THEN c.sales ELSE 0 END),0) AS EUROPE,
FORMAT(SUM(CASE WHEN c.region = 'OCEANIA' THEN c.sales ELSE 0 END),0) AS OCEANIA,
FORMAT(SUM(CASE WHEN c.region = 'SOUTH_AMERICA' THEN c.sales ELSE 0 END),0) AS SOUTH_AMERICA,
FORMAT(SUM(CASE WHEN c.region = 'USA' THEN c.sales ELSE 0 END),0) AS USA
FROM clean_weekly_sales c
GROUP BY c.calendar_year, c.month_number
ORDER BY c.calendar_year DESC, c.month_number;
```

**Answers:**

<img width="692" alt="Screen Shot 2024-01-23 at 13 57 52" src="https://github.com/chile2706/8-week-sql/assets/147631781/121fbe24-eff2-4ee2-a789-e84d3dc46658">

#### 5. What is the total count of transactions for each platform
- Use `GROUP BY` with `platform` and `SUM()` to get the total count of transactions of each PLATFORM

```mysql
SELECT c.platform, FORMAT(SUM(c.transactions),0) AS total_transactions
FROM clean_weekly_sales c
GROUP BY c.platform;
```

**Answers:**

<img width="150" alt="Screen Shot 2024-01-23 at 14 06 18" src="https://github.com/chile2706/8-week-sql/assets/147631781/8c43d5f7-f9dd-4266-ab5a-d0b7fa0c4c5e">


#### 6. What is the percentage of sales for Retail vs Shopify for each month?
- `GROUP BY` and `CASE WHEN`

```mysql
SELECT c.calendar_year, c.month_number,
  ROUND(SUM(CASE WHEN c.platform = 'Shopify' THE c.sales ELSE 0 end)/SUM(c.sales)*100,2) AS shopify,
  ROUND(SUM(CASE WHEN c.platform = 'Retail' THE c.sales ELSE 0 end)/SUM(c.sales)*100,2) AS retail
FROM clean_weekly_sales c
GROUP BY c.calendar_year, c.month_number
ORDER BY c.calendar_year DESC, c.month_number;
```

**Answers:**

<img width="255" alt="Screen Shot 2024-01-23 at 14 06 56" src="https://github.com/chile2706/8-week-sql/assets/147631781/6538f7b3-0715-41e1-9b98-f41c32de6814">


#### 7. What is the percentage of sales by demographic for each year in the dataset?
- `GROUP BY` and `CASE WHEN`

```mysql
SELECT c.calendar_year, 
ROUND(SUM(CASE WHEN c.demographic = 'Couples' THEN c.sales ELSE 0 END)/SUM(c.sales)*100,2) AS couples,
ROUND(SUM(CASE WHEN c.demographic = 'Families' THEN c.sales ELSE 0 END)/SUM(c.sales)*100,2) AS families,
ROUND(SUM(CASE WHEN c.demographic IS NULL THEN c.sales ELSE 0 END)/SUM(c.sales)*100,2) AS 'null'
FROM clean_weekly_sales c
GROUP BY c.calendar_year
ORDER BY c.calendar_year ASC;
```

**Answers:**

<img width="211" alt="Screen Shot 2024-01-23 at 14 11 21" src="https://github.com/chile2706/8-week-sql/assets/147631781/b07fa8d6-aa90-4863-b703-586fc7af2558">


#### 8. Which `age_band` and `demographic` values contribute the most to Retail sales?
- Use `GROUP BY` with `age_band` to calculate total sales of each age_band and then use `ORDER BY` combined with `LIMIT 1` to get the `age_band` that contributes the most
- Use `UNION ALL` to combine both `age_band` and `demographic` in one query

```mysql
SELECT 'age_band' AS category, a.age_band AS value
FROM
  (SELECT c.age_band, FORMAT(SUM(c.sales),0) AS total_sales
  FROM clean_weekly_sales c
  WHERE c.platform = 'Retail'
  AND c.age_band IS NOT NULL
  GROUP BY c.age_band
  ORDER BY total_sales DESC
  LIMIT 1) a
UNION ALL
SELECT 'demographic', b.demographic
FROM
  (SELECT c.demographic, FORMAT(SUM(c.sales),0) AS total_sales
  FROM clean_weekly_sales c
  WHERE c.platform = 'Retail'
  AND c.demographic IS NOT NULL
  GROUP BY c.demographic
  ORDER BY total_sales desc
  LIMIT 1) b;
```

**Answers:**

<img width="143" alt="Screen Shot 2024-01-23 at 14 18 18" src="https://github.com/chile2706/8-week-sql/assets/147631781/d158239f-02fc-4869-8898-7d1e6a386a91">
Thus, if not taking unkown `demographic` and `age_band` into account, `Middled Aged` and `Families` contribute the most to `total_sales`

#### 9. Can we use the `avg_transaction` column to find the average transaction size for each year for Retail vs Shopify? If not - how would you calculate it instead?
- We can not use the `avg_transaction` to find the average transactionn size for each year
- Instead, we need to use `GROUP BY` and `CASE WHEN` to get the sum of total_sales and then number of transactions for each platform for each year

```mysql
SELECT c.calendar_year,
ROUND(SUM(CASE WHEN c.platform = 'Retail' THEN c.sales ELSE 0 END)/SUM(CASE WHEN c.platform = 'Retail' THEN c.transactions ELSE 0 END),2) AS retail,
ROUND(SUM(CASE WHEN c.platform = 'Shopify' THEN c.sales ELSE 0 END)/SUM(CASE WHEN c.platform = 'Shopify' THEN c.transactions ELSE 0 END),2) AS shopify
FROM clean_weekly_sales c
GROUP BY c.calendar_year;
```

**Answers:**
<img width="160" alt="Screen Shot 2024-01-24 at 13 35 34" src="https://github.com/chile2706/8-week-sql/assets/147631781/16a6ba7c-9d75-4b4d-a188-08ae7d4cabaa">

### B. Before & After Analysis
Taking the `week_date` value of `2020-06-15` as the baseline week where the Data Mart sustainable packaging changes came into effect.

We would include all `week_date` values for `2020-06-15` as the start of the period **after** the change and the previous week_date values would be **before**

Using this analysis approach - answer the following questions:

#### 1. What is the total sales for the 4 weeks before and after 2020-06-15? What is the growth or reduction rate in actual values and percentage of sales?
- Use `DATE_ADD()`, `DATE_SUB()` and `CASE` to get data for 4 weeks before and after `2020-06-15`
```mysql
SELECT
  FORMAT(a.sale_after - a.sale_before, 0) AS actual_value,
  ROUND((a.sale_after - a.sale_before)/a.sale_before*100, 2) AS percentage
FROM
  (SELECT 
    SUM(CASE WHEN c.week_date BETWEEN '2020-06-15' AND DATE_ADD('2020-06-15', INTERVAL 3 WEEK) THEN c.sales ELSE 0 END) AS sale_after,
    SUM(CASE WHEN c.week_date BETWEEN DATE_SUB('2020-06-15', INTERVAL 4 WEEK) AND DATE_SUB('2020-06-15', INTERVAL 1 WEEK)
      THEN c.sales ELSE 0 END) AS sale_before
  FROM clean_weekly_sales c) a;
```

**Answers:**

<img width="217" alt="Screen Shot 2024-01-24 at 13 42 00" src="https://github.com/chile2706/8-week-sql/assets/147631781/f639820b-87f8-483c-8ce8-569be5c90f4f">

#### 2. What about the entire 12 weeks before and after?
- Same approach as Q1: `DATE_ADD()`, `DATE_SUB()` and `CASE`
  
```mysql
SELECT
  FORMAT(a.sale_after - a.sale_before, 0) AS actual_value,
  ROUND((a.sale_after - a.sale_before)/a.sale_before*100, 2) AS percentage
FROM
  (SELECT 
    SUM(CASE WHEN c.week_date BETWEEN '2020-06-15' AND DATE_ADD('2020-06-15', INTERVAL 11 WEEK) THEN c.sales ELSE 0 END) AS sale_after,
    SUM(CASE WHEN c.week_date BETWEEN DATE_SUB('2020-06-15', INTERVAL 12 WEEK) AND DATE_SUB('2020-06-15', INTERVAL 1 WEEK)
      THEN c.sales ELSE 0 END) AS sale_before
  FROM clean_weekly_sales c) a;
```

**Answers:**

<img width="217" alt="Screen Shot 2024-01-24 at 13 48 25" src="https://github.com/chile2706/8-week-sql/assets/147631781/30f9c66d-073e-4039-949a-41aca6508c1e">

#### 3. How do the sale metrics for these 2 periods before and after compare with the previous years in 2018 and 2019?
- Use `GROUP BY` with `calendar_year` and `CASE` to get the data for 2 periods for each year
```mysql
SELECT c.calendar_year,
  FORMAT(SUM(CASE WHEN MONTH(c.week_date) > 6 OR (MONTH(c.week_date) = 6 AND DAY(c.week_date) >= 15) THEN c.sales ELSE 0 END),0) AS sale_after,
  FORMAT(SUM(CASE WHEN MONTH(c.week_date) < 6 OR (MONTH(c.week_date) = 6 AND DAY(c.week_date) < 15) THEN c.sales ELSE 0 END),0) AS sale_before
FROM clean_weekly_sales c
GROUP BY c.calendar_year;
```

**Answers:**
