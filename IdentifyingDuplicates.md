**Identifying Duplicates**

When working with real-life messy data, duplicate records can be found almost everywhere. In this practical file, I have utilized health data to examine certain 
instances of duplicated information.

**Let's inspect snapshot to view the frist 10 rows from the user_logs table**
''''sql 
select * from 
health.user_logs limit 10;
''''
--Record Count 
select count(*) record_count
from health.user_logs;

-- Unique Column Counts (how many unique id values are there in this dataset) ?

select count(DISTINCT(id)) as unique_id
from health.user_logs;

-- Inspect most frequent values in measure column

select measure , count(measure) as frequency
from health.user_logs
group by measure;

-- Add the percentage column 
select measure , count(measure) as frequency ,
round(
    100 * count(measure) / sum(count(measure)) OVER (),
    2
  ) as percentage
from health.user_logs
group by measure
order by frequency desc;

-- Let's add the id column too and limit the output just for the first 10 rows 

select id, measure, count(measure) as frequency ,
round(
    100 * count(measure) / sum(count(measure)) OVER (),
    2
  ) as percentage
from health.user_logs
group by 1,2
order by 3 desc limit 10 ;



--- Let's inspect most frequent values for each column
-- a) Measure Column 

select measure_value, count(measure_value) as frequency  from
health.user_logs 
group by measure_value 
order by 2 desc;
-- b) systolic
select systolic, count(systolic) as frequency from 
health.user_logs 
group by systolic
order by 2 desc ;
-- c)diastolic
select diastolic, count(*) as frequency from 
health.user_logs 
group by diastolic
order by 2 desc; 
-- 






