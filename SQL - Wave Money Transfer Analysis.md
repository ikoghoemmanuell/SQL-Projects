# Wave Money Transfer Analysis

I will be using SQL to analyze a money transfer business operated by a company called Wave. The provided schema is a slightly simplified glimpse of some tables from Wave's actual PostgreSQL schema.

# Schema Overview

The schema consists of several tables that store relevant data about the money transfer business. Here's an overview of the key tables:

**users**: Contains information about the users, including their user ID, name, and other relevant details.

**transfers**: Stores details about the transfers made by users, including the transfer ID, sender ID, recipient ID, transfer amount, and other relevant information.

**agents**: Contains information about the agents facilitating transactions, including the agent ID, city, and other relevant details.

**wallets**: Stores details about the wallets associated with users, including the wallet ID, user ID, country, and other relevant information.

.

## 1. How many users does Wave have?

```sql
SELECT COUNT(u_id) AS TotalUsers 
FROM wave.users;
```
![image](https://github.com/ikoghoemmanuell/SQL-Projects/assets/102419217/e96919fb-e335-4b87-9400-2942125c90c5)

This query retrieves the count of all users from the users table.

.

## 2. How many transfers have been sent in the currency CFA?

```sql
SELECT COUNT(send_amount_currency) AS TotalTransfersInCfa 
FROM wave.transfers
WHERE send_amount_currency = 'CFA';
```
![image](https://github.com/ikoghoemmanuell/SQL-Projects/assets/102419217/6b0d6385-6d10-4531-a227-5ec938d7ab20)

This query retrieves the count of all transfers from the transfers table where the currency is CFA.

.

## 3. How many different users have sent a transfer in CFA?

```sql
SELECT  COUNT(DISTINCT user_id) AS TotalUsers, send_amount_currency AS CurrencySent 
FROM wave.transfers
WHERE send_amount_currency = 'CFA';
```
![image](https://github.com/ikoghoemmanuell/SQL-Projects/assets/102419217/65edfc18-97eb-4215-bc58-6393a23f5292)

This query retrieves the count of distinct sender IDs from the transfers table where the currency is CFA.

.

## 4. How many agent_transactions did we have in the months of 2022 (broken down by month)?

```sql
SELECT MONTH(when_created),
COUNT(atx_id) AS TotalTransactions
FROM wave.agent_transactions
WHERE when_created LIKE '%2022%'
GROUP BY MONTH(when_created)
ORDER BY MONTH(when_created);
```
![image](https://github.com/ikoghoemmanuell/SQL-Projects/assets/102419217/b04a28cb-ef8b-4cf3-be70-84d827b75072)

This query retrieves the count of agent transactions from the agent_transactions table in each month of 2022.

.

## 5. Over the course of the first half of 2022, how many Wave agents were "net depositors" vs. "net withdrawers"?

```sql
SELECT agent_id, SUM(amount)
FROM wave.agent_transactions
WHERE MONTH(when_created)<7 AND when_created LIKE '%2022%'
GROUP BY agent_id
ORDER BY SUM(amount);
```
![image](https://github.com/ikoghoemmanuell/SQL-Projects/assets/102419217/49b20891-45fe-4e42-9f35-346bc388d499)

This query retrieves the count of distinct agent IDs from the agent_transactions table in the first half of 2022, grouped by whether they were net depositors or net withdrawers.

.

## 6. Build an "atx volume city summary" table: find the volume of agent transactions created in the first half of 2022, grouped by city.

```sql
SELECT city, SUM(amount) AS Volume
FROM wave.agent_transactions
JOIN agents
ON agent_transactions.agent_id = agents.agent_id
WHERE MONTH(agent_transactions.when_created)<7 AND agent_transactions.when_created LIKE '%2022%'
GROUP BY city;
```
![image](https://github.com/ikoghoemmanuell/SQL-Projects/assets/102419217/15f1faf4-cbb9-4951-beb5-0268e419b8f3)

This query retrieves the total volume of agent transactions in the first half of 2022, grouped by the city where the transactions took place.

.

## 7. Now separate the atx volume by country as well (so your columns should be country, city, volume).

```sql
SELECT country, city, SUM(amount) AS Volume
FROM wave.agent_transactions
JOIN agents
ON agent_transactions.agent_id = agents.agent_id
WHERE MONTH(agent_transactions.when_created)<7 AND agent_transactions.when_created LIKE '%2022%'
GROUP BY city;
```
![image](https://github.com/ikoghoemmanuell/SQL-Projects/assets/102419217/5055bbc7-9494-48c3-86f2-815b8d328e09)

This query retrieves the total volume of agent transactions in the first half of 2022, separated by country and city.

.

## 8. Build a "send volume by country and kind" table: find the total volume of transfers sent in the first half of 2022, grouped by country and transfer kind.

```sql
SELECT ledger_location AS country, kind AS transfer_kind, SUM(send_amount_scalar) AS volume
FROM wave.transfers
JOIN wallets
ON wallets.wallet_id = transfers.dest_wallet_id
WHERE MONTH(transfers.when_created)<7 AND transfers.when_created LIKE '%2022%'
GROUP BY ledger_location,kind;
```
![image](https://github.com/ikoghoemmanuell/SQL-Projects/assets/102419217/27c17e4c-93f4-4441-9b9d-d07f51792256)

This query retrieves the total volume of transfers sent in the first half of 2022, grouped by country and transfer kind.

.

## 9. Then add columns for transaction count and number of unique senders (still broken down by country and transfer kind).

```sql
SELECT ledger_location AS country, kind AS transfer_kind, SUM(send_amount_scalar) AS volume,
COUNT(transfer_id), COUNT(DISTINCT user_id)
FROM wave.transfers
JOIN wallets
ON wallets.wallet_id = transfers.dest_wallet_id
WHERE MONTH(transfers.when_created)<7 AND transfers.when_created LIKE '%2022%'
GROUP BY ledger_location,kind;
```
![image](https://github.com/ikoghoemmanuell/SQL-Projects/assets/102419217/3327a8a7-933a-44aa-98ab-c75a794d60ed)

This query retrieves the transaction count, number of unique senders, and total volume of transfers sent in the first half of 2022, broken down by country and transfer kind.

.

## 10. Finally, which wallets sent more than 1,000,000 CFA in transfers in the first quarter (as identified by the source_wallet_id column on the transfers table), and how much did they send?

```sql
SELECT source_wallet_id, send_amount_scalar 
FROM wave.transfers
WHERE send_amount_currency = 'CFA' 
AND send_amount_scalar > 1000000
AND MONTH(when_created) < 4;
```
![image](https://github.com/ikoghoemmanuell/SQL-Projects/assets/102419217/39457906-bc15-49e9-8293-cbeb69e87e79)

This query retrieves the wallets (identified by the source_wallet_id) that sent more than 1,000,000 CFA in transfers during the first quarter of 2022 and the corresponding total amount sent.

## Instructions

1. To run the project, you will need to have a SQL database installed and running.
2. Open a SQL client and connect to the database.
3. Copy and paste the queries into the SQL client.
4. Run the queries one at a time.
5. The results of the queries will be displayed in the SQL client.

## Note

The provided queries assume the presence of appropriate tables and their relationships as described in the schema overview. Please adjust the table and column names according to your specific database setup.

```

```
