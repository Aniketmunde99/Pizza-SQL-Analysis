# SQL Project: Data Analysis for Pizza Sales

## Overview

This project demonstrates my SQL problem-solving skills through the analysis of data for Pizaa sales, a popular food delivery company in India. The project involves setting up the database, importing data, handling null values, and solving a variety of business problems using complex SQL queries.

## Project Structure

- **Database Setup:** Creation of the `pizza_db` database and the required tables.
- **Data Import:** Inserting sample data into the tables.
- **Data Cleaning:** Handling null values and ensuring data integrity.
- **Business Problems:** Solving 12 specific business problems using SQL queries.


## Database Setup
```sql
CREATE DATABASE pizza_db;
```

### Creating Tables
```sql
CREATE TABLE order_details (
    order_details_id INT PRIMARY KEY,
    order_id INT NOT NULL,
    pizza_id VARCHAR(50) NOT NULL,
    quantity INT NOT NULL
);

CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    date DATE,
    time TIME
);

CREATE TABLE pizza_types (
    pizza_type_id VARCHAR(20) PRIMARY KEY,
    name VARCHAR(100),
    category VARCHAR(50),
    ingredients TEXT
);

CREATE TABLE pizzas (
    pizza_id VARCHAR(20) PRIMARY KEY,
    pizza_type_id VARCHAR(20),
    size CHAR(1),
    price DECIMAL(5, 2),
    FOREIGN KEY (pizza_type_id) REFERENCES pizza_types(pizza_type_id)
);

```
### 1.Retrieve the total number of orders placed.
```sql
SELECT 
    COUNT(order_id) AS total_orders
FROM
    orders
```
### 2.Calculate the total revenue generated from pizza sales.
```sql
SELECT 
    ROUND(SUM(od.quantity * p.price)) AS total_revenue
FROM
    order_details od
        JOIN
    pizzas p ON od.pizza_id = p.pizza_id
```
### 3.Identify the highest-priced pizza
```sql
select pt.name ,p.price 
from pizza_types pt join pizzas p
on pt.pizza_type_id = p.pizza_type_id
order by p.price desc 
limit 1
```
### 4.Identify the most common pizza size ordered.
```sql
select p.size,count(od.order_details_id) order_count
from order_details od 
join pizzas p 
on p.pizza_id = od.pizza_id
group by p.size
order by order_count desc
limit 1
```

### 5.List the top 5 most ordered pizza types along with their quantities.
```sql
select pt.name,sum(od.quantity) order_count
from order_details od 
join pizzas p 
on p.pizza_id = od.pizza_id
join pizza_types pt
on pt.pizza_type_id = p.pizza_type_id
group by pt.name
order by order_count desc
limit 5
```
### 6.Join the necessary tables to find the total quantity of each pizza category and thir revenue.
```sql
select pt.category,sum(od.quantity) as order_count,
round(sum(od.quantity*p.price)) revenue
from order_details od 
join pizzas p 
on p.pizza_id = od.pizza_id
join pizza_types pt
on pt.pizza_type_id = p.pizza_type_id
group by pt.category
order by order_count desc
```
### 7.Determine the distribution of orders by hour of the day.
```sql
select hour(order_time) as hrs , count(order_id) as orders_count
from orders 
group by hrs
```
### 8.Group the orders by date and calculate the average number of pizzas ordered per day.
```sql
select round(avg(quantity),0) avg_order_per_day
 from
(select date(order_date) as date , sum(od.quantity) as quantity
from orders o
join order_details od
on o.order_id=od.order_id
group by  date
order by date) as order_quantity ;
```
### 9.Determine the top 3 most ordered pizza types based on revenue
```sql
select pt.name,round(sum(od.quantity*p.price)) total_revenue
from order_details od 
join pizzas p 
on p.pizza_id = od.pizza_id
join pizza_types pt
on pt.pizza_type_id = p.pizza_type_id
group by pt.name
order by total_revenue desc
limit 3
```
### 10.Calculate the percentage contribution of each pizza type to total revenue.
```sql
select pt.category,round(
(sum(od.quantity*p.price)*100)/(
SELECT 
    SUM(od.quantity * p.price) AS total_revenue
FROM
    order_details od
        JOIN
    pizzas p ON od.pizza_id = p.pizza_id),2)total_revenue
from order_details od 
join pizzas p 
on p.pizza_id = od.pizza_id
join pizza_types pt
on pt.pizza_type_id = p.pizza_type_id
group by pt.category
order by total_revenue desc
```
### 11. Analyze the cumulative revenue generated over time.
```sql
select (order_date),
sum(revenue) over(order by order_date) as cum
from 
(select (o.order_date)  , 
round(sum(od.quantity * p.price),0) AS revenue
FROM
    order_details od
        JOIN
    pizzas p ON od.pizza_id = p.pizza_id
join orders o 
on o.order_id=od.order_id
group by o.order_date) as sales
```
### 12.Determine the top 3 most ordered pizza types based on revenue for each pizza category.
```sql
select category ,name,total_revenue,rnk
from
(select category ,name,total_revenue,
rank() over(partition by category order by total_revenue desc) as rnk
from
(select pt.category,pt.name,round(sum(od.quantity*p.price)) total_revenue
from order_details od 
join pizzas p 
on p.pizza_id = od.pizza_id
join pizza_types pt
on pt.pizza_type_id = p.pizza_type_id
group by pt.name,pt.category 
order by category asc,total_revenue desc ) as A) as B
where rnk <= 3
```








