# Solution Write up


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

### Explaination: 

Utilizing the realationship between product_id shared between sales and menu the tables can be inner joined to show which customer spent what each day. Using the aggregate sum followed by a grouping of customer_id we can see exactly how much each customer spent.



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


### Explaination:

Unlike the last solution all the information we need is located in the sales table meaning no joins are needed. To get the number of visits we need to use the COUNT aggreagate function to find the amount of visits and group that by the customer_id. On first glance this appears to be the solution, but it fails to account for customers that visit multiple times a day. This is easily fixed by adding DISTINCT to the COUNT function, only counting unique visits. 


## Question #3: **What was the first item from the menu purchased by each customer?**



This problem revealed some serious gaps in my knowledge. My first attempt was to find the minumum dates of order by customer. But I quickly realized this doesn't return half of what it needs to, and fails to recoqnize that two orders were placed on the minimum day for Customer A. 

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


After a bit of brainstorming and research I remembered subqueries/WITH statements and implented a more thourough approach, this returns a correct answer, but runs on a LIMIT of 4 which only works if the results are physically countable (AKA brute force). 

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

Aftter a bit more research I stumbled on RANK and DENSERANK, which return a ranked list based on parameters set. USING DENSERANK for this problme would solve the problme of customer A needing two different meals. 

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


### Question #4: **What is the most purchased item on the menu and how many times was it purchased by all customers?**



```SQL

SELECT menu.product_name as Food, COUNT(sales.product_id) as Amount
FROM sales
JOIN menu on sales.product_id = menu.product_id
GROUP BY menu.product_name
ORDER BY Amount DESC
Limit 1;

```

## Output:

|Food | Amount|
|---------|------------|
|Ramen       | 8          |

### Explaination: 

Find the amount of product sold using count. Joining this to the menu on product_id allows you to group by the products name. This then shows a list of the amount of products sold. Ordering this so that the most sold is at the top using DESC and limit it to 1 result. 

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
|A        | ramen        | 3
|B       | curry          |2
|B        | sushi      |2
|B        | ramen     |2
|C        |ramen         |3

### Explaination: 

This was another difficult problem for me. The problem boils down to making a subquery which DENSE_RANKS each customer based on the amount of times their ID shows up in the list. Using DESC this means that a dense rank of 1 is a customers favorite food and is retruned along with the food and amunt of it sold to what customer.

## Question #6: Which item was purchased first by the customer after they became a member?
An important assumption here is that the customer signed up before ordering on the day they signed up. Meaning the date that they signed up is the date they ordered after!

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

## Output:

|Customer | Product_After_Membership|
|---------|------------|
|A     | curry         |
|B    | sushi         |

### Explaination: 

The same methodoogy applies to this problem. A subquery is created which ranks the date ordered after OR equal to their sign up date. This is then selected where the rank is the highest returning their first order after signign up

## Question #7: **Which item was purchased just before the customer became a member?**
The assumption applies here as well, and if two orderes happen in the same day both are counted due to not knowing specifically which was first

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

## Output:

|Customer | Order_Before|
|---------|------------|
|A     |sushi        |
|A     | curry         |
|B    | sushi         |

## Explaination:

This is effectively the same as question 6 except the inequality is changed to be less than the join date, and the RANk is done from DESC date.
