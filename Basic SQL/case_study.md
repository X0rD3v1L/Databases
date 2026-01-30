### Sales Order Dataset

```
 id |   product_name    | price  | released_date 
----+-------------------+--------+---------------
  1 | iPhone 17         | 102900 | 2025-09-19
  2 | iPhone 17 Pro     | 130900 | 2025-09-19
  3 | iPhone 17 Pro Max | 149900 | 2025-09-19
  4 | iPhone 15         |    800 | 2023-08-22
  5 | Macbook Pro       |   2100 | 2022-10-12
  6 | Apple Watch 9     |    550 | 2022-09-04
  7 | iPad              |    400 | 2020-08-25
  8 | AirPods           |    420 | 2024-03-30
(8 rows)

 id |     name      |      email       
----+---------------+------------------
  1 | Meghan Harley | mharley@demo.com
  2 | Rosa Chan     | rchan@demo.com
  3 | Logan Short   | lshort@demo.com
  4 | Zaria Duke    | zduke@demo.com
(4 rows)

 id |    name     
----+-------------
  1 | Nina Kumari
  2 | Abrar Khan
  3 | Irene Costa
(3 rows)

 order_id | order_date | quantity | prod_id |  status   | customer_id | emp_id 
----------+------------+----------+---------+-----------+-------------+--------
        1 | 2024-01-01 |        2 |       1 | Completed |           1 |      1
        2 | 2024-01-01 |        3 |       1 | Pending   |           2 |      2
        3 | 2024-01-02 |        3 |       2 | Completed |           3 |      2
        4 | 2024-01-03 |        3 |       3 | Completed |           3 |      2
        5 | 2024-01-04 |        1 |       1 | Completed |           3 |      2
        6 | 2024-01-04 |        1 |       3 | completed |           2 |      1
        7 | 2024-01-04 |        1 |       2 | On Hold   |           2 |      1
        8 | 2024-01-05 |        4 |       2 | Rejected  |           1 |      2
        9 | 2024-01-06 |        5 |       5 | Completed |           1 |      2
       10 | 2024-01-06 |        1 |       1 | Cancelled |           1 |      1
(10 rows)
```

1. Identify the total no of products sold

```
demo=# SELECT SUM(quantity) AS "Products Sold" FROM sales_order WHERE status = 'Completed';
 Products Sold 
---------------
            14
(1 row)
```

2. Other than Completed, display the available delivery statuses

```
demo=# SELECT DISTINCT status FROM sales_order WHERE status NOT IN ('Completed', 'completed');
  status   
-----------
 Pending
 Rejected
 On Hold
 Cancelled
(4 rows)
```


3. Display the order id, order_date and product_name for all the completed orders.

```
demo=# SELECT 
    so.order_id   AS "Order ID",
    so.order_date AS "Order Date",
    p.product_name AS "Product Name"
FROM sales_order so
INNER JOIN products p
ON so.prod_id = p.id
WHERE lower(so.status) = 'completed';
;
 Order ID | Order Date |   Product Name    
----------+------------+-------------------
        5 | 2024-01-04 | iPhone 17
        1 | 2024-01-01 | iPhone 17
        3 | 2024-01-02 | iPhone 17 Pro
        6 | 2024-01-04 | iPhone 17 Pro Max
        4 | 2024-01-03 | iPhone 17 Pro Max
        9 | 2024-01-06 | Macbook Pro
(6 rows)
```

4. Sort the above query to show the earliest orders at the top. Also, display the customer who purchased these orders.

