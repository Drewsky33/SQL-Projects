--Count how many records 
``` sql

SELECT 
  COUNT(*) AS row_count
FROM dvd_rentals.film_list;

```


-- Extracting unique values

``` sql

SELECT DISTINCT
  rating
FROM dvd_rentals.film_list;

```

-- Count unique values
``` sql

SELECT
  COUNT(DISTINCT category) AS unique_category_count
FROM dvd_rentals.film_list;

```

-- Using Group BY: dataset divided into groups based on values in selected columns
-- Simplified table example

``` sql

SELECT
  fid,
  title,
  category,
  rating,
  price
FROM dvd_rentals.film_list
LIMIT 10;

```

-- Group BY query

``` sql
WITH example_table AS (
  SELECT
    fid, 
    title, 
    category,
    rating,
    price
  FROM dvd_rentals.film_list
  LIMIT 10
)

```

``` sql

SELECT
  rating, 
  COUNT(*) AS record_count
FROM example_table
GROUP BY rating
ORDER BY record_count DESC;

```

-- Group by Example 2:

``` sql

SELECT
  rating,
  COUNT(*) AS frequency
FROM dvd_rentals.film_list
GROUP BY rating;

``` 

-- Using ORDER BY with group by

``` sql
SELECT
  rating,
  COUNT(*) AS frequency
FROM dvd_rentals.film_list 
GROUP BY rating 
ORDER BY frequency DESC;

```

-- Adding a percentage column 

``` sql

SELECT
  rating,
  COUNT(*) AS frequency,
  COUNT(*)::NUMERIC / SUM(COUNT(*)) OVER () AS percentage
FROM dvd_rentals.film_list 
GROUP BY rating 
ORDER BY frequency DESC;

```

-- Adding a percentage

``` sql

SELECT
  rating,
  COUNT(*) AS frequency,
  ROUND(
    100 * COUNT(*)::NUMERIC / SUM(COUNT(*)) OVER (),
    2
  ) AS percentage
FROM dvd_rentals.film_list 
GROUP BY rating 
ORDER BY frequency DESC;

```

-- Counts for Multiple Column Combinations 

``` sql

SELECT
  rating, 
  category,
  COUNT(*) AS frequency
FROM dvd_rentals.film_list 
GROUP BY rating, category
ORDER BY frequency DESC 
LIMIT 10;

```

-- Positional numbers instead of column names

``` sql

SELECT
  rating, 
  category, 
  COUNT(*) AS frequency 
FROM dvd_rentals.film_list 
GROUP BY 1, 2
ORDER BY frequency DESC;

```

-- Practice exercises

-- 1. Which actor id has most unique film_id records in the film_actors table?

``` sql

SELECT
  actor_id,
  COUNT(DISTINCT film_id) AS unique_films
FROM dvd_rentals.film_actor
GROUP BY actor_id
ORDER BY unique_films DESC;


```

-- 2. How many distinct fid values are there for the 3rd most common price value in the dvd_rentals.nicer_but_slower_film_list table?

``` sql

SELECT
  price, 
  COUNT(DISTINCT fid) AS unique_fid_count
FROM dvd_rentals.nicer_but_slower_film_list
GROUP BY price
ORDER BY unique_fid_count DESC;

```

-- 3 How many unique countr_id values exist in the dvd_rentals.city table?

``` sql

SELECT
  COUNT(DISTINCT country_id) AS unique_country_id
FROM dvd_rentals.city;

```

-- 4. What percentage of overall total_slaes does the sports category make up in the dvd_rentals.sales_by_film_category table?

``` sql

SELECT
  category,
  total_sales,
  ROUND(
    100 * total_sales::NUMERIC / SUM(total_sales) OVER() ,
    2
    ) AS percentage
FROM dvd_rentals.sales_by_film_category;

```

-- 5. What percentage of unique fid values are in the children category in the dvd_rentals.film_list table?

``` sql

SELECT
  category,
  ROUND(
    100 * COUNT(DISTINCT fid)::NUMERIC/ SUM(COUNT(DISTINCT fid)) OVER (),
    2
  ) AS percentage
FROM dvd_rentals.film_list 
GROUP BY category 
ORDER BY category;

```
