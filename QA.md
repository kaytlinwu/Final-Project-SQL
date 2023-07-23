## What are your risk areas? 
Identify and describe them.
* **Null values** - A majority of the data set (in the analytics table and the all_sessions table) are null values. This is especially noticed in the revenue columns along with several other columns. This poses the greatest risk as large amounts of data is missing from the data set.
* **Nonsensical columns without data** - There are many different columns for revenue:  ```revenue```,  ```item_revenue```,  ```product_revenue```,   ```transaction_revenue```, and  ```total_transaction_revenue```. These columns have many null values and the returned rows are not the same. The ```products_sold``` column is also a NULL column. 
* **Discrepancies in the data** - Analysis of the data set revealed that there were rows that showed ```"orderedQuantity"``` > ```"stockLevel"```. This does not make sense as the ordered quantity should be smaller than the stock level as it implies that a surplus of products were ordered. This data may be incorrect and would need to be further investigated.
* **Shared columns with conflicitng data** - Analysis of shared columns revealed that shared columns between the analytics and all_sessions table were not completely identical. Some of the rows that were returned had conflicting values. This greatly reduces the integrity of the data itself. It is unclear which of the 2 inputted values for the column is correct. 
* **Missing data not shared between columns**- When comparing shared columns, using the except function, it was found a large amount of rows were only found in the all_sessions table and not found in the analytics table. 


Due to the amount of NULL values in the revenue columns that were provided in the dataset, the revenue calculation was performed. The revenue used in our analysis was calculated utilizing average ```"unit_price"``` and ```"total_ordered"``` to address the NULL values in the revenue column. However, the calculated value may not be accurate. In the revenue calculation, 
```
Revenue = unit price * products sold * (1 - discount)
```
Discount is not addressed because there is no discount in the tables that were provided. Therefore, we must make an assumption that all the products had no discount. As there were multiple different ```unit_price``` for the same ```productSKU```, the average ```unit_price``` was taken.
This revenue calculation returns fewer rows due to having to join multiple tables together. As a result, there is a risk that although the calculation was performed, a large portion of the data still remains missing.  

Multiple NULL columns were disregarded in this analysis because they did not pertain to answering questions relevent to this project. 

## QA Process
Describe your QA process and include the SQL queries used to execute it.

1. First, preliminary data exploration was performed to view the tables and find what information was given in each table. 
2. NULL values, misformatted columns, glaring discrepancies, shared columns, and date ranges were noted and when possible were cleaned. These items were subject to further QA checks (as seen below). 
4. Each item was tackled one by one and the queries that were used were documented below. Any significant findings are noted underneath each query. 
> If more areas to QA was found, another bullet point was added to the list. These areas were also verified/checked before doing analysis as seen below.
> As the list is analyzed, if there are any risk areas that need to be addressed it was be described in the question above.  

## Test Cases 
Verify that columns match.

### Test Case 1
Is ```sales_by_sku."total_ordered"``` = ```sales_report."total_ordered"```?

*  **Query -** Get ```total_ordered``` from sales_by_sku table
```
SELECT "total_ordered"
FROM sales_by_sku;
```
> Returns 462 rows
*  **Query -** Get ```total_ordered``` from sales_report table
```
SELECT "total_ordered"
FROM sales_report;
```
> Returns 454 rows.
*  **Query -** Get ```total_ordered``` from sales_report table, joining sales_by_sku on ```productSKU```
where the ```total_ordered``` value is the same 
```
SELECT sr."total_ordered"
FROM sales_report sr
JOIN sales_by_sku s ON sr."productSKU" = s."productSKU"
WHERE sr."total_ordered" = s."total_ordered";
```
> Returns 454 rows

**Test Result - PASS** 
* All the rows in ```"total_ordered"``` from the sales_report table can be found in the ```sales_by_SKU``` table.

- - - -
### Test Case 2
Is ```sales_report.stockLevel``` = ```products.stockLevel```?
*  **Query -** Get ```stockLevel``` from sales_report table
```
SELECT "stockLevel"
FROM sales_report;
```
> Returns 454 rows

