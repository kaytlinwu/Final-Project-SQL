# Final-Project-Transforming-and-Analyzing-Data-with-SQL
![](https://github.com/kaytlinwu/Final-Project-SQL/blob/main/schema.png)

## Project/Goals
The goal of my project is to see what the top selling products were in this ecommerce database, which countries had the most products ordered, and to see what categories these products were in.

This will help the company determine what kind of products customers are interested in buying, where the customers are coming from (which countries they are purchasing from), and to see if there are any trends in this data.

This information would be helpful for future growth of the company, including investments in product streams that are high in demand, developing business in areas of high customer activity, and generally improving the logistics of conducting an online retail business.

## Process
* To start off the project, project files were copied from github and the excel sheets were opened to see what kind of data was given. This data was used to determine the datatype in which the columns would be imported into pgadmin.

* Excel sheets were imported into pgAdmin using the pgAdmin interface. Each excel sheet was made into a separate table. 

* Once the data was imported into pgAdmin, data exploration and data cleaning was performed on the ecommerce database. With this, multiple areas of discrepancies and deficiencies in the data were found primarily due to NULL values and missing data. 
     * In particular, there were multiple NULL columns that had no data in them. 
     * These columns were mostly ignored for the analysis in this project but were noted during data cleaning. Columns (i.e revenue), containing numerical values, also returned a vast majority of NULL rows. These problems were noted and addressed in the QA.md file.

* After noting areas of interest and areas that needed to be cleaned, the data was cleaned and documented in the cleaning_data.md file. This included fixing strings, changing data types to the appropriate type (i.e integer to date), converting unit_price to the correct unit, analyzing duplicate rows, and analyzing NULL rows. 

* When the data was sufficiently cleaned, the data was QA'd to determine whether shared columns between tables matched. This was to ensure that rows that contained the same data were the same between tables.
 
 * Further analysis was performed on the multiple revenue columns in the analytics and all_sessions table. 
It was noticed after performing a query excluding NULL values that the revenue columns provided in the database did not produce many rows in comparison to the data set as a whole. To mitigate this, a new column was made that contained a calculated revenue value given the information from the database. The dates in this data set were queried and were acceptable and appropriate. 

* Many deficiencies were noted during the QA and cleaning process.

* After exploring, cleaning, and QAing the dataset, queries for the ```starting_with_questions``` was performed. 
    * For question 1, "```revenue"``` from the analytics table was used. After analysis, it was determined that the calculated revenue column ws more accurate to use because it returned more rows for the country's revenue. Using just the revenue column, only a few countries returned their respective revenue. 

* When the ```starting_with_questions.md ```were complete, queries were made to further look into the pageviews, how many unique customers were using the database, and to see what the top 5 products that were sold in the database were. Questions in order to gain more insights on the products that were sold was created and answered. 

## Results

*  Pageviews were analyzed by country to see which countries accessed the database the most. 
    * The database was visited most by customers in United States and Canada. 

* Number of unique visitors in 2016 was compared to the number of unique visitors in 2017. 
    * The difference between the number of visitors represents the growth/decline of unique visitors to the database. The database had a growth of 1324 unique visitors from 2016-2017.

* Using a query for the average total_ordered for each city/country, the average number of products ordered from each city/country was 21 products

* The highest revenue per county/city was queried. To see the impact the revenue had, we compared the revenue to the total products ordered in the total_ordered column.
    * The country with the highest revenue was the United States.
    * The city with the highest revenue was Mountainview, United States.
    * The cities with the highest revenue and the highest total products ordered were from the United States.
    * The countries and cities with the highest revenue generally had the most products ordered in the database.

* The top selling product amongst country/city was queried. Analyzing the trends in categories, the keyword 'home' existed in 56 out of 69 distinct categories.  
    * The top selling product by country was 22 oz Bottle Infuser. 
    * The top selling product by city was The Hard cover journal 
    * Items returned between the country and city queries were similar. One slight difference was that in the top products sold by country query, there were more customizable products (i.e custom decals) were sold.
    * A query was performed to find the most popular product categories. The most popular category was: Home/Electronics/Accessories/Drinkware/
    * The most popular items bought mainly had home as the primary category along with 1 or more subcategories.
    * Most products sold were in categories that are high in daily use. I.e. bottles (Drinkware), stationary (notebooks and pens), and office supplies.

* The top 5 products containing a stock of > 10 in 2016 was queried. 
    * Top 5 products with stock > 10 were: Ballpoint LED Light Pen, 17oz Stainless Steel Sport Bottle, Leatherette Journal, Spiral Journal with Pen, and Foam Can and Bottle Cooler. 
    * Products were purchases most from United States, Canada, United Kingdom, India, and Germany. These countries match the countries found with the highest revenue and most products ordered in the ecommerce database.

## Challenges 
* The biggest challenge in the project was addressing the NULL values and duplicates presented in the data set. It seems like a lot of the data was missing and could not be extrapolated from other columns. Additionally, identically named columns between tables did not reproduce the same results in shared columns. This may be due to the fact that duplicate columns were not removed as explained in the QA.md file. 
    * Unable to remove duplicate values because the rows were not completely identical. 
    * As the duplicates were not removed, it affected the analysis downstream (i.e QA checks and may have affected the results). The results may be slightly skewed . In order to address this issue, tried to create querieies utilizing the DISTINCT function.
* With the amount of NULL values within the dataset, it is difficult to ascertain the validity of the queries that were performed. 
* Data may not be an accurate reflection of the database as a whole due to the amount of NULL columns present, missing revenue values (majority of the data was missing), nonsensical columns (multiple NULL columns), and inconsistent data between columns.
* Columns were not labelled or described in the database. Unclear what data was presented (ie. the time column. Unsure what the original datatype is).


## Future Goals
If more time was provided the following problems would be addressed.
* Further clean data. Fix the time column convert data type into hour/minute format.
* Remove completely duplicate rows and re-analyze the dataset to see if there are any changes.

