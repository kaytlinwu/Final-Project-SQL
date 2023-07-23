Answer the following questions and provide the SQL queries used to find the answer.

    
# Question 1
Which cities and countries have the highest level of transaction revenues on the site?


**SQL Queries**

**Query 1-** Highest level of transaction revenues for country:
```
SELECT DISTINCT country AS "Country", 
       SUM(a."revenue") AS "TransactionRev"
FROM all_sessions al
JOIN analytics a ON a."fullvisitorId" = al."fullVisitorId"
GROUP BY country
HAVING SUM("revenue") > 0
ORDER BY SUM("revenue") DESC;
```

Using the ```"revenue"``` column from the analytics table, it only returns 5 countries without NULL values as seen in the table above.  This is not a good indicator of which country has the highest level of transaction revenues on the site. 

Instead, the ```"calc_revenue"``` column in the all_sessions table will be used:

**Revised Query 1-** Highest level of transaction revenues for country:
```
SELECT DISTINCT country As "Country",
                SUM("calc_revenue") as TransactionRev
FROM all_sessions 
GROUP BY country
HAVING SUM("calc_revenue") is NOT NULL
ORDER BY SUM("calc_revenue") DESC;
```
Using ```"calc_revenue"``` column, the query returns 127 countries. This is a great improvement from the previous query. Therefore, it is used in the next analysis to determine the highest revenues on the site.

**Query 2-** Highest level of transaction revenues for city:

```
SELECT DISTINCT city As "City",
                SUM("calc_revenue") as "TransactionRev"
FROM all_sessions 
GROUP BY city
ORDER BY SUM("calc_revenue") DESC;
```

**Answer**
* The country with the highest level of transaction revenues is United States
* The city with the highest level of transaction revenues is Mountain View
> Note that duplicate values were removed. Data for the city "not available in demo dataset" were replaced with NULL values and were not considered in this analysis.

# Question 2
What is the average number of products ordered from visitors in each city and country?

**SQL Queries**

**Query 1-** Average products ordered in total for all countries/cities:
```
SELECT round(AVG(s."total_ordered")
FROM all_sessions al
JOIN sales_by_sku s ON al."productSKU" = s."productSKU"
```
* Note that ```total_ordered``` was put as sales in query
* **Result:** Average number of products ordered = 21 

**Query 2-** Average products ordered by country (descending order):
```
SELECT al.country, 
       round(AVG(s."total_ordered")) AS "Country_Avg_Sales"
FROM all_sessions al
JOIN sales_by_sku s ON al."productSKU" = s."productSKU"
GROUP BY country
ORDER BY AVG(s.total_ordered) DESC;
```

**Query 3-** Average products ordered by city (descending order):
```
SELECT al.city, 
       round(AVG(s."total_ordered")) AS "City_Avg_Sales"
FROM all_sessions al
JOIN sales_by_sku s ON al."productSKU" = s."productSKU"
GROUP BY city
ORDER BY AVG(s.total_ordered) DESC;
```

**Answer**
* The average number of products ordered by visitors in each country/city is 21 orders.
* Note that the query was modified to search for a further breakdown based on a visitor's country/city for further analysis. Please refer to query 2 and 3. 


# Question 3
Is there any pattern in the types (product categories) of products ordered from visitors in each city and country?

**SQL Queries**

**Query 1 -** Average number of products ordered by country:
```
SELECT "v2ProductCategory", 
       country, 
       round(AVG(s."total_ordered")) AS "Country_Avg_Sales"
FROM all_sessions al
JOIN sales_by_sku s ON al."productSKU" = s."productSKU"
GROUP BY "v2ProductCategory", country
ORDER BY AVG(s.total_ordered) DESC, "v2ProductCategory"; 
```

**Query 2 -** Average number of products ordered by city:
```
SELECT "v2ProductCategory",
       city,
       round(AVG(s."total_ordered")) AS "City_Avg_Sales"
FROM all_sessions al
JOIN sales_by_sku s ON al."productSKU" = s."productSKU"
GROUP BY "v2ProductCategory", city
ORDER BY AVG(s.total_ordered) DESC, "v2ProductCategory";
```

