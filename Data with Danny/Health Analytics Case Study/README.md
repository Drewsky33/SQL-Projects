
<img width="182" alt="image" src="https://user-images.githubusercontent.com/77873198/175190899-b30562a3-302f-4299-a352-71baebe3766f.png">

## Introduction

This mini case study is part of the Data with Danny Serious SQL course. Danny Ma's designed this case study to be a lesson in debugging SQL code. 

## Problem

The GM of Health Analytics has asked us to answer a few questions for an upcoming board meeting and they want to find out who their most active users are. 

Before answering the business questions that will be asked of us, it's best to have a look at the dataset to get an idea of its shape, column names, and the types of values that are in each column. 

``` sql
SELECT
  * 
FROM health.user_logs;

```

**OUTPUT**:
<img width="1290" alt="image" src="https://user-images.githubusercontent.com/77873198/175191501-d7229774-59fc-4cf8-93e0-fbff19decd83.png">

Based on the output from above we can see that the fields we have are:
- ID: unique identifer for a client.
- log_date: a date.
- measure: type of measure being taken.
- measure_value: the value for the measure that has been taken.
- systolic: a measurement that goes into calculating blood pressure. 
- diastolic: another measurement involved in calculating blood pressure. 

I want to know which unique values are in the measures column. 

``` sql
SELECT
 (DISTINCT measure)
FROM health.user_logs;

```

**OUTPUT**:

<img width="290" alt="Screen Shot 2022-06-22 at 8 09 40 PM" src="https://user-images.githubusercontent.com/77873198/175192609-fc3726d3-44a6-4fbc-93af-2d29a356a69f.png">

It looks like we have 3 main measures which are:
- blood_glucose
- blood_pressure
- weight

Lastly I want to know what the dimensions of the table are. 

``` sql
SELECT
 COUNT(*)
FROM health.user_logs;

```
**OUTPUT**:


<img width="185" alt="image" src="https://user-images.githubusercontent.com/77873198/175192529-c5f789db-a35b-47ff-a47a-4b5d8ffabdaa.png">

Now that I know the shape of the data, the type of values in each column, and the number of rows in the dataset I'm going to try answer the business questions that the GM has asked us to address. 

1. How many unique users exist in the logs_dataset?
The initial query was:


``` sql

SELECT
 COUNT DISTINCT user_id
FROM health.user_logs;
```

When looking at the query and the column names earlier, I noticed that the reason the query wasn't running was because there is no column called `user_id` in the dataset. To fix this we can run the query:

``` sql
SELECT
 COUNT(DISTINCT id)
FROM health.user_logs;

```

**OUTPUT**:


<img width="176" alt="image" src="https://user-images.githubusercontent.com/77873198/175193382-9d64632a-d961-4be1-a9a4-86d79e742a27.png">


** For the next few questions, we needed to create a temporary table. **

``` sql
-- Create a temporary table
DROP TABLE IF EXISTS user_measure_count;
CREATE TEMP TABLE user_measure_count AS (
  SELECT
    id,
    COUNT(*) AS measure_count,
    COUNT(DISTINCT measure) AS unique_measures
  FROM health.user_logs
  GROUP BY 1
);

```

This will create a temporary table for which we can quickly for the next few questions. The `DROP TABLE IF EXISTS` command ensures that we won't create duplicate tables. 


2. How many total measurements do we have per user on average?
**Initial Query**:
``` sql
SELECT
 ROUND(MEAN(measure_count))
FROM health.user_logs;
```
Right away I noticed that the reason the query didn't execute because there is no function called MEAN, but rather the mean is calculated using the AVG() function. 


``` sql
SELECT
  ROUND(AVG(measure_count)) AS measurements_per_user
FROM user_measure_count;
```

**OUTPUT**:

<img width="359" alt="image" src="https://user-images.githubusercontent.com/77873198/175194349-303dab25-3b22-4908-aa88-b29884099598.png">




This first question is helping us identify how many measurements an active user might have and we found that there were around 79 measurements per user on average.


3. What about the median number of measurements per user?
**Initial Query**:

``` sql
SELECT
 PERCENTILE_CONTINUOUS(0.5) WITHIN GROUP (ORDER BY id) AS median_value
FROM user_measure.count;

```
The above query doesn't work because PERCENTILE_CONTINUOUS isn't the name of the function, additionally, it doesn't make sense to ORDER BY the id, but rather the measure_count. The query below will get the correct output. 

``` sql
SELECT
  PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY measure_count) AS median_value
FROM user_measure_count;
```


**OUTPUT**:


<img width="247" alt="image" src="https://user-images.githubusercontent.com/77873198/175195519-92e225c8-901a-4659-a028-1a772e54cb59.png">


As you can see the median number of measurements per user is significantly different than the mean amount of measurments. 

4. How many users have 3 or more measurements?
**Initial query:**

``` sql
SELECT
 COUNT(*)
FROM user_measure_count
HAVING measure >= 3;
```

