📊 SQL Analysis Project: Customer Segmentation and Plan Engagement

🔍 Overview

This project analyzes customer engagement with savings and investment plans using MySQL. The goal was to identify high-value customers who have funded both savings and investment plans  a key insight for cross-selling opportunities.

The project involved querying multiple related tables:

users_customuser (customer details)

savings_savingsaccount (savings transactions)

plans_plan 

🧠 Per-Question Explanations


❓Q1: Identify customers with at least one funded savings plan and one funded investment plan

📘 Per-Question Explanation:

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


CODE:

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
    





