
**Case Danny's Dinner**

 

As a data analyst, I am presented with a case study to assist Danny in analyzing data for his restaurant that specializes in sushi, curry, and ramen. Danny has collected information over a few months of operation, but he requires my expertise in utilizing this data to drive growth for his business.

Problem Statement
Danny want that i use his data to answer few simple qustions about his customers 
* Vissiting patterns
* how much money they've spent 
* which menu items are their favourite 
 
Danny has shared with me  3 key datasets for this case study:
* sales
* menu
* members

Uncovering Business Insights: Key Questions for Danny's Restaurant

#### What is the total amount each customer spent at the restaurant?
-- Customer Id, Price , Total Amount of each customer 
````sql
select S.customer_id, sum(price) as total_amount  from 
dannys_diner.sales S inner join dannys_diner.menu M on 
S.product_id = M.product_id 
group by 1
order by 2 desc;
````

### How many days has each customer visited the restaurant?
````sql
select customer_id,count( DISTINCT order_date) total_days 
from dannys_diner.sales 
group by 1
order by 2 desc;
````

### What was the first item from the menu purchased by each customer?
````sql
select S.customer_id , M.product_name ,S.order_date
from dannys_diner.sales S inner join dannys_diner.menu M 
on S.product_id = M.product_id
group by 1,2,3
order by 3 asc;
````

### What is the most purchased item on the menu and how many times was it purchased by all customers?
````sql
select S.product_id ,M.product_name,count(order_date) as number_of_orders
from dannys_diner.sales S inner join dannys_diner.menu M
on M.product_id = S.product_id
group by 1,2
order by 3 desc
limit 1;
````

### Which item was the most popular for each customer?

````sql
select S.customer_id,M.product_name,count(S.product_id) as order_count from 
dannys_diner.sales S
inner join dannys_diner.menu M on M.product_id = S.product_id
group by 1, 2
order by order_count desc;

````
### Which item was purchased first by the customer after they became a member?
````sql
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

````

### Which item was purchased just before the customer became a member?

````sql
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

````

### What is the number of unique menu items and total amount spent for each member ? before they became a member?
```sql
select s.customer_id,count(DISTINCT s.product_id) as unique_menu_items,
sum(m.price) as total_price
from dannys_diner.sales s inner join dannys_diner.menu m
on s.product_id =m.product_id
inner join dannys_diner.members C on s.customer_id = C.customer_id
where s.order_date <c.join_date 
group by 1
;
```
### If each $1 spent equates to 10 points and sushi has a 2x points multiplier -  how many points would each customer have ?

```sql
DROP TABLE IF EXISTS data_info;
create table data_info as (
Select * ,
  CASE WHEN product_id =1 THEN price * 20 
    ELSE price * 10
    END as Points
  from
  dannys_diner.menu
  )

select s.customer_id,sum(d.points) as Total_Points 
  from data_info d inner join dannys_diner.sales s
    on s.product_id =d.product_id
    group by 1;
```

### Join All The Things - Recreate the table with: customer_id, order_date, product_name, price, member (Y/N)

```sql
select s.customer_id, s.order_date, m.product_name, m.price,
  CASE 
      WHEN c.join_date > s.order_date THEN 'N'
      WHEN c.join_date <= s.order_date THEN 'Y'
      ELSE 'N'
  END as Member
  from dannys_diner.sales s
    LEFT join dannys_diner.menu m
  on m.product_id = s.product_id
    LEFT join dannys_diner.members c 
  on s.customer_id = c.customer_id  
  order by s.customer_id , s.order_date;
 ``` 
 
 ### Rank All The Things - Danny also requires further information about the ranking of customer products, but he purposely does not need the ranking for non-member purchases so he expects null ranking values for the records when customers are not yet part of the loyalty program.
 
```sql 
WITH summary_cte AS 
(
   SELECT s.customer_id, s.order_date, m.product_name, m.price,
      CASE
      WHEN mm.join_date > s.order_date THEN 'N'
      WHEN mm.join_date <= s.order_date THEN 'Y'
      ELSE 'N' END AS member
   FROM dannys_diner.sales AS s
   LEFT JOIN dannys_diner.menu AS m
      ON s.product_id = m.product_id
   LEFT JOIN dannys_diner.members AS mm
      ON s.customer_id = mm.customer_id
)

SELECT *, CASE
   WHEN member = 'N' then NULL
   ELSE
      RANK () OVER(PARTITION BY customer_id, member
      ORDER BY order_date) END AS ranking
FROM summary_cte;

```