*  **Query -** Get ```stockLevel``` from products table
```
SELECT "stockLevel"
FROM products;
```
> Returns 1092 rows

*  **Query -** Get ```stockLevel``` from products table, joining sales_by_sku on ```productSKU``` 
where the ```stockLevel``` values are identical
```
SELECT sr."stockLevel"
FROM products p
JOIN sales_report sr ON p."SKU" = sr."productSKU"
WHERE p."stockLevel" = sr."stockLevel";
```
> Returns 454 rows.

**Test Result - PASS**
* All the rows in ```"stockLevel"``` from the sales_report table can be found in the products table.

- - - -
### Test Case 3
Is ```sales_report.restockingLeadTime``` = ```products.restockingLeadTime```?

*  **Query -** Get ```restockingLeadTime``` from sales_report table
```
SELECT "restockingLeadTime" 
FROM sales_report
```
> Returns 454 rows

*  **Query -** Get ```restockingLeadTime``` from products table
```
SELECT "restockingLeadTime" 
FROM products;
```
> Returns 1092 rows

*  **Query -** Get ```restockingLeadTime``` from products table, 
joining sales_by_sku on ```productSKU ``` where the ```restockingLeadTime``` is the same value
```
SELECT sr."restockingLeadTime" 
FROM products p
JOIN sales_report sr ON p."SKU" = sr."productSKU"
WHERE p."restockingLeadTime" = sr."restockingLeadTime";
```
> Returns 454 rows. 

**Test Result - PASS**
* All the rows in ```"restockingLeadTime"``` from the sales_report table can be found in the products table.

- - - -
### Test Case 4
Is ```sales_report.sentimentScore``` = ```products.sentimentScore```?
*  **Query -** Get ```sentimentScore``` from sales_report table
```
SELECT "sentimentScore"
FROM sales_report;
```
> Returns 454 rows
*  **Query -** Get ```sentimentScore``` from products table
```
SELECT "sentimentScore"
FROM products;
```
> Returns 1092 rows
*  **Query -** Get ```sentimentScore``` from products table after joining
 with the sales_report on ```productSKU``` where the ```sentimentScore``` is the same value
```
SELECT sr."sentimentScore" 
FROM products p
JOIN sales_report sr ON p."SKU" = sr."productSKU"
WHERE p."sentimentScore" = sr."sentimentScore";
```
> Returns 454 rows. 

**Test Result - PASS**
* All the rows in ```"sentimentScore"``` from the sales_report table can be found in the products table.

- - - -
### Test Case 5
Is ```sales_report.sentimentMagnitude``` = ```products.sentimentMagnitude```?

*  **Query -** Get ```sentimentMagnitude``` from sales report table
```
SELECT "sentimentMagnitude"
FROM sales_report;
```
> Returns 454 rows.
*  **Query -** Get ```sentimentMagnitude``` from products table
```
SELECT "sentimentMagnitude"
FROM products;
```
> Returns 1092 rows 
*  **Query -** Get ```sentimentMagnitude``` from products table after joining
 with the sales_report on ```productSKU``` where ```sentimentMagnitude``` is the same value
```
SELECT sr."sentimentMagnitude"
FROM products p
JOIN sales_report sr ON p."SKU" = sr."productSKU"
WHERE p."sentimentMagnitude" = sr."sentimentMagnitude";
```
> Returns Returns 454 rows. 

**Test Result - PASS**
* All the rows in "sentimentMagnitude" from the sales_report table can be found in the products table.

- - - -
### Test Case 6
Is ```all_sessions."fullvisitorid"``` = ```analaytics."fullvisitorid"```? 

*  **Query -** Get ```fullvisitorId``` from the analytics table
```
SELECT "fullvisitorId"
FROM analytics;
```
> Returns 4301122 rows

*  **Query -** Get ```fullvisitorId``` from the analytics table where ```fullvisitorId``` is NULL
```
SELECT "fullvisitorId"
FROM analytics
WHERE "fullvisitorId" is NULL;
```
> Returns 0 rows. No NULL values. 

