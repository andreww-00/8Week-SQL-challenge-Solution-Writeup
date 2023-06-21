## Introduction and Problem Statement:  
(Summaries of introduction and Problem from [Case Study # 1](https://8weeksqlchallenge.com/case-study-1/))

- Danny opened a restaurant that sells sushi, curry, and ramen. Danny has collected some basic data over the past two months and needs help developing data-driven insights.

- Danny wants to answer questions like 'what are his customers visiting patterns?' or 'How much money they've spent at the restaurant?' to help him deliver a better experience to his customers.

- His main goal is to use these insights to determine if he should expand his current customer loyalty program.

- Danny has given me some sample data about his customers to work with.

## Data:
The data I have been given is in 3 tables and encapsulates simple aspects of the customers. These datasets are as follows.

- Sales: Shows which customer bought what on which day.
- Menu: Maps the product ID to their menu name and price.
- Members: Shows which customer has joined the rewards program on which day.

  
# Solution Write-up
(All questions are answered in MySQL)

## Question #1: **What is the total amount each customer spent at the restaurant?**



```SQL

SELECT sales.customer_id as Customer, SUM(menu.price) as Total_Spent
FROM sales 
JOIN menu ON sales.product_id = menu.product_id
GROUP BY customer_id;

```
### Output:
 
|Customer | Total_Spent|
|---------|------------|
|A        | 76         |
|B        | 74         |
|C        | 36         |

### Explanation: 

1.	Inner Join product_id between sales and menu table to show what each customer spent on a given day
2.	Using the aggregate sum followed by grouping customer_id we can see exactly how much each customer spent.


## Question #2: **How many days has each customer visited the restaurant?**



``` SQL
SELECT sales.customer_id as Customer, COUNT(DISTINCT sales.order_date) as Num_Visits
FROM sales
GROUP BY sales.customer_id;

```

### Output:

|Customer | Num_Visits |
|---------|------------|
|A        | 4          |
|B        | 6          |
|C        | 2          |


### Explanation:

1.	Use COUNT aggregate function to find the number of visits and group by customer_id
2.	This appears to be the solution but fails to account for the customers that visit multiple time per day
3.	Add distinct too the count function to count unique visits to fix this.


## Question #3: **What was the first item from the menu purchased by each customer?**



This problem revealed some serious gaps in my knowledge. My first attempt was to find the minimum dates of order by customer. But I quickly realized this doesn't return half of what it needs to and fails to recognize that two orders were placed on the minimum day for Customer A.

``` SQL

SELECT sales.customer_id, MIN(sales.order_date) as First_Sale
FROM sales
GROUP BY sales.customer_id;

```
|customer_id | First_Sale |
|---------|------------|
|A        | 2021-01-01          |
|B        | 2021-01-01        |
|C        | 2021-01-01        |


After a bit of brainstorming and research I remembered subqueries/WITH/CTE statements and implanted a more thorough approach, this returns a correct answer, but runs on a LIMIT of 4 which only works if the results are physically countable (AKA brute force).

``` SQL

WITH f_sale  as (SELECT sales.product_id, sales.customer_id, MIN(sales.order_date) as First_Sale
FROM sales
GROUP BY sales.customer_id, sales.product_id
ORDER BY First_sale
LIMIT 4
)

SELECT f_sale.customer_id as Customer, menu.product_name as First_Order
FROM f_sale
JOIN menu ON f_sale.product_id = menu.product_id;

```
|Customer | First_Sale |
|---------|------------|
|A        | sushi         |
|A        | curry          |
|B        | curry        |
|C        |ramen         |



``` SQL

WITH f_sale  as (SELECT sales.product_id, 
sales.customer_id, 
DENSE_RANK() OVER(PARTITION BY sales.customer_id ORDER BY order_date) AS score,
sales.order_date
FROM sales
ORDER BY score
)

SELECT DISTINCT f_sale.customer_id as Customer, menu.product_name as First_Order
FROM f_sale
JOIN menu ON f_sale.product_id = menu.product_id
WHERE score = 1;
```

### Output:

|Customer | First_Order |
|---------|------------|
|A        | sushi         |
|A        | curry          |
|B        | curry        |
|C        |ramen         |

After a bit more research I stumbled on RANK and DENSERANK, which return a ranked list based on parameters set. USING DENSERANK for this problem would solve the problem of customer A needing two different meals.

### Explanation:
1. 	Create a CTE which uses a DENSE_RANK() windows function to assign a rank to the order date partitioned by the customer_id. Effectively storing the first order of each customer as 1
2.	Then calling from the CTE to return all values where the rank is 1 to show only the first order

## Question #4: **What is the most purchased item on the menu and how many times was it purchased by all customers?**



```SQL

SELECT menu.product_name as Food, COUNT(sales.product_id) as Amount
FROM sales
JOIN menu on sales.product_id = menu.product_id
GROUP BY menu.product_name
ORDER BY Amount DESC
Limit 1;

```

### Output:

|Food | Amount|
|---------|------------|
|Ramen       | 8          |

### Explanation: 

1.	Find the amount of product sold, using the COUNT function. 
2.	Join this result to the menu table using product_id and group by product's name. This will give you a list of the amount of each product sold.
3.	Display the product with the highest sales at the top using the ORDER BY clause with the DESC keyword. 
4.	Use LIMIT of 1 to only show the highest sold item


## Question #5: **Which item was the most popular for each customer?**



```SQL

WITH favs as (
SELECT 
menu.product_name,
sales.customer_id,
COUNT(sales.product_id) as Amount_Sold,
DENSE_RANK() OVER(PARTITION BY sales.customer_id ORDER BY COUNT(sales.customer_id) DESC) as SCORE
FROM sales
JOIN menu ON sales.product_id = menu.product_id 
GROUP BY sales.customer_id, menu.product_name
ORDER BY SCORE
)

SELECT 
favs.customer_id as Customer, 
favs.product_name as Fav_Food, 
favs.Amount_Sold
FROM favs
WHERE SCORE = 1
ORDER BY customer_id;

```

### Output:

|Customer | Fav_Food | Amount_Sold |
|---------|------------|---------------|
|A        | ramen        | 3 |
|B       | curry          |2 |
|B        | sushi      |2 |
|B        | ramen     |2 | 
|C        |ramen         |3 |

### Explanation: 

This was another difficult problem for me. 
1.	Create a CTE which DENSE_RANKS each customer based on the COUNT of their ID making sure to use DESC. 
2.	This CTE is then referenced where the ranks are 1 showing the customers favorite products


## Question #6: Which item was purchased first by the customer after they became a member?

An important assumption here is that the customer signed up before ordering on the day they signed up, meaning the date that they signed up is the date they ordered after!

```SQL

WITH ranked_orders AS (
SELECT sales.customer_id,
sales.order_date as Order_Date,
menu.product_name,
members.join_date as Join_date,
RANK() OVER(PARTITION BY sales.customer_id ORDER BY sales.Order_Date) AS score
FROM sales 
JOIN members ON sales.customer_id = members.customer_id
JOIN menu ON sales.product_id = menu.product_id 
WHERE Order_date >= Join_date
ORDER BY customer_id
)

SELECT 
ranked_orders.customer_id AS Customer,
ranked_orders.product_name AS Product_After_Membership
FROM ranked_orders
WHERE SCORE = 1;

```

### Output:

|Customer | Product_After_Membership|
|---------|------------|
|A     | curry         |
|B    | sushi         |

### Explanation: 

1.	Create CTE which ranks the dates ordered after OR equal to their sign-up date. 
2.	This is then selected where the rank is the highest, returning their first order after signing up.
3.	(DENSE_RANK and RANK can be used interchangeably here)


## Question #7: **Which item was purchased just before the customer became a member?**

The assumption applies here as well, and if two orders happen in the same day, both are counted due to not knowing specifically which was first.

```SQL

WITH favs AS (
SELECT sales.customer_id,
sales.order_date as Order_Date,
menu.product_name,
members.join_date as Join_date,
RANK() OVER(PARTITION BY sales.customer_id ORDER BY sales.Order_Date DESC) AS score
FROM sales 
JOIN members ON sales.customer_id = members.customer_id
JOIN menu ON sales.product_id = menu.product_id 
WHERE Order_date < Join_date
ORDER BY customer_id
)

SELECT 
favs.customer_id AS Customer,
favs.product_name AS Order_Before
FROM favs
WHERE score = 1;

```

### Output:

|Customer | Order_Before|
|---------|------------|
|A     |sushi        |
|A     | curry         |
|B    | sushi         |

### Explanation:

This is effectively the same as question 6, except the inequality is changed to be less than the join date, and the RANK is done from DESC date.

# Question #8: **What is the total items and amount spent for each member before they became a member?**

```SQL

SELECT sales.customer_id AS Customer,
COUNT(sales.order_date) AS Total_Sales_Before,
SUM(menu.price) AS Amount_Spent_Before
FROM sales
JOIN menu on sales.product_id = menu.product_id
JOIN members on sales.customer_id = members.customer_id
WHERE sales.order_date < members.join_date
GROUP BY Customer
ORDER BY Customer; 

```
### Output:

|Customer | Total_Sales | Amount_Spent |
|---------|------------|---------------|
|A        | 2       | 25
|B       | 3         |40

### Explanation:

1. Find the COUNT and SUM and group them by the customer
2. Set WHERE statement so that the order date is before they became members
3. JOIN sales on menu and again on members 



## Question #9: **If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?**

```SQL

WITH item_points AS (
SELECT s.customer_id, 
m.product_name,
m.price,
CASE
	WHEN m.product_name = 'sushi' THEN m.price * 20
	ELSE m.price * 10
END AS points
FROM sales AS s 
JOIN menu AS m on s.product_id = m.product_id
)

SELECT item_points.customer_id AS Customer,
SUM(item_points.points) AS Total_Points
FROM item_points
GROUP BY item_points.customer_id;

```

### Output:

|Customer | Total Points|
|---------|------------|
|A     |860       |
|B    | 940       |
|C   | 360     |

### Explanation:


1.	Create a CTE which selects a column for price using a CASE/WHEN/ELSE to account for multipliers and points
2.	Reference this CTE, SUM points and group by customer 


## Question #10: **In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?**
Important note on original read through of this question I assumed that sushi would not get the 2x multiplier after the week.
```SQL

WITH item_points AS (
SELECT s.customer_id,
m.price,
CASE 
	WHEN ABS(s.order_date - mem.join_date) <= 7 THEN m.price * 20 
    ELSE m.price * 10
END as Points,
s.order_date,
mem.join_date
FROM sales AS s
JOIN menu AS m on s.product_id = m.product_id
JOIN members AS mem ON s.customer_id = mem.customer_id
WHERE s.order_date > 2021-02-01
)

SELECT item_points.customer_id AS Customer,
SUM(item_points.Points)
FROM item_points 
GROUP BY Customer
ORDER BY Customer;

```

### Output:

|Customer | Points|
|---------|------------|
|A     | 1520      |
|B    | 1210      |

### Explanation:

1.	Create a CTE which selects a column for price using a CASE/WHEN/ELSE to account for multipliers and points and conditions in which these points apply
2.	Reference this CTE, SUM points and group by customer 

