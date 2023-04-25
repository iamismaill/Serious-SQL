
**Case Danny's Dinner**

#### 1- What is the total amount each customer spent at the restaurant?
-- Customer Id, Price , Total Amount of each customer 
````sql
select S.customer_id, sum(price) as total_amount  from 
dannys_diner.sales S inner join dannys_diner.menu M on 
S.product_id = M.product_id 
group by 1
order by 2 desc;
````

### 2.How many days has each customer visited the restaurant?
````sql
select customer_id,count( DISTINCT order_date) total_days 
from dannys_diner.sales 
group by 1
order by 2 desc;
````

--3. What was the first item from the menu purchased by each customer?
select S.customer_id , M.product_name ,S.order_date
from dannys_diner.sales S inner join dannys_diner.menu M 
on S.product_id = M.product_id
group by 1,2,3
order by 3 asc

--4. What is the most purchased item on the menu and how many times was it purchased by all customers?
select S.product_id ,M.product_name,count(order_date) as number_of_orders
from dannys_diner.sales S inner join dannys_diner.menu M
on M.product_id = S.product_id
group by 1,2
order by 3 desc
limit 1;

--5. Which item was the most popular for each customer?
select S.customer_id,M.product_name,count(S.product_id) as order_count from 
dannys_diner.sales S
inner join dannys_diner.menu M on M.product_id = S.product_id
group by 1, 2
order by order_count desc;
--6. Which item was purchased first by the customer after they became a member?
create TABLE first_purchased as (
 SELECT s.customer_id, m.join_date, s.order_date, s.product_id,
      DENSE_RANK() OVER(PARTITION BY s.customer_id
      ORDER BY s.order_date) AS rank
   FROM dannys_diner.sales  s
   JOIN dannys_diner.members  m
      ON s.customer_id = m.customer_id
   WHERE s.order_date >= m.join_date )
   
   
SELECT s.customer_id, s.order_date, m.product_name 
FROM first_purchased AS s
JOIN dannys_diner.menu  m
   ON s.product_id = m.product_id
WHERE rank = 1;


---7. Which item was purchased just before the customer became a member?



create TABLE after_purchased as (
 SELECT s.customer_id, m.join_date, s.order_date, s.product_id,
      DENSE_RANK() OVER(PARTITION BY s.customer_id
      ORDER BY s.order_date) AS rank
   FROM dannys_diner.sales  s
   JOIN dannys_diner.members  m
      ON s.customer_id = m.customer_id
   WHERE s.order_date < m.join_date )


SELECT s.customer_id, s.order_date, m.product_name 
FROM after_purchased s 
JOIN dannys_diner.menu m
   ON s.product_id = m.product_id
WHERE rank = 1;






