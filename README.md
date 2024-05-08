# Pizza_Sales_Analysis

Converted CSV files into Database then try solve some questions using MySql Workbench.

## Basic:

1. Retrieve the total number of orders placed.

SELECT COUNT(ORDER_ID) as total_orders FROM orders;

2. Calculate the total revenue generated from pizza sales.

SELECT 
    SUM((order_details.quantity) * (pizzas.price)) AS TOTAL_REVENUE
FROM
    order_details
        JOIN
    PIZZAS ON ORDER_DETAILS.PIZZA_ID = PIZZAS.PIZZA_ID;
3. Identify the highest-priced pizza.

SELECT 
    pizza_types.name, pizzas.price
FROM
    PIZZA_TYPES
        JOIN
    PIZZAS ON PIZZA_TYPES.PIZZA_TYPE_ID = PIZZAS.PIZZA_TYPE_ID
WHERE
    PIZZAS.PRICE = (SELECT 
            MAX(PRICE)
        FROM
            PIZZAS);
            
4. Identify the most common pizza size ordered.

SELECT 
    pizzas.size,
    COUNT(order_details.order_detail_id) AS size_count
FROM
    pizzas
        JOIN
    order_details ON pizzas.pizza_id = order_details.pizza_id
GROUP BY pizzas.size
ORDER BY size_count DESC
LIMIT 1;

5. List the top 5 most ordered pizza types along with their quantities.

SELECT 
    pizza_types.name, SUM(order_details.quantity) AS quantity
FROM
    pizza_types
        JOIN
    pizzas ON pizza_types.pizza_type_id = pizzas.pizza_type_id
        JOIN
    order_details ON order_details.pizza_id = pizzas.pizza_id
GROUP BY pizza_types.name
ORDER BY quantity DESC
LIMIT 5;


## Intermediate:

1. Join the necessary tables to find the total quantity of each pizza category ordered.

SELECT pt.category, SUM(od.quantity) AS total_quantity
FROM Order_details od
JOIN Pizzas p ON od.pizza_id = p.pizza_id
JOIN Pizza_types pt ON p.pizza_type_id = pt.pizza_type_id
GROUP BY pt.category;

2. Determine the distribution of orders by hour of the day.

SELECT EXTRACT(HOUR FROM order_time) AS hour_of_day, COUNT(*) AS order_count
FROM Orders
GROUP BY hour_of_day
ORDER BY hour_of_day;

3. Join relevant tables to find the category-wise distribution of pizzas.

SELECT pt.category, COUNT(*) AS pizza_count
FROM Pizzas p
JOIN Pizza_types pt ON p.pizza_type_id = pt.pizza_type_id
GROUP BY pt.category;

4. Group the orders by date and calculate the average number of pizzas ordered per day.

SELECT order_date, AVG(total_pizzas) AS avg_pizzas_per_day
FROM (
    SELECT order_date, COUNT(*) AS total_pizzas
    FROM Orders o
    JOIN Order_details od ON o.order_id = od.order_id
    GROUP BY order_date
) AS orders_per_day
GROUP BY order_date;

5. Determine the top 3 most ordered pizza types based on revenue.

SELECT pt.name AS pizza_name, SUM(od.quantity * p.price) AS total_revenue
FROM Order_details od
JOIN Pizzas p ON od.pizza_id = p.pizza_id
JOIN Pizza_types pt ON p.pizza_type_id = pt.pizza_type_id
GROUP BY pt.name
ORDER BY total_revenue DESC
LIMIT 3;


## Advanced:

1. Calculate the percentage contribution of each pizza type to total revenue.

SELECT pt.name AS pizza_name, 
       SUM(od.quantity * p.price) AS total_revenue,
       (SUM(od.quantity * p.price) / total_total_revenue.total_revenue) * 100 AS revenue_percentage
FROM Order_details od
JOIN Pizzas p ON od.pizza_id = p.pizza_id
JOIN Pizza_types pt ON p.pizza_type_id = pt.pizza_type_id
CROSS JOIN (
   SELECT SUM(od.quantity * p.price) AS total_revenue
   FROM Order_details od
   JOIN Pizzas p ON od.pizza_id = p.pizza_id
) AS total_total_revenue
GROUP BY pt.name, total_total_revenue.total_revenue
ORDER BY total_revenue DESC;



2. Analyze the cumulative revenue generated over time.

SELECT order_date, SUM(total_revenue) OVER (ORDER BY order_date) AS cumulative_revenue
FROM (
    SELECT o.order_date, SUM(od.quantity * p.price) AS total_revenue
    FROM Orders o
    JOIN Order_details od ON o.order_id = od.order_id
    JOIN Pizzas p ON od.pizza_id = p.pizza_id
    GROUP BY o.order_date
) AS revenue_by_date;

3. Determine the top 3 most ordered pizza types based on revenue for each pizza category.

WITH RankedPizzaTypes AS (
   SELECT pt.name AS pizza_name, 
          pt.category,
          SUM(od.quantity * p.price) AS total_revenue,
          ROW_NUMBER() OVER(PARTITION BY pt.category ORDER BY SUM(od.quantity * p.price) DESC) AS ranking
   FROM Order_details od
   JOIN Pizzas p ON od.pizza_id = p.pizza_id
   JOIN Pizza_types pt ON p.pizza_type_id = pt.pizza_type_id
   GROUP BY pt.name, pt.category
)
SELECT pizza_name, category, total_revenue
FROM RankedPizzaTypes
WHERE ranking <= 3;