**Query 3 -** Finding patterns in categories:
* Average of the total number of prodcuts ordered per category
```
SELECT "v2ProductCategory",
       round(AVG(s."total_ordered"))
FROM all_sessions al
JOIN sales_by_sku s ON al."productSKU" = s."productSKU"
GROUP BY "v2ProductCategory"
ORDER BY AVG(s.total_ordered) DESC;
```

**Query 4 -** Finding number of unique categories:
```
SELECT DISTINCT "v2ProductCategory"
FROM all_sessions al
JOIN sales_by_sku s ON al."productSKU" = s."productSKU";
```
> Query Result: Returns 69 rows
 
**Query 5 -** Finding number of unique categories containing the keyword home: 
```
SELECT DISTINCT "v2ProductCategory"
FROM all_sessions al
JOIN sales_by_sku s ON al."productSKU" = s."productSKU"
WHERE "v2ProductCategory" LIKE '%Home%';
```
> Query Result: Returns 56 rows

**Answer**

The top categories obtained from the all_sessions table from query 3 are:
 1. Home/Electronics/Accessories/Drinkware/
 2. Nest-USA
 3. Home/Lifestyle/Fun/
 4. Home/Office/Notebooks & Journals
 5. Home/Drinkware/Water Bottles and Tumblers

Looking at the query results above, the most popular products bought were items that had more than 1 subcategory, and most products had a main category with the keyword "home" in them. Additionally the top selling product categories were usuable daily items found in the  workplace or at home. 

Using query 1 and 2, the top categories by country and city the following was found:
* **Country** - The highest average sales were found for products with the main category as "Home". Most categories contained "Home" as the main category with at least 1 or 2 additional subcategories. Home was a keyword found in 56 out of 69 distinct categories.  
* **City** - Same findings as country. 

The queries returned very similar data. It seemed to match the categories presented in query 3. 

# Question 4
What is the top-selling product from each city/country? Can we find any pattern worthy of noting in the products sold?

**SQL Queries**

**Query -** Top-selling product by country:
```
WITH Country_top_Ranks AS (SELECT sr."name" as item_name, 
		   al.country as country,  
		   RANK () OVER (PARTITION BY al.country ORDER BY SUM(sr."total_ordered") DESC) as Rnk
	FROM sales_report sr
	JOIN all_sessions al ON al."productSKU" = sr."productSKU"
	JOIN products p ON sr."productSKU" = p."SKU"
	GROUP BY al.country, sr.name
	ORDER BY sr.name
	 )
SELECT item_name, country, rnk
FROM Country_top_Ranks
WHERE Rnk = 1;
```

**Query -** Count of how many times the top-selling product is repeated per country:
```
WITH Rank1product_country AS (WITH Country_top_Ranks AS (SELECT sr."name" as item_name, 
		   al.country as country,  
		   RANK () OVER (PARTITION BY al.country ORDER BY SUM(sr."total_ordered") DESC) as Rnk
	FROM sales_report sr
	JOIN all_sessions al ON al."productSKU" = sr."productSKU"
	JOIN products p ON sr."productSKU" = p."SKU"
	GROUP BY al.country, sr.name
	ORDER BY sr.name
			)
SELECT item_name, country, rnk
FROM Country_top_Ranks
WHERE Rnk = 1)

SELECT item_name, count(item_name) as country_count
FROM Rank1product_country
GROUP BY item_name
ORDER BY count(item_name) DESC;
```

**Query -** Top-selling product by city:
```
WITH City_top_Ranks AS (SELECT sr."name" as item_name, 
		   al.city as city,  
		   SUM(sr."total_ordered") as total,
		   RANK () OVER (PARTITION BY al.city ORDER BY SUM(sr."total_ordered") DESC) as Rnk
	FROM sales_report sr
	JOIN all_sessions al ON al."productSKU" = sr."productSKU"
	JOIN products p ON sr."productSKU" = p."SKU"
	GROUP BY al.city, sr.name)
SELECT item_name, city, total, rnk
FROM City_top_Ranks
WHERE Rnk = 1;
```

