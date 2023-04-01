**Health Analytics Mini Case Study** 


In this mini case study, it present an untidy dataset from health analytics and an urgent request from the general manager of the Analytics team. 
They require our assistance with their analysis of the "health.user_logs" dataset.

The dataset contains various health-related metrics- My task is to inspect the data and provide insights that answer the following business questions:

1. How many unique users exist in the logs dataset?
2. How many total measurements do we have per user on average?
3. What about the median number of measurements per user?
4. How many users have 3 or more measurements?
5. How many users have 1,000 or more measurements?
Looking at the logs data - what is the number and percentage of the active user base who:
6. Have logged blood glucose measurements?
7. Have at least 2 types of measurements?
8. Have all 3 measures - blood glucose, weight and blood pressure?
For users that have blood pressure measurements:
9	What is the median systolic/diastolic blood pressure values??

By cleaning and analyzing the data, I aim to provide meaningful insights that can inform decision-making and improve health outcomes.


### 1. How many unique users exist in the logs dataset?
````sql

select count(DISTINCT(id)) as unique_users from 
health.user_logs ; 

````
<img width="299" alt="Screen Shot 2023-04-01 at 10 10 37 PM" src="https://user-images.githubusercontent.com/51711008/229291179-28e8aede-01f5-40e1-9dca-3851a2eef817.png">

**In order to Answer From Q2 to Q8 - let's create temporary table**

````sql
Create TEMP TABLE user_measure_info  as (
select 
id, 
count(*) as measure_count,
count(DISTINCT (measure)) as unique_measure_count 
from health.user_logs
group by id
);
````
**So now , let's inspect the data according to the questions**
**2. How many total measurements do we have per user on average?**
````sql
select 
avg(measure_count) as avg_measurement 
from user_measure_info ;
````
**Answer**
<img width="366" alt="Screen Shot 2023-04-01 at 10 15 55 PM" src="https://user-images.githubusercontent.com/51711008/229291323-b17f01f5-bc71-4e02-b0c8-bf311613d363.png">

**What about the median number of measurements per user ?**
````sql
select 
PERCENTILE_CONT(0.5) WITHIN group (order by measure_count ) as median_value
from user_measure_info ; 
````
**Answer**
<img width="366" alt="Screen Shot 2023-04-01 at 10 15 55 PM" src="https://user-images.githubusercontent.com/51711008/229291391-16785b98-d588-4363-8389-6325a3abda4e.png">

**How many users have 3 or more measurements?**
````sql
select count(id)  from 
user_measure_info where measure_count >= 3 ; 

````
<img width="213" alt="Screen Shot 2023-04-01 at 10 19 12 PM" src="https://user-images.githubusercontent.com/51711008/229291471-60682b62-1ccd-4c43-911a-32b3d0d1b9a8.png">





