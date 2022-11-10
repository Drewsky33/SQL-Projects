-- Calculating the average

``` sql

SELECT
  AVG(measure_value)
FROM health.user_logs;

```

-- Counting the records for measures

``` sql

SELECT
  measure,
  COUNT(*) AS counts 
FROM health.user_logs
GROUP BY measure 
ORDER BY counts DESC;

```

-- The mean is calculated using the average function in PostgreSQL

``` sql

SELECT
  AVG(measure_value) 
FROM health.user_logs;

```

-- Calculate the average values across each measure

``` sql

SELECT
  measure, 
  COUNT(*) AS counts, 
  ROUND(AVG(measure_value), 2) AS average
FROM health.user_logs
GROUP BY measure 
ORDER BY counts DESC;

```

-- The median: all number lined up from smallest to largest in uniformity. Number in the middle or the average of the two most middle numbers. 
-- Ther mode: Similar to a group by and count aggregation function 
-- Practice:

``` sql

WITH sample_data (example_values) AS (
  VALUES
  (82), (51), (144), (84), (120), (148), (148), (108), (160), (86)
)

SELECT
  AVG(example_values) AS mean_value, 
  PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY example_values) AS median_value, 
  MODE() WITHIN GROUP (ORDER BY example_values) AS mode_value
FROM sample_data;

```
-- Apply these to our dataset

``` sql

SELECT
  AVG(measure_value) AS mean_value,
  PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY measure_value) AS median_value,
  MODE() WITHIN GROUP (ORDER BY measure_value) AS mode_value
FROM health.user_logs
WHERE measure = 'weight';

```


-- Spread statistics: The min, max, and the range 
-- Calculating them for the measure value column

``` sql

SELECT
  MIN(measure_value) AS minimum_value, 
  MAX(measure_value) AS maximum_value,
  MAX(measure_value) - MIN(measure_value) AS range_value
FROM health.user_logs
WHERE measure = 'weight';

```

-- Improving efficiency

``` sql

EXPLAIN ANALYZE
WITH min_max_values AS (
  SELECT
    MIN(measure_value) AS minimum_value, 
    MAX(measure_value) AS maximum_value
  FROM health.user_logs
  WHERE measure = 'weight'
)

SELECT
  minimum_value,
  maximum_value,
  maximum_value - minimum_value AS range_value
FROM min_max_values;

```

-- Variance and Standard Deviation : Used to describe the spread of the data and the proportion of which parts of data fall within different levels of spread.

``` sql

WITH sample_data (example_values) AS (
  VALUES
  (82), (51), (144), (84), (120), (148), (148), (108), (160), (86)
)

SELECT
  ROUND(VARIANCE(example_values), 2) AS variance_value, 
  ROUND(STDDEV(example_values), 2) AS standard_dev_value, 
  ROUND(AVG(example_values), 2) AS mean_value, 
  PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY example_values) AS median_value, 
  MODE() WITHIN GROUP (ORDER BY example_values) AS mode_value
FROM sample_data;

```

-- Practical example: Calculating all of the summary statistics 

``` sql

SELECT
  'weight' AS measure, 
  ROUND(MIN(measure_value), 2) AS min_value, 
  ROUND(MAX(measure_value), 2) AS max_value, 
  ROUND(AVG(measure_value), 2) AS mean_value,
  ROUND(
    CAST(PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY measure_value) AS NUMERIC),
    2
  ) AS median_value, 
  ROUND(
    MODE() WITHIN GROUP (ORDER BY measure_value),
    2
  ) AS mode_value, 
  ROUND(STDDEV(measure_value), 2) AS standard_deviation, 
  ROUND(VARIANCE(measure_value), 2) AS variance_value
FROM health.user_logs
WHERE measure = 'weight';

```

-- Exercise 1: What is the average, median, and mode values of blood glucose values to 2 decimal places

``` sql

SELECT
  ROUND(AVG(measure_value), 2) AS average_value,
  ROUND(
    CAST(PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY measure_value) AS NUMERIC),
  2
  ) AS median_value, 
  ROUND(
    MODE() WITHIN GROUP (ORDER BY measure_value),
  2
  ) AS mode_value
FROM health.user_logs
WHERE measure = 'blood_glucose';

```


-- Exercise 2: What is themost frequently occuring measure_value for all blood glucose measurements

``` sql

SELECT
  measure_value, 
  COUNT(*) AS counts
FROM health.user_logs
WHERE measure = 'blood_glucose'
GROUP BY 1
ORDER BY 2 DESC
LIMIT 10;

```


-- Exercise 3: Calculate the two coefficients of skewness for blood glucose measures given the following formulas 

``` sql


WITH cte_blood_glucose AS (
 SELECT
  AVG(measure_value) AS mean_value,
  PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY measure_value) AS median_value,
  MODE() WITHIN GROUP (ORDER BY measure_value) AS mode_value,
  STDDEV(measure_value) AS stddev_value
 FROM health.user_logs
 WHERE measure = 'blood_glucose'
)

SELECT
  mean_value, 
  median_value, 
  mode_value,
  stddev_value,
  (mean_value - mode_value) / stddev_value AS pearson_1,
  3 * (mean_value - median_value) / stddev_value AS pearson_2
FROM cte_blood_glucose;

```

-- Cumulative distribution functions 

-- Reverse engineering: 1. Order all of the values from largest to smallest, split them into 100 buckets for percentiles, for each bucket calc the min, max, count records 
-- Count how many records we had when the measure was equal to weight

``` sql

SELECT
  COUNT(*) AS counts
FROM health.user_logs
WHERE measure = 'weight';

```

