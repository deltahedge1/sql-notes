# SQL Aggregations

## Introduction

In this lesson, we will cover and you will be able to:

- Deal with `NULL` values
- Create aggregations in your SQL Queries including
- `COUNT`
- `SUM`
- `MIN` & `MAX`
- `AVG`
- `GROUP BY`
- `DISTINCT`
- `HAVING`
- Create `DATE` functions
- Implement `CASE` statements

## Summary of Aggregations
|Clause|Description|Null Treatment|Alpha treatement|
|------|-----------|--------------|----------------|
|`COUNT`|Counts up all items in a column|ignores `NULL` values|Can be used|
|`SUM`|Adds up all numbers in a column|ignores `NULL` values| Cannot be used|
|`MIN`|returns the lowest number, earliest date, or non-numerical value as early in the alphabet as possible (a- Z)|ignores `NULL` values| Can be used|
|`MAX`|returns the highest number, earliest date, or non-numerical value as early in the alphabet as possible (Z- a)|ignores `NULL` values| Can be used|
|`AVG`|returns the mean of the data|ignores `NULL` values| Cannot be used|

## NULL

Datatype that specifiies that there is no data exists i.e. it does not necessarily mean 0 it is the absence of data.

__Protip__ We don't use =, because NULL isn't considered a value in SQL. Rather, it is a property of the data.

Filtering for `NULL`

```sql
SELECT *
FROM table
WHERE col_1 IS NULL
```

Filtering for `NOT NULL`
```sql
SELECT *
FROM table
WHERE col_1 IS NOT NULL
```

## Aggregations

### COUNT

`COUNT` over all rows in a table

```sql
SELECT COUNT(*)
FROM accounts;
```

`COUNT` over speciic column (this will count only non `NULL` values)

```sql
SELECT COUNT(accounts.id)
FROM accounts;
```

### SUM

```sql
SELECT SUM(standard_qty) AS standard,
       SUM(gloss_qty) AS gloss,
       SUM(poster_qty) AS poster
FROM orders
```

### MIN and MAX

```sql
SELECT MIN(standard_qty) AS standard_min,
       MIN(gloss_qty) AS gloss_min,
       MIN(poster_qty) AS poster_min,
       MAX(standard_qty) AS standard_max,
       MAX(gloss_qty) AS gloss_max,
       MAX(poster_qty) AS poster_max
FROM   orders
```

### AVG

```sql
SELECT AVG(standard_qty) AS standard_avg,
       AVG(gloss_qty) AS gloss_avg,
       AVG(poster_qty) AS poster_avg
FROM orders
```

__`AVG` when you want to include NULLs as 0__

- Use the `COALESCE` function

```sql
SELECT AVG(COALESCE(amount, 0)) AS average_amount
FROM table
```

The innermost subquery retrieves the column_name values, assigns a unique row_num to each row using the ROW_NUMBER() function, and calculates the total number of rows in the table using the COUNT(*) OVER() function.

```sql
/* output of subquery */
+-----------+--------+------------+
| column_name | row_num | total_rows |
+-----------+--------+------------+
| 50.00 |      5 |         10 |
| 60.00 |      6 |         10 |
+-----------+--------+------------+
```

The middle subquery filters the rows that have a row_num equal to either FLOOR((total_rows + 1) / 2.0) or CEILING((total_rows + 1) / 2.0).

If the number of rows in the table is odd, FLOOR((total_rows + 1) / 2.0) returns the exact middle row number, and the WHERE clause selects that row.

If the number of rows in the table is even, FLOOR((total_rows + 1) / 2.0) and CEILING((total_rows + 1) / 2.0) return the two middle row numbers, and the WHERE clause selects those two rows.

The outermost query calculates the median by taking the AVG() of the selected rows.

In this case, since the sales table has an even number of rows, the middle subquery selects rows 5 and 6 with sales amounts of 50.00 and 60.00.

### Median (no function out the box)

`MEDIAN` is more advanced to calculate

```sql
SELECT column_name, row_num, total_rows
FROM (
  SELECT column_name, 
  ROW_NUMBER() OVER (ORDER BY column_name) AS row_num, 
  COUNT(*) OVER () AS total_rows
  FROM sales
) subquery
WHERE row_num = FLOOR((total_rows + 1) / 2.0) OR row_num = CEILING((total_rows + 1) / 2.0);
```

### GROUP BY

- Used to aggregate subsets of data e.g. by region, age, gender
- Any column not in the `SELECT` statement that is not within an aggregator must be in the `GROUP BY` clause
- Goes between `WHERE` and `ORDER BY`
```sql
SELECT col1, col2, SUM(col2) total
FROM table
WHERE col2 = 'M'
GROUP BY col1
ORDER BY total
```
- `ORDER BY` is temporary and will not change the table
- Evaluated before the `LIMIT` so all aggregations will be performed so there are no errors, but that means that if you are using `LIMIT` for performance reasons you won't get any benefit
```sql
/* if you want a performance benefit use a subquery */
SELECT col1, col2, SUM(col2) total
FROM (SELECT * FROM table LIMIT 100) as t1
WHERE col2 = 'M'
GROUP BY col1
ORDER BY total
```
- You can `GROUP BY` multiple columns at once
    - order of the columns in the `ORDER BY` does not matter
    - you can subsitute numbers for columns

