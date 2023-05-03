-- 1. How many customers has Foodie-Fi ever had?
  select 
    count (DISTINCT customer_id) as unique_customer
  from foodie_fi.subscriptions;
--- 2. What is the monthly distribution of trial plan start_date values for our dataset - use the start of the month as the group by value

select 
  DATE_PART('month' , start_date) as Month_date,
  TO_CHAR(start_date,'Month') as Month_name,
  Count(*) as trail_subscriptions
from foodie_fi.subscriptions
where plan_id =0
group by 1,2
order by 1;
-- March has the highest number of trial plans, whereas February has the lowest number of trial plans.
--- 3.What plan start_date values occur after the year 2020 for our dataset? Show the breakdown by count of events for each plan_name.

SELECT 
  p.plan_id,
  p.plan_name,
  COUNT(*) AS events
FROM foodie_fi.subscriptions s
JOIN foodie_fi.plans p
  ON s.plan_id = p.plan_id
WHERE s.start_date >= '2021-01-01'
GROUP BY p.plan_id, p.plan_name
ORDER BY p.plan_id;

--4.What is the customer count and percentage of customers who have churned rounded to 1 decimal place?
SELECT count(*) AS churn_count,
round(100 * count(*)::NUMERIC / (
    SELECT count(DISTINCT customer_id) 
    FROM foodie_fi.subscriptions),1) AS churn_percentage
FROM foodie_fi.subscriptions s
JOIN foodie_fi.plans p
  ON s.plan_id = p.plan_id
WHERE s.plan_id = 4;

--5. How many customers have churned straight after their initial free trial - what percentage is this rounded to the nearest whole number?
-- To find ranking of the plans by customers and plans
WITH ranking_customers AS (
SELECT  s.customer_id, s.plan_id, p.plan_name,ROW_NUMBER() OVER (PARTITION BY s.customer_id  ORDER BY s.plan_id) AS plan_rank 
FROM foodie_fi.subscriptions s
JOIN foodie_fi.plans p
  ON s.plan_id = p.plan_id)
SELECT 
  COUNT(*) AS churn_count, ROUND(100 * COUNT(*) / (SELECT COUNT(DISTINCT customer_id)  FROM foodie_fi.subscriptions),0) AS churn_percentage
FROM ranking_customers
WHERE plan_id = 4  
AND plan_rank = 2  

-- 6.What is the number and percentage of customer plans after their initial free trial?
WITH next_plan_subscription AS (
select  customer_id,  plan_id, LEAD(plan_id, 1) OVER( PARTITION BY customer_id ORDER BY plan_id) as next_plan
FROM foodie_fi.subscriptions)
SELECT next_plan, COUNT(*) AS conversions,ROUND(100 * COUNT(*)::NUMERIC / (SELECT COUNT(DISTINCT customer_id) FROM foodie_fi.subscriptions),1) AS conversion_percentage
FROM next_plan_subscription
WHERE next_plan IS NOT NULL 
AND plan_id = 0
GROUP BY next_plan
ORDER BY next_plan;
