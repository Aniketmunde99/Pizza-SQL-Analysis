# Pizza-SQL-Analysis

# 1.Retrieve the total number of orders placed.

SELECT 
    COUNT(order_id) AS total_orders
FROM
    orders
# 2.Calculate the total revenue generated from pizza sales.

SELECT 
    ROUND(SUM(od.quantity * p.price)) AS total_revenue
FROM
    order_details od
        JOIN
    pizzas p ON od.pizza_id = p.pizza_id

# 3.Identify the most common pizza size ordered.
select p.size,count(od.order_details_id) order_count
from order_details od 
join pizzas p 
on p.pizza_id = od.pizza_id
group by p.size
order by order_count desc
limit 1

# 4.List the top 5 most ordered pizza types along with their quantities.
select pt.name,sum(od.quantity) order_count
from order_details od 
join pizzas p 
on p.pizza_id = od.pizza_id
join pizza_types pt
on pt.pizza_type_id = p.pizza_type_id
group by pt.name
order by order_count desc
limit 5

# 5.Join the necessary tables to find the total quantity of each pizza category and thir revenue.
select pt.category,sum(od.quantity) as order_count,
round(sum(od.quantity*p.price)) revenue
from order_details od 
join pizzas p 
on p.pizza_id = od.pizza_id
join pizza_types pt
on pt.pizza_type_id = p.pizza_type_id
group by pt.category
order by order_count desc

# 6.Determine the distribution of orders by hour of the day.
select hour(order_time) as hrs , count(order_id) as orders_count
from orders 
group by hrs

# 7.Group the orders by date and calculate the average number of pizzas ordered per day.
select round(avg(quantity),0) avg_order_per_day
 from
(select date(order_date) as date , sum(od.quantity) as quantity
from orders o
join order_details od
on o.order_id=od.order_id
group by  date
order by date) as order_quantity ;

# 8.Determine the top 3 most ordered pizza types based on revenue
select pt.name,round(sum(od.quantity*p.price)) total_revenue
from order_details od 
join pizzas p 
on p.pizza_id = od.pizza_id
join pizza_types pt
on pt.pizza_type_id = p.pizza_type_id
group by pt.name
order by total_revenue desc
limit 3

# 9.Calculate the percentage contribution of each pizza type to total revenue.
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

# 10. Analyze the cumulative revenue generated over time.
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

# 11.Determine the top 3 most ordered pizza types based on revenue for each pizza category.
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









