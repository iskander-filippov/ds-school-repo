# Домашнее задание №6

## Задача №1
**Постановка:** *выберите заказчиков из Германии, Франции и Мадрида, выведите их название, страну и адрес*

**Решение:**
```
SELECT
    CustomerName,
    Country,
    Address
FROM Customers
WHERE Country IN (
    'Germany',
    'France'
)
   OR City = 'Madrid'
```

## Задача №2
**Постановка:** *выберите топ 3 страны по количеству заказчиков, выведите их названия и количество записей*

**Решение:**
```
SELECT
    Country,
    COUNT(*) AS CUST_CNT
FROM Customers
GROUP BY Country
ORDER BY CUST_CNT DESC
LIMIT 3
```

## Задача №3
**Постановка:** *выберите перевозчика, который отправил 10-й по времени заказ, выведите его название, и дату отправления*

**Решение:**
```
SELECT
    S.ShipperName,
    O.OrderDate
FROM Orders O 
JOIN Shippers S ON O.ShipperID = S.ShipperID
ORDER BY O.OrderDate
LIMIT 1
OFFSET 9
```

## Задача №4
**Постановка:** *выберите самый дорогой заказ, выведите список товаров с их ценами*

**Решение:**
```
WITH MOST_XPNSV_ORDER AS (
    SELECT
        OD.OrderID,
        SUM(OD.Quantity * P.Price) AS Total
    FROM OrderDetails OD
    JOIN Products P ON OD.ProductID = P.ProductID
    GROUP BY OD.OrderID
    ORDER BY Total DESC
    LIMIT 1
)

SELECT
    OD.ProductID,
    P.ProductName,
    P.Price
FROM OrderDetails OD
JOIN Products P ON OD.ProductID = P.ProductID
JOIN MOST_XPNSV_ORDER MXO ON OD.OrderID = MXO.OrderID
```

## Задача №5
**Постановка:** *какой товар больше всего заказывали по количеству единиц товара, выведите его название и количество единиц в каждом из заказов*

**Решение:**
```
WITH MOST_ORDRBL_PRD AS (
    SELECT
        OD.ProductID,
        P.ProductName,
        SUM(OD.Quantity) AS Total
    FROM OrderDetails OD
    JOIN Products P ON OD.ProductID = P.ProductID
    GROUP BY OD.ProductID, P.ProductName
    ORDER BY Total DESC
    LIMIT 1
)

SELECT
    MOP.ProductName,
    OD.OrderID,
    OD.Quantity
FROM OrderDetails OD
JOIN MOST_ORDRBL_PRD MOP ON OD.ProductID = MOP.ProductID
```

## Задача №6
**Постановка:** *выведите топ 5 поставщиков по количеству заказов, выведите их названия, страну, контактное лицо и телефон*

**Решение:**
```
WITH MOST_PPLR_SPPLRS AS (
    SELECT
        P.SupplierID,
        COUNT(OD.OrderID) AS Total
    FROM OrderDetails OD
    JOIN Products P ON OD.ProductID = P.ProductID
    GROUP BY P.SupplierID
    ORDER BY TOTAL DESC
    LIMIT 5
)

SELECT
    SupplierName,
    Country,
    ContactName,
    Phone
FROM Suppliers SP
JOIN MOST_PPLR_SPPLRS MPS ON SP.SupplierID = MPS.SupplierID
```

## Задача №7
**Постановка:** *какую категорию товаров заказывали больше всего по стоимости в Бразилии, выведите страну, название категории и сумму*

**Решение:**
```
WITH MOST_VLBL_CAT AS (
    SELECT
        P.CategoryID,
        SUM(OD.Quantity * P.Price) AS Total
    FROM Orders O
    JOIN Customers C ON O.CustomerID = C.CustomerID AND C.Country = 'Brazil'
    JOIN OrderDetails OD ON O.OrderID = OD.OrderID
    JOIN Products P ON OD.ProductID = P.ProductID
    GROUP BY P.CategoryID
    ORDER BY TOTAL DESC
    LIMIT 1
)

SELECT
    'Brazil' AS Country,
    C.CategoryName,
    MVC.Total
FROM Categories C
JOIN MOST_VLBL_CAT MVC ON C.CategoryID = MVC.CategoryID
```

## Задача №8
**Постановка:** *какая разница в стоимости между самым дорогим и самым дешевым заказом из США*

**Решение:**
```
WITH USA_ORDERS AS (
    SELECT
        OD.OrderID,
        SUM(OD.Quantity * P.Price) AS Total
    FROM Orders O
    JOIN Customers C ON O.CustomerID = C.CustomerID AND C.Country = 'USA'
    JOIN OrderDetails OD ON OD.OrderID = O.OrderID
    JOIN Products P ON OD.ProductID = P.ProductID
    GROUP BY OD.OrderID
)

SELECT
    MAX(Total) - MIN(Total) AS Difference
FROM USA_ORDERS
```

## Задача №9
**Постановка:** *выведите количество заказов у каждого их трех самых молодых сотрудников, а также имя и фамилию во второй колонке*

**Решение:**
```
SELECT
    COUNT(O.OrderID) AS Orders,
    E.FirstName || ' ' ||  E.LastName AS Name
FROM Orders O
JOIN Employees E ON O.EmployeeID = E.EmployeeID AND E.BirthDate >= (
    SELECT BirthDate FROM Employees ORDER BY BirthDate DESC LIMIT 1 OFFSET 2
)
GROUP BY E.FirstName || ' ' ||  E.LastName
```

## Задача №10
**Постановка:** *cколько банок крабового мяса всего было заказано*

**Решение:**
```
SELECT
    SUM(Quantity * 24) AS Total_tins
FROM OrderDetails OD
JOIN Products P ON OD.ProductID = P.ProductID AND P.ProductName LIKE '%rab%eat%'
```