**Query -** Count of how many times the top-selling product is repeated per city:
```
WITH Rank1product_city AS (WITH City_top_Ranks AS (SELECT sr."name" as item_name, 
		   al.city as city,  
		   SUM(sr."total_ordered") as total,
		   RANK () OVER (PARTITION BY al.city ORDER BY SUM(sr."total_ordered") DESC) as Rnk
	FROM sales_report sr
	JOIN all_sessions al ON al."productSKU" = sr."productSKU"
	JOIN products p ON sr."productSKU" = p."SKU"
	GROUP BY al.city, sr.name
	ORDER BY sr.name
			)
SELECT item_name, city, total, rnk
FROM City_top_Ranks
WHERE Rnk = 1)

SELECT item_name, count(item_name) as city_count
FROM Rank1product_city
GROUP BY item_name
ORDER BY count(item_name) DESC;
```

**Answer**

* **By number of countries:** The most sold items were the following:
  * 1 - 22 oz Bottle Infuser (in 17 countries)
  * 2 - Hard Cover Journal (in 16 countries)
  * 3 - BallpointLED Light Pen (in 10 countries).


* **By number of cities:** The most sold items were the following:
  * 1 - The Hard cover journal (in 13 cities)
  * 2 - 17oz Stainless Steel Sport Bottle (in 11 cities)
  * 3 - Leatherette Journal
  * 4 - Blackout Cap (in 9 cities)

Looking at the returned results from the queries, the top 10 products include mostly stationery supplies (journals, pens) and athletic wear (Men's short sleeve tee, sport bottle, cap). 

Between items returned in the countries and cities query, the countries query included items that were more customizable ie. custom decals, keyboard DOT stickers, Android sports bottle. Otherwise, the categories and items returned were very similar. 

## Question 5
Can we summarize the impact of revenue generated from each city/country?

**SQL Queries**

**Query 1 -** Sum of total number of products ordered by country
```
SELECT al.country,
       SUM(s."total_ordered")
FROM all_sessions al
JOIN sales_by_sku s ON al."productSKU" = s."productSKU"
GROUP BY al.country
ORDER BY SUM(s."total_ordered") DESC;
```

**Query 2 -** Sum of total number of products ordered by city
```
SELECT al.city,
       SUM(s."total_ordered")
FROM all_sessions al
JOIN sales_by_sku s ON al."productSKU" = s."productSKU"
GROUP BY al.city
ORDER BY SUM(s."total_ordered") DESC;
```

**Answer**

From query 1, in descending order, the countries with the most products ordered by country were the following: 
* 1 - United States
* 2 - Canada
* 3 - United Kingdom
* 4 - India
* 5 - Germany

Using revised query 1 from **Question 1**, the countries with the highest revenues were found to be United States, United Kingdom, Canada, India, and Germany. The higher revenue countries corresponded to  higher number of products that the customers in that country ordered. These queries produce a similar table with slight differences in the ranking on the list. 

From query 2, in descending order, the cities with the most products ordered by city were the following: 
* 1- Mountain View
* 2- New York
* 3 - San Francisco
* 4- Sunnyvale
* 5 - Los Angeles.
 
Using the query 2 from **Question 1**, the cities with the highest revenues were Mountain View, San Francisco, New York, Sunnyvale and Palo Alto. We can see here that the higher the revenue, the higher the number of products that customers in that city ordered. 

Another inference from this query is that the cities with the highest revenue and amount of sales were from the United States. This matched our previous finding that United States had the highest revenue and highest number of products ordered on this database. These queries produce a similar table with slight differences in the ranking on the list. 

Utilizing the information from this query, and query number 1, we can see that United States accounted for the most sales on this ecommerce database for data collected in 2016-2017. 
