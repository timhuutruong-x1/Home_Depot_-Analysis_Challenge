# Home_Depot_-Analysis_Challenge
Independent 10-query SQL analysis of 3,598 Home Depot products using SQLite and DBeaver
-- Full Analysis Challenege 

=============================================================================================================

--What are the top 5 brands by total number of products?

SELECT COUNT(hd.brand), hd.brand
FROM home_depot hd
WHERE hd.brand IS NOT NULL
GROUP BY hd.brand
ORDER BY COUNT(hd.brand) DESC
LIMIT 5;

-- What is the average price per brand for brands with more than 15 products?

SELECT hd.brand, AVG(hd.price)
FROM home_depot hd 
GROUP BY hd.brand
HAVING COUNT(*)>15;

-- Which products are priced above their brand's average price?

SELECT hd.title, hd.price
FROM home_depot hd 
WHERE hd.price > (
	SELECT AVG(hd2.price)
	FROM home_depot hd2 
	WHERE hd2.brand = hd.brand
	)
GROUP BY hd.title;

-- What percentage of products fall into each price tier (Budget, Mid Range, Premium)?

SELECT 
	CASE 
		WHEN price < 50 THEN 'Budget'
		WHEN price < 200 THEN ' Mid Range'
		ELSE  'Premium'
	END AS tier, COUNT(*) AS num_products, 
	ROUND(COUNT(*) * 100.0 / (SELECT COUNT(*) FROM home_depot), 1) AS pct_of_catalog
FROM home_depot
WHERE price IS NOT NULL
GROUP BY tier;

-- Which brand has the highest price range (max - min)?

SELECT brand, MAX(price)-MIN(price) AS price_range
FROM home_depot hd 
WHERE brand IS NOT NULL
GROUP BY brand 
ORDER BY price_range DESC
LIMIT 5;

-- Find the top 3 most expensive products in each brand that has more than 10 products

WITH big_brands AS ( 
	SELECT brand
	FROM home_depot hd 
	GROUP BY brand HAVING COUNT(*) > 10
	),
ranked AS (
	SELECT hd.brand, hd.title, hd.price,
	RANK() OVER (PARTITION BY hd.brand ORDER BY hd.price DESC) AS price_rank
	FROM home_depot hd 
	WHERE hd.brand IN (SELECT brand FROM big_brands)
	)
SELECT *
FROM ranked WHERE price_rank <=3
ORDER BY brand, price_rank;

-- Which brands have ALL their products priced below $100?

SELECT hd.brand, MAX(hd.price) AS maxprice
FROM home_depot hd 
WHERE hd.brand IS NOT NULL
GROUP BY hd.brand
HAVING maxprice < 100 
ORDER BY maxprice DESC;

-- Find products where the price is in the top 10% of the entire catalog

WITH ranked AS (
	SELECT hd.title, hd.brand, hd.price,
	PERCENT_RANK() OVER (ORDER BY hd.price) AS tp
	FROM home_depot hd 
	WHERE hd.price IS NOT NULL
)
SELECT *
FROM ranked
WHERE tp >= .9
ORDER BY price DESC;

-- What is the cumulative count of products as you go from most expensive to cheapest?

SELECT title, brand, price,   
		COUNT(*) OVER (ORDER BY price DESC
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS cumulative_count
FROM home_depot 
WHERE price IS NOT NULL
ORDER BY price DESC;

-- Create a brand summary showing: brand name, product count, avg price, price 
-- range, and a label of Premium', 'Mid', or 'Budget' based on their average price

SELECT brand, COUNT(title), AVG(price), MAX(price)-MIN(price) AS price_range,
	CASE
		WHEN AVG(price) < 50 THEN 'Budget'
		WHEN AVG(price) < 200 THEN 'Mid Range'
		ELSE 'Premium'
	END AS tier, COUNT(*) AS num_products
FROM home_depot hd
WHERE hd.price IS NOT NULL 
GROUP BY hd.brand
HAVING COUNT(hd.title) > 5
ORDER BY AVG(price) DESC;
	

-- Create a brand summary showing: brand name, product count, avg price, price range, and a label of 'Premium', 'Mid', or 'Budget' based on their average price
