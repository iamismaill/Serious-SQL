
**Case Danny's Dinner**

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
<img width="505" alt="Screen Shot 2023-04-26 at 5 03 46 PM" src="https://user-images.githubusercontent.com/51711008/234510808-6d7fbfb0-f7f6-49e4-a943-d02ae983e1a9.png">

- Customer A spent $76.
- Customer B spent $74.
- Customer C spent $36.

*** 

### How many days has each customer visited the restaurant?
````sql
select customer_id,count( DISTINCT order_date) total_days 
from dannys_diner.sales 
group by 1
order by 2 desc;
````

<img width="506" alt="Screen Shot 2023-04-26 at 5 09 06 PM" src="https://user-images.githubusercontent.com/51711008/234511933-6c95a70a-c85f-40d5-9b3e-12784f0353c8.png">

- Customer A Visited 4 days 
- Cusstomer B Visited 6 days 
- Customer C Vistied 2 days
*** 

### What was the first item from the menu purchased by each customer?
````sql
select S.customer_id , M.product_name ,S.order_date
from dannys_diner.sales S inner join dannys_diner.menu M 
on S.product_id = M.product_id
group by 1,2,3
order by 3 asc;
````

<img width="660" alt="Screen Shot 2023-04-26 at 5 11 29 PM" src="https://user-images.githubusercontent.com/51711008/234512619-74b317b0-5a11-4455-8f5f-70e8688cb864.png">

***

### What is the most purchased item on the menu and how many times was it purchased by all customers?
````sql
select S.product_id ,M.product_name,count(order_date) as number_of_orders
from dannys_diner.sales S inner join dannys_diner.menu M
on M.product_id = S.product_id
group by 1,2
order by 3 desc
limit 1;
````
<img width="671" alt="Screen Shot 2023-04-26 at 5 13 51 PM" src="https://user-images.githubusercontent.com/51711008/234513177-41ce560f-2af2-4a21-8e22-0b17f7e45f69.png">


***

### Which item was the most popular for each customer?

````sql
select S.customer_id,M.product_name,count(S.product_id) as order_count from 
dannys_diner.sales S
inner join dannys_diner.menu M on M.product_id = S.product_id
group by 1, 2
order by order_count desc;

````
<img width="1269" alt="Screen Shot 2023-04-26 at 5 36 01 PM" src="https://user-images.githubusercontent.com/51711008/234518928-25dd47b1-9278-4bba-b886-bde240e7fd7e.png">

***

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
<img width="1152" alt="Screen Shot 2023-04-26 at 5 37 27 PM" src="https://user-images.githubusercontent.com/51711008/234519239-673d845d-a6f6-432c-86f1-40b08a406518.png">



***
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

<img width="1147" alt="Screen Shot 2023-04-26 at 5 38 18 PM" src="https://user-images.githubusercontent.com/51711008/234519524-7d6565c2-8289-4d27-a438-838e1e32d55a.png">


*** 


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
<img width="1151" alt="Screen Shot 2023-04-26 at 5 39 43 PM" src="https://user-images.githubusercontent.com/51711008/234519822-e58f285d-ae76-4dbf-ad02-f370ca91ae4d.png">


**** 
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

<img width="517" alt="Screen Shot 2023-04-26 at 5 40 29 PM" src="https://user-images.githubusercontent.com/51711008/234520028-0171df45-0a02-4e7e-8f99-f6a5cccba00b.png">


*** 
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
 
 <img width="1155" alt="Screen Shot 2023-04-26 at 5 41 49 PM" src="https://user-images.githubusercontent.com/51711008/234520418-8cdfa08d-2856-4588-a370-ce82e0d895fe.png">

 
 *** 
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

<img width="1268" alt="Screen Shot 2023-04-26 at 5 43 17 PM" src="https://user-images.githubusercontent.com/51711008/234520754-484eb013-16a0-4ef6-9d56-811d49b73e0c.png">

****




