
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

**OUTPUT**:
<img width="1290" alt="image" src="https://user-images.githubusercontent.com/77873198/175191501-d7229774-59fc-4cf8-93e0-fbff19decd83.png">
