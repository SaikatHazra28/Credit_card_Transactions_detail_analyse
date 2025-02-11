Credit Card Transactions SQL Analysis 🚀
This project contains SQL queries analyzing credit card transactions to derive insights such as top-spending cities, highest expense categories, and transaction trends.

📂 Dataset Details
The dataset (database.xlsx) contains transaction records with fields like:

Transaction ID
Transaction Date
City
Card Type (Gold, Platinum, Silver, Signature)
Expense Type (Entertainment, Grocery, Travel, etc.)
Amount
Gender
... and more!
📊 SQL Queries & Questions
1️⃣ Top 5 Cities by Credit Card Spends
👉 Question: Find the top 5 cities with the highest total spending and their percentage contribution to total credit card spends.

sql
Copy
Edit
WITH A AS (
    SELECT city, SUM(amount) AS city_spend
    FROM credit_card_transcations 
    GROUP BY city
), 
B AS (
    SELECT SUM(amount) AS total_spend FROM credit_card_transcations
)
SELECT TOP 5 city, city_spend * 1.0 / total_spend * 100 AS percentage_contribution
FROM A, B
ORDER BY city_spend DESC;
2️⃣ Highest Spending Month for Each Card Type
👉 Question: Identify the month with the highest total spending for each card type.

sql
Copy
Edit
WITH cte AS (
    SELECT card_type, 
           YEAR(transaction_date) AS year, 
           MONTH(transaction_date) AS month, 
           SUM(amount) AS amount_spent
    FROM credit_card_transcations
    GROUP BY card_type, YEAR(transaction_date), MONTH(transaction_date)
)
SELECT * FROM (
    SELECT *, RANK() OVER(PARTITION BY card_type ORDER BY amount_spent DESC) AS rn
    FROM cte
) A
WHERE rn = 1;
3️⃣ First ₹1,000,000 Cumulative Spend Per Card Type
👉 Question: Find the first transaction where cumulative spending reaches ₹1,000,000 for each card type.

sql
Copy
Edit
WITH cte AS (
    SELECT *, SUM(amount) OVER(PARTITION BY card_type ORDER BY transaction_date, transaction_id) AS total_spent
    FROM credit_card_transcations
)
SELECT * FROM (
    SELECT *, RANK() OVER(PARTITION BY card_type ORDER BY total_spent) AS rn
    FROM cte WHERE total_spent >= 1000000
) A 
WHERE rn = 1;
4️⃣ City with Lowest Percentage Spend on Gold Cards
👉 Question: Identify the city where Gold cards have the lowest spending contribution.

sql
Copy
Edit
WITH A AS (
    SELECT city, 
           SUM(amount) AS city_spend,
           SUM(CASE WHEN card_type = 'Gold' THEN amount END) AS total_spent_gold
    FROM credit_card_transcations 
    GROUP BY city  
)
SELECT TOP 1 city, total_spent_gold * 1.0 / city_spend AS ratio
FROM A
WHERE total_spent_gold IS NOT NULL
ORDER BY ratio ASC;
5️⃣ Highest & Lowest Expense Type per City
👉 Question: Retrieve the highest and lowest expense type per city.

sql
Copy
Edit
WITH cte AS (
    SELECT city, MAX(amount) AS max_amount, MIN(amount) AS min_amount
    FROM credit_card_transcations
    GROUP BY city
), 
cte1 AS (
    SELECT c.city, c.amount, c.exp_type, 
           cte.max_amount, cte.min_amount
    FROM credit_card_transcations c
    INNER JOIN cte 
        ON c.city = cte.city 
       AND (c.amount = cte.max_amount OR c.amount = cte.min_amount)
)
SELECT DISTINCT city,
       MAX(CASE WHEN amount = max_amount THEN exp_type END) AS highest_expense_type,
       MAX(CASE WHEN amount = min_amount THEN exp_type END) AS lowest_expense_type
FROM cte1
GROUP BY city
ORDER BY city;
6️⃣ Female Spending Contribution by Expense Type
👉 Question: Find the percentage contribution of female spending for each expense type.

sql
Copy
Edit
SELECT exp_type,
       SUM(CASE WHEN gender='F' THEN amount ELSE 0 END) * 1.0 / SUM(amount) AS percentage_female_contribution 
FROM credit_card_transcations
GROUP BY exp_type
ORDER BY percentage_female_contribution DESC;
7️⃣ Highest Month-over-Month Growth (Jan 2014)
👉 Question: Find the card type & expense type combination that had the highest month-over-month growth in January 2014.

sql
Copy
Edit
WITH cte AS (
    SELECT card_type, exp_type,
           YEAR(transaction_date) AS year, 
           MONTH(transaction_date) AS month,
           SUM(amount) AS total_spent
    FROM credit_card_transcations 
    GROUP BY card_type, exp_type, YEAR(transaction_date), MONTH(transaction_date) 
)
SELECT card_type, exp_type, (total_spent - pm) * 1.0 / pm AS mom_growth 
FROM (
    SELECT *, LAG(total_spent, 1) OVER(PARTITION BY card_type, exp_type ORDER BY year, month) AS pm
    FROM cte
) A
WHERE pm IS NOT NULL AND year = 2014 AND month = 1
ORDER BY mom_growth DESC;
8️⃣ City with Highest Weekend Spend-to-Transaction Ratio
👉 Question: Find the city with the highest spend-to-transaction ratio on weekends.

sql
Copy
Edit
SELECT TOP 1 city, SUM(amount) * 1.0 / COUNT(1) AS ratio
FROM credit_card_transcations
WHERE DATEPART(weekday, transaction_date) IN (6, 0)
GROUP BY city
ORDER BY ratio DESC;
9️⃣ Fastest City to Reach 500 Transactions
👉 Question: Which city took the least number of days to reach its 500th transaction?

sql
Copy
Edit
WITH cte AS (
    SELECT *, ROW_NUMBER() OVER(PARTITION BY city ORDER BY transaction_date, transaction_id) AS rn
    FROM credit_card_transcations
)
SELECT TOP 1 city, DATEDIFF(DAY, MIN(transaction_date), MAX(transaction_date)) AS days_to_500
FROM cte
WHERE rn = 1 OR rn = 500 
GROUP BY city
HAVING COUNT(1) = 2
ORDER BY days_to_500;
📜 How to Use This Project
Upload the dataset (database.xlsx) into SQL Server.
Run the SQL scripts provided in this repository to generate insights.
Modify queries as needed for further analysis.
🛠️ Technologies Used
SQL Server (for query execution)
Git & GitHub (for version control)
Excel (for data storage & import)
📢 Contribute!
Feel free to suggest improvements or add more queries to this project! 🚀
