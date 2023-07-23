## What issues will you address by cleaning the data?

1. Fix the ```unit_price``` column
2. Fix formatting for names in columns
3. Analyze duplicate rows 
4. Analyze null values
5. Handle missing data in ```city``` column
6. Analyze revenue columns
7. Change ```analytics.date``` from  integer type to date type
8. Additional clean up

- - - -
### 1. Fix unit price

*  **Query -** Fix the ```unit_price``` and divide ```unit_price``` by 1,000,000 as instructed by the assignment rubric.

```
SELECT round("unit_price"/100000) as "Unit_price_rd"
FROM analytics
```

- - - -
### 2. Fix Name Formatting
*  **Query -** Trim the space in front of the name in the products and sales_report table:
```
SELECT TRIM ("name") as "name"
FROM products;
```
```
SELECT TRIM ("name") as "name"
FROM sales_report;
```

- - - -
### 3. Analysis of Duplicates
*  **Query -** Find if duplicate rows are present in all_sessions table
```
SELECT "fullVisitorId",
       count("fullVisitorId")
FROM all_sessions  
GROUP BY "fullVisitorId"
HAVING count("fullVisitorId") >1;
```
> Returns: 794 columns

*  **Query -** Find if duplicate rows are present in analytics table`
```
SELECT "fullvisitorId",
       count("fullvisitorId")
FROM analytics 
GROUP BY "fullvisitorId"
HAVING count("fullvisitorId") >1;
```
> Returns: 1000 rows

From the above queries, it was found that in the ```all_sessions."fullVisitorId"``` and ```analytics."fullvisitorId"``` column duplicate rows are present. Continuing the analysis, the duplicate rows should be queried to determine if there are any differences in the duplicate rows. 

*  **Query -** Return duplicate rows in all_sessions table
```
---In all_sessions table

SELECT *
FROM all_sessions al
JOIN (SELECT al."fullVisitorId", COUNT(*)
FROM all_sessions al
GROUP BY al."fullVisitorId"
HAVING count(*) > 1 ) b
ON  al."fullVisitorId" = b."fullVisitorId"
ORDER BY al."fullVisitorId";
```
> Returns 1705 rows of duplicate ```fullvisitorId```

From analysis, the rows of duplicate ```fullvisitorID``` from the all_sessions table contained different values in the ```productSKU``` and ```date``` columns. These records cannot be merged to a single record because they are not identical. In some cases of duplicates, one record contained a value for the ```city```, while the other record was null. The missing city cannot be assumed as identical because purchases were made on different days. This purchase has the possibility of being purchased in a different city.

After analysis, it was determined that the duplicate rows should not be deleted. Multiple duplicate entries for each ```fullVisitorId``` exists because the visitor can make multiple purchases and purchase more than 1 product. Each row contains a different ```productSKU``` for different products that were purchased. Additionally, these products can be purchased on different days.

This may be valuable information to the company. If we do remove these duplicates, it should be put into another table so that the data is not lost. We may be able to remove completely duplicated rows in which all columns are exactly the same. This was not preformed due to time constraint.  

```
---In the analytics table:

SELECT a.*
FROM analytics a
JOIN (SELECT a."fullvisitorId", COUNT(*)
FROM analytics a
GROUP BY a."fullvisitorId"
HAVING count(*) > 1 ) b
ON a."fullvisitorId" = b."fullvisitorId"
ORDER BY a."fullvisitorId";
```
> Returns 4299678 rows of duplicate ```fullvisitorId```

```
SELECT DISTINCT ("fullvisitorId")
FROM analytics;
```
> Returns 120018 rows of unique ```fullvisitorId```

Upon analysis, duplicate rows return the same column values except for ```"unit_price"```. The ```"unit_price"``` can be averaged together to amalgamate all the duplicates into a single row. 

The average ```"unit_price"``` should be used because it is unclear which ```"unit_price"``` is correct. This implies that the ```"unit_price"``` may change over time. 

*  **Query -** Combine duplicate rows by taking the average ```unit_price```
```
SELECT DISTINCT a."fullvisitorId", 
       AVG(unit_price) as avg_unit_price
