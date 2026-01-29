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