*  **Query -** Get distinct values of ```fullvisitorId``` from the analytics table
```
SELECT DISTINCT ("fullvisitorId")
FROM analytics;
```
> Returns 120018 rows

Query returned less rows when using distinct to filter for unique ```fullvisitorId``` numbers. 
This implies that there are duplicate rows are present in the analytics table.

*  **Query -** Get ```fullvisitorId``` from the all_sessions table
```
SELECT "fullVisitorId"
FROM all_sessions;
```
> Returns: 15134 rows

*  **Query -** Get ```fullvisitorId``` from the all_sessions table where ```fullvisitorId``` is NULL
```
SELECT "fullVisitorId"
FROM all_sessions
WHERE "fullVisitorId" is NULL;
```
> Returns 0 rows. No NULL values. 

*  **Query -** Get distinct values of ```fullvisitorId``` from the all_sessions table
```
SELECT DISTINCT ("fullVisitorId")
FROM all_sessions;
```
> Returns 14223 rows

Query returned less rows when using distinct to filter for unique ```fullvisitorId``` numbers. 
This implies that there are duplicate rows are present in the all_sessions table.

*  **Query -** Get ```fullVisitorId``` from all_sessions after inner joining with the analytics table
on ```fullVisitorId```
```
SELECT al."fullVisitorId"
FROM all_sessions al
JOIN  analytics a ON a."fullvisitorId" = al."fullVisitorId";
```
> Returns 234508 rows. 

*  **Query -** Get distinct ```fullVisitorId``` from all_sessions after inner joining with the analytics table
on ```fullVisitorId```
```
SELECT DISTINCT al."fullVisitorId"
FROM all_sessions al
JOIN  analytics a ON a."fullvisitorId" = al."fullVisitorId
```
> Returns 3895 rows. 

**Test Result - FAIL**
* ```al."fullvisitorId"``` != ```a."fullvisitorId"```. The number of rows returned is less than the number of distinct ```fullvisitorId``` present in both columns. 

The query returns 234508 rows between the tables (this includes all the duplicate ```"fullvisitorId"```s). Of the 234508 rows, both tables share 3896 rows with unique ```al."fullVisitorId"```. 

From the rows returned above, it can be inferred that not all of the ```"fullvistorId"```s are present in both tables. For the next test cases below, we are joining the all_sessions table and the analytics table using ```"fullVisitorId"```. As a result, we will only be able to return rows pertaining to the distinct 3896 matching ```al."fullvisitorId"``` rows that are present in between the 2 tables.

- - - -
### Test Case 7
Is ```all_sessions."channelGrouping"``` = ```analytics."channelGrouping"```?

*  **Query -** Retrieve rows that only exists in all_sessions table when compared to the analytics table:
```
SELECT DISTINCT "fullVisitorId", "channelGrouping"
FROM all_sessions
EXCEPT 
SELECT DISTINCT "fullvisitorId", "channelGrouping"
FROM analytics;
```
> Returns 10425 rows that only exists in the all_sessions table not in the analytics table.

*  **Query -** Retrieve rows that exists in both all_sessions and analytics table:
```
SELECT DISTINCT "fullVisitorId", "channelGrouping"
FROM all_sessions
INTERSECT
SELECT DISTINCT "fullvisitorId", "channelGrouping"
FROM analytics;
```
**Test Result - PASS**
* Returns 3845 rows out of 3846 possible rows that exists in both all_sessions and analytics table. 
* Fails for the 1 unique ```fullvisitorId``` that shared between the 2 tables that does not have the same channel grouping. 

- - - -
### Test Case 8
Is ```analytics."timeonsite"``` = ```all_sessions."timeonsite"```

*  **Query -** Retrieve the rows that only exists in all_sessions table when compared to the analytics table:
```
SELECT DISTINCT "fullVisitorId", "timeOnSite" 
FROM all_sessions
EXCEPT 
SELECT DISTINCT "fullvisitorId", "timeonsite"
FROM analytics;
```
> Returns 10831 rows that exists in both all_sessions only. 

