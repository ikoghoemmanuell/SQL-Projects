# SQL Project by Emmanuel Ikogho

This project uses SQL to answer five questions about a fictional online retail company. The data for the project is provided in the Northwind database.

## 1. What are the total sums of freights of ship cities that have orders from 2 or more customers?

```sql
SELECT ShipCity,
SUM(Freight) AS TotalFreight
FROM 'Order'
GROUP BY ShipCity
HAVING COUNT(DISTINCT CustomerId)>=2
ORDER BY TotalFreight DESC;
```

This query retrieves ship cities along with the total number of unique customers and the sum of freight costs. It only includes ship cities that have orders from two or more customers, and the results are sorted in descending order of total freight.

.

## 2. Generate a table of employees and the number of customers they handle.

```sql
SELECT e.FirstName, e.LastName, COUNT(DISTINCT o.CustomerId) AS Customers
FROM 'Order' o
LEFT JOIN Employee e ON o.EmployeeId = e.Id
GROUP BY e.FirstName, e.LastName
ORDER BY Customers DESC;
```

This query creates a table that lists employees along with the number of unique customers they handle. It joins the "Order" and "Employee" tables based on the employee ID and groups the results by the employee's first name and last name. The table is sorted in descending order of the number of customers.

.

## 3. How many orders were made each year?

```sql
SELECT strftime('%Y', OrderDate) AS 'Year', COUNT(Id) AS Orders
FROM 'Order'
GROUP BY 'Year'
ORDER BY 'Year';
```

This query retrieves the number of orders made each year. It uses the strftime function to extract the year from the OrderDate column and groups the results by the year. The results are sorted in ascending order of the year.

.

## 4. What quantity of the product ‘Chang’ was shipped in 2014?

```sql
SELECT p.ProductName, SUM(od.Quantity) AS Quantity
FROM 'Order' o
LEFT JOIN OrderDetail od ON o.Id = od.OrderId
LEFT JOIN Product p ON od.ProductId = p.Id
WHERE strftime('%Y', OrderDate) = '2014' AND p.ProductName = 'Chang'
GROUP BY p.ProductName;
```

This query retrieves the quantity of the product 'Chang' that was shipped in the year 2014. It joins the "Order," "OrderDetail," and "Product" tables based on the corresponding IDs and filters the results to include only orders from 2014 and the product 'Chang.' The results are grouped by the product name.

.

## 5. What are the revenues per supplier in 2013 before and after discount applied? Show both values in a single table.

```sql
SELECT s.CompanyName AS Supplier,
       SUM(od.UnitPrice * od.Quantity) AS 'Revenue Before Discount',
       SUM((od.UnitPrice * od.Quantity) - (od.UnitPrice * od.Quantity * od.Discount)) AS 'Revenue After Discount'
FROM OrderDetail od
LEFT JOIN 'Order' o ON od.OrderId = o.Id
LEFT JOIN Product p ON od.ProductId = p.Id
LEFT JOIN Supplier s ON p.SupplierId = s.Id
WHERE strftime('%Y', o.OrderDate) = '2013'
GROUP BY Supplier
ORDER BY 'Revenue Before Discount' DESC;
```

This query calculates the revenues per supplier in the year 2013 before and after discounts are applied. It joins the "OrderDetail," "Order," "Product," and "Supplier" tables based on the corresponding IDs and filters the results to include only orders from 2013. The revenues before and after discount are calculated and grouped by the supplier. The results are sorted in descending order of revenue before discount.


## Instructions

1. To run the project, you will need to have a SQL database installed and running.
2. Open a SQL client and connect to the database.
3. Copy and paste the queries into the SQL client.
4. Run the queries one at a time.
5. The results of the queries will be displayed in the SQL client.

## Notes

* The queries in this project are written for the `Northwind` database. If you are using a different database, you may need to modify the queries to reflect the schema of your database.
* The results of the queries may vary depending on the data in your database.