### DISTINCT

- Provides unique rows for all columns
- `DISTINCT` is always used in `SELECT` statements
- Only use `DISTINCT` once
    - Correct
    ```sql
    SELECT DISTINCT column1, column2
    FROM table
    ```
    - Wrong!
    ```sql
    SELECT DISTINCT column1, DISTINCT column2 /* do not do this */
    FROM table
    ```

Case study

Example data

| sale_id | customer_name | product_name | sale_date  |
|---------|---------------|--------------|------------|
| 1       | John          | T-shirt      | 2022-01-01 |
| 2       | John          | T-shirt      | 2022-01-03 |
| 3       | Sarah         | Hoodie       | 2022-01-02 |
| 4       | Jane          | Jeans        | 2022-01-04 |
| 5       | Jane          | Jeans        | 2022-01-05 |
| 6       | John          | Jeans        | 2022-01-03 |
| 7       | Sarah         | T-shirt      | 2022-01-04 |
| 8       | Jane          | Hoodie       | 2022-01-05 |

Distinct on one column

```sql
SELECT DISTINCT customer_name
FROM sales;
```

Output


| customer_name |
|---------------|
| John          |
| Sarah         |
| Jane          |

Distinct on two columns

```sql
SELECT DISTINCT customer_name, product_name
FROM sales;
```

| customer_name | product_name |
|---------------|--------------|
| John          | T-shirt      |
| Sarah         | Hoodie       |
| Jane          | Jeans        |
| John          | Jeans        |
| Sarah         | T-shirt      |
| Jane          | Hoodie       |

Using Distinct with aggregations

```sql
SELECT COUNT(DISTINCT customer_name), COUNT(DISTINCT product_name)
FROM orders 
```

output

| COUNT(DISTINCT customer_name)| COUNT(DISTINCT product_name)|
|-----------------------------|------------------------------|
|            3                |             3                |


### HAVING

- Filtering a query that has aggregations
- Anytime you want to perform a `WHERE` on an aggregation you must use  `HAVING`

| WHERE | HAVING|
|-------|-------|
|`WHERE` subsets the returned data based on logical condition| works on logical statements involving aggregations|
|appears after the `FROM`, `JOIN` and `ON` but before `GROUP BY`| appears after the `GROUP BY` but before `ORDER BY`|

simple example

```sql
SELECT account_id,
       SUM(total_amt_usd) AS sum_total_amt_usd
FROM orders
GROUP BY 1
HAVING SUM(total_amt_usd) >= 250000
```

example of `WHERE` and `HAVING`

```sql
SELECT customer_id, COUNT(*) as num_orders, SUM(order_total) as total_sales
FROM orders
WHERE order_date BETWEEN '2022-01-01' AND '2022-12-31' -- Only orders in 2022
GROUP BY customer_id
HAVING COUNT(*) > 5 AND SUM(order_total) > 1000
ORDER BY total_sales
```

### DATE FUNCTIONS

Database format `YYYY MM DD`
- dates sorted alphabetically are in chronological order


#### DATE_TRUNC
- truncates the date to the required position
- syntax
    ```sql
    DATE_TRUNC('<date option e.g. day>', column_name)
    ```
- options
    - `day`
    - `month`
    - `year`

![Date Trunc](images/date_trunc.png)

example to groupby the year
```sql
 SELECT DATE_FUNC('year', occurred_at) ord_year,  SUM(total_amt_usd) total_spent
 FROM orders
 GROUP BY 1
 ORDER BY 2 DESC;
```

#### DATE_PART
- extracts only that part of the `DATE` e.g. the month
- syntax
    ```sql
    DATE_PART('<date option e.g. day>', column_name)
    ```
- options
    - `dow` i.e. day of week
    - `day`
    - `month`
    - `year`

![Date Part](images/date_part.png)

Example for month from any year

```sql
SELECT DATE_PART('month', occurred_at) ord_month, SUM(total_amt_usd) total_spent
FROM orders
WHERE occurred_at BETWEEN '2014-01-01' AND '2017-01-01'
GROUP BY 1
ORDER BY 2 DESC; 
```

### CASE

- The CASE statement always goes in the SELECT clause.
- CASE must include the following components: WHEN, THEN, and END. ELSE is an optional component to catch cases that didn’t meet any of the other previous CASE conditions.
- You can make any conditional statement using any conditional operator (like WHERE) between WHEN and THEN. This includes stringing together multiple conditional statements using AND and OR.
- You can include multiple WHEN statements, as well as an ELSE statement again, to deal with any unaddressed conditions.

```sql
SELECT account_id, CASE WHEN standard_qty = 0 OR standard_qty IS NULL THEN 0
                        ELSE standard_amt_usd/standard_qty END AS unit_price
FROM orders
LIMIT 10;
```

```sql
SELECT account_id,
       occurred_at,
       total,
       CASE WHEN total > 500 THEN 'Over 500'
            WHEN total > 300 THEN '301 - 500'
            WHEN total > 100 THEN '101 - 300'
            ELSE '100 or under' END AS total_group
FROM orders
```