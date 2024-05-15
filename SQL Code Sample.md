-- Calculate total sales for 6 months
SELECT ROUND(SUM(unit_price * transaction_qty),2) AS total_sale
FROM transaction

-- Calculate total sales for each month
SELECT EXTRACT(MONTH FROM transaction_date) AS Month,
  ROUND(SUM(unit_price * transaction_qty),2) AS total_sale
FROM transaction
GROUP BY Month

-- Let's see how sale change each month
WITH cte1 AS (
  SELECT 
    Month, 
    total_sale, 
    LAG(total_sale, 1, 0) OVER(ORDER BY Month ASC) AS sale_last_month,
FROM (
  SELECT EXTRACT(MONTH FROM transaction_date) AS Month,
      ROUND(SUM(unit_price * transaction_qty),2) AS total_sale,
  FROM transaction
  GROUP BY Month
  )
)
SELECT  
  *, 
  ROUND((total_sale - sale_last_month), 2) AS compare_last_month,
  CASE WHEN sale_last_month = 0 THEN 0
    ELSE ROUND(((total_sale - sale_last_month)* 100/sale_last_month), 2)  END AS %MoM
FROM cte1
ORDER BY Month ASC

-- Calculate total sales by Day of Week
SELECT 
  *, 
  RANK() OVER(ORDER BY total_sale DESC) AS Ranking
FROM (
  SELECT 
    EXTRACT(DAYOFWEEK FROM transaction_date) AS DoW,
    ROUND(SUM(unit_price * transaction_qty),2) AS total_sale, 
  FROM transaction
  GROUP BY DoW
)
ORDER BY DoW ASC

-- Calculate total sales by store location
SELECT 
  s.store_location AS store_location,
  ROUND(SUM(t.unit_price * t.transaction_qty),2) AS total_sale
FROM transaction AS t
JOIN store AS s
ON t.store_id = s.store_id
GROUP BY store_location
ORDER BY total_sale DESC

-- Report total sales by store location by month
SELECT *
FROM
(
SELECT 
  EXTRACT(MONTH FROM transaction_date) AS Month,
  s.store_location AS store_location,
  t.unit_price * t.transaction_qty AS total_sale
FROM transaction AS t
JOIN store AS s
ON t.store_id = s.store_id
) AS source
PIVOT(
  SUM(total_sale)
  FOR store_location IN("Hell's Kitchen", "Lower Manhattan", "Astoria")
) AS pivot

-- Sales by product categories in descending order
WITH cte2 AS (
  SELECT 
    p.product_category AS product_category,
    ROUND(SUM(t.unit_price * t.transaction_qty),1) AS total_sale
  FROM transaction AS t
  JOIN product AS p
  ON t.product_id = p.product_id
  GROUP BY product_category
)
SELECT 
  product_category,
  total_sale,
  ROUND(total_sale * 100/store_sale, 2) AS perc
FROM (
  SELECT 
    *,
    ROUND(SUM(total_sale) OVER(), 2) AS store_sale
  FROM cte2
  )
ORDER BY total_sale DESC

-- Product category sales by month
SELECT *
FROM
(
SELECT 
  EXTRACT(MONTH FROM t.transaction_date) AS Month,
  p.product_category AS product_category,
  t.unit_price * t.transaction_qty AS total_sale
FROM transaction AS t
JOIN product AS p
ON t.product_id = p.product_id
) AS source
PIVOT(
  SUM(total_sale)
  FOR product_category IN ("Coffee", "Tea", "Bakery", "Drinking Chocolate", "Coffee beans", "Branded", "Loose Tea", "Flavours", "Packaged Chocolate")
) AS pivot

-- Top 10 product types acquire the highest sale
SELECT
  p.product_category AS product_category,
  p.product_type AS product_type,
  ROUND(SUM(t.unit_price * t.transaction_qty),1) AS total_sale
FROM transaction AS t
JOIN product AS p
ON t.product_id = p.product_id
GROUP BY product_category, product_type
ORDER BY total_sale DESC
LIMIT 10

-- Let's dive a bit further by ranking the quantity sold
SELECT
  *,
  RANK() OVER(ORDER BY total_quantity DESC) AS rank_quantity
FROM (
SELECT
  p.product_category AS product_category,
  p.product_type AS product_type,
  ROUND(SUM(t.unit_price * t.transaction_qty),1) AS total_sale,
  SUM(transaction_qty) AS total_quantity
FROM transaction AS t
JOIN product AS p
ON t.product_id = p.product_id
GROUP BY product_category, product_type
)
ORDER BY total_sale DESC
LIMIT 10

-- Open and Close Hour
SELECT 
  MIN(EXTRACT (HOUR FROM transaction_time)) AS open_hour,
  MAX((EXTRACT (HOUR FROM transaction_time) + 1))  AS close_hour
FROM transaction 

-- The busiest hour. 
SELECT 
  EXTRACT (HOUR FROM transaction_time) AS hour,
  SUM(transaction_qty) AS total_quantity,
  ROUND(SUM(unit_price * transaction_qty),2) AS total_sale
FROM transaction
GROUP BY hour
ORDER BY total_quantity DESC

-- Unit price sales contribute
SELECT 
  CASE WHEN unit_price <= 5 THEN '0-5'
       WHEN unit_price <= 10 THEN '5-10'
       WHEN unit_price <= 15 THEN '10-15'
       WHEN unit_price <= 20 THEN '15-20'
       WHEN unit_price <= 25 THEN '20-25'
       WHEN unit_price <= 25 THEN '20-25'
       WHEN unit_price <= 30 THEN '25-30'
       ELSE '>30' END AS unit_price_bin,
  ROUND(SUM(unit_price * transaction_qty),2) AS total_sale
FROM transaction
GROUP BY unit_price_bin
ORDER BY unit_price_bin ASC


SELECT 
  CASE WHEN unit_price <= 1 THEN '0-1'
       WHEN unit_price <= 2 THEN '1-2'
       WHEN unit_price <= 3 THEN '2-3'
       WHEN unit_price <= 4 THEN '3-4'
       WHEN unit_price <= 5 THEN '4-5'
       END AS unit_price_bin,
  ROUND(SUM(unit_price * transaction_qty),2) AS total_sale
FROM transaction
GROUP BY unit_price_bin
ORDER BY unit_price_bin ASC
