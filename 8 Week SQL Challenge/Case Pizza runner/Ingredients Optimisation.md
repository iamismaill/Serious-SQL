--What are the standard ingredients for each pizza?

WITH pizza_names AS (
  SELECT
    pizza_id,
    REGEXP_SPLIT_TO_TABLE(toppings, E'[,\s]+')::INTEGER AS topping_id
  FROM pizza_runner.pizza_recipes
)
SELECT
  pizza_id,
  STRING_AGG(t2.topping_name, ',') AS standard_ingredients
FROM cte_split_pizza_names AS t1
JOIN pizza_runner.pizza_toppings AS t2
  ON t1.topping_id = t2.topping_id
GROUP BY pizza_id
ORDER BY pizza_id;


--What was the most commonly added extra?

WITH extras_info AS (
  SELECT
    REGEXP_SPLIT_TO_TABLE(extras, E'[,\s]+')::INTEGER AS topping_id
  FROM pizza_runner.customer_orders
  WHERE extras IS NOT NULL AND extras != 'null' AND extras != ''
)
SELECT
  topping_name,
  COUNT(*) AS extras_count
FROM cte_extras
JOIN pizza_runner.pizza_toppings
  ON cte_extras.topping_id = pizza_runner.pizza_toppings.topping_id
GROUP BY topping_name
ORDER BY extras_count DESC;

--What was the most common exclusion?

WITH exlusions_info AS (
  SELECT
    REGEXP_SPLIT_TO_TABLE(exclusions, E'[,\s]+')::INTEGER AS topping_id
  FROM pizza_runner.customer_orders
  WHERE exclusions IS NOT NULL AND exclusions != 'null' AND exclusions != ''
)
SELECT
  topping_name,
  COUNT(*) AS exclusions_count
FROM cte_exclusions
JOIN pizza_runner.pizza_toppings
  ON cte_exclusions.topping_id = pizza_runner.pizza_toppings.topping_id
GROUP BY topping_name
ORDER BY exclusions_count DESC;