```
demo=# SELECT 
    so.order_id   AS "Order ID",
    so.order_date AS "Order Date",
    p.product_name AS "Product Name",
    c.name AS "Customer Name"
FROM sales_order so
INNER JOIN products p
ON so.prod_id = p.id
INNER JOIN customers c
ON so.customer_id = c.id
WHERE lower(so.status) = 'completed'
ORDER BY so.order_date ASC;
 Order ID | Order Date |   Product Name    | Customer Name 
----------+------------+-------------------+---------------
        1 | 2024-01-01 | iPhone 17         | Meghan Harley
        3 | 2024-01-02 | iPhone 17 Pro     | Logan Short
        4 | 2024-01-03 | iPhone 17 Pro Max | Logan Short
        5 | 2024-01-04 | iPhone 17         | Logan Short
        6 | 2024-01-04 | iPhone 17 Pro Max | Rosa Chan
        9 | 2024-01-06 | Macbook Pro       | Meghan Harley
(6 rows)
```

5. Display the total no of orders corresponding to each delivery status

```
demo=# SELECT status, COUNT(*) FROM sales_order GROUP BY status;
  status   | count 
-----------+-------
 Pending   |     1
 completed |     1
 Completed |     5
 Rejected  |     1
 On Hold   |     1
 Cancelled |     1
(6 rows)
```

6. How many orders are still not completed for orders purchasing more than 1 item?

```
demo=# SELECT COUNT(*) AS "Not completed orders" FROM sales_order WHERE quantity > 1 AND lower(status) = 'completed';
 Not completed orders 
----------------------
                    4
(1 row)
```

7. Find the total number of orders corresponding to each delivery status by ignoring the case in the delivery status. The status with highest no of orders should be at the top.

```
demo=# SELECT UPPER(status), COUNT(*) AS "count" FROM sales_order GROUP BY UPPER(status) ORDER BY count DESC;
   upper   | count 
-----------+-------
 COMPLETED |     6
 PENDING   |     1
 REJECTED  |     1
 ON HOLD   |     1
 CANCELLED |     1
(5 rows)
```

8. Write a query to identify the total products purchased by each customer.

```
demo=# SELECT 
    c.name AS "Customer Name",
    SUM(so.quantity) AS "Total Quantity"
FROM customers c
JOIN sales_order so ON so.customer_id = c.id
GROUP BY c.name;
 Customer Name | Total Quantity 
---------------+----------------
 Meghan Harley |             12
 Logan Short   |              7
 Rosa Chan     |              5
(3 rows)
```

9. Display the total sales and average sales done for each day. 

```
demo=# 
      SELECT order_date AS "Date", 
      SUM(quantity*price) AS "TOTAL SALES", AVG(quantity*p.price) AS "AVG SALES" 
      FROM sales_order so JOIN products p ON p.id = so.prod_id 
      GROUP BY order_date
      ORDER BY order_date;

    Date    | TOTAL SALES | AVG SALES 
------------+-------------+-----------
 2024-01-01 |      514500 |    257250
 2024-01-02 |      392700 |    392700
 2024-01-03 |      449700 |    449700
 2024-01-04 |      383700 |    127900
 2024-01-05 |      523600 |    523600
 2024-01-06 |      113400 |     56700
(6 rows)
```

10. Display the customer name, employee name, and total sale amount of all orders which are either on hold or pending.

```
demo=# SELECT c.name AS "Customer Name", e.name AS "Employee Name", SUM(so.quantity*p.price) AS "Total Sales"
FROM sales_order so
JOIN customers c ON c.id = so.customer_id
JOIN employees e ON e.id = so.emp_id
JOIN products p ON p.id = so.prod_id
WHERE so.status IN ('On Hold', 'Pending')
GROUP BY c.name, e.name;

 Customer Name | Employee Name | Total Sales 
---------------+---------------+-------------
 Rosa Chan     | Abrar Khan    |      308700
 Rosa Chan     | Nina Kumari   |      130900
(2 rows)
```

11. Fetch all the orders which were neither completed/pending or were handled by the employee Abrar. Display employee name and all details of order.

