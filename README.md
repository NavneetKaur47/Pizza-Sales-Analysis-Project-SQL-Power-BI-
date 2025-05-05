# Pizza-Sales-Analysis-Project-SQL-Power-BI-
Used SQL and Power BI to analyze sales data for a pizza restaurant. Extracted key performance metrics such as total revenue, most popular pizza size, hourly order distribution, and top-selling categories. 
-- Retrieve the total number of orders placed --
select * from orders;
select count(order_id) as total_orders from orders;

-- calculate the total revenue generated from pizza sales --

SELECT ROUND(SUM(order_details.quantity * pizzas.price),2) AS total_sales
FROM order_details JOIN
pizzas ON order_details.pizza_id = pizzas.pizza_id;
    
-- Identify the highest priced pizza --
select * from pizza_types;
select * from pizzas;
select pizza_types.name , pizzas.price from pizza_types
join pizzas on pizza_types.pizza_type_id = pizzas.pizza_type_id
order by pizzas.price desc limit 1;

-- identify the most common pizza size ordered --
SELECT pizzas.size, COUNT(order_details.order_details_id) AS total_orders
FROM pizzas
JOIN order_details ON pizzas.pizza_id = order_details.pizza_id
GROUP BY pizzas.size
ORDER BY total_orders DESC
LIMIT 1;
 

-- list the top 5 most ordered pizza types along with their quantities --
select pizza_types.name , sum(order_details.quantity) as quantity
from pizza_types join pizzas
on pizza_types.pizza_type_id = pizzas.pizza_type_id
join order_details
on order_details.pizza_id = pizzas.pizza_id
group by pizza_types.name
order by quantity desc limit 5;

-- Join the necessary tables to find the  total quantity of each pizza category ordered --
select pizza_types.category , sum(order_details.quantity) as quantity
from pizza_types join pizzas 
on pizza_types.pizza_type_id = pizzas.pizza_type_id
join order_details
on order_details.pizza_id = pizzas.pizza_id
group by pizza_types.category order by quantity desc;

-- determine the distribution of orders by hour of the day --
select hour(order_time), count(order_id) from orders
group by hour(order_time);

-- join relevant tables to find the category- wise distribution of pizzas
select category , count(name) from pizza_types
group by category;
use pizzahut;

-- group the orders by date and calculate the average no. of pizzas order per day
select round(avg(quantity),0) as avg_pizzas_order_per_day from
(select orders.order_date, sum(order_details.quantity) as quantity from orders
join order_details
on orders.order_id = order_details.order_id
group by orders.order_date) as order_quantity;

-- determine the top 3 most ordered pizza type based on revenue.
select pizza_types.name, sum(order_details.quantity * pizzas.price) as revenue
from pizza_types join pizzas
on pizzas.pizza_type_id = pizza_types.pizza_type_id
join order_details
on order_details.pizza_id = pizzas.pizza_id
group by pizza_types.name order by revenue desc limit 3;

-- calculate the percentage contribution of each pizza type to total revenue
select pizza_types.category, 
sum(order_details.quantity * pizzas.price) / (select round(sum(order_details.
quantity * pizzas.price),2) as total_sales 
from order_details
join pizzas on pizzas.pizza_id = order_details.pizza_id) * 100 as revenue
from pizza_types join pizzas
on pizza_types.pizza_type_id = pizzas.pizza_type_id
join order_details
on order_details.pizza_id = pizzas.pizza_id
group by pizza_types.category order by revenue desc;

-- analyse the cumulative revenue generated over time
select order_date,
sum(revenue) over(order by order_date) as cum_revenue
from
(select orders.order_date,
sum(order_details.quantity * pizzas.price) as revenue
from order_details join pizzas
on order_details.pizza_id = pizzas.pizza_id
join orders
on orders.order_id = order_details.order_id
group by orders.order_date) as sales;
