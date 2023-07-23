## Question 1
Which 3 countries had the most amount of page views per year?

**SQL Queries**
```
SELECT DISTINCT country,  
		extract(year from "date") as year, 
		SUM("pageviews") OVER (PARTITION by extract(year from "date"), country)  as Pageviews
FROM all_sessions
ORDER BY year, pageviews desc;
```
**Answer** 
* In 2016: United States, Canada, United Kingdom
* In 2017: United States, Canada, India


## Question 2
Find the difference in the number of unique visitors (```'fullVisitorID'```) between 2016 and 2017?

**SQL Queries**
```
SELECT count(DISTINCT("fullVisitorId")) as visitor_count, 
       extract(year from "date") as year
FROM all_sessions
GROUP BY extract(year from "date");
```
Subtract number of unique visitors from year 2017 by the number of unique visitors from 2016. These values are obtained by the query.

**Asnwer** 
* 1324



## Question 3
What is the top 5 products sold in 2016 with a product stock level > 10?

**SQL Queries**
```
SELECT sr."name",
       SUM(sr."total_ordered")
FROM sales_report sr
JOIN all_sessions al ON al."productSKU" = sr."productSKU"
JOIN products p ON sr."productSKU" = p."SKU"
WHERE extract(year from date) = 2016 AND p."stockLevel" > 10
GROUP BY sr."name",sr."total_ordered"
ORDER BY sr."total_ordered" DESC
LIMIT 5;
```

**Answer** 
* 1 - Ballpoint LED Light Pen
* 2 - 17oz Stainless Steel Sport Bottle
* 3 - Leatherette Journal
* 4 - Spiral Journal with Pen
* 5 - Foam Can and Bottle Cooler

## Question 4
Using the top 5 products sold in question 3, which 5 countries sold the most top 5 products?

**SQL Queries**
```
WITH top_5_products AS(
    SELECT sr."name",
           SUM(sr."total_ordered")
    FROM sales_report sr
    JOIN all_sessions al ON al."productSKU" = sr."productSKU"
    JOIN products p ON sr."productSKU" = p."SKU"
    WHERE extract(year from date) = 2016 AND p."stockLevel" > 10
    GROUP BY sr."name",sr."total_ordered"
    ORDER BY sr."total_ordered" DESC
    LIMIT 5
)
SELECT al.country,
       SUM(sr."total_ordered")
FROM all_sessions al
JOIN sales_report sr on al."productSKU" = sr."productSKU"
WHERE sr."name" IN (SELECT sr."name" FROM top_5_products)
GROUP BY al.country
ORDER BY SUM(sr."total_ordered") DESC;
```
**Answer** 
* 1 - United States
* 2 - Canada
* 3 - United Kingdom
* 4 - India
* 5 - Germany
