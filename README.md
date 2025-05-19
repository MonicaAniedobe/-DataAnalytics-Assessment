# SQL Analysis Project: Customer Segmentation and Plan Engagement

## Overview

This project analyzes customer engagement with savings and investment plans using MySQL. The goal was to identify high-value customers who have funded both savings and investment plans  a key insight for cross-selling opportunities.

The project involved querying multiple related tables:

users_customuser (customer details)

savings_savingsaccount (savings transactions)

plans_plan 

 ## Assessment_Q1.sql : Identify customers with at least one funded savings plan and one funded investment plan

 ### Per-Question Explanation:

1️⃣ Selecting User Details

Selected id and name from users_customuser as basic identifiers for each customer.

2️⃣ Counting Number of Savings and Investment Plans

Used a SUM(CASE WHEN … THEN 1 ELSE 0 END) pattern to count how many transactions each customer has under a savings plan and an investment plan.

Applied LOWER() to standardize case in plan names.

3️⃣ Calculating Total Deposits

Used SUM(s.amount) to get total amount deposited per customer, but only for transactions with transaction_status = 'funded' (filtered in the WHERE clause).

4️⃣ Joining Related Tables

Joined users_customuser to savings_savingsaccount via owner_id.

Joined savings_savingsaccount to plans_plan via plan_id to access plan names (e.g., 'savings' or 'investment').

5️⃣ Grouping Results by Customer

Applied GROUP BY u.id, u.name to aggregate per customer.

6️⃣ Filtering Customers with At Least One of Each Plan Type

Used a HAVING clause to only include customers with at least one savings plan and at least one investment plan.

7️⃣ Ordering Results

Ordered final results by total_deposits in descending order to see highest depositors first.


## CODE:

SELECT 

    u.id AS owner_id,
    
    u.name,
    
    SUM(CASE WHEN LOWER(p.name) = 'savings' THEN 1 ELSE 0 END) AS savings_count,
    
    SUM(CASE WHEN LOWER(p.name) = 'investment' THEN 1 ELSE 0 END) AS investment_count,
    
    SUM(s.amount) AS total_deposits
    
FROM 

    users_customuser u
    
JOIN 

    savings_savingsaccount s ON u.id = s.owner_id
    
JOIN 

    plans_plan p ON s.plan_id = p.id
    
WHERE 

    s.transaction_status = 'funded'
    
GROUP BY 

    u.id, u.name
    
HAVING 

    SUM(CASE WHEN LOWER(p.name) = 'savings' THEN 1 ELSE 0 END) >= 1
    
    AND SUM(CASE WHEN LOWER(p.name) = 'investment' THEN 1 ELSE 0 END) >= 1
    
ORDER BY 

    total_deposits DESC;
    
## Challenges Encountered:

🛑 Schema Difference / Column Mapping Assumptions
Initially assumed the join would be via savings_savingsaccount.owner_id to users_customuser.id and savings_savingsaccount.plan_id to plans_plan.id.

### Resolution:
Confirmed the schema relationships via table definitions in the database navigator to ensure correct columns were joined.

## Assessment_Q2.sql : Calculate the average number of transactions per customer per month and categorize them:
### "High Frequency" (≥10 transactions/month)
### "Medium Frequency" (3-9 transactions/month)
### "Low Frequency" (≤2 transactions/month)

### Per-Question Explanation:

This query calculates the total number of funded transactions per customer.

It divides the total number of transactions by the number of distinct months those transactions occurred to get an average per month.

Based on this average, customers are categorized into High, Medium, or Low Frequency.

## CODE :

SELECT 

    frequency_category,
    COUNT(DISTINCT owner_id) AS customer_count,
    AVG(avg_txn_per_month) AS avg_transactions_per_month
FROM (

    SELECT 
        s.owner_id,
        COUNT(s.id) / COUNT(DISTINCT DATE_FORMAT(s.transaction_date, '%Y-%m')) AS avg_txn_per_month,
        
        CASE 
        
            WHEN COUNT(s.id) / COUNT(DISTINCT DATE_FORMAT(s.transaction_date, '%Y-%m')) >= 10 THEN 'High Frequency'
            WHEN COUNT(s.id) / COUNT(DISTINCT DATE_FORMAT(s.transaction_date, '%Y-%m')) BETWEEN 3 AND 9 THEN 'Medium Frequency'
            
            ELSE 'Low Frequency'
            
        END AS frequency_category
        
    FROM savings_savingsaccount s
    
    WHERE s.transaction_status = 'funded'
    
    GROUP BY s.owner_id
    
) AS txn_summary

GROUP BY frequency_category;

## Challanges :
The savings_savingsaccount table currently has no records with transaction_status = 'funded'.

As a result, the query returns no rows.

Solution: Confirmed the query syntax is correct by testing with mock data

## Assessment_Q3.sql : Find all active accounts (savings or investments) with no transactions in the last 1 year (365 days) .

## Approach:

LEFT JOIN is used to pair each plan with its corresponding transactions (if any).

MAX(s.transaction_date) finds the latest transaction date per account.

DATEDIFF(CURDATE(), MAX(s.transaction_date)) calculates inactivity in days.

The HAVING clause filters:

Accounts with no transactions (MAX(s.transaction_date) IS NULL)

Or accounts whose last transaction is older than 365 days

Grouped by plan ID and owner ID to ensure unique plan-account combinations.

## CODE :

SELECT 
    p.id AS plan_id,
    
    p.owner_id,
    
    MAX(s.transaction_date) AS last_transaction_date,
    
    DATEDIFF(CURDATE(), MAX(s.transaction_date)) AS inactivity_days
    
FROM plans_plan p

LEFT JOIN savings_savingsaccount s 

    ON s.savings_id = p.id
    
GROUP BY p.id, p.owner_id

HAVING 

    MAX(s.transaction_date) IS NULL
    
    OR DATEDIFF(CURDATE(), MAX(s.transaction_date)) > 365;

# Assessment_Q4.sql:  For each customer, assuming the profit_per_transaction is 0.1% of the transaction value, calculate:
Account tenure (months since signup)
Total transactions
Estimated CLV (Assume: CLV = (total_transactions / tenure) * 12 * avg_profit_per_transaction)
Order by estimated CLV from highest to lowest.

## 📌Approach :
TIMESTAMPDIFF(MONTH, u.date_joined, CURDATE()) → Calculates the number of months since the user joined.

COUNT(s.id) → Counts all transactions the user has made.

AVG(s.amount) → Computes the average transaction value.

The CLV formula scales average monthly transactions to yearly, multiplies it by 0.001 (0.1%) of the average transaction value, and rounds it to 2 decimal places.

LEFT JOIN is used to ensure users with no transactions are still included (with total transactions as 0).

HAVING tenure_months > 0 protects against division by zero when calculating average transactions per month.

ORDER BY estimated_clv DESC ranks customers by their projected value to the business.

## QUERY :

SELECT 
    u.id AS customer_id,
    
    u.name,
    
    TIMESTAMPDIFF(MONTH, u.date_joined, CURDATE()) AS tenure_months,
    
    COUNT(s.id) AS total_transactions,
    
    ROUND(
        (COUNT(s.id) / TIMESTAMPDIFF(MONTH, u.date_joined, CURDATE())) * 12 
        * (0.001 * AVG(s.amount)),
        
        2
    ) AS estimated_clv
    
FROM users_customuser u

LEFT JOIN savings_savingsaccount s 

    ON s.savings_id = u.id
    
GROUP BY u.id, u.name

HAVING tenure_months > 0 -- avoid division by zero

ORDER BY estimated_clv DESC;








