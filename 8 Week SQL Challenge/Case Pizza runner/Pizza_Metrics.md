# 🍧 Case Study Danny's Diner 

## Pizza Metrics 

Uncovering Business Insights: Key Questions

### How many pizzas were ordered?

````sql

select count(order_id) as number_of_orders 
  from pizza_runner.customer_orders ;

````
### How many unique customer orders were made?
````sql

select count(DISTINCT order_id) as unique_customers 
  from pizza_runner.customer_orders;
  
````

### How many successful orders were delivered by each runner?

````sql

select runner_id,count(DISTINCT order_id ) as successfull_orders   
  from pizza_runner.runner_orders
where distance <> 'null' 
group by runner_id;

````

### How many of each type of pizza was delivered?

````sql
select  p.pizza_name,count(c.pizza_id) as successfull
  from pizza_runner.runner_orders r 
inner join pizza_runner.customer_orders c 
  on r.order_id = c.order_id 
inner join pizza_runner.pizza_names p 
  on c.pizza_id = p.pizza_id
where r.distance <> 'null'
group by 1;

````
### How many Vegetarian and Meatlovers were ordered by each customer?  1-- Meatlovers , 2. Vegetarian 

````sql
select c.customer_id, p.pizza_name, count(c.pizza_id) as number_of_orders
  from pizza_runner.customer_orders c
inner join pizza_runner.pizza_names p 
  on c.pizza_id = p.pizza_id
group by 1,2
order by 1 ;

````
 ### What was the maximum number of pizzas delivered in a single order?
 --order_ID , pizza_id ,delivered 
 
 ````sql
Create table delivered_pizzas as (
select c.order_id , count(c.pizza_id) as pizza_id 
  from pizza_runner.customer_orders c 
inner join pizza_runner.runner_orders r
  on c.order_id = r.order_id
  where r.distance <> 'null'
group by 1
  );
  
select order_id , max(pizza_id) as Max_pizza_ordered from 
delivered_pizzas
group by order_id
order by order_id ;

 ````

### For each customer, how many delivered pizzas had at least 1 change and how many had no changes?

````sql
SELECT 
  c.customer_id,
     
  SUM(
    CASE WHEN c.exclusions IN('null','')   THEN 1
    ELSE 0
    END) AS at_least_1_change,
  SUM(
    CASE WHEN c.exclusions IN('null','')  THEN 1 
    ELSE 0
    END) AS no_change
FROM pizza_runner.customer_orders  c
JOIN pizza_runner.runner_orders  r
  ON c.order_id = r.order_id
WHERE r.distance <> 'null' 
GROUP BY c.customer_id
ORDER BY c.customer_id;

 ````
 
### How many pizzas were delivered that had both exclusions and extras?

 ````sql
 
WITH delivered_pizzas_both as (
select * 
  from pizza_runner.customer_orders 
where exclusions not in('','null') and extras not in ('','null')
)

select  count(*) as pizza_countwithexlussionsandextras
  from delivered_pizzas_both d
inner join pizza_runner.runner_orders r 
  on d.order_id = r.order_id
where r.distance <> 'null';

````
### What was the total volume of pizzas ordered for each hour of the day?

 ````sql
select 
  DATE_PART('hour', pickup_time::TIMESTAMP) AS hour_of_day,
  count(*) as pizza_count 
from pizza_runner.runner_orders
where pickup_time <> 'null'
group by 1 
order by 1

````
### What was the volume of orders for each day of the week? 

 ````sql
SELECT
  TO_CHAR(order_time, 'Day') AS day_of_week,
  SUM(order_id) AS pizza_count
FROM pizza_runner.customer_orders
GROUP BY day_of_week, DATE_PART('dow', order_time)
ORDER BY DATE_PART('dow', order_time);

 ````