```
demo=# SELECT e.name AS employee, so.*
FROM sales_order so
JOIN employees e ON e.id = so.emp_id
WHERE LOWER(e.name) LIKE '%abrar%'
   OR LOWER(so.status) NOT IN ('completed', 'pending');
  employee   | order_id | order_date | quantity | prod_id |  status   | customer_id | emp_id 
-------------+----------+------------+----------+---------+-----------+-------------+--------
 Abrar Khan  |        2 | 2024-01-01 |        3 |       1 | Pending   |           2 |      2
 Abrar Khan  |        3 | 2024-01-02 |        3 |       2 | Completed |           3 |      2
 Abrar Khan  |        4 | 2024-01-03 |        3 |       3 | Completed |           3 |      2
 Abrar Khan  |        5 | 2024-01-04 |        1 |       1 | Completed |           3 |      2
 Nina Kumari |        7 | 2024-01-04 |        1 |       2 | On Hold   |           2 |      1
 Abrar Khan  |        8 | 2024-01-05 |        4 |       2 | Rejected  |           1 |      2
 Abrar Khan  |        9 | 2024-01-06 |        5 |       5 | Completed |           1 |      2
 Nina Kumari |       10 | 2024-01-06 |        1 |       1 | Cancelled |           1 |      1
(8 rows)
```

12. Fetch the orders which cost more than 2000 but did not include the MacBook Pro. Print the total sale amount as well.

```
demo=# SELECT (so.quantity * p.price) AS total_sale, so.*
FROM sales_order so
JOIN products p ON p.id = so.prod_id
WHERE p.product_name <> 'Macbook Pro'
AND (so.quantity * p.price) > 2000;
 total_sale | order_id | order_date | quantity | prod_id |  status   | customer_id | emp_id 
------------+----------+------------+----------+---------+-----------+-------------+--------
     205800 |        1 | 2024-01-01 |        2 |       1 | Completed |           1 |      1
     308700 |        2 | 2024-01-01 |        3 |       1 | Pending   |           2 |      2
     392700 |        3 | 2024-01-02 |        3 |       2 | Completed |           3 |      2
     449700 |        4 | 2024-01-03 |        3 |       3 | Completed |           3 |      2
     102900 |        5 | 2024-01-04 |        1 |       1 | Completed |           3 |      2
     149900 |        6 | 2024-01-04 |        1 |       3 | completed |           2 |      1
     130900 |        7 | 2024-01-04 |        1 |       2 | On Hold   |           2 |      1
     523600 |        8 | 2024-01-05 |        4 |       2 | Rejected  |           1 |      2
     102900 |       10 | 2024-01-06 |        1 |       1 | Cancelled |           1 |      1
(9 rows)
```

13. Identify the customers who have not purchased any product yet.

```
demo=# SELECT *
FROM customers c
WHERE NOT EXISTS (
    SELECT 1
    FROM sales_order so
    WHERE so.customer_id = c.id
);
 id |    name    |     email      
----+------------+----------------
  4 | Zaria Duke | zduke@demo.com
(1 row)

demo=# SELECT c.*
FROM customers c
LEFT JOIN sales_order so ON so.customer_id = c.id
WHERE so.customer_id IS NULL;
 id |    name    |     email      
----+------------+----------------
  4 | Zaria Duke | zduke@demo.com
(1 row)
```

14. Write a query to identify the total products purchased by each customer. Return all customers irrespective of whether they have made a purchase or not. Sort the result with the highest no of orders at the top.

```
SELECT c.id AS "Customer Id",
       c.name AS "Customer Name",
       COALESCE(SUM(so.quantity), 0) AS "Total items purchased"
FROM customers c
LEFT JOIN sales_order so ON so.customer_id = c.id
GROUP BY c.id, c.name
ORDER BY "Total items purchased" DESC;


 Customer Id | Customer Name | Total items purchased 
-------------+---------------+-----------------------
           1 | Meghan Harley |                    12
           3 | Logan Short   |                     7
           2 | Rosa Chan     |                     5
           4 | Zaria Duke    |                     0
(4 rows)
```

15. Corresponding to each employee, display the total sales they made of all the completed orders. Display total sales as 0 if an employee made no sales yet.


