--1. How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)

SELECT 
  DATE_PART('week', registration_date) AS registration_week,
  COUNT(runner_id) AS runner_signup
FROM pizza_runner.runners
GROUP BY 1;
--2. What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?
WITH cte_pickup_minutes AS (
  SELECT DISTINCT
    t1.order_id,
    DATE_PART('minutes', AGE(t1.pickup_time::TIMESTAMP , t2.order_time))::INTEGER AS pickup_minutes
  FROM pizza_runner.runner_orders AS t1
  INNER JOIN pizza_runner.customer_orders AS t2
    ON t1.order_id = t2.order_id
  WHERE t1.pickup_time  <> 'null'
    --AND AGE(t2.order_time, t1.pickup_time::TIMESTAMP) > INTERVAL '0 minutes'
)
SELECT
  ROUND(AVG(pickup_minutes), 3) AS avg_pickup_minutes
FROM cte_pickup_minutes;
--Is there any relationship between the number of pizzas and how long the order takes to prepare?

WITH prep_time_cte AS
(
  SELECT 
    c.order_id, 
    COUNT(c.order_id) AS pizza_order, 
    c.order_time, 
    r.pickup_time, 
    DATE_PART('min', AGE(r.pickup_time::TIMESTAMP, c.order_time::TIMESTAMP))::INTEGER AS pickup_minutes

  FROM pizza_runner.customer_orders AS c
  JOIN pizza_runner.runner_orders AS r
    ON c.order_id = r.order_id
  WHERE r.distance <>'null'
  GROUP BY c.order_id, c.order_time, r.pickup_time
)

SELECT 
  pizza_order, 
  AVG(pickup_minutes) AS avg_prep_time_minutes
FROM prep_time_cte
WHERE pickup_minutes > 1
GROUP BY pizza_order;


--What was the difference between the longest and shortest delivery times for all orders?

WITH duration AS (
  SELECT
  CAST(SUBSTRING(duration from '[0-9]+') as Integer) as duration 
  FROM pizza_runner.runner_orders
  WHERE pickup_time <> 'null'
)
SELECT
  MAX(DURATION) - MIN(duration) AS max_difference
FROM duration ;
--What was the average speed for each runner for each delivery and do you notice any trend for these values?
WITH average_speed as (
SELECT 
  r.runner_id, 
  c.customer_id, 
  c.order_id, 
  COUNT(c.order_id) AS pizza_count, 
  CAST(SUBSTRING(distance from '[0-9]+') as INTEGER) as distance,
  CAST(SUBSTRING(duration from '[0-9]+')as INTEGER) as duration 
FROM pizza_runner.runner_orders AS r
JOIN pizza_runner.customer_orders AS c
  ON r.order_id = c.order_id
WHERE distance <> 'null'
GROUP BY r.runner_id, c.customer_id, c.order_id, r.distance,r.duration
ORDER BY c.order_id
)
select 
  runner_id, 
  customer_id, 
  order_id, 
  pizza_count  
  distance, (duration / 60) AS duration_hr , 
  ROUND((distance/duration * 60), 2) AS avg_speed 
  from average_speed

--What is the successful delivery percentage for each runner?
SELECT 
  runner_id, 
  ROUND(100 * SUM(
    CASE WHEN CAST(SUBSTRING(distance from '[0-9]+')as INTEGER)  = 0 THEN 0
    ELSE 1 END) / COUNT(*), 0) AS success_perc
FROM pizza_runner.runner_orders
GROUP BY runner_id;
