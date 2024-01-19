# 8-week-sql-challenge
8 week sql challenge is a set of 8 different types of databases . solution of challenge#3 Foodie-fi 
# 🥑 Case Study #3: Foodie-Fi

<img src="https://user-images.githubusercontent.com/81607668/129742132-8e13c136-adf2-49c4-9866-dec6be0d30f0.png" width="500" height="520" alt="image">

## 📚 Table of Contents
- [Business Task](#business-task)
- [Entity Relationship Diagram](#entity-relationship-diagram)
- [Question and Solution](#question-and-solution)

Please note that all the information regarding the case study has been sourced from the following link: [here](https://8weeksqlchallenge.com/case-study-3/). 

***

## Business Task
Danny and his friends launched a new startup Foodie-Fi and started selling monthly and annual subscriptions, giving their customers unlimited on-demand access to exclusive food videos from around the world.

This case study focuses on using subscription style digital data to answer important business questions on customer journey, payments, and business performances.

## Entity Relationship Diagram

![image](https://user-images.githubusercontent.com/81607668/129744449-37b3229b-80b2-4cce-b8e0-707d7f48dcec.png)

**Table 1: `plans`**

<img width="207" alt="image" src="https://user-images.githubusercontent.com/81607668/135704535-a82fdd2f-036a-443b-b1da-984178166f95.png">

There are 5 customer plans.

- Trial — Customer sign up to an initial 7 day free trial and will automatically continue with the pro monthly subscription plan unless they cancel, downgrade to basic or upgrade to an annual pro plan at any point during the trial.
- Basic plan — Customers have limited access and can only stream their videos and is only available monthly at $9.90.
- Pro plan — Customers have no watch time limits and are able to download videos for offline viewing. Pro plans start at $19.90 a month or $199 for an annual subscription.

When customers cancel their Foodie-Fi service — they will have a Churn plan record with a null price, but their plan will continue until the end of the billing period.

**Table 2: `subscriptions`**

<img width="245" alt="image" src="https://user-images.githubusercontent.com/81607668/135704564-30250dd9-6381-490a-82cf-d15e6290cf3a.png">

Customer subscriptions show the **exact date** where their specific `plan_id` starts.

If customers downgrade from a pro plan or cancel their subscription — the higher plan will remain in place until the period is over — the `start_date` in the subscriptions table will reflect the date that the actual plan changes.

When customers upgrade their account from a basic plan to a pro or annual pro plan — the higher plan will take effect straightaway.

When customers churn, they will keep their access until the end of their current billing period, but the start_date will be technically the day they decided to cancel their service.

***

## Question and Solution

If you have any questions, reach out to me on [LinkedIn](https://www.linkedin.com/in/katiehuangx/).


## B. Data Analysis Questions

### 1. How many customers has Foodie-Fi ever had?

To determine the count of unique customers for Foodie-Fi, I utilize the `COUNT()` function wrapped around `DISTINCT`.

```sql
SELECT COUNT(DISTINCT customer_id) AS num_of_customers
FROM foodie_fi.subscriptions;
```
- Foodie-Fi has 1,000 unique customers.

### 2. What is the monthly distribution of trial plan start_date values for our dataset - use the start of the month as the group by value

In other words, the question is asking for the monthly count of users on the trial plan subscription.
- To start, extract the numerical value of month from `start_date` column using the `month()` function, specifying the 'month' of a date. 
- Filter the results to retrieve only users with trial plan subscriptions (`plan_id = 0). 

```sql
SELECT
 select month(start_date) as month,monthname(start_date) as month_name,count(customer_id) as total_customers from subscriptions where plan_id =0 
group by month(start_date),monthname(start_date)
order by month(start_date);

```

**Answer:**  

Among all the months, March has the highest number of trial plans, while February has the lowest number of trial plans.

### 3. What plan start_date values occur after the year 2020 for our dataset? Show the breakdown by count of events for each plan_name.

 /*Question is asking to know the number of plans in jan 2021 and then group by number of plans */

CREATE VIEW event21 AS
    (SELECT 
        subscriptions.plan_id, plan_name, COUNT(*) AS event_21
    FROM
        subscriptions
            JOIN
        plans ON plans.plan_id = subscriptions.plan_id
    WHERE
        start_date >= '2021-01-01'
    GROUP BY plan_name , subscriptions.plan_id
    ORDER BY subscriptions.plan_id);

CREATE VIEW event20 AS
    (SELECT 
        subscriptions.plan_id, plan_name, COUNT(*) AS event_20
    FROM
        subscriptions
            JOIN
        plans ON plans.plan_id = subscriptions.plan_id
    WHERE
        start_date >= '2020-01-01'
    GROUP BY plan_name , subscriptions.plan_id
    ORDER BY subscriptions.plan_id
    );

### to compare data 0f 2020 and 2021
                                                                           
select event20.plan_id,event20.plan_name,event_20,ifnull(event_21,0) as ev21 from event20 
left join event21 on event21.plan_id =event20.plan_id;

### 4. What is the customer count and percentage of customers who have churned rounded to 1 decimal place
  
  Let's analyze the question:
- First, we need to determine
- The number of customers who have churned, meaning those who have discontinued their subscription.
- The total number of customers, including both active and churned ones.

- To calculate the churn rate, we divide the number of churned customers by the total number of customers. The result should be rounded to one decimal place.

```sql

   select 
          COUNT(DISTINCT sub.customer_id) AS churned_customers,
          round(100*count(*)/(select count(distinct customer_id) from subscriptions),1) 
as churned_percentage from subscriptions where plan_id =4;      -- Filter results to customers with churn plan only
```
**Answer:**

- Out of the total customer base of Foodie-Fi, 307 customers have churned. This represents approximately 30.7% of the overall customer count.

### 5. How many customers have churned straight after their initial free trial - what percentage is this rounded to the nearest whole number?

Within a CTE called `ranked_cte`, determine which customers churned immediately after the trial plan by utilizing `ROW_NUMBER()` function to assign rankings to each customer's plans. 

In this scenario, if a customer churned right after the trial plan, the plan rankings would appear as follows:
- Trial Plan - Rank 1
- Churned - Rank 2

In the outer query:
- Apply 2 conditions in the WHERE clause:
  - Filter `plan_id = 4`. 
  - Filter for customers who have churned immediately after their trial with `row_num = 2`.
- Count the number of customers who have churned immediately after their trial period using a `CASE` statement by checking if the row number is 2 (`row_num = 2`) and the plan name is 'churn' (`plan_name = 'churn'`). 
- Calculate the churn percentage by dividing the `churned_customers` count by the total count of distinct customer IDs in the `subscriptions` table. Round percentage to a whole number.

```sql
with ranked_cte as
(
select customer_id,plan_name, 
row_number() over( partition by customer_id order by start_date asc) as rn
from subscriptions s
inner join
plans p on 
p.plan_id =s.plan_id
)

select count(customer_id) as churned_customers_after_trial,
round((count(customer_id)/(select count(distinct customer_id) from subscriptions))*100,0) as churned_percentage  from ranked_cte 
where rn =2 and plan_name= 'churn';


- A total of 92 customers churned immediately after the initial free trial period, representing approximately 9% of the entire customer base.

### 6. What is the number and percentage of customer plans after their initial free trial?

```sql
with cte as
(
  select customer_id,s.plan_id,plan_name, row_number() over(partition by customer_id order by start_date) as rn
from subscriptions s
join plans p
on p.plan_id =s.plan_id
)
select plan_name,count(customer_id) as customers, 
round((count(customer_id)/(select count(distinct customer_id) from subscriptions))*100,1) as conversion_percentage
  from cte 
  where rn =2
  group by plan_name
  order by conversion_percentage;

**Answer:**
| plan_id | converted_customers | conversion_percentage |
| ------- | ------------------- | --------------------- |
| 1       | 546                 | 54.6                  |
| 2       | 325                 | 32.5                  |
| 3       | 37                  | 3.7                   |
| 4       | 92                  | 9.2                   |

- More than 80% of Foodie-Fi's customers are on paid plans with a majority opting for Plans 1 and 2. 
- There is potential for improvement in customer acquisition for Plan 3 as only a small percentage of customers are choosing this higher-priced plan.

### 7. What is the customer count and percentage breakdown of all 5 plan_name values at 2020-12-31?

In the cte called `next_dates`, we begin by filtering the results to include only the plans with start dates on or before '2020-12-31'. To identify the next start date for each plan, we utilize the `LEAD()` window function.

In the outer query,  we filter the results where the `next_date` is NULL. This step helps us identify the most recent plan that each customer subscribed to as of '2020-12-31'. 

Lastly, we perform calculations to determine the total count of customers and the percentage of customers associated with each trial plan. 

```sql
                                                                           
with cte as
(
 select *,row_number() over(partition by customer_id order by start_date desc) as rn from subscriptions 
where start_date <='2020-12-31'

)
select plan_name, count(customer_id),
round(count(customer_id)*100/(select count(distinct customer_id) from cte),1) as customersPercentage from cte 
inner join plans
on plans.plan_id=cte.plan_id
where rn =1
 group by plan_name;
 
**Answer:**
### 8. How many customers have upgraded to an annual plan in 2020?

```sql
SELECT COUNT(DISTINCT customer_id) AS num_of_customers
FROM subscriptions
WHERE plan_id = 3
  AND start_date <= '2020-12-31';
```

**Answer:**
- 196 customers have upgraded to an annual plan in 2020.

### 9. How many days on average does it take for a customer to upgrade to an annual plan from the day they join Foodie-Fi?

This question is straightforward and the query provided is self-explanatory. 

````sql
WITH trial_plan AS (
-- trial_plan CTE: Filter results to include only the customers subscribed to the trial plan.
  SELECT 
    customer_id, 
    start_date AS trial_date
  FROM subscriptions
  WHERE plan_id = 0
), annual_plan AS (
-- annual_plan CTE: Filter results to only include the customers subscribed to the pro annual plan.
  SELECT 
    customer_id, 
    start_date AS annual_date
  FROM subscriptions
  WHERE plan_id = 3
)
-- Find the average of the differences between the start date of a trial plan and a pro annual plan.
SELECT 
  ROUND(
    AVG(
      annual.annual_date - trial.trial_date)
  ,0) AS avg_days_to_upgrade
FROM trial_plan AS trial
JOIN annual_plan AS annual
  ON trial.customer_id = annual.customer_id;
````

**Answer:**

- On average, customers take approximately 105 days from the day they join Foodie-Fi to upgrade to an annual plan.

### 10. Can you further breakdown this average value into 30 day periods (i.e. 0-30 days, 31-60 days etc)

To understand how the `WIDTH_BUCKET()` function works in creating buckets of 30-day periods, you can refer to this [StackOverflow](https://stackoverflow.com/questions/50518548/creating-a-bin-column-in-postgres-to-check-an-integer-and-return-a-string) answer.

```sql
with trial as
(
 select customer_id,start_date as trial_date from subscriptions 
 where plan_id =0
),

annual as
(
 select customer_id,start_date as annual_date from subscriptions 
 where plan_id =3
)

select  
 case 
 when datediff(annual_date,trial_date)<=30 then '0-30'
 when datediff(annual_date,trial_date)<=60 then '31-60'
 when datediff(annual_date,trial_date)<=90 then '61-90'
 when datediff(annual_date,trial_date)<=120 then '91-120'
 when datediff(annual_date,trial_date)<=150 then '121-150'
 when datediff(annual_date,trial_date)<=180 then '151-180'
 when datediff(annual_date,trial_date)<=210 then '181-210'
 when datediff(annual_date,trial_date)<=240 then '211-240'
 when datediff(annual_date,trial_date)<=270 then '241-270'
 when datediff(annual_date,trial_date)<=300 then '271-300'
 when datediff(annual_date,trial_date)<=330 then '301-330'
 when datediff(annual_date,trial_date)<=360 then '331-360'
 
end as breakdown,count(trial.customer_id) as customers
from trial 
join annual on annual.customer_id = trial.customer_id 
group by 1;


**Answer:**

| bucket         | num_of_customers |
| -------------- | ---------------- |
| 0 - 30 days    | 49               |
| 30 - 60 days   | 24               |
| 60 - 90 days   | 35               |
| 90 - 120 days  | 35               |
| 120 - 150 days | 43               |
| 150 - 180 days | 37               |
| 180 - 210 days | 24               |
| 210 - 240 days | 4                |
| 240 - 270 days | 4                |
| 270 - 300 days | 1                |
| 300 - 330 days | 1                |
| 330 - 360 days | 1                |

### 11. How many customers downgraded from a pro monthly to a basic monthly plan in 2020?

```sql
with pro_monthly as 
(
 select customer_id,start_date as promonthly_date from subscriptions 
 where plan_id =2
),

basic_monthly as 
(
 select customer_id,start_date as basicmonthly_date from subscriptions 
 where plan_id =1
)
select p.customer_id, promonthly_date,basicmonthly_date
from pro_monthly as p
join basic_monthly as b
on p.customer_id =b.customer_id
where promonthly_date<basicmonthly_date and year(basicmonthly_date) ='2020';

**Answer:**
In 2020, there were no instances where customers downgraded from a pro monthly plan to a basic monthly plan.
***

