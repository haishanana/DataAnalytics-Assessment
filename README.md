# DataAnalytics-Assessment
## Overview
This document provides an overview of the tables in the database, outlining their purpose and the type of data they store. The database is designed to manage customer information, financial transactions, and savings plans.
+ Questions
 
2. Transaction Frequency Analysis
Scenario: The finance team wants to analyze how often customers transact to segment them (e.g., frequent vs. occasional users).
Task: Calculate the average number of transactions per customer per month and categorize them:
"High Frequency" (≥10 transactions/month)
"Medium Frequency" (3-9 transactions/month)
"Low Frequency" (≤2 transactions/month)
Tables:
users_customuser
savings_savingsaccout
#### THE APPROACH
- To analyze transaction frequency and segment customers into different categories based on their activity, here’s the approach:

1. Understand the Required Metrics
We need to determine:

The total number of transactions per customer.

The average transactions per month per customer.

The frequency category based on predefined thresholds.

2. Identify Relevant Tables
users_customuser: Contains customer details (id as owner_id).

savings_savingsaccount: Stores deposit transactions (confirmed_amount field to track inflows).

3. Define Transaction Scope
Since we are analyzing frequency per customer per month, we will:

Count transactions for each customer.

Calculate the average number of transactions per month.

Categorize customers based on the given frequency thresholds.

#### CHALLENGES
+  Data Gaps & Accuracy Issues
Missing Transactions: Some accounts might have incomplete or missing transaction records, leading to inaccurate inactivity flags.

Data Delays: Transaction timestamps may be incorrect due to system delays, impacting the reliability of "last activity" calculations.



## SQL QUERIES
```SQL
SELECT 
    u.id AS customer_id,
    u.name AS customer_name,
    COUNT(s.id) / TIMESTAMPDIFF(MONTH, u.date_joined, NOW()) AS avg_transactions_per_month,
    CASE 
        WHEN COUNT(s.id) / TIMESTAMPDIFF(MONTH, u.date_joined, NOW()) >= 10 THEN 'High Frequency'
        WHEN COUNT(s.id) / TIMESTAMPDIFF(MONTH, u.date_joined, NOW()) BETWEEN 3 AND 9 THEN 'Medium Frequency'
        ELSE 'Low Frequency'
    END AS transaction_category
FROM users_customuser u
LEFT JOIN savings_savingsaccount s ON u.id = s.owner_id
WHERE u.date_joined IS NOT NULL
GROUP BY u.id, u.name
ORDER BY avg_transactions_per_month DESC;
```
 3. Account Inactivity Alert
Scenario: The ops team wants to flag accounts with no inflow transactions for over one year.
Task: Find all active accounts (savings or investments) with no transactions in the last 1 year (365 days) .
Tables:
plans_plan
savings_savingsaccount
### THE APPROACH
1. Identify Relevant Tables
savings_savingsaccount: Stores deposit transactions (confirmed inflows).

plans_plan: Tracks active financial plans (both savings and investments).

2. Define Inactivity Criteria
We need to:

Find active accounts (savings or investments).

Check if the last recorded inflow (confirmed_amount) was more than a year ago.

## SQL QUERIES 
```SQL
SELECT u.id AS customer_id, 
       u.name AS customer_name, 
       p.id AS investment_plan_id, 
       s.id AS savings_account_id, 
       p.last_charge_date AS last_investment_transaction, 
       s.transaction_date AS last_savings_transaction
FROM users_customuser u
LEFT JOIN plans_plan p ON u.id = p.owner_id
LEFT JOIN savings_savingsaccount s ON u.id = s.owner_id
WHERE u.is_active = 1 -- Only active users
AND (
    (p.last_charge_date IS NULL OR p.last_charge_date < DATE_SUB(NOW(), INTERVAL 1 YEAR)) 
    OR
    (s.transaction_date IS NULL OR s.transaction_date < DATE_SUB(NOW(), INTERVAL 1 YEAR))
)
ORDER BY customer_name;
```
4

### Tables
#### users_customuser
Description: Stores customer demographic and contact information.

+ Key Fields:
id (Primary Key) – Unique identifier for each customer
name – Full name of the customer
email – Contact email
phone_number – Customer's phone number
address – Residential or mailing address
