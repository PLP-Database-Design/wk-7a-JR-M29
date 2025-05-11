# 📝 Assignment: Database Design and Normalization

## 🎯 **Learning Objectives**
* 🛠️ **Understand the principles of good database design** and **normalization**.
* 💡 **Apply normalization techniques** to improve database structure and efficiency.
* 🔍 **Learn First, Second, and Third Normal Forms** (1NF, 2NF, 3NF) to eliminate redundancy and optimize data storage.

---

## 📋 **What You'll Need**
* 💻 A computer with internet access.
* ✍️ A code editor (e.g., Visual Studio Code).
* 🖥️ MySQL Workbench or another SQL database environment.

---


## 📝 Submission Instructions  
📂 Write all your SQL queries in the **answers.sql** file.  
✍️ Answer each question concisely and make sure your queries are clear and correct.  
🗣️ Structure your responses clearly, and use comments if necessary to explain your approach.

--- 

## 📚 Assignment Questions

---

### Question 1 Achieving 1NF (First Normal Form) 🛠️
Task:
- You are given the following table **ProductDetail**:

| OrderID | CustomerName  | Products                        |
|---------|---------------|---------------------------------|
| 101     | John Doe      | Laptop, Mouse                   |
| 102     | Jane Smith    | Tablet, Keyboard, Mouse         |
| 103     | Emily Clark   | Phone                           |


- In the table above, the **Products column** contains multiple values, which violates **1NF**.
- **Write an SQL query** to transform this table into **1NF**, ensuring that each row represents a single product for an order

--- 

### Question 2 Achieving 2NF (Second Normal Form) 🧩

- You are given the following table **OrderDetails**, which is already in **1NF** but still contains partial dependencies:

| OrderID | CustomerName  | Product      | Quantity |
|---------|---------------|--------------|----------|
| 101     | John Doe      | Laptop       | 2        |
| 101     | John Doe      | Mouse        | 1        |
| 102     | Jane Smith    | Tablet       | 3        |
| 102     | Jane Smith    | Keyboard     | 1        |
| 102     | Jane Smith    | Mouse        | 2        |
| 103     | Emily Clark   | Phone        | 1        |

- In the table above, the **CustomerName** column depends on **OrderID** (a partial dependency), which violates **2NF**. 

- Write an SQL query to transform this table into **2NF** by removing partial dependencies. Ensure that each non-key column fully depends on the entire primary key.

---
Good luck 🚀



```
-- Create a new table to store the normalized data
CREATE TABLE ProductDetail_1NF (
    OrderID INT,
    CustomerName VARCHAR(255),
    Product VARCHAR(255)
);

-- Insert data into the new table, splitting the Products column
INSERT INTO ProductDetail_1NF (OrderID, CustomerName, Product)
SELECT 
    OrderID,
    CustomerName,
    TRIM(value) AS Product
FROM 
    ProductDetail,
    JSON_TABLE(
        CONCAT('[', REPLACE(Products, ', ', '","'), ']'),
        '$[*]' COLUMNS (value VARCHAR(255) PATH '$')
    ) AS ProductsSplit;

-- Alternatively, if your database system supports a string split function
-- You might use something like the following (the exact syntax may vary):
-- INSERT INTO ProductDetail_1NF (OrderID, CustomerName, Product)
-- SELECT 
--     OrderID,
--     CustomerName,
--     SPLIT_PART(Products, ', ', index) AS Product
-- FROM 
--     ProductDetail,
--     GENERATE_SERIES(1, ARRAY_LENGTH(SPLIT(Products, ', '))) AS index;

-- Select the normalized data
SELECT * FROM ProductDetail_1NF;
```


```
-- Create a new table to store order information
CREATE TABLE Orders (
    OrderID INT PRIMARY KEY,
    CustomerName VARCHAR(255)
);

-- Create a new table to store order details
CREATE TABLE OrderDetails_2NF (
    OrderID INT,
    Product VARCHAR(255),
    Quantity INT,
    PRIMARY KEY (OrderID, Product),
    FOREIGN KEY (OrderID) REFERENCES Orders(OrderID)
);

-- Insert data into the Orders table
INSERT INTO Orders (OrderID, CustomerName)
SELECT DISTINCT OrderID, CustomerName
FROM OrderDetails;

-- Insert data into the OrderDetails_2NF table
INSERT INTO OrderDetails_2NF (OrderID, Product, Quantity)
SELECT OrderID, Product, Quantity
FROM OrderDetails;

-- Select the normalized data
SELECT * FROM Orders;
SELECT * FROM OrderDetails_2NF;
```

