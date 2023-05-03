## Customer Journey

Uncovering Business Insights: Key Questions


### Based off the 8 sample customers provided in the sample subscriptions table below, write a brief description about each customer’s onboarding journey.

 
````sql

select 
  s.customer_id ,p.plan_id,p.plan_name,s.start_date
  from 
  foodie_fi.subscriptions s
  inner join foodie_fi.plans p 
  on s.plan_id = p.plan_id
  where s.customer_id in (1,2,3,4,5,6,7,8); 
 
 ````

  From the sample I choose 3 customers 
  
  ### Customer One 
  
  ````sql

  select 
  s.customer_id ,p.plan_id,p.plan_name,s.start_date
  from 
  foodie_fi.subscriptions s
  inner join foodie_fi.plans p 
  on s.plan_id = p.plan_id
  where s.customer_id in =1; 

````

### Customer Four 
````sql

  select 
  s.customer_id ,p.plan_id,p.plan_name,s.start_date
  from 
  foodie_fi.subscriptions s
  inner join foodie_fi.plans p 
  on s.plan_id = p.plan_id
  where s.customer_id =4; 
  ````
### Customer Three

````sql

  select 
  s.customer_id ,p.plan_id,p.plan_name,s.start_date
  from 
  foodie_fi.subscriptions s
  inner join foodie_fi.plans p 
  on s.plan_id = p.plan_id
  where s.customer_id in =3; 
  
  ````
