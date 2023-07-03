# Foodie Fi write up 

## Introduction and Problem Statement:  
(Summaries of introduction and Problem from [Case Study # 3](https://8weeksqlchallenge.com/case-study-3/))

- Danny created a streaming service exclusively for food content
- Danny wants to use his data to grow his business by keeping track of his subscription model

## Data:
Danny supplied 2 different tables. The tables correspond to plan information, and when customers joined the plans 


  
# Solution Write-up
(All questions are answered in MySQL)

## A: Customer Journey:
Below is a combined version of the two data sets to easily describe the journey of each of the customers. I will only be covering a few of the customer journeys as most of them follow a similar pattern. 

| Customer ID | Plan Name         | Start Date   |
|-------------|-------------------|--------------|
| 1           | trial             | 2020-08-01   |
| 1           | basic monthly     | 2020-08-08   |
| 2           | trial             | 2020-09-20   |
| 2           | pro annual        | 2020-09-27   |
| 11          | trial             | 2020-11-19   |
| 11          | churn             | 2020-11-26   |
| 13          | trial             | 2020-12-15   |
| 13          | basic monthly     | 2020-12-22   |
| 13          | pro monthly       | 2021-03-29   |
| 15          | trial             | 2020-03-17   |
| 15          | pro monthly       | 2020-03-24   |
| 15          | churn             | 2020-04-29   |
| 16          | trial             | 2020-05-31   |
| 16          | basic monthly     | 2020-06-07   |
| 16          | pro annual        | 2020-10-21   |
| 18          | trial             | 2020-07-06   |
| 18          | pro monthly       | 2020-07-13   |
| 19          | trial             | 2020-06-22   |
| 19          | pro monthly       | 2020-06-29   |
| 19          | pro annual        | 2020-08-29   |

- Customer 1 started the free trial and after the week upgraded to the monthly basic plan
- Customer 2 did the same but bought the pro plan
- Customer 11 started the free trial and canceled their plan when it ended

## B. Data Analysis

### Question 1: How many customers has Foodie-Fi ever had?


```SQL 
 SELECT COUNT(DISTINCT customer_id) AS Unique_customers
  FROM subscriptions; 
```

### Output

| Unique Customers |
|-----------------|
| 1000            |


### Explanation
Use COUNT and DISTINCT to find the value of unique customers 

### Question 2: What is the monthly distribution of trial plan start_date values for our dataset - use the start of the month as the group by value


```SQL 
 SELECT  monthname(start_date) AS months, 
  COUNT(customer_id) as amount
  FROM subscriptions 
  WHERE plan_id = 0
  GROUP BY months
  ORDER BY amount;

```

### Output

| Month     | Count |
|-----------|-------|
| February  | 68    |
| November  | 75    |
| June      | 79    |
| October   | 79    |
| April     | 81    |
| December  | 84    |
| September | 87    |
| August    | 88    |
| January   | 88    |
| May       | 88    |
| July      | 89    |
| March     | 94    |

### Explanation
1. Use WHERE To filter the data to only the ones with the starting plan
2. Extract month using monthname() function 
3. Count customers and group by the month name


### Question 3: What plan start_date values occur after the year 2020 for our dataset? Show the breakdown by count of events for each plan_name


```SQL
SELECT COUNT(customer_id) AS events_after2020, 
  plan_name
  FROM subscriptions AS s 
  JOIN plans AS p ON s.plan_id = p.plan_id
  WHERE start_date >= '2021-01-01'
  GROUP BY plan_name
  ORDER BY events_after2020;

```

### Output 

| Events After 2020 | Plan Name      |
|------------------ |------------------|
| 8                 | basic monthly    |
| 60                | pro monthly      |
| 63                | pro annual       |
| 71                | churn            |


### Explanation
1. Filter data for start_dates greater than 2020 
2. COUNT customer_id and group by the plan_name


### Question 4: What is the customer count and percentage of customers who have churned rounded to 1 decimal place?

	
```SQL 
SELECT COUNT(DISTINCT customer_id) AS churn_customers,
ROUND(COUNT(customer_id) / (SELECT COUNT(DISTINCT customer_id) FROM subscriptions) * 100, 1) AS churn_perc	
FROM subscriptions AS s 
JOIN plans AS p ON s.plan_id = p.plan_id
WHERE plan_name = 'churn';

```

### Output


| Churn Customers | Churn Percentage |
|-----------------|-----------------|
| 307             | 30.7           |



### Explanation
1. Find COUNT of customers that churned 
2. Find percentage by dividing the regular count by a subquery of the total amount of customers
3. Make sure to round this to 1 decimal point using the ROUND() function.

### Question 5: How many customers have churned straight after their initial free trial - what percentage is this rounded to the nearest whole number?


```SQL 
WITH ranked AS (SELECT customer_id, 
plan_name,
 start_date,
DENSE_RANK() OVER (PARTITION BY customer_id  ORDER BY start_date ) AS ranks	
FROM subscriptions AS s
JOIN plans AS p ON s.plan_id = p.plan_id
)

SELECT
COUNT(customer_id) AS amount_churned,
ROUND(COUNT(customer_id) / (SELECT COUNT(DISTINCT customer_id) FROM subscriptions) * 100) AS perc_churned_after_trial
FROM ranked
WHERE plan_name = 'churn' AND ranks = 2;

  
```
### Output

| Amount Churned | Perc Churned After Trial |
|---------------|-------------------------|
| 92            | 9                       |

### Explanation

1. Create a CTE which uses DENSE_RANK to rank the order in which their plan history took place. The logic here being that the trail will get a value of 1, and if they churn directly after they will get a rank of 2. 
2. Then in the main query call the CTE WHERE the plan is trial and has a rank of 2, this shows only the customers that churned after the trial
3. COUNT the number of these, and then do the same as the question above to find percent. 

### Question 6: What is the number and percentage of customer plans after their initial free trial?


``` SQL 
WITH ranked as (SELECT customer_id,
plan_name,
start_date,
s.plan_id,
DENSE_RANK() OVER (PARTITION BY customer_id ORDER BY start_date) AS ranks
FROM subscriptions AS s
JOIN plans AS p on s.plan_id = p.plan_id
)

SELECT 
plan_id,
plan_name,
COUNT(DISTINCT customer_id) AS customers_after_after,
ROUND(COUNT(DISTINCT customer_id) / (SELECT COUNT(DISTINCT customer_id) FROM subscriptions) * 100) as perc
FROM ranked 
WHERE ranks = 2
GROUP BY plan_name, plan_id
ORDER BY plan_id

```

### Output 


| Plan ID | Plan Name       | Amount of Cust | Perc Customers |
|---------|-----------------|----------------|----------------|
| 1       | basic monthly   | 546            | 55            |
| 2       | pro monthly     | 325            | 33           |
| 3       | pro annual      | 37             | 4             |
| 4       | churn           | 92             | 9             |



### Explanation
1. Similar approach as above, create a CTE that ranks the values of the plans
2. SELECT from the CTE with a where statement of 2 which selects the rank of the customer right after their trial
3. Same COUNT and percentage calculation as above

### Question 7: What is the customer count and percentage breakdown of all 5 plan_name values at 2020-12-31?

``` SQL 
WITH cte AS (SELECT 
s.plan_id,
customer_id,
plan_name,
start_date,
LEAD(start_date) OVER (PARTITION BY customer_id ORDER BY start_date) AS date_next
FROM subscriptions AS s
JOIN plans AS p ON s.plan_id = p.plan_id
WHERE start_date <= '2020-12-31'
)

SELECT
plan_id,
plan_name,
COUNT(customer_id) AS amount,
ROUND(COUNT(DISTINCT customer_id) / (SELECT COUNT( DISTINCT customer_id) FROM subscriptions) * 100) AS perc
FROM cte 
WHERE date_next IS NULL
GROUP BY plan_name, plan_id
ORDER BY plan_id;

```

### Output 


| Plan ID | Plan Name        | Amount | Perc |
|---------|------------------|--------|------|
| 0       | trial            | 19     | 2    |
| 1       | basic monthly    | 224    | 22   |
| 2       | pro monthly      | 326    | 33   |
| 3       | pro annual       | 195    | 20   |
| 4       | churn            | 236    | 24   |

### Explanation
1. Create a CTE which uses LEAD() function to show the next start date partitioned by customer_id
2. Call the CTE where the date_next is NULL as this shows the most recent update to the plan
3. Do the same select statements as above to find amount and percentage.

### Question 8:How many customers have upgraded to an annual plan in 2020?


``` SQL 
SELECT 
COUNT(*) AS customers_to_annual_plan
FROM subscriptions 
WHERE year(start_date) = 2020 AND plan_id = 3;

```

### Output
| customers_to_annual | 
|---------|
| 195       |

### Explanation 
1. SET where to have a year of 2020 and a plan_id of 3 
2. COUNT * these values 

### Question 9: How many days on average does it take for a customer to an annual plan from the day they join Foodie-Fi?


``` SQL 
WITH join_date AS (
SELECT start_date, customer_id
FROM subscriptions
WHERE plan_id = 0 
),
annual_date AS (
SELECT start_date AS date_of_upgrade, customer_id
FROM subscriptions
WHERE plan_id = 3
)

SELECT 
ROUND(AVG(DATEDIFF(date_of_upgrade,start_date))) AS avg_days_till_annual
FROM join_date as j
JOIN annual_date as a ON j.customer_id = a.customer_id

```

### Output

|avg_days_till_annual| 
|---------|
| 105      | 

### Explanation
- This problem gave me a little trouble, but it all came together once I realized you could utilize more than one CTE
1. Create two CTE’s one which tracks the trial and the other which tracks the annual pro plan
2. Join these two tables together on customer_id to show when each upgrades
3. Find the average of the difference in dates.


