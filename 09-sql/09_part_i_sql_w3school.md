# Challenge Set 9
## Part I: W3Schools SQL Lab 

*Introductory level SQL*

--


This challenge uses the [W3Schools SQL playground](http://www.w3schools.com/sql/trysql.asp?filename=trysql_select_all). Please add solutions to this markdown file and submit.

Database Info (as displayed on screen)
Tablename	Records
Customers	91
Categories	8
Employees	10
OrderDetails	518
Orders	196
Products	77
Shippers	3
Suppliers	29

-----
```
1. Which customers are from the UK?

SELECT (*)
    FROM Customers
    WHERE Country = 'UK';

--Output:

CustomerID	CustomerName	ContactName	Address	City	PostalCode	Country

4	Around the Horn	Thomas Hardy	120 Hanover Sq.	London	WA1 1DP	UK

11	B's Beverages	Victoria Ashworth	Fauntleroy Circus	London	EC2 5NT	UK

16	Consolidated Holdings	Elizabeth Brown	Berkeley Gardens 12 Brewery	London	WX1 6LT	UK

19	Eastern Connection	Ann Devon	35 King George	London	WX3 6FW	UK

38	Island Trading	Helen Bennett	Garden House Crowther Way	Cowes	PO31 7PJ	UK

53	North/South	Simon Crowther	South House 300 Queensbridge	London	SW7 1RZ	UK

72	Seven Seas Imports	Hari Kumar	90 Wadhurst Rd.	London	OX15 4NB	UK


2. What is the name of the customer who has the most orders?

SELECT
    CustomerName,
    COUNT(*) as OrderCount
FROM
    Customers 
  JOIN
    Orders 
  ON
    Customers.CustomerID = Orders.CustomerID
GROUP BY
    Customers.CustomerID
ORDER BY
    OrderCount DESC
LIMIT 10;
--Ernst Handel has the most orders (10).

3. Which supplier has the highest average product price?
SELECT
    SupplierName,
    AVG(p.Price) as AvgPrice
FROM
    Suppliers as s
  JOIN
    Products as p
  ON 
    s.SupplierID = p.SupplierID
GROUP BY
    s.SupplierID
ORDER BY
    AvgPrice DESC;

--Aux joyeux ecclésiastiques, at 140.75.

4. How many different countries are all the customers from? (*Hint:* consider [DISTINCT](http://www.w3schools.com/sql/sql_distinct.asp).)

SELECT COUNT(DISTINCT(Country))
FROM Customers;

--21 different countries.

5. What category appears in the most orders?
#/dt show all tables in SQL
#indentation to help readability
#JOIN ON is one command! keep close together

SELECT CategoryName, COUNT(Quantity) as OrderCount
FROM Categories AS c
  JOIN Products as p
    ON c.CategoryID = p.CategoryID
  JOIN OrderDetails as od
    ON od.ProductID = p.ProductID
GROUP BY
    c.CategoryID
ORDER BY
    OrderCount DESC;

--So Dairy Products appear in the most orders.


6. What was the total cost for each order?
SELECT
    od.OrderID,
    SUM(od.Quantity * p.Price) as TotalCost   
FROM
    OrderDetails as od
  JOIN
    Products as p
  ON
    od.ProductID = p.ProductID
GROUP BY
    od.OrderID
ORDER BY
    TotalCost DESC;

--outputs the total cost for all 196 orders. Here are the top 3:
OrderID TotalCost
10372	15353.6
10424	14366.5
10417	14104


7. Which employee made the most sales (by total price)?

SELECT
    e.EmployeeID, e.FirstName, e.LastName,
    SUM(od.Quantity * p.Price) as TotalSalesPrice   
FROM
    Orders as o
    JOIN
    Employees as e
    ON
    o.EmployeeID = e.EmployeeID
    JOIN
    OrderDetails as od
    ON 
    od.OrderID = o.OrderID
    JOIN
    Products as p
    ON 
    p.ProductID = od.ProductID

GROUP BY
    e.EmployeeID
ORDER BY
    TotalSalesPrice DESC;

--Employee ID number 4 (Margaret Peacock) has the most sales, at 105696.49999999999.

8. Which employees have BS degrees? (*Hint:* look at the [LIKE](http://www.w3schools.com/sql/sql_like.asp) operator.)
SELECT
    *
FROM 
    Employees 
WHERE 
    Notes 
LIKE 
    '%BS%';

--Janet Leverling and Steven Buchanan have BS degrees.	
    


9. Which supplier of three or more products has the highest average product price? (*Hint:* look at the [HAVING](http://www.w3schools.com/sql/sql_having.asp) operator.)

SELECT
    SupplierName,
    COUNT(*) as NumProducts,
    AVG(p.Price) as MeanPrice
FROM
    Suppliers AS s
  JOIN
    Products AS p
  ON
    s.SupplierID = p.SupplierID
GROUP BY
    s.SupplierID
HAVING
    NumProducts >= 3
ORDER BY
 	MeanPrice DESC;

--Tokyo Traders. 	
```

