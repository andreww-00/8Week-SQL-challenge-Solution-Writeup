### Pizza Runner write up 

## Introduction and Problem Statement:  
(Summaries of introduction and Problem from [Case Study # 2](https://8weeksqlchallenge.com/case-study-1/))

- Danny created a business idea that combines Uber and Pizza called pizza runner

- Danny wants to use his data to best determine the scheduling of his runners

## Data:
Danny supplied 6 different tables - some of which need cleaning. These tables are best summarized in the following entity relationship diagram. 


  
# Solution Write-up
(All questions are answered in MySQL)

### Data Clean Up
Before getting started some tables need to be cleaned 
Customer_Orders has lots of empty values and the values that are present don’t all take the same form of null. Below shows the table prior to clean up.

| order_id | customer_id | pizza_id | exclusions | extras | order_time |
|--------|-------------|----------|------------|--------|------------|
| 1      | 101         | 1        |            |        |            |
| 2      | 101         | 1        |            |        |            |
| 3      | 102         | 1        |            |        |            |
| 3      | 102         | 2        |            |        | NULL       |
| 4      | 103         | 1        | 4          |        |            |
| 4      | 103         | 1        | 4          |        |            |
| 4      | 103         | 2        | 4          |        |            |
| 5      | 104         | 1        | 'null'     | 1      |            |
| 6      | 101         | 2        | 'null'     | 'null' |            |
| 7      | 105         | 2        | 'null'     | 1      |            |
| 8      | 102         | 1        | 'null'     | 'null' |            |
| 9      | 103         | 1        | 4          | 1,5    |            |
| 10     | 104         | 1        | 'null'     | 'null' |            |
| 10     | 104         | 1        | 2,6        | 1,4    |            |


```SQL 

-- Create table that will contain cleaned information
CREATE TABLE customer_orders_cleaned AS 
SELECT * FROM customer_orders;

-- Cleans NULL type values from exclusions and extras
UPDATE customer_orders_cleaned
SET exclusions = coalesce(exclusions, ''),
	extras = coalesce(extras, '');

-- Removes 'null' strings and replaces them with empty strings
UPDATE customer_orders_cleaned
SET
  exclusions = CASE WHEN exclusions = 'null' THEN '' ELSE exclusions END,
  extras = CASE WHEN extras = 'null' THEN '' ELSE extras END;

  SELECT * FROM customer_orders_cleaned

```

Running the three queries above unifies the data in the exclusions and extras columns by changing all NULL/’null’ values into empty strings. The output is shown below

| order_id | customer_id | pizza_id | exclusions | extras | order_time |
|--------|-------------|----------|------------|--------|------------|
| 1      | 101         | 1        |            |        |            |
| 2      | 101         | 1        |            |        |            |
| 3      | 102         | 1        |            |        |            |
| 3      | 102         | 2        |            |        |            |
| 4      | 103         | 1        | 4          |        |            |
| 4      | 103         | 1        | 4          |        |            |
| 4      | 103         | 2        | 4          |        |            |
| 5      | 104         | 1        |            | 1      |            |
| 6      | 101         | 2        |            |        |            |
| 7      | 105         | 2        |            | 1      |            |
| 8      | 102         | 1        |            |        |            |
| 9      | 103         | 1        | 4          | 1,5    |            |
| 10     | 104         | 1        |            |        |            |
| 10     | 104         | 1        | 2,6        | 1,4    |            |


Runner_orders: there is also a lot of cleaning needed in this table too. There are null strings, null values, and empty strings that need to be unified. The duration and distance need their units removed as well. The data types for order time, distance, and duration also need to be corrected
Below is the table before changes

| order_id | runner_id | pickup_time         | distance | duration | cancellation          |
|----------|-----------|---------------------|----------|----------|-----------------------|
| 1        | 1         | 2020-01-01 18:15:34 | 20km     | 32 minutes |                       |
| 2        | 1         | 2020-01-01 19:10:54 | 20km     | 27 minutes |                       |
| 3        | 1         | 2020-01-03 00:12:37 | 13.4km   | 20 mins  |      NULL                 |
| 4        | 2         | 2020-01-04 13:53:03 | 23.4     | 40       |    NULL                   |
| 5        | 3         | 2020-01-08 21:10:57 | 10       | 15       |         NULL              |
| 6        | 3         | null                | null     | null     | Restaurant Cancellation |
| 7        | 2         | 2020-01-08 21:30:45 | 25km     | 25mins   | null                  |
| 8        | 2         | 2020-01-10 00:15:02 | 23.4 km  | 15 minute | null                  |
| 9        | 2         | null                | null     | null     | Customer Cancellation |
| 10       | 1         | 2020-01-11 18:50:20 | 10km     | 10minutes | null                  |


```SQL

-- Creates table using CASE to unify null statements to empty strings
CREATE TABLE runner_orders_cleaned AS
SELECT
order_id,
runner_id,
CASE
WHEN pickup_time LIKE 'null' THEN NULL
ELSE pickup_time
END AS pickup_time,
CASE
WHEN distance LIKE 'null' THEN NULL
ELSE distance
END AS distance,
CASE
WHEN duration LIKE 'null' THEN NULL
ELSE duration
END AS duration,
CASE
WHEN cancellation LIKE 'null' OR CANCELLATION LIKE '' THEN NULL
ELSE cancellation
END as cancellation
FROM runner_orders;

-- removes units after distance and duration
UPDATE runner_orders_cleaned
SET distance = REGEXP_REPLACE(distance, '[^0-9.]+', ''),
duration = REGEXP_REPLACE(duration, '[^0-9.]+', '')
WHERE distance REGEXP '[^0-9.]+' OR duration REGEXP '[^0-9.]+';

-- corrects data types for pickup time, distance, and duration
ALTER TABLE runner_orders_cleaned 
MODIFY COLUMN pickup_time TIMESTAMP,
MODIFY COLUMN distance INT,
MODIFY COLUMN duration INT;

```

Running this three queries fixes all the problems discussed above and following is the corrected table


| order_id | runner_id | pickup_time         | distance | duration | cancellation          |
|----------|-----------|---------------------|----------|----------|-----------------------|
| 1        | 1         | 2020-01-01 18:15:34 | 20       | 32       | NULL                  |
| 2        | 1         | 2020-01-01 19:10:54 | 20       | 27       | NULL                  |
| 3        | 1         | 2020-01-03 00:12:37 | 13       | 20       | NULL                  |
| 4        | 2         | 2020-01-04 13:53:03 | 23       | 40       | NULL                  |
| 5        | 3         | 2020-01-08 21:10:57 | 10       | 15       | NULL                  |
| 6        | 3         | NULL                | NULL     | NULL     | Restaurant Cancellation |
| 7        | 2         | 2020-01-08 21:30:45 | 25       | 25       | NULL                  |
| 8        | 2         | 2020-01-10 00:15:02 | 23       | 15       | NULL                  |
| 9        | 2         | NULL                | NULL     | NULL     | Customer Cancellation |
| 10       | 1         | 2020-01-11 18:50:20 | 10       | 10       | NULL                  |


Now the data is ready to reveal its insights!

### A. Pizza Metrics

### Question 1: How many pizzas were ordered?

```SQL 
SELECT COUNT(*) as amount_of_orders FROM customer_orders_cleaned;
```

###Output:

|amount_of_orders|
|------|
|14|

### Explanation
Using COUNT(*) counts the number of rows and ignores null rows as well as counting duplicate rows, as some customers might have ordered two pizzas

### Question 2: How many unique customer orders were made?

```SQL 
SELECT COUNT(DISTINCT order_id) as Unique_orders
FROM customer_orders_cleaned;

```

###Output:

|unique_orders|
|------|
|10|


### Explanation
Use to find the amount of orders made, use distinct to make sure those values are each distinct.

### Question 3: How many successful orders were delivered by each runner?

```SQL 

SELECT runner_id AS Runner, 
COUNT(order_id) AS amount_delivered
FROM runner_orders_cleaned
WHERE runner_orders_cleaned.cancellation IS NULL
GROUP BY runner_id;

```

###Output:

|Runner| amount_delivered |
|------|-------|
|1| 4  |
|2| 3  |
|3| 1  |

### Explanation
Use Count and group by runner_id to find the amount of orders each runner took
Set a where to show which cancellation values were NULL to show successful orders.


### Question 4: How many of each type of pizza was delivered?

```SQL

SELECT pizza_name AS pizza_name, 
COUNT(pizza_name) AS amount
FROM runner_orders_cleaned
JOIN customer_orders_cleaned ON runner_orders_cleaned.order_id = customer_orders_cleaned.order_id 
JOIN pizza_names ON customer_orders_cleaned.pizza_id = pizza_names.pizza_id
WHERE runner_orders_cleaned.cancellation IS NULL
GROUP BY pizza_name
ORDER BY amount ASC;

```

###Output:

|pizza_name| amount |
|------|-------|
|Vegetarian| 3  |
|Meatlovers| 9  |

### Explanation
JOIN on order_id and then by pizza_id to show the pizza names
COUNT the pizzas and group them their name
Don’t forget where IS NULL to show only successful deliveries

### Question 5: How many Vegetarian and Meatlovers were ordered by each customer?

```SQL 

SELECT customer_id AS customer,
pizza_name AS pizza_type,
COUNT(customer_id) AS amount_of_orders
FROM runner_orders_cleaned
JOIN customer_orders_cleaned ON runner_orders_cleaned.order_id = customer_orders_cleaned.order_id 
JOIN pizza_names ON customer_orders_cleaned.pizza_id = pizza_names.pizza_id
GROUP BY pizza_name, customer_id
order by customer_id;

```

### Output

| Customer | Pizza Type  | Amount of Orders |
|--------- |------------ |----------------- |
| 101      | Meatlovers  | 2                |
| 101      | Vegetarian  | 1                |
| 102      | Vegetarian  | 1                |
| 102      | Meatlovers  | 2                |
| 103      | Vegetarian  | 1                |
| 103      | Meatlovers  | 3                |
| 104      | Meatlovers  | 3                |
| 105      | Vegetarian  | 1                |


### Explanation
Join customer_orders to runner_orders to pizza_names to show the type of pizza that was ordered
COUNT customer_id and then group by customer_id and pizza_type

### Question 6: What was the maximum number of pizzas delivered in a single order?

```SQL 

WITH amount_per_order AS (
SELECT customer_orders_cleaned.order_id AS orders,
COUNT(customer_id) AS amount_orders
FROM customer_orders_cleaned
JOIN runner_orders_cleaned ON customer_orders_cleaned.order_id = runner_orders_cleaned.order_id
WHERE cancellation IS NULL 
GROUP BY customer_orders_cleaned.order_id
)

SELECT MAX(amount_orders) AS most_pizzas_in_order
FROM amount_per_order;


```

### Output 

|most_pizzas_in_order|
|------|
|3|

### Explanation

Create a CTE which lists the counts of each customers order
Join onto the runner_orders table to be able to filter out canceled orders
Call CTE again to find the max number of pizzas in a given order. 

### Question 7: For each customer, how many delivered pizzas had at least 1 change and how many had no changes?

```SQL 



WITH is_change AS (
SELECT customer_id,
CASE 
	WHEN exclusions = '' AND extras = '' THEN 1
    ELSE 0
END AS no_changes,
CASE 
	WHEN exclusions != '' OR extras != '' THEN 1
    ELSE 0
END AS changes
FROM customer_orders_cleaned
JOIN runner_orders_cleaned ON customer_orders_cleaned.order_id = runner_orders_cleaned.order_id
WHERE cancellation IS NULL
)

SELECT customer_id,
SUM(no_changes) AS amount_without_changes,
SUM(changes) AS amount_with_changes
FROM is_change
GROUP BY customer_id
ORDER BY customer_id;


```

### Output

| Customer ID | Amount without Changes | Amount with Changes |
|-------------|-----------------------|--------------------|
| 101         | 2                     | 0                  |
| 102         | 3                     | 0                  |
| 103         | 0                     | 3                  |
| 104         | 1                     | 2                  |
| 105         | 0                     | 1                  |

### Explanation

Create a CTE which generates a table showing which orders have changes and no changes using CASE statements.
Call the CTE and run SUM grouping by customer ID to show which customers had an amount of orders with or without changes. 
NOTE: CTE is not required here but helps to clean the code up, having the SUM statements on top of the CASE ruins readability in my opinion.  

### Question 8: How many pizzas were delivered that had both exclusions and extras?

```SQL
SELECT
SUM(CASE 
	WHEN exclusions = "" OR extras = "" THEN 0
    ELSE 1
    END) AS Amount_exclusions_Extras 
FROM customer_orders_cleaned
JOIN runner_orders_cleaned ON customer_orders_cleaned.order_id = runner_orders_cleaned.order_id
WHERE cancellation IS NULL;

```

### Output

|Amount_exclusions_Extras|
|------|
|1|

### Explanation

Join customer_orders onto runner_orders, WHERE cancellation IS NULL to find the orders that actually got delivered. 
Set up case statement to determine which orders have both extras and exclusions
Add SUM to case statement to find amount of orders with extras and exclusions

### Question 9: What was the total volume of pizzas ordered for each hour of the day?

```SQL

SELECT HOUR(order_time) as hours, 
COUNT(HOUR(order_time)) AS orders_in_hour
FROM customer_orders_cleaned
GROUP BY hours
ORDER BY hours;

```

### Output

| Hour | Orders in Hour |
|------|----------------|
| 11   | 1              |
| 13   | 3              |
| 18   | 3              |
| 19   | 1              |
| 21   | 3              |
| 23   | 3              |

### Explanation 
Use hours function to return the hour of each order
Do this again but count the hours GROUPING BY hours to show volume per hour


### Question 10: What was the volume of orders for each day of the week?


```SQL 
SELECT DAYNAME(order_time) as days, 
COUNT(DAYNAME(order_time)) AS orders_in_weekday
FROM customer_orders_cleaned
GROUP BY days
ORDER BY orders_in_weekday;

```

### Output

| Day       | Orders in Weekday |
|-----------|------------------|
| Friday    | 1                |
| Thursday  | 3                |
| Wednesday | 5                |
| Saturday  | 5                |


### Explanation
This solution is exactly the same switching HOURS function for DAYNAME function; these functions are all found under MySQL Dev docs. 



## B. Runner and Customer Experience 

### Question 1: How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)

```SQL
SELECT 
WEEK(pickup_time) AS weeks,
COUNT(DISTINCT runner_id) AS amount_of_runners
FROM runner_orders_cleaned
WHERE cancellation IS NULL
GROUP BY weeks;

```

### Output

| Weeks       | Amount_of_runners |
|-----------|------------------|
|0    | 2                |
| 1  | 3                |


### Explanation
Use the week function to return the value of which week the runner is signed up
Use COUNT DISTINCT and group by weeks to show the amount of runners signed up per week.

### Question 2: What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pick up the order?

```SQL 
WITH time_diff AS (
SELECT 
runner_id,
TIMEDIFF(pickup_time, order_time) AS diff
FROM runner_orders_cleaned AS r 
JOIN customer_orders_cleaned AS c on r.order_id = c.order_id
WHERE cancellation IS NULL
)

SELECT runner_id as runner,
(AVG(diff)/60) AS avg_time_to_pickup
FROM time_diff
GROUP BY runner;

```

### Output

| Runner | Avg Time to Pickup |
|--------|-------------------|
| 1      | 25.90000000       |
| 2      | 39.32000000       |
| 3      | 17.13333333       |


### Explanation
Create a CTE that uses the TIMEDIFF function to return the time taken to pick up an order.
SELECT the AVG time difference from the CTE grouping by runner_id
In this case the avg defaulted to seconds, dividing the value by 60 will reveal minutes.

### Question 3: Is there any relationship between the number of pizzas and how long the order takes to prepare?

```SQL

WITH pizzas AS (
SELECT 
order_time,
pickup_time, 
COUNT(c.order_id) AS pizzas_in_order,
TIMEDIFF(pickup_time, order_time) AS diff
FROM customer_orders_cleaned AS c
JOIN runner_orders_cleaned AS r ON c.order_id = r.order_id
WHERE cancellation IS NULL
GROUP BY order_time, pickup_time	
)

SELECT 
pizzas_in_order,
(AVG(diff)/60) AS time_to_prepare
FROM pizzas
GROUP BY pizzas_in_order;

``` 

### Output

| Pizzas in Order | Avg. Time to Prepare |
|-----------------|---------------------|
| 1               | 20.35666667         |
| 2               | 30.37500000         |
| 3               | 48.61666667         |


### Explanation
COUNT order_id GROUPING BY both order_time and pickup_time. This returns a table with the amount of pizzas in the order
Use TIMEDIFF on order_time and pick_up time to return the time to prepare the order
Turn that table into a CTE and call it using the AVG function to find the average time to prepare for the amount of pizzas
As shown, the more pizzas in an order the longer the prep time takes.

### Question 4. What was the average distance traveled for each customer?

```SQL 

SELECT 
customer_id as Customer,
AVG(distance) AS avg_distance
FROM runner_orders_cleaned AS r 
JOIN customer_orders_cleaned AS c ON r.order_id = c.order_id
WHERE cancellation IS NULL
GROUP BY customer_id;

```
### Output

| Customer | Avg Distance |
|----------|--------------|
| 101      | 20.0000      |
| 102      | 16.3333      |
| 103      | 23.0000      |
| 104      | 10.0000      |
| 105      | 25.0000      |


### Explanation
JOIN customer_order to runner_orders to access both customer_id and distance attributes
Use AVG function on distance and group by customer_id 
USE WHERE to filter out the canceled orders. 


### Question 5: What was the difference between the longest and shortest delivery times for all orders?

```SQL 
SELECT 
MAX(duration)-MIN(duration) as time_dif
FROM runner_orders_cleaned
WHERE cancellation IS NULL;

```

### Output 

| time_diff | 
|----------|
| 30     |

### Explanation

Using MAX and MIN function and subtract those values in a single select statement
Add the normal WHERE clause to extract non canceled orders


### Question 6: What was the average speed for each runner for each delivery and do you notice any trend for these values?

```SQL 
SELECT 
runner_id,
order_id,
AVG(distance/(duration/60)) AS speed,
HOUR(pickup_time) as times
FROM runner_orders_cleaned
WHERE cancellation IS NULL
GROUP BY order_id, runner_id, pickup_time
ORDER BY times;

```

### Output


| Runner ID | Order ID | Speed        | Times |
|-----------|----------|--------------|-------|
| 1         | 3        | 39.00000000  | 0     |
| 2         | 8        | 92.00000000  | 0     |
| 2         | 4        | 34.50000000  | 13    |
| 1         | 1        | 37.50000000  | 18    |
| 1         | 10       | 60.00000000  | 18    |
| 1         | 2        | 44.44440000  | 19    |
| 3         | 5        | 40.00000000  | 21    |
| 2         | 7        | 60.00000000  | 21    |




### Explanation
Speed is equal to distance/time, converting time to hours so the units are more readable.
Take the average of that speed and group by runner_id and order_id
I was curious to explore if the time of day had any impact on the average speed of the drivers. I used HOUR and ORDERED it to see.
Doing this you can see a loose correlation that the drivers drive faster at night then they do midday, 
 

### Question 7: What is the successful delivery percentage for each runner?

```SQL

SELECT runner_id,
(SUM(CASE 
	WHEN cancellation IS NULL THEN 1
    ELSE 0	END) / count(*)) * 100 AS perc
FROM runner_orders_cleaned
GROUP BY runner_id;

```

### Output

| Runner ID | Perc     |
|-----------|----------|
| 1         | 100.0000 |
| 2         | 75.0000  |
| 3         | 50.0000  |


### Explanation
Create a Sum case statement which adds up all orders that aren't canceled. 
Divide this by COUNT(*) and multiply it all by 100 to get percent.
GROUP BY runner_id to show per runner. 



