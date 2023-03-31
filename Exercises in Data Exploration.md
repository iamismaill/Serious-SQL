# 📚 Serious SQL

## 🌎 Data Exploration

### 📌 Select and Sort Data

**What is the name of the category with the highest category_id in the dvd_rentals.category table?**

- Use `LIMIT` to restrict result to the highest category_id only.

````sql
select * from
 dvd_rentals.category
 Order by category_id desc 
 limit 1 ;
````

**For the films with the longest length, what is the title of the “R” rated film with the lowest replacement_cost in dvd_rentals.film table?**
 

```sql 
SELECT
  title,
  replacement_cost,
  length,
  rating
FROM dvd_rentals.film
ORDER BY length DESC, replacement_cost
LIMIT 10;
````

**Who was the manager of the store with the highest total_sales in the dvd_rentals.sales_by_store table?**

````sql 
select manager,total_sales from 
dvd_rentals.sales_by_store
order by total_sales desc ;  
````

**What is the postal_code of the city with the 5th highest city_id in the dvd_rentals.address table?**


````sql

select postal_code,city_id from dvd_rentals.address
order by city_id desc limit 5

````

***

## 📌  Record Counts & Distincy Values


**Which actor_id has the most number of unique film_id records in the dvd_rentals.film_actor table?**
  ````sql
  select actor_id,count(DISTINCT(film_id))
  from dvd_rentals.film_actor 
  group by actor_id
  order by 2  desc 
  limit 1;
  ````
**How many distinct fid values are there for the 3rd most common price value in the dvd_rentals.nicer_but_slower_film_list table?**
 
 ````sql
 select price, count(DISTINCT(fid)) as distinct_value  from
 dvd_rentals.nicer_but_slower_film_list
 group by 1
 order by 2 desc;
 
 ````
 
**How many unique country_id values exist in the dvd_rentals.city table?**
````sql

  select count(DISTINCT(country_id)) as number_of_unique_country_id
  from dvd_rentals.city ;
  
  ````
 **What percentage of overall total_sales does the Sports category make up in the dvd_rentals.sales_by_film_category table?**
  
  ````sql
  
select category,
Round(
100 * total_sales / sum(total_sales) OVER(),2 )as percentage
from dvd_rentals.sales_by_film_category;
  
 ````
**What percentage of unique fid values are in the Children category in the dvd_rentals.film_list table?**
````sql
select category, count(DISTINCT(fid))
from dvd_rentals.film_list
group by 1;

````
**Let's make sure this point**

````sql

select count(fid) from 
dvd_rentals.film_list where category ='Children';

````
-- now since we make sure , let's find the percentage 
````sql
SELECT
  category,
  ROUND(
    100 * COUNT(DISTINCT fid) / SUM(COUNT(DISTINCT fid)) OVER (),
    2
  ) AS percentage
from dvd_rentals.film_list
group by 1;
````
## 📌  Identifying Duplicates 
**Which id value has the most number of duplicate records in the health.user_logs table ?**

````sql
WITH temporary_query as (
select id,log_date,measure,measure_value,systolic,diastolic ,
count(*) as frequency 
from health.user_logs
group by 1,2,3,4,5,6 )

select id, sum(frequency) as total_duplicates
from temporary_query 
where frequency >1
group by 1
order by total_duplicates DESC
limit 10;

````


**Which log_date value had the most duplicate records after removing the max duplicate id value from question 1 ?**
````sql

WITH temporary_query as (
select id,log_date,measure,measure_value,systolic,diastolic ,
count(*) as frequency 
from health.user_logs
where id != '054250c692e07a9fa9e62e345231df4b54ff435d'
group by 1,2,3,4,5,6 )

select id, sum(frequency) as total_duplicates
from temporary_query 
where frequency >1
group by 1
order by total_duplicates DESC
limit 10;

````
**Which measure_value had the most occurences in the health.user_logs value when measure = 'weight'?**
-- So basically the question is asking for us the mode of the measure_value when the measure is 'weight'
````sql

select measure_value , count(measure_value) as frequency 
from health.user_logs
where measure = 'weight'
group by measure_value
order by frequency desc
limit 1;

````

**How many single duplicated rows exist when measure = 'blood_pressure' in the health.user_logs? How about the total number of duplicate records in the same table**

````sql 

WITH temporary_table as (
select id, 
log_date,
measure,
measure_value,
systolic,
diastolic,
count(*) frequency
from health.user_logs
group by 1,2,3,4,5,6
)
select count(*) duplicte_rows, sum(frequency) from temporary_table 
where measure = 'blood_pressure' and frequency >1;

````

**What percentage of records measure_value = 0 when measure = 'blood_pressure' in the health.user_logs table? How many records are there also for this same condition**

````sql 

with temporary_table as (

select measure_value,
count(*) as total_records,
SUM(COUNT(*)) OVER () AS overall_total
from health.user_logs
where measure = 'blood_pressure'
group by 1

)
select overall_total, total_records,
round(100 * total_records / overall_total ,2) as percentage
from temporary_table
where measure_value =0;

````



<img width="440" alt="image" src="https://user-images.githubusercontent.com/81607668/128623800-dc689fdd-2de3-4c89-9e55-75be3c4a941c.png">

<img width="676" alt="image" src="https://user-images.githubusercontent.com/81607668/128623822-3a3f3a91-fd67-45ff-985a-79fa0312e59a.png">


***
