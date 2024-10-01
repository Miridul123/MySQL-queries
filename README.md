# MySQL-queries
## Case-based
create database Ques1;

use question2;

show tables;

create table covid(city varchar(50),days date,cases int);
delete from covid;

insert into covid values('DELHI','2022-01-01',100),
('DELHI','2022-01-02',200),
('DELHI','2022-01-03',300),

('MUMBAI','2022-01-01',100),
('MUMBAI','2022-01-02',100),
('MUMBAI','2022-01-03',300),

('CHENNAI','2022-01-01',100),
('CHENNAI','2022-01-02',200),
('CHENNAI','2022-01-03',150),

('BANGALORE','2022-01-01',100),
('BANGALORE','2022-01-02',300),
('BANGALORE','2022-01-03',200),
('BANGALORE','2022-01-04',400);
### select * from covid;
select city, days,cases,
   case 
      when cases<150 then 'low'
      when cases between 150 and 300 then 'medium'
      else 'high'
      end as case_category
      from covid;
 ### 2.Write a query that labels the first day a city reported cases as "First Reported" and other days as "Subsequent Report".
    SELECT city, days, cases,
       CASE
           WHEN days = (SELECT MIN(days) FROM covid AS c WHERE c.city = covid.city)
           THEN 'First Reported'
           ELSE 'Subsequent Report'
       END AS report_status
FROM covid;
### 3.Write a query that marks cities with more than 200 cases on any day as Hotspot and those with 200 or fewer cases as Normal. 
select city,days,cases ,
   case
   when cases>200 then 'hotspot'
   else 'normal'
   end as city_status
   from covid;
###   4.Write a query that compares the case count of each day with the previous day for each city. If the case count increased, label it as "Increase"; otherwise, label it as "No Increase". 


with ranked_data as (select city, days,cases,rank() over( partition by city order by days) as d_c from covid)
select r1.city,r1.days,r1.cases ,
case  when r1.cases>r2.cases then 'incerase'
     else 'no increase'
     end as case_trend
from ranked_data r1 join ranked_data r2 on r1.city=r2.city and r1.d_c=r2.d_c +1;
 ### 3.Write a query that calculates the total number of cases for each city and classifies cities as Low Risk (total cases < 500), Moderate Risk (total cases between 500 and 1000), or High Risk (total cases > 1000). 
select city,sum(cases) as total_cases,
case
  when sum(cases)<500 then 'low'
  when sum(cases) between 500 and 800 then 'medium'
  else 'high'
  end as risk_level
  from covid  group by city;
### 4.Write a query to label a city as New Outbreak if the number of cases increased by more than 100 from the previous day. 
with ranked_data as(
select city, days,cases, rank() over(partition by city order by days) as r_d from covid)
select r1.city,r1.days,r1.cases ,
    case 
        when r1.cases-r2.cases>100 then 'outbreak'
        else 'stable'
        end as rank_out_brak
	from ranked_data r1 join ranked_data r2 
    on r1.city=r2.city and r1.r_d=r2.r_d+1;
### Rank-Based

#### select * from covid;

Add id

ALTER TABLE covid
ADD COLUMN id INT AUTO_INCREMENT PRIMARY KEY;

Delete repeated row

DELETE t1
FROM covid t1
INNER JOIN (
    SELECT MIN(id) AS id, city, days, cases 
    FROM covid
    GROUP BY city, days, cases
) t2 ON t1.city = t2.city AND t1.days = t2.days AND t1.cases = t2.cases
WHERE t1.id > t2.id;
select * from covid;
alter table covid drop column id;

### Get total cases per city
select city ,sum(cases) as total_cases from covid group by city;
### 2. Find the day with the highest number of cases for each city
select city,days,max(cases)as max_cases from covid group by city, days order by city ,max_cases desc;
### Find the cities where cases are greater than 200 on any day
select distinct city from covid where cases>200;
### 3.Count the total number of days recorded for each city
select city, count(distinct days) as total_days from covid group by city;
### 4. List all cities that have more than 3 entries in the table
select city ,count(*) as total_count from covid  group by city having total_count>3;
### 5.Find the average number of cases per city
select city,avg(cases) as avg_cases from covid group by city;
### 6.Retrieve the top 3 cities with the most total cases 
select city, sum(cases) as total_sum from covid group by city order by total_sum desc limit 3;
### 7.Retrieve the top 3 cities with the most total case
select city, count(cases) as total_sum from covid group by city order by total_sum desc;
### 8.Find the cities where the number of cases increased day by day
SELECT city, days, cases
FROM covid c1
WHERE cases > (
    SELECT cases
    FROM covid c2
    WHERE c1.city = c2.city
    AND c2.days < c1.days
    ORDER BY c2.days DESC
    LIMIT 1
)
ORDER BY city, days;
### 9. Rank cities based on total cases per city
select city,sum(cases) as total_cases,
rank() over(order by sum(cases) desc) as city_rank
 from covid group by city ;
 ### 10. Rank cities by their daily cases for each day 
 select city, days,cases ,rank() over(partition by days order by cases desc) as total_rank from covid;
 ### 11.Find the DENSE_RANK() of each city's total cases 
 select city,sum(cases) as total_cases,dense_rank() over (order by sum(cases) desc) as dense_cases from covid group by city; #----with total of two city are equal
 ### 12.Rank cities by cases for each city on each day (using DENSE_RANK()) 
 select city,days,cases,dense_rank() over(partition by city order by cases desc) as total_dense_rank from covid;
 ### 13. Rank cities based on their daily cases, but include only the top 3 ranks 
 select city,days,cases,dense_rank() over ( partition by days order by cases desc) as total_d_r from covid where  dense_rank() over ( partition by days order by cases desc)>=3;
SELECT city, days, cases, rank_by_cases
FROM (
    SELECT city, days, cases,
           RANK() OVER (PARTITION BY days ORDER BY cases DESC) AS rank_by_cases
    FROM covid
) ranked_data
WHERE rank_by_cases >= 3;
### 14.Get the RANK() of cities based on their average cases per day 
select city,avg(cases) as avg_casey,rank() over(order by avg(cases) desc) as rank_avg from covid group by city; 
### 15.Use RANK() to find cities with the most significant increase in cases day by day 
WITH ranked_data AS (
    SELECT city, days, cases,
           RANK() OVER (PARTITION BY city ORDER BY days) AS rank_by_day
    FROM covid
)
SELECT r1.city, r1.days, r1.cases,
       r1.cases - r2.cases AS case_increase
FROM ranked_data r1
JOIN ranked_data r2
ON r1.city = r2.city AND r1.rank_by_day = r2.rank_by_day + 1;
### 16.Find cities ranked based on cases but partitioned by city and grouped by day 
select city, days cases, rank() over( partition by city,days order by cases desc) as t_r from covid;
### 17.Get the DENSE_RANK() of cities with unique case counts per city and day 
select city,days,cases,dense_rank() over ( partition by city order by cases) as c_r_t from covid;


    