FROM analytics a
JOIN (SELECT a."fullvisitorId", COUNT(*)
FROM analytics a
GROUP BY a."fullvisitorId"
HAVING count(*) > 1 ) b
ON a."fullvisitorId" = b."fullvisitorId"
GROUP BY DISTINCT a."fullvisitorId"
ORDER BY a."fullvisitorId";
```

- - - -
### 4. Dealing with NULL values

There are many NULL values in the all_sessions table in different columns. 
*  **Query -**  find number of rows in ```productVariant``` column that are not null in all_sessions table
```
SELECT "productVariant"
FROM all_sessions
WHERE "productVariant" not like '(not set)';
```
> Returns 40 rows that are not NULL

*  **Query -**  find number of rows in ```itemQuantity``` column that are not null in all_sessions table
```
SELECT "itemQuantity"
FROM all_sessions
WHERE "itemQuantity" is not NULL;
```
> All rows are NULL

*  **Query -**  find number of rows in ```transactionId``` column that are not null in all_sessions table
```
SELECT "transactionId"
FROM all_sessions
WHERE "transactionId" is not NULL;
```
> Only 9 rows are not NULL

*  **Query -**  find number of rows in ```searchKeyword``` column that are not null in all_sessions table
```
SELECT "searchKeyword" 
FROM all_sessions
WHERE "searchKeyword"  is not NULL;
```
> All rows are NULL

Although these columns are mostly NULL, they are not relevant in the analysis done in this project. However it is good to note that these columns are missing data for this dataset should further analysis be performed that require these columns.

- - - -
### 5. Missing Data in City Column
Missing data from the ```city``` column in the all_sessions table was set from 'Not available in demo dataset' or '(not set)' to NULL.

*  **Query -**  Update all_sessions city column to set 'Not available in demo dataset' to NULL
```
UPDATE all_sessions
SET "city" = NULL
WHERE city ='Not available in demo dataset';
```
*  **Query -**  Update all_sessions city column to set '(not set)' to NULL
```
UPDATE all_sessions
SET "city" = NULL
WHERE "city"= '(not set)';
```
- - - -
### 6. Analysis of Revenue Columns 

In the **QA.md** file, the dataset was found to contain multiple revenue columns, of which many of these revenue columns contained NULL values. 
To decrease the amount of missing data, a new revenue column will be created using a calculation.
```
Revenue = unit price * products sold * (1 - discount)
```
Revenue is calculated using ```analytics."unit_price"``` and ```sales_by_sku."total_ordered".``` No discount has been provided in the data set and therefore it is assumed that discount = 0.

* Note that the ```analytics."units_sold"``` is not used due to it containing only NULL values. 
* The tables are joined using ```"fullvisitorId"```.
```
SELECT s."productSKU", 
       a."unit_price", 
       s."total_ordered", (unit_price * s."total_ordered") as calc_revenue
	FROM all_sessions al
JOIN analytics a ON a."fullvisitorId" = al."fullVisitorId"
JOIN sales_by_sku s ON al."productSKU" = s."productSKU"
GROUP BY s."productSKU", a."unit_price"
ORDER BY s."productSKU";
```
Noticed in this query that a few items with the same ```"productSKU"``` have multiple different unit prices. 
As a result, to get results that are more accurate, we will take the ```AVG(unit_price)``` and multiply it to the ```"total_ordered"``` to calculate revenue.
```
ALTER TABLE all_sessions
ADD revcalccte numeric;

WITH revcalccte AS (SELECT s."productSKU", 
    		    round(AVG(a."unit_price")), 
      		    s."total_ordered", 
		        (round(AVG(a.unit_price)) * s."total_ordered") as calc_revenue
FROM all_sessions al
JOIN analytics a ON a."fullvisitorId" = al."fullVisitorId"
JOIN sales_by_sku s ON al."productSKU" = s."productSKU"
GROUP BY s."productSKU"
ORDER BY s."productSKU", sum(a.unit_price))

UPDATE all_sessions
SET "calc_revenue" = revcalccte."calc_revenue"
FROM revcalccte WHERE all_sessions."productSKU" = revcalccte."productSKU";
```
- - - -
### 7. Change analytics.date type from integer to time.

```
ALTER TABLE analytics
ALTER COLUMN time TYPE VARCHAR;

SELECT TO_DATE(date, 'YYYYMMDD') FROM analytics;
```
- - - -
### 8. Additional Clean Up

Other things that can be cleaned are formatting the ```"pagePathLevel1"``` column. We can also clean up the ```time``` column and change the time format to 
to hour-date format. Because these two columns were determined to be not relevant in this project, clean up is noted for future cleaning.

Can clean up ```"productVariant"``` column in the all_sessions table and set all rows containing 'not set' to NULL. This was not performed because this column was not used in analysis. 

With the duplicate data, we can delete completely duplicate data in the all_sessions table if each column is exactly the same.

*  **Query -** Retrieve duplicate rows that are completely identical
```
Select *,
count(*) from all_sessions
GROUP BY (each column) 
HAVING COUNT(*) > 1;
```

For the project, the duplicates were not deleted because many duplicate columns were not exactly identical. 
Additional statistical analysis can also be performed to see if there were any outliers in the dataset, however, this analysis may not be accurate due to the amount of missing data in the dataset. Due to time restraints, this was not performed. 