-- SQL implementation of this process 

``` sql


SELECT
  measure_value, 
  NTILE(100) OVER (
    ORDER BY 
      measure_value
  ) AS percentile
FROM health.user_logs
WHERE measure = 'weight';

```

-- Part 2:

``` sql


WITH percentile_values AS (
  SELECT
    measure_value,
    NTILE(100) OVER (
      ORDER BY measure_value
    ) AS percentile
  FROM health.user_logs
  WHERE measure = 'weight'
)

SELECT
  percentile,
  MIN(measure_value) AS floor_value, 
  MAX(measure_value) AS ceiling_value, 
  COUNT(*) AS percentile_counts
FROM percentile_values
GROUP BY percentile
ORDER BY percentile;

```

-- Using window functions to sort values 
-- Application: Diving into the 100th percentile outliers


``` sql

WITH percentile_values AS (
  SELECT
    measure_value, 
    NTILE(100) OVER (
      ORDER BY 
        measure_value
      ) AS percentile
  FROM health.user_logs
  WHERE measure = 'weight'
)

SELECT
  measure_value, 
  -- These are examples of window functions below 
  ROW_NUMBER() OVER (ORDER BY measure_value DESC) AS row_number, 
  RANK() OVER (ORDER BY measure_value DESC) AS rank_order, 
  DENSE_RANK() OVER (ORDER BY measure_value DESC) AS dense_rank_order
FROM percentile_Values 
WHERE percentile = 100
ORDER BY measure_value DESC;

```

-- Practice Exercise: What is the difference between ROW_NUMBER, RANK, DENSE_RANK window functions 
-- Next: Investigating small outliers

``` sql

WITH percentile_values AS (
  SELECT
    measure_value, 
    NTILE(100) OVER (
      ORDER BY
        measure_value
    ) AS percentile
  FROM health.user_logs
  WHERE measure = 'weight'
)

SELECT
  measure_value, 
  ROW_NUMBER() OVER (ORDER BY measure_value) AS row_number_order,
  RANK() OVER (ORDER BY measure_value) AS rank_order, 
  DENSE_RANK() OVER (ORDER BY measure_value) AS dense_rank_order
FROM percentile_values
WHERE percentile = 1
ORDER BY measure_value;


```


-- Observing large outliers

``` sql

WITH percentile_values AS (
  SELECT
    measure_value, 
    NTILE(100) OVER (
      ORDER BY
        measure_value 
  ) AS percentile
  FROM health.user_logs
  WHERE measure= 'weight'
)

SELECT
  measure_value, 
  ROW_NUMBER() OVER (ORDER BY measure_value DESC) AS row_number_order, 
  RANK() OVER (ORDER BY measure_value DESC) AS rank_number_order, 
  DENSE_RANK() OVER (ORDER BY measure_value DESC) AS dense_rank_order
FROM percentile_values
WHERE percentile = 100
ORDER BY measure_value DESC;

```

-- Checking at small outliers as well

``` sql


WITH percentile_values AS (
  SELECT
    measure_value, 
    NTILE(100) OVER (
      ORDER BY 
        measure_value
    ) AS percentile
  FROM health.user_logs 
  WHERE measure = 'weight'
) 

SELECT
  measure_value, 
  ROW_NUMBER() OVER (ORDER BY measure_value) AS row_number,
  RANK() OVER (ORDER BY measure_value) AS rank_order, 
  DENSE_RANK() OVER (ORDER BY measure_value) AS dense_rank_order
FROM percentile_values
WHERE percentile = 1
ORDER BY measure_value;

```

-- Removing outliers

``` sql


DROP TABLE IF EXISTS clean_weight_logs;
CREATE TEMP TABLE clean_weight_logs AS (
  SELECT *
  FROM health.user_logs
  WHERE measure = 'weight'
    AND measure_value > 0
    AND measure_value < 201
);

```


-- Calculate summary statistics on filtered data

``` sql

SELECT
  ROUND(MIN(measure_value), 2) AS minimum_value, 
  ROUND(MAX(measure_value), 2) AS maximum_value,
  ROUND(AVG(measure_value), 2) AS mean_value, 
  ROUND(
    CAST(PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY measure_value) AS NUMERIC),
    2
  ) AS median_value, 
  ROUND(
    MODE() WITHIN GROUP (ORDER BY measure_value),
    2
  ) AS mode_value, 
  ROUND(STDDEV(measure_value), 2) AS standard_dev_value, 
  ROUND(VARIANCE(measure_value), 2) AS variance_value
FROM clean_weight_logs;

```

-- Use the cumulative distribution functions from before with the treated/cleaned data

``` sql

WITH percentile_values AS (
  SELECT
    measure_value, 
    NTILE(100) OVER (
      ORDER BY 
        measure_value
    ) AS percentile
  FROM clean_weight_logs
)

SELECT
  percentile, 
  MIN(measure_value) AS floor_value,
  MAX(measure_value) AS ceiling_value,
  COUNT(*) AS percentile_counts
FROM percentile_values
GROUP BY percentile
ORDER BY percentile;

``` 


-- Create ranges for bucketing function

``` sql

SELECT
  MIN(measure_value) AS minimum_value,
  MAX(measure_value) AS maximum_value
FROM clean_weight_logs;

```

-- Choosing the min, max, and number of buckets for viz

``` sql

SELECT
  WIDTH_BUCKET(measure_value, 0, 200, 50) AS bucket, 
  AVG(measure_value) AS mean_value, 
  COUNT(*) AS count
FROM clean_weight_logs
GROUP BY bucket
ORDER BY bucket;

```