*  **Query -** Retrieve the rows that exists in both all_sessions and analytics table:
```
SELECT DISTINCT "fullVisitorId", "timeOnSite" 
FROM all_sessions
INTERSECT
SELECT DISTINCT "fullvisitorId", "timeonsite"
FROM analytics;
```
**Test Result - FAIL**
* Returns 3687 rows out of 3846 possible unique ```fullvisitId``` rows that exists in both all_sessions and analytics table.
* There is a discrepancy here as some of the rows do not have matching ```"timeOnsite"```.
* As duplicate rows were not removed, duplicate rows may contain different ```timeOnsite```.  May have conflicting values in shared columns.

- - - -
### Test Case 9
Is ```analytics."pageviews"``` = ```all_sessions."pageviews"```?

*  **Query -** Retrieves the rows that only exists in all_sessions table when compared to the analytics table:
```
SELECT DISTINCT "fullVisitorId", "pageviews" 
FROM all_sessions
EXCEPT 
SELECT DISTINCT "fullvisitorId", "pageviews"
FROM analytics;
```
> Returns 10754 rows that only exists in the all_sessions table. 

*  **Query -** Retrieves the rows that exists in both all_sessions and analytics table:
```
SELECT DISTINCT "fullVisitorId", "pageviews" 
FROM all_sessions
INTERSECT
SELECT DISTINCT "fullvisitorId", "pageviews"
FROM analytics;
```
**Test Result - FAIL**
* Returned 3728 rows out 3846 rows of possible unique ```fullvisitId``` rows that exists in both all_sessions and analytics table.
* The majority of data intersects but there are rows that do not intersect. This could since the duplicates were not removed from the original dataset. 
* The ```fullvisitorId``` had multiple duplicate values containing rows with a different number of pageviews. 
* May have conflicting values in shared columns.

- - - -
### Test Case 10
Is ```analytics."date"``` = ```all_session."date"```?

*  **Query -** Retrieve the rows that only exists in all_sessions table when compared to the analytics table:
```
WITH analytics_date AS (
SELECT "fullvisitorId", TO_DATE(date, 'YYYYMMDD') FROM analytics)

SELECT DISTINCT "fullVisitorId", "date" 
FROM all_sessions
EXCEPT 
SELECT DISTINCT "fullvisitorId", "to_date"
FROM analytics_date;
```
> Returns 10857 rows that only exists in the all_sessions table. 

*  **Query -** Retrieve the rows that exists in both all_sessions and analytics table:
```
WITH analytics_date AS (
SELECT "fullvisitorId", TO_DATE(date, 'YYYYMMDD') FROM analytics)

SELECT DISTINCT "fullVisitorId", "date" 
FROM all_sessions
INTERSECT
SELECT DISTINCT "fullvisitorId", "to_date"
FROM analytics_date;
```
**Test Result - FAIL**
* Returns 3655 rows out of 3846 rows of possible unique ```fullvisitId``` that exists between both all_sessions and analytics table. 
* The majority of data intersects, but there are rows that do not intersect. This could since the duplicates were not removed from the original dataset. 
* The ```fullvisitorId``` had multiple duplicate values containing rows with different dates when the customer placed an order. 
* May have conflicting values in shared columns.

- - - -
### Test Case 11
Is ```analytics."visitId"``` = ```all_sessions."visitId"```

*  **Query -** Retrieve the rows that only exists in all_sessions table when compared to the analytics table:
```
SELECT DISTINCT "fullVisitorId", "visitId" 
FROM all_sessions
EXCEPT
SELECT DISTINCT "fullvisitorId", "visitId"
FROM analytics;
```
> Returns 10900 rows that only exists in the all_sessions table.