This query wouldn't execute because of the presence of the HAVING clause. A having clause is used to query on groups. Using a WHERE call will filter the data as we want it. 

``` sql
SELECT
 COUNT(*)
FROM user_measure_count
WHERE measure_counts >= 3;

```
**OUTPUT**:


<img width="162" alt="image" src="https://user-images.githubusercontent.com/77873198/175196088-6bb3a670-703f-4077-a10e-21f27b0ac505.png">

It looks like there are 209 users who have over 3 measurements. 

5. How many users have 1,000 or more measurements
**Initial Query**:

``` sql
SELECT
 SUM(id)
FROM user_measure_count
WHERE measure_count >= 1000;

```
This query doesn't produce the required output because of the SUM call and additionally, what purpose is there for the ID column in this query. We want the count for the users who have over 1,000 measurments so we should use a COUNT(*) call. 

``` sql
SELECT
 COUNT(*)
FROM user_measure_count
WHERE measure_counts >= 1000;

```

**OUTPUT**:

<img width="171" alt="image" src="https://user-images.githubusercontent.com/77873198/175196772-9a3e1626-4b80-408f-b6a7-482f7f8139f1.png">


In the case of this call query it looks like we have 5 users who have over 1000 measure counts. 

6. How many users have logged blood glucose measurements?
**Initial Query**

``` sql
SELECT
 COUNT DISTINCT id
FROM health.user_logs
WHERE measure is 'blood_sugar';
```
This query doesn't run for several reasons. The DISTINCT call need to be inside parentheses and measure needs to be followed by an `=` sign along with the string `blood_glucose`.

``` sql
SELECT
 COUNT(DISTINCT id)
FROM health.user_logs
WHERE measure = 'blood_glucose';

```

**OUTPUT**

<img width="168" alt="image" src="https://user-images.githubusercontent.com/77873198/175197583-f0d54ad2-1257-4c11-97d2-e0e2fea1591b.png">


We have 325 users who have logged blood glucose measurements. 


7. How many users have at least two types of measurements?
**Initial Query**

``` sql
SELECT
 COUNT(*)
FROM user_measure_count
WHERE COUNT(DISTINCT measures) >= 2;

```

The COUNT(DISTINCT measures) call doesn't exist in the table. We provided that call with the lable unique_measures when we created out temporary table earlier. 

``` sql
SELECT
 COUNT(*)
FROM user_measure_count
WHERE unique_measures >= 2;
```
**OUTPUT**:

<img width="175" alt="image" src="https://user-images.githubusercontent.com/77873198/175198046-2f8c0d97-a3d0-4f3a-82c7-b8e1777e70e5.png">


It looks like we have 204 people who have had at least 2 different types of measurement on their health. 

8. Have all 3 measures- blood glucose, weight, and blood pressure?
**Initial Query**

``` sql
SELECT
 COUNT(*)
FROM usr_measure_count
WHERE unique_measures = 3;

```

This query doesn't execute because there is a typo. It should be `user_measure_count` not `usr_measure_count`. 

``` sql
SELECT
  COUNT(*)
FROM user_measure_count
WHERE unique_measures = 3;

```

**OUTPUT**

<img width="175" alt="image" src="https://user-images.githubusercontent.com/77873198/175198502-f020c0b3-afc1-4935-b260-2da4bac8f30c.png">


50 users have recorded the 3 different types of measurements our company offers.

9. What is the median systolic/diastolic blood pressure values?
**Initial Query**:

``` sql
SELECT
  PERCENTILE_CONT(0.5) WITHIN (ORDER BY systolic) AS median_systolic
  PERCENTILE_CONT(0.5) WITHIN (ORDER BY diastolic) AS median_diastolic
FROM health.user_logs
WHERE measure is blood_pressure;

```

This query won't run because the WITHIN clause is missing the latter half of the call which is a GROUP call. It should be WITHIN GROUP. Additionally, blood pressure should be passed in as a string like this `blood_pressure`.

``` sql
SELECT
  PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY systolic) AS median_systolic,
  PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY diastolic) AS median_diastolic
FROM health.user_logs
WHERE measure = 'blood_pressure';

```

**OUTPUT:**

<img width="433" alt="image" src="https://user-images.githubusercontent.com/77873198/175199367-f237ea63-305d-4667-a159-5e62a1751ce5.png">


It looks like the median systolic value was 126 and the median diastolic value was 79. 

## Final Findings
- We found that there were 554 unique users in the dataset. 
- The average user had around 79 measurements. 
- The median amount of measurement per user was 2. 
- 209 users had more than 3 measurements. 
- 5 users had more than 1000 measurements. 
- 325 users have had a blood glucose measurement.
- 204 users have had at least 2 different types of measurment. 
- 50 users had all 3 measruments. 
- The median blood pressure values were 129 systolic and 79 diastolic. 
## Conclusions

This mini case study allowed for me to practically apply some of the SQL functions I've started to learn. Additionally, it gave me some practice with 
- Debugging code
- Sorting/Filtering Data
- Applying Summary Statistics
- Using CTE's and TEMP TABLES
