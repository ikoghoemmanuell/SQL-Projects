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
SELECT COUNT(*) AS TotalUsers
FROM users;
```

This query retrieves the count of all users from the users table.

.

## 2. How many transfers have been sent in the currency CFA?

```sql
SELECT COUNT(*) AS TotalTransfers
FROM transfers
WHERE currency = 'CFA';
```

This query retrieves the count of all transfers from the transfers table where the currency is CFA.

.

## 3. How many different users have sent a transfer in CFA?

```sql
SELECT COUNT(DISTINCT sender_id) AS UniqueUsers
FROM transfers
WHERE currency = 'CFA';
```

This query retrieves the count of distinct sender IDs from the transfers table where the currency is CFA.

.

## 4. How many agent_transactions did we have in the months of 2022 (broken down by month)?

```sql
SELECT strftime('%m', created_at) AS Month, COUNT(*) AS TotalTransactions
FROM agent_transactions
WHERE strftime('%Y', created_at) = '2022'
GROUP BY Month;
```

This query retrieves the count of agent transactions from the agent_transactions table in each month of 2022.

.

## 5. Over the course of the first half of 2022, how many Wave agents were "net depositors" vs. "net withdrawers"?

```sql
SELECT is_net_depositor, COUNT(DISTINCT agent_id) AS TotalAgents
FROM agent_transactions
WHERE strftime('%Y-%m', created_at) BETWEEN '2022-01' AND '2022-06'
GROUP BY is_net_depositor;
```

This query retrieves the count of distinct agent IDs from the agent_transactions table in the first half of 2022, grouped by whether they were net depositors or net withdrawers.

.

## 6. Build an "atx volume city summary" table: find the volume of agent transactions created in the first half of 2022, grouped by city.

```sql
SELECT agents.city, SUM(amount) AS TotalVolume
FROM agent_transactions
JOIN agents ON agent_transactions.agent_id = agents.agent_id
WHERE strftime('%Y-%m', created_at) BETWEEN '2022-01' AND '2022-06'
GROUP BY agents.city;
```

This query retrieves the total volume of agent transactions in the first half of 2022, grouped by the city where the transactions took place.

.

## 7. Now separate the atx volume by country as well (so your columns should be country, city, volume).

```sql
SELECT wallets.country, agents.city, SUM(agent_transactions.amount) AS TotalVolume
FROM agent_transactions
JOIN agents ON agent_transactions.agent_id = agents.agent_id
JOIN wallets ON agent_transactions.wallet_id = wallets.wallet_id
WHERE strftime('%Y-%m', agent_transactions.created_at) BETWEEN '2022-01' AND '2022-06'
GROUP BY wallets.country, agents.city;
```

This query retrieves the total volume of agent transactions in the first half of 2022, separated by country and city.

.

## 8. Build a "send volume by country and kind" table: find the total volume of transfers sent in the first half of 2022, grouped by country and transfer kind.

```sql
SELECT wallets.ledger_location AS Country, transfers.kind AS TransferKind, SUM(transfers.send_amount_scalar) AS TotalVolume
FROM transfers
JOIN wallets ON transfers.source_wallet_id = wallets.wallet_id
WHERE strftime('%Y-%m', transfers.created_at) BETWEEN '2022-01' AND '2022-06'
GROUP BY Country, TransferKind;
```

This query retrieves the total volume of transfers sent in the first half of 2022, grouped by country and transfer kind.

.

## 9. Then add columns for transaction count and number of unique senders (still broken down by country and transfer kind).

```sql
SELECT
  wallets.ledger_location AS Country,
  transfers.kind AS TransferKind,
  COUNT(*) AS TransactionCount,
  COUNT(DISTINCT transfers.sender_id) AS UniqueSenders,
  SUM(transfers.send_amount_scalar) AS TotalVolume
FROM transfers
JOIN wallets ON transfers.source_wallet_id = wallets.wallet_id
WHERE strftime('%Y-%m', transfers.created_at) BETWEEN '2022-01' AND '2022-06'
GROUP BY Country, TransferKind;
```

This query retrieves the transaction count, number of unique senders, and total volume of transfers sent in the first half of 2022, broken down by country and transfer kind.

.

## 10. Finally, which wallets sent more than 1,000,000 CFA in transfers in the first quarter (as identified by the source_wallet_id column on the transfers table), and how much did they send?

```sql
SELECT transfers.source_wallet_id, SUM(transfers.send_amount_scalar) AS TotalAmount
FROM transfers
WHERE strftime('%Y-%m', transfers.created_at) BETWEEN '2022-01' AND '2022-03'
GROUP BY transfers.source_wallet_id
HAVING TotalAmount > 1000000;
```

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