*  **Query -** Retrieve the rows that exists in both all_sessions and analytics table:
```
SELECT DISTINCT "fullVisitorId", "visitId" 
FROM all_sessions
INTERSECT
SELECT DISTINCT "fullvisitorId", "visitId"
FROM analytics;
```
**Test Result- FAIL**
* Returns 3661 rows out of 3846 rows of possible unique ```fullvisitId``` that exists between both all_sessions and analytics table.
* The majority of data intersects but there are rows that do not intersect.
* Duplicate rows were not removed which may have resulted in this disrepancy. 
* Rows that are shared may not contain the same information. The same ```fullvisitId``` may have duplicate rows, each row containing a different ```visitId``` representing different visits.  

- - - -
### Ordered Quantity vs. Stock Level
Is ```orderedQuantity``` > ```stocklevel``` ?
*  **Query -** Find rows from products table where ```orderedQuantity``` > ```stocklevel```
```
SELECT "SKU"
FROM products
WHERE "orderedQuantity" > "stockLevel";
```
> Returns 15 items

**FINAL RESULT - FAIL** 
* The ```Total_Ordered``` should not be more than the stock available for the product. This may be an indicator of incorrect data. We would have to verify whether this transaction was possible as it implies that more product was purchased than we have in stock in the database. 

## Multiple columns with similar data in the database

Found that these columns shared very similar names: 
* ```"itemRevenue"```
* ```"transactionRevenue"```
* ```"productRevenue"```
* ```"totalTransactionRevenue``` 
in all_sessions table and ```"revenue"``` in analytics table. 

We can perform a quick query to compare the data.

*  **Query -** Fetch ```itemRevenue```, ```transactionRevenue```, ```productRevenue```, and ```totalTransactionRevenue```
columns from the all_sessions table for visual analysis.
```
SELECT "itemRevenue",
       "transactionRevenue",
       "productRevenue",
       "totalTransactionRevenue"
FROM all_sessions;
```
> All columns contain a lot NULL values. This implies that we are missing large amounts of data from the dataset. 

Quickly querying for the numbers of rows that are not NULL in each column:
*  **Query -** Query all_sessions for number of rows that are not null in ```itemRevenue``` column
```
SELECT "itemRevenue"
FROM all_sessions
WHERE "itemRevenue" is NOT NULL; 
```
> Returns 0 rows. The ```itemRevenue``` column is completely NULL.


*  **Query -** Query all_sessions for number of rows that are not null in ```transactionRevenue``` column
```
SELECT "transactionRevenue"
FROM all_sessions
WHERE "transactionRevenue" is NOT NULL; 
```
> Returns only 4 rows.

*  **Query -** Query all_sessions for number of rows that are not null in ```productRevenue``` column
```
SELECT "productRevenue"
FROM all_sessions
WHERE "productRevenue" is NOT NULL; 
```
> Returns only 4 rows. 

The ```"transactionRevenue"``` and ```"productRevenue"``` column returned the same amount of queries. Are they the same values? 

* **Query** - Find number of rows in the all_sessions table where the values are non-null and ```totalTransactionRevenue``` = ```productRevenue```
```
SELECT "productRevenue", "transactionRevenue"
FROM all_sessions
WHERE "productRevenue" is NOT NULL AND
      "transactionRevenue" is NOT NULL AND
      "transactionRevenue" = "productRevenue";
```
> Returns 0 rows. The returned rows are not the same

* **Query** - Find number of records of all_sessions where ```totalTransactionRevenue``` is not null
```
SELECT "totalTransactionRevenue"
FROM all_sessions
WHERE "totalTransactionRevenue" is NOT NULL; 
```
> Returns 81 rows. 
> This query returns the most amount of rows in the all_sessions table, but a vast majority of the data is NULL. 

Are there any rows shared between the ```"totalTransactionRevenue"``` column and ```"productRevenue"``` column?

* **Query** - Find records of of all_sessions where the non-null values of the ```productRevenue``` = ```productRevenue```
```
SELECT "productRevenue", "totalTransactionRevenue" 
FROM all_sessions
WHERE "productRevenue" is NOT NULL AND
      "totalTransactionRevenue" is NOT NULL AND
      "totalTransactionRevenue" = "productRevenue";
```
> Returns 0 rows. 
> There are no shared rows. 

