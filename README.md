# 📊 Credit Card Transactions SQL Analysis

## 🔥 Project Overview
This project analyzes **credit card transactions** using SQL queries to derive insights such as:
- Top 5 cities with highest spending  
- Month-wise spending trends by card type  
- Highest and lowest expense types in each city  
- Gender-wise spending contribution  
- Month-over-month growth analysis
- ## 🗂️ Files Included  
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

### **1️⃣ Top 5 Cities by Total Spend & % Contribution**
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
