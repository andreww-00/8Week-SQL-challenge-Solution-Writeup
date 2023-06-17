# Solution Write up


### Question 1:

What is the total amount each customer spent at the restaurant?

```SQL

SELECT sales.customer_id as Customer, SUM(menu.price) as Total_Spent
FROM sales 
JOIN menu ON sales.product_id = menu.product_id
GROUP BY customer_id;

```
Output:

|Customer | Total_Spent|
|---------|------------|
|A        | 76         |
|B        | 74         |
|C        | 36         |

Explaination: 

Utilizing the realationship between product_id shared between sales and menu the tables can be inner joined to show which customer spent what each day. Using the aggregate sum followed by a grouping of customer_id we can see exactly how much each customer spent.



### Question #2:

How many days has each customer visited the restaurant?

``` SQL
SELECT sales.customer_id as Customer, COUNT(DISTINCT sales.order_date) as Num_Visits
FROM sales
GROUP BY sales.customer_id;

```

Output:

|Customer | Num_Visits |
|---------|------------|
|A        | 4          |
|B        | 6          |
|C        | 2          |


Explaination:

Unlike the last solution all the information we need is located in the sales table meaning no joins are needed. To get the number of visits we need to use the COUNT aggreagate function to find the amount of visits and group that by the customer_id. On first glance this appears to be the solution, but it fails to account for customers that visit multiple times a day. This is easily fixed by adding DISTINCT to the COUNT function, only counting unique visits. 