How about the ```"transactionRevenue"``` and the ```"totalTransactionRevenue"``` column?

* **Query -** Find non null values and see if ```transactionRevenue``` = ```totalTransactionRevenue``` from the all_sessions table
```
SELECT "transactionRevenue", "totalTransactionRevenue" 
FROM all_sessions
WHERE "transactionRevenue"  is NOT NULL AND
      "totalTransactionRevenue" is NOT NULL AND
      "totalTransactionRevenue" = "transactionRevenue";
```
> Returns 4 rows. 
* Yes, all the non-NULL columns in the ```"transactionRevenue"``` column are in the ```"totalTransactionRevenue"``` column.
* Since the ```"totalTransactionRevenue"``` column returned more rows, it represents the data more accurately. (81 rows vs. 4 rows)

The analytics table also has a revenue column. Pull to see revenue column from Analytics table:

**Query -** Get ```revenue``` values from analytics
```
SELECT "revenue"
FROM analytics;
```
> Upon inspection, the query results also contain NULL values. 

**Query -** Get ```revenue``` values from analytics where the ```revenue``` is not NULL
```
SELECT "revenue"
FROM analytics
WHERE "revenue" is not NULL;
```
> Returns 15355 columns. Has the most rows returned that share the keyword "revenue".

How about the ```analytics."revenue"``` vs. ```all_sessions."totalTransactionRevenue"``` column? 

* **Query** - Find values where ```a."revenue"``` = ```al."totalTransactionRevenue"``` after joining the 
analytics table with the all_sessions talbe on ```fullvisitorId```
```
SELECT a."revenue", al."totalTransactionRevenue"
FROM analytics a
JOIN all_sessions al ON a."fullvisitorId" = al."fullVisitorId"
WHERE a."revenue" = al."totalTransactionRevenue";
```
> Returns 12 rows where ```a."revenue"``` = ```al."totalTransactionRevenue"```

* **Query** - Find values where ```a."revenue"``` != ```al."totalTransactionRevenue"``` after joining the 
analytics table with the all_sessions table on ```fullvisitorId```
```
SELECT a."revenue", al."totalTransactionRevenue"
FROM analytics a
JOIN all_sessions al ON a."fullvisitorId" = al."fullVisitorId"
WHERE a."revenue" != al."totalTransactionRevenue";
```
> Returns 54 rows returned where ```a."revenue"``` does not equal to ```al."totalTransactionRevenue"```

```a."revenue"``` and ```al."totalTransactionRevenue"``` may share some values/rows. 


## How to work with the NULL columns

Create a new revenue column to account for NULL values:

```revenue = unit_price * product_sold * 1-discount```

* ```product_sold``` column is NULL. 
* Use ```total_ordered``` column instead from products table. 

Looking at the multiple revenue columns, a majority of the dataset is NULL. After comparing the columns between each other, there is no definitive correlation.There are columns that have shared values but it seems negligible as it only accounts for less than 100 rows.
```item_revenue``` is completely NULL and was disregarded in analysis. ```analytics."revenue"``` seems to be the best indicator of revenue provided in this dataset. However, a large subset of data is still missing. Calculating revenue may fill in some of the missing gaps.
As the difference between these multiple "Revenue" columns are unclear, making correlations between each column may not be a good assumption. 

Other NULL columns that have no data include:
From all_sessions table: 
* ```"productVariant"```
* ```"itemQuantity"```
* ```"transactionId"```
* ```"searchKeyword"```

Please see **data.cleaning.md** for SQL codes.

## Checking date

Check to see that date range is acceptable

* **Query -** Find the minimum and maximum date from all_sessions table
```
SELECT MIN(date) as min_date, MAX(date) as max_date
FROM all_sessions 

max_date = 2017-08-01
min_date = 2016-08-01
```
> Data has been gathered from 2016-08-01 to 2017-08-01 in the all_sessions table.
The date range is acceptable. 