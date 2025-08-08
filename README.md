# zomato_sql_project

# SQL Project: Data Analysis for Zomato - A Food Delivery Company

## Overview

This project demonstrates my SQL problem-solving skills through the analysis of data for Zomato, a popular food delivery company in India. The project involves setting up the database, importing data, handling null values, and solving a variety of business problems using complex SQL queries.

![ERD](https://github.com/MDABRAR-Git/zomato_sql_project/blob/main/ER%20Diagram.png)
## Project Structure

- **Database Setup:** Creation of the `zomato_db` database and the required tables.
- **Data Import:** Inserting sample data into the tables.
- **Data Cleaning:** Handling null values and ensuring data integrity.
- **Business Problems:** Solving 15 specific business problems using SQL queries.



## Database Setup
```sql
CREATE DATABASE zomato_db;
```

### 1. Dropping Existing Tables
```sql
DROP TABLE IF EXISTS deliveries;
DROP TABLE IF EXISTS Orders;
DROP TABLE IF EXISTS customers;
DROP TABLE IF EXISTS restaurants;
DROP TABLE IF EXISTS riders;

CREATE TABLE customers(
	   customer_id INT PRIMARY KEY,
	   customer_name varchar(35),
	   reg_date DATE
	);
	
	
	CREATE TABLE restaurants(
	   restaurant_id int primary key,
	   restaurant_name varchar(55),
	   city varchar(20),
	   opening_hours varchar(40)
	
	);
	
	CREATE TABLE orders(
	     order_id int primary key,
		 customer_id int ,
		 restaurant_id int,
		 order_item varchar(20),
		 order_date DATE,
		 order_time TIME,
		 order_status varchar(55),
		 total_amount float,
		 FOREIGN KEY (customer_id) REFERENCES customers(customer_id),
		 FOREIGN KEY (restaurant_id) REFERENCES restaurants(restaurant_id)
	);
	alter table orders
	drop column order_item,
	add order_item varchar(35);
	
	CREATE TABLE riders(
	rider_id int primary key,
	rider_name varchar(50),
	sign_up date
	);
	
	CREATE TABLE deliveries(
	delivery_id int primary key,
	order_id int,
	delivery_status varchar(50),
	delivery_time TIME,
	rider_id int,
	FOREIGN KEY (order_id) REFERENCES orders(order_id),
	FOREIGN KEY (rider_id) REFERENCES riders(rider_id)
	);
```

## Data Import

## Data Cleaning and Handling Null Values

Before performing analysis, I ensured that the data was clean and free from null values where necessary. For instance:

```sql
--1.For customers table
select count(*) from customers
where customer_id is null
or customer_name is null
or reg_date is null;


--2.for orders table 
select count(*) from orders
where order_id is null
or order_item is null
or customer_id is null
or  restaurant_id is null
or 	 order_item is null
or 	 order_date is null
or 	 order_status is null
or 	 total_amount is null;



--2.for restaurants table 
select count(*) from restaurants
where restaurant_id is null
or restaurant_name is null
or   city is null
 or  opening_hours is null;


select count(*) from riders
where
rider_id is null
or rider_name is null
or sign_up is null;


select count(*) from deliveries
where
 delivery_id is null
 or order_id is null
or delivery_status is null
or delivery_time is null
or rider_id is null;
```

## Business Problems Solved

### 1. Write a query to find the top 5 most frequently ordered dishes by customer called "Arjun Mehta" in the last 1 year.

```sql
SELECT 


SELECT
    customer_id,
    customer_name,
    dishes,
    total_order,
    DENSE_RANK() OVER (ORDER BY total_order DESC) AS rank
FROM (
    SELECT
        c.customer_id,
        c.customer_name,
        o.order_item AS dishes,
        COUNT(*) AS total_order
    FROM
        customers AS c
        JOIN orders AS o ON c.customer_id = o.customer_id
    WHERE
        c.customer_name = 'Sameer Khan'
        AND o.order_date > CURRENT_DATE - INTERVAL '720 days'
    GROUP BY
        c.customer_id,
        c.customer_name,
        o.order_item
		) as per_dish

ORDER BY
    total_order DESC;

```

### 2. Popular Time Slots
-- Question: Identify the time slots during which the most orders are placed. based on 2-hour intervals.

**Approach 1:**

```sql
-- Approach 1
SELECT 
	FLOOR(EXTRACT(HOUR FROM order_time)/2)*2 as start_time,
	FLOOR(EXTRACT(HOUR FROM order_time)/2)*2 + 2 as end_time,
	COUNT(*) as total_orders
FROM orders
GROUP BY 1, 2
ORDER BY 3 DESC;
```

**Approach 2:**

```sql
SELECT
    CASE
        WHEN EXTRACT(HOUR FROM order_time) BETWEEN 0 AND 1 THEN '00:00 - 02:00'
        WHEN EXTRACT(HOUR FROM order_time) BETWEEN 2 AND 3 THEN '02:00 - 04:00'
        WHEN EXTRACT(HOUR FROM order_time) BETWEEN 4 AND 5 THEN '04:00 - 06:00'
        WHEN EXTRACT(HOUR FROM order_time) BETWEEN 6 AND 7 THEN '06:00 - 08:00'
        WHEN EXTRACT(HOUR FROM order_time) BETWEEN 8 AND 9 THEN '08:00 - 10:00'
        WHEN EXTRACT(HOUR FROM order_time) BETWEEN 10 AND 11 THEN '10:00 - 12:00'
        WHEN EXTRACT(HOUR FROM order_time) BETWEEN 12 AND 13 THEN '12:00 - 14:00'
        WHEN EXTRACT(HOUR FROM order_time) BETWEEN 14 AND 15 THEN '14:00 - 16:00'
        WHEN EXTRACT(HOUR FROM order_time) BETWEEN 16 AND 17 THEN '16:00 - 18:00'
        WHEN EXTRACT(HOUR FROM order_time) BETWEEN 18 AND 19 THEN '18:00 - 20:00'
        WHEN EXTRACT(HOUR FROM order_time) BETWEEN 20 AND 21 THEN '20:00 - 22:00'
        WHEN EXTRACT(HOUR FROM order_time) BETWEEN 22 AND 23 THEN '22:00 - 00:00'
    END AS time_slot,
    COUNT(order_id) AS order_count
FROM Orders
GROUP BY time_slot
ORDER BY order_count DESC;
```

### 3. Order Value Analysis
-- Question: Find the average order value per customer who has placed more than 750 orders.
-- Return customer_name, and aov(average order value)

```sql

select c.customer_name,c.customer_id,avg(total_amount)
  FROM
        customers AS c
        JOIN orders AS o ON c.customer_id = o.customer_id
group by 1,2
having count(customer_name)>750;

```

### 4. High-Value Customers
-- Question: List the customers who have spent more than 100K in total on food orders.
-- return customer_name, and customer_id!

```sql

select c.customer_name,c.customer_id,sum(total_amount)
  FROM
        customers AS c
        JOIN orders AS o ON c.customer_id = o.customer_id
group by 1,2
having sum(total_amount)>100000;

```

### 5. Orders Without Delivery
-- Question: Write a query to find orders that were placed but not delivered. 
-- Return each restuarant name, city and number of not delivered orders 

```sql

select r.restaurant_name,r.city,d.delivery_status,
count(*) as not_delivered_items
from 
orders as o
left join restaurants as r ON o.restaurant_id=r.restaurant_id
left join deliveries as d on o.order_id=d.order_id
where d.delivery_status is null
group by 1,2,3
order by 4 desc;

```


### 6. Restaurant Revenue Ranking: 
-- Rank restaurants by their total revenue from the last year, including their name, 
-- total revenue, and rank within their city.

```sql

select r.city,r.restaurant_name,sum(o.total_amount),
rank() over (partition by r.city order by sum(o.total_amount) desc)  as rank
FROM
restaurants as r 
JOIn orders as o 
ON r.restaurant_id=o.restaurant_id
where o.order_date < current_date - interval '720 days'
group by 1,2
order by 1,3 desc
;


```

### 7. Most Popular Dish by City: 
-- Identify the most popular dish in each city based on the number of orders.

```sql

with ranking_table as 
(select r.city,
      order_item,
	  count(o.customer_id) as pop_dish,
	  rank() over (partition by r.city order by count(o.customer_id) desc) as rank
from orders as o 
join customers as c on c.customer_id=o.customer_id
join restaurants as r on o.restaurant_id=r.restaurant_id
group by 1,2
order by 1,3,4)
select * from ranking_table where rank=1 ;
```

### 8. Customer Churn: 
-- Find customers who haven’t placed an order in 2024 but did in 2023.

```sql

select c.customer_name,c.customer_id from 
orders as o 
join customers as c On o.customer_id=c.customer_id
group by c.customer_id,c.customer_name
having
      sum(case when o.order_date>='2023-01-01' and o.order_date<'2024-01-01' then 1 else 0 end)>0
	  and
	  sum(case when o.order_date>='2024-01-01' and o.order_date<'2025-01-01' then 1 else 0 end)=0;


```


### 9. Rider Average Delivery Time: 
-- Determine each rider's average delivery time.

```sql

select d.rider_id,avg(d.delivery_time)
from deliveries as d 
join riders as r on r.rider_id=d.rider_id
where d.delivery_status='Delivered'
group by 1
order by 1,2 desc;

```


### 10. Customer Segmentation: 
-- Customer Segmentation: Segment customers into 'Gold' or 'Silver' groups based on their total spending 
-- compared to the average order value (AOV). If a customer's total spending exceeds the AOV, 
-- label them as 'Gold'; otherwise, label them as 'Silver'. Write an SQL query to determine each segment's 
-- total number of orders and total revenue

```sql

select segment,
       sum(total_spend), 
       sum(total_order) 
from 
(select customer_id,
       sum(total_amount) as total_spend,
	   count(order_id) as total_order,
	   (case when Sum(total_amount)>(select avg(total_amount) from orders) then 'gold'
	   else 'silver'
	   end) as segment
from orders
group by 1) as t1

group by 1;
```

### 11. Rider Monthly Earnings: 
-- Calculate each rider's total monthly earnings, assuming they earn 8% of the order amount.

```sql


select  rider_id,
		to_char(o.order_date,'mm-yy') as monthly,
        sum(total_amount) as total_revenue,
       sum(total_amount)*0.08 as mnthly_earning
from 
orders as o 
join deliveries as d on o.order_id=d.order_id
group by 1,2
order by 1,2,3 desc
;
```


### 12. Q.15 Order Frequency by Day: 
-- Analyze order frequency per day of the week and identify the peak day for each restaurant.

```sql
SELECT * FROM
(
	SELECT 
		r.restaurant_name,
		-- o.order_date,
		TO_CHAR(o.order_date, 'Day') as day,
		COUNT(o.order_id) as total_orders,
		RANK() OVER(PARTITION BY r.restaurant_name ORDER BY COUNT(o.order_id)  DESC) as rank
	FROM orders as o
	JOIN
	restaurants as r
	ON o.restaurant_id = r.restaurant_id
	GROUP BY 1, 2
	ORDER BY 1, 3 DESC
	) as t1
WHERE rank = 1;
```



### 13. Monthly Sales Trends: 
-- Identify sales trends by comparing each month's total sales to the previous month.

```sql
SELECT 
	EXTRACT(YEAR FROM order_date) as year,
	EXTRACT(MONTH FROM order_date) as month,
	SUM(total_amount) as total_sale,
	LAG(SUM(total_amount), 1) OVER(ORDER BY EXTRACT(YEAR FROM order_date), EXTRACT(MONTH FROM order_date)) as prev_month_sale
FROM orders
GROUP BY 1, 2;
```



### 14. Order Item Popularity: 
-- Track the popularity of specific order items over time and identify seasonal demand spikes.

```sql
SELECT 
	order_item,
	seasons,
	COUNT(order_id) as total_orders
FROM 
(
SELECT 
		*,
		EXTRACT(MONTH FROM order_date) as month,
		CASE 
			WHEN EXTRACT(MONTH FROM order_date) BETWEEN 4 AND 6 THEN 'Spring'
			WHEN EXTRACT(MONTH FROM order_date) > 6 AND 
			EXTRACT(MONTH FROM order_date) < 9 THEN 'Summer'
			ELSE 'Winter'
		END as seasons
	FROM orders
) as t1
GROUP BY 1, 2
ORDER BY 1, 3 DESC;
```

### 15. Rank each city based on the total revenue for last year 2023
```sql

select 
      -- order_date=<urrent_date-interval between '720 days',
       city,
	   sum(total_amount) as total_revenue,
	   rank() over (order by sum(total_amount) desc) as ranking
from orders as o
join restaurants as r on o.restaurant_id=r.restaurant_id
where o.order_date > CURRENT_DATE - INTERVAL '720 days'
group by 1
order by 1,2 desc;

```

## Conclusion

This project highlights my ability to handle complex SQL queries and provides solutions to real-world business problems in the context of a food delivery service like Zomato. The approach taken here demonstrates a structured problem-solving methodology, data manipulation skills, and the ability to derive actionable insights from data.

## Notice 
All customer names and data used in this project are computer-generated using AI and random functions. They do not represent real data associated with Zomato or any other entity. This project is solely for learning and educational purposes, and any resemblance to actual persons, businesses, or events is purely coincidental.
