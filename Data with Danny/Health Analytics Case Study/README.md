
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




