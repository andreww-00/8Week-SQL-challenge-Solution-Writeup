# Solution Write up


Question 1:

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
