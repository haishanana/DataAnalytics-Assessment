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
#### CHALLENGES
+  Data Gaps & Accuracy Issues
Missing Transactions: Some accounts might have incomplete or missing transaction records, leading to inaccurate inactivity flags.

+ Data Delays: Transaction timestamps may be incorrect due to system delays, impacting the reliability of "last activity" calculations.


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
4.Customer Lifetime Value (CLV) Estimation
Scenario: Marketing wants to estimate CLV based on account tenure and transaction volume (simplified model).
Task: For each customer, assuming the profit_per_transaction is 0.1% of the transaction value, calculate:
Account tenure (months since signup)
Total transactions
Estimated CLV (Assume: CLV = (total_transactions / tenure) * 12 * avg_profit_per_transaction)
Order by estimated CLV from highest to lowest

Tables:
users_customuser
savings_savingsaccount
### APPROACH
To estimate Customer Lifetime Value (CLV), we need to calculate:

Account Tenure (months since signup)

Use the difference between the signup date (created_at in users_customuser) and today.

Convert this difference into months.

Total Transactions

Count all transactions (confirmed_amount > 0) in savings_savingsaccount for each customer.

Estimated CLV Calculation

Using the formula:

C
𝐿
𝑉
=
(
total transactions
tenure
)
×
12
×
avg profit per transaction
Assume profit per transaction is 0.1% of transaction value.

Order by CLV (Descending)

Sort customers based on their estimated CLV from highest to lowest.

### CHALLENGES
+ Edge Cases in Tenure Calculation
If tenure is zero months (new accounts), division by zero error must be handled (NULLIF function prevents this).
Alternative: Convert to days, then average transactions per day instead of per month.

## SQL QUERIES 
```SQL
SELECT 
    u.id AS customer_id, 
    u.name AS customer_name, 
    TIMESTAMPDIFF(MONTH, u.date_joined, NOW()) AS account_tenure_months, 
    COUNT(s.id) AS total_transactions, 
    (COUNT(s.id) / NULLIF(TIMESTAMPDIFF(MONTH, u.date_joined, NOW()), 0)) * 12 * 0.001 * AVG(s.fee_in_kobo) AS estimated_CLV
FROM users_customuser u
LEFT JOIN savings_savingsaccount s ON u.id = s.owner_id
WHERE u.date_joined IS NOT NULL
GROUP BY u.id, u.name
ORDER BY estimated_CLV DESC;
```



### Tables
#### users_customuser
Description: Stores customer demographic and contact information.

+ Key Fields:
id (Primary Key) – Unique identifier for each customer
name – Full name of the customer
email – Contact email
phone_number – Customer's phone number
address – Residential or mailing address

#### 2. savings_savingsaccount
Description: Records of deposit transactions for customers' savings accounts.

Key Fields:

account_id (Primary Key) – Unique identifier for each savings account

owner_id (Foreign Key → users_customuser.id) – Links the savings account to a customer

balance – Current balance of the savings account

confirmed_amount – Amount deposited into the account

created_at – Timestamp of when the account was created

#### 3. plans_plan
Description: Stores records of financial plans created by customers.

Key Fields:

    |id | name | description | amount | start_date | last_charge_date | next_charge_date | created_on | frequency_id | owner_id | status_id | interest_rate | withdrawal_date | default_plan | plan_type_id | goal | locked | next_returns_date | last_returns_date | cowry_amount | debit_card | is_archived | is_deleted | is_goal_achieved | is_a_goal | is_interest_free | plan_group_id | is_deleted_from_group | is_a_fund | purchased_fund_id | is_a_wallet | currency_is_dollars | is_auto_rollover | is_vendor_plan | plan_source | is_donation_plan | donation_description | donation_expiry_date | donation_link | link_code | charge_payment_fee | donation_is_approved | is_emergency_plan | is_personal_challenge | currency_id | is_a_usd_index | usd_index_id | open_savings_plan | new_cycle | recurrence | is_bloom_note | is_managed_portfolio | portfolio_holdings_id | is_fixed_investment | is_regular_savings |

#### 4. withdrawals_withdrawal
Description: Records all customer withdrawal transactions.

Key Fields:

   |id | amount | amount_withdrawn | transaction_reference | transaction_date | new_balance | bank_id | owner_id | plan_id | transaction_channel_id | transaction_status_id | transaction_type_id | fee_in_kobo | description | gateway | gateway_response | session_id | currency | fee_in_cents | payment_id | created_on | updated_on | withdrawal_intent_id |


### THE FILE 





--Transaction Frequency Analysis
Scenario: The finance team wants to analyze how often customers transact to segment them (e.g., frequent vs. occasional users).
Task: Calculate the average number of transactions per customer per month and categorize them:
"High Frequency" (≥10 transactions/month)
"Medium Frequency" (3-9 transactions/month)
"Low Frequency" (≤2 transactions/month)

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

High-Value Customers with Multiple Products
Scenario: The business wants to identify customers who have both a savings and an investment plan (cross-selling opportunity).
Task: Write a query to find customers with at least one funded savings plan AND one funded investment plan, sorted by total deposits.
Tables:
users_customuser
savings_savingsaccount
plans_plan
SELECT 
    u.id AS customer_id, 
    u.name, 
    COUNT(DISTINCT s.account_id) AS savings_count, 
    COUNT(DISTINCT p.plan_id) AS investment_count, 
    SUM(s.confirmed_amount) AS total_deposits
FROM users_customuser u
JOIN savings_savingsaccount s ON u.id = s.owner_id
JOIN plans_plan p ON u.id = p.owner_id
WHERE s.confirmed_amount > 0 AND p.is_a_fund = 1
GROUP BY u.id, u.name
HAVING COUNT(DISTINCT s.account_id) > 0 AND COUNT(DISTINCT p.plan_id) > 0
ORDER BY total_deposits DESC;










 ISE.sql…]()

