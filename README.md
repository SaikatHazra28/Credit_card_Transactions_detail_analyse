# 📊 Credit Card Transactions SQL Analysis

## 🔥 Project Overview
This project analyzes **credit card transactions** using SQL queries to derive insights such as:
- Top 5 cities with the highest spending.
- Month-wise spending trends by card type.
- Highest and lowest expense types in each city.
- Gender-wise spending contribution.
- Month-over-month growth analysis.

## 📂 Files Included  
- **database.xlsx** → Contains raw transaction data (used to create the SQL table).  
- **questions.txt** → Contains all problem statements.  
- **queries.sql** → SQL solutions for each problem.  

---

## 📌 SQL Table Structure
The dataset consists of the following columns:
- `transaction_id` (INT) - Unique transaction ID  
- `card_type` (VARCHAR) - Type of credit card (Silver, Gold, Platinum, etc.)  
- `exp_type` (VARCHAR) - Expense category (Food, Travel, Entertainment, etc.)  
- `amount` (BIGINT) - Transaction amount  
- `city` (VARCHAR) - City where the transaction happened  
- `gender` (VARCHAR) - Gender of the cardholder  
- `transaction_date` (DATE) - Date of the transaction  

---

## 🚀 SQL Questions & Solutions

### **1️⃣ Write a query to print the top 5 cities with the highest spends and their percentage contribution of total credit card spends**
```sql
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
```

### **2️⃣ Write a query to print the highest spend month and amount spent in that month for each card type**
```sql
WITH cte AS (
    SELECT card_type, YEAR(transaction_date) AS yt, MONTH(transaction_date) AS mt,
           SUM(amount) AS amount_spend
    FROM credit_card_transcations
    GROUP BY card_type, YEAR(transaction_date), MONTH(transaction_date)
)
SELECT * FROM (
    SELECT *, RANK() OVER (PARTITION BY card_type ORDER BY amount_spend DESC) AS rn
    FROM cte
) A
WHERE rn = 1;
```

### **3️⃣ Write a query to print the transaction details (all columns) for each card type when it reaches a cumulative of 1,000,000 total spends**
```sql
WITH cte AS (
    SELECT *,
           SUM(amount) OVER (PARTITION BY card_type ORDER BY transaction_date, transaction_id) AS total_spent
    FROM credit_card_transcations
)
SELECT * FROM (
    SELECT *, RANK() OVER (PARTITION BY card_type ORDER BY total_spent) AS rn
    FROM cte
    WHERE total_spent >= 1000000
) A
WHERE rn = 1;
```

### **4️⃣ Write a query to find the city with the lowest percentage spend for the Gold card type**
```sql
WITH A AS (
    SELECT city, SUM(amount) AS city_spend,
           SUM(CASE WHEN card_type = 'Gold' THEN amount END) AS total_spent_gold
    FROM credit_card_transcations
    GROUP BY city
)
SELECT TOP 1 city,
       total_spent_gold * 1.0 / city_spend AS ratio
FROM A
WHERE total_spent_gold IS NOT NULL
ORDER BY ratio ASC;
```

### **5️⃣ Write a query to print 3 columns: city, highest_expense_type, lowest_expense_type**
```sql
WITH cte AS (
    SELECT city, MAX(amount) AS max_amount, MIN(amount) AS min_amount
    FROM credit_card_transcations
    GROUP BY city
),
cte1 AS (
    SELECT c.city, c.amount, c.exp_type, cte.max_amount, cte.min_amount
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
```

### **6️⃣ Write a query to find the percentage contribution of spends by females for each expense type**
```sql
SELECT exp_type,
       SUM(CASE WHEN gender = 'F' THEN amount ELSE 0 END) * 1.0 / SUM(amount) AS percentage_female_contribution
FROM credit_card_transcations
GROUP BY exp_type
ORDER BY percentage_female_contribution DESC;
```

### **7️⃣ Which card and expense type combination saw the highest month-over-month growth in Jan 2014?**
```sql
WITH cte AS (
    SELECT card_type, exp_type,
           DATEPART(YEAR, transaction_date) AS Yt,
           DATEPART(MONTH, transaction_date) AS Mt,
           SUM(amount) AS total_spent
    FROM credit_card_transcations
    GROUP BY card_type, exp_type, DATEPART(YEAR, transaction_date), DATEPART(MONTH, transaction_date)
)
SELECT card_type, exp_type, (total_spent - pm) * 1.0 / pm AS mom_growth
FROM (
    SELECT *,
           LAG(total_spent, 1) OVER (PARTITION BY card_type, exp_type ORDER BY Yt, Mt) AS pm
    FROM cte
) A
WHERE pm IS NOT NULL AND Yt = 2014 AND Mt = 1
ORDER BY mom_growth DESC;
```

### **8️⃣ During weekends, which city has the highest total spend to total number of transactions ratio?**
```sql
SELECT TOP 1 city, SUM(amount) * 1.0 / COUNT(1) AS ratio
FROM credit_card_transcations
WHERE DATEPART(WEEKDAY, transaction_date) IN (6, 0)
GROUP BY city
ORDER BY ratio DESC;
```

### **9️⃣ Which city took the least number of days to reach its 500th transaction after the first transaction in that city?**
```sql
WITH cte AS (
    SELECT *,
           ROW_NUMBER() OVER (PARTITION BY city ORDER BY transaction_date, transaction_id) AS rn
    FROM credit_card_transcations
)
SELECT TOP 1 city, DATEDIFF(DAY, MIN(transaction_date), MAX(transaction_date)) AS days_to_500
FROM cte
WHERE rn = 1 OR rn = 500
GROUP BY city
HAVING COUNT(1) = 2
ORDER BY days_to_500;
```

---

## 💪 Conclusion
This project demonstrates how SQL can be used to extract meaningful insights from **credit card transaction data**. By using **window functions, ranking, CTEs, and aggregation**, we efficiently analyze spending trends, growth patterns, and customer behaviors.

---

## 👉 How to Use
- Clone this repository.
- Load `database.xlsx` into SQL Server.
- Run `queries.sql` to generate insights.

💡 Feel free to contribute, suggest improvements, or star this repo if you find it useful!