```
demo=# SELECT e.name AS "Employee Name",
       COALESCE(SUM(so.quantity * p.price), 0) AS "Total Sales"
FROM employees e
LEFT JOIN sales_order so
       ON so.emp_id = e.id
      AND LOWER(so.status) = 'completed'
LEFT JOIN products p
       ON p.id = so.prod_id
GROUP BY e.name
ORDER BY "Total Sales" DESC;
 Employee Name | Total Sales 
---------------+-------------
 Abrar Khan    |      955800
 Nina Kumari   |      355700
 Irene Costa   |           0
(3 rows)
```

16. Re-write the above query to display the total sales made by each employee corresponding to each customer. If an employee has not served a customer yet then display "-" under the customer.

```
demo=# SELECT
    e.name AS employee,
    COALESCE(c.name, '-') AS customer,
    COALESCE(SUM(p.price * so.quantity), 0) AS total_sale
FROM employees e
LEFT JOIN sales_order so
       ON so.emp_id = e.id
      AND LOWER(so.status) = 'completed'
LEFT JOIN customers c
       ON c.id = so.customer_id
LEFT JOIN products p
       ON p.id = so.prod_id
GROUP BY e.name, c.name
ORDER BY total_sale DESC;
  employee   |   customer    | total_sale 
-------------+---------------+------------
 Abrar Khan  | Logan Short   |     945300
 Nina Kumari | Meghan Harley |     205800
 Nina Kumari | Rosa Chan     |     149900
 Abrar Khan  | Meghan Harley |      10500
 Irene Costa | -             |          0
(5 rows)
```

17. Re-write the above query to display only those records where the total sales are above 1000

```
demo=# SELECT
    e.name AS employee,
    COALESCE(c.name, '-') AS customer,
    COALESCE(SUM(p.price * so.quantity), 0) AS total_sale
FROM employees e
LEFT JOIN sales_order so
       ON so.emp_id = e.id
      AND LOWER(so.status) = 'completed'
LEFT JOIN customers c
       ON c.id = so.customer_id
LEFT JOIN products p
       ON p.id = so.prod_id
GROUP BY e.name, c.name
HAVING SUM(p.price * so.quantity) > 11000
ORDER BY total_sale DESC;
  employee   |   customer    | total_sale 
-------------+---------------+------------
 Abrar Khan  | Logan Short   |     945300
 Nina Kumari | Meghan Harley |     205800
 Nina Kumari | Rosa Chan     |     149900
(3 rows)
```

18. Identify employees who have served more than 2 customers.

```
demo=# SELECT e.name AS "Employee Name",
       COUNT(DISTINCT c.name) AS "Total customers"
FROM sales_order so
JOIN employees e ON e.id = so.emp_id
JOIN customers c ON c.id = so.customer_id
GROUP BY e.name
HAVING COUNT(DISTINCT c.name) > 2;
 Employee Name | Total customers 
---------------+-----------------
 Abrar Khan    |               3
(1 row)
```

19. Identify the customers who have purchased more than 5 products

```
demo=# SELECT c.id AS "Customer Id",
       c.name AS "Customer Name",
       SUM(so.quantity) AS "Total products purchased"
FROM sales_order so
LEFT JOIN customers c ON so.customer_id = c.id
GROUP BY c.id, c.name
HAVING SUM(so.quantity) > 5;
 Customer Id | Customer Name | Total products purchased 
-------------+---------------+--------------------------
           1 | Meghan Harley |                       12
           3 | Logan Short   |                        7
(2 rows)
```

20. Identify customers whose average purchase cost exceeds the average sale of all the orders.

```
demo=# SELECT
    c.id AS "Customer Id",
    c.name AS "Customer Name"
FROM sales_order so
JOIN customers c ON c.id = so.customer_id
JOIN products p ON p.id = so.prod_id
GROUP BY c.id, c.name
HAVING AVG(so.quantity * p.price) >
       (
           SELECT AVG(so2.quantity * p2.price)
           FROM sales_order so2
           JOIN products p2 ON p2.id = so2.prod_id
       );
 Customer Id | Customer Name 
-------------+---------------
           3 | Logan Short
(1 row)
```