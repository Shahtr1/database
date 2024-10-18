# SQL Interview Questions

1. What are the different types of SQL commands (DDL, DML, DCL, TCL)?
    - DDL (Data Definition Language):
      - DDL commands are used to define or modify the structure of database objects, such as tables, indexes, and views. 
      - These commands affect the schema of the database.
      - CREATE, ALTER, DROP, TRUNCATE, RENAME
    - DML (Data Manipulation Language):
      - DML commands are used to manipulate data stored in the database. 
      - INSERT, UPDATE, DELETE, SELECT
    - DCL (Data Control Language):
      - DCL commands are used to control access to the database by granting or revoking permissions.
      - GRANT, REVOKE
      - `GRANT SELECT ON Employees TO John;`
      - `REVOKE SELECT ON Employees FROM John;`
    - TCL (Transaction Control Language):
      - TCL commands are used to manage transactions in the database. 
      - A transaction is a group of operations that must either all succeed or all fail.
      - COMMIT, ROLLBACK
      - SAVEPOINT: Creates a point within a transaction to which you can roll back.
      ```sql
        -- Start a transaction
        BEGIN;
        
        -- Insert a new employee
        INSERT INTO Employees (id, name, department) VALUES (2, 'Bob', 'IT');
        
        -- Commit the transaction
        COMMIT;
        
        -- Rollback the changes in case of an error
        ROLLBACK;
        
        -- Create a savepoint within a transaction
        SAVEPOINT emp_save;
        INSERT INTO Employees (id, name, department) VALUES (3, 'Charlie', 'Sales');
        ROLLBACK TO emp_save;  -- Undo changes after the savepoint
        ```
      - SET TRANSACTION: Sets the properties of the current transaction (e.g., isolation level).
         -  Isolation Levels:
            `READ COMMITTED`: A query reads only committed data.
            
            `REPEATABLE READ`: A query can repeatedly read the same data within the transaction.
        
            `SERIALIZABLE`: Ensures complete isolation (most strict but can reduce performance).
        ```sql
        BEGIN;
        
        SET TRANSACTION READ ONLY ISOLATION LEVEL READ COMMITTED;
        
        -- Try to read data (this will work)
        SELECT * FROM Employees;
        
        -- Try to modify data (this will fail because the transaction is read-only)
        UPDATE Employees SET department = 'Finance' WHERE id = 1;
        
        COMMIT;
        ```
        ```sql
        BEGIN;
        
        SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
        
        -- Query 1: Select all employees
        SELECT * FROM Employees;
        
        -- Another concurrent transaction tries to modify the same table (but is blocked due to SERIALIZABLE level).
        
        -- Commit the transaction to release the lock
        COMMIT;
        ```

2. What is the difference between `WHERE` and `HAVING` clauses?
    - The `WHERE` clause is used to filter rows before any grouping or aggregation.
    - The `HAVING` clause filters the result of a GROUP BY query. It is used after aggregation to filter groups based on aggregate values.
    ```sql
    CREATE TABLE Employees (
    employee_id INT PRIMARY KEY,
    employee_name VARCHAR(100),
    department VARCHAR(50),
    salary DECIMAL(10, 2)
    );
    ---
    INSERT INTO Employees (employee_id, employee_name, department, salary) VALUES
    (1, 'Alice', 'IT', 60000.00),
    (2, 'Bob', 'IT', 45000.00),
    (3, 'Charlie', 'Finance', 70000.00),
    (4, 'David', 'Finance', 52000.00),
    (5, 'Eva', 'Finance', 51000.00),
    (6, 'Frank', 'HR', 48000.00),
    (7, 'Grace', 'HR', 51000.00),
    (8, 'Hank', 'IT', 72000.00);
    ---
    SELECT department, COUNT(employee_id) AS total_employees
    FROM Employees
    WHERE salary > 50000  -- Filters employees with salary > 50000
    GROUP BY department
    HAVING COUNT(employee_id) > 2;  -- Filters departments with more than 2 employees
    ```
   | **department** | **total_employees** |
   |----------------|---------------------|
   | Finance        | 3                   |

3. What is a JOIN? List the different types of JOINs in SQL.
    - `INNER JOIN`: Returns only the matching rows from both tables.
    - `LEFT (OUTER) JOIN`: Returns all rows from the left table, and matching rows from the right table.
    - `RIGHT (OUTER) JOIN`: Returns all rows from the right table, and matching rows from the left table.
    - `FULL (OUTER) JOIN`: Returns all rows when there is a match in either left or right table.
    - `CROSS JOIN`: Returns the Cartesian product of two tables (all possible combinations).
    - `SELF JOIN`: Joins a table to itself.

4. What is a unique key, and how is it different from a primary key?
    - A Unique Key ensures that all values in the column (or group of columns) are unique across all rows in the table.
    - Null values are allowed (one per column) in Unique Key
    - Unique Key creates a non-clustered index while as primary key creates a clustered one

5. What is the difference between `DELETE`, `TRUNCATE`, and `DROP`?
    - The `DELETE` command is used to remove specific rows from a table based on a condition.
    - The `TRUNCATE` command removes all rows from a table without logging individual row deletions. This makes it faster than `DELETE` for large tables.
    - The `DROP` command is used to remove the entire table (or other database objects) along with all its data and structure.

6. How do you fetch the top 5 highest salaries from a table?
   - Here's how
     ```sql
        CREATE TABLE Employees (
        employee_id INT PRIMARY KEY,
        employee_name VARCHAR(100),
        salary DECIMAL(10, 2)
        ); 
     INSERT INTO Employees (employee_id, employee_name, salary) VALUES
      (1, 'Alice', 120000.00),
      (2, 'Bob', 110000.00),
      (3, 'Charlie', 105000.00),
      (4, 'David', 100000.00),
      (5, 'Eva', 95000.00),
      (6, 'Frank', 95000.00),  -- Same salary as Eva (testing ties)
      (7, 'Grace', 85000.00);

     select employee_name, salary from employees order by salary desc limit 5;
     select top 5 employee_name, salary from employees order by salary desc;
     -- If there are ties (employees with the same salary)
     select employee_name, salary from (
        select employee_name, salary, RANK() OVER (ORDER BY salary DESC) as rnk
        from employees  
        ) as ranked_employees where rnk <= 5;
     ```
     | **employee_name** | **salary**  |
     |-------------------|-------------|
     | Alice             | 120000.00   |
     | Bob               | 110000.00   |
     | Charlie           | 105000.00   |
     | David             | 100000.00   |
     | Eva               | 95000.00    |
     | Frank             | 95000.00    |

7. What are SQL constraints?
    - `NOT NULL`
    - `UNIQUE`
    - `PRIMARY KEY`
    - `FOREIGN KEY`
    - `CHECK`
    - `DEFAULT`

8. How to Work with NULL in SQL?
   - Checking for NULL Values Using `IS NULL` or `IS NOT NULL`:
   - `COALESCE(email, 'Not Provided') AS email`
   - `IFNULL(email, 'No Email') AS email`
   - `SELECT AVG(salary) FROM Employees;  -- Ignores NULL salaries`

9. What is the purpose of the `ALTER` command?
    - The ALTER command in SQL is used to modify the structure of an existing database object
    ```sql
    ALTER TABLE table_name
    ADD | MODIFY | DROP | RENAME column_name ...;
    ```

10. What are aggregate functions in SQL? Provide examples.
    - aggregate functions perform a calculation on a set of values and return a single result.
    - `COUNT()`, `SUM()`, `AVG()`, `MIN()`, `MAX()`

11. Types of Indexes
    - Single-Column Index: 
    ```sql
    CREATE INDEX idx_employee_name ON Employees (employee_name);
    ```
    - Composite Index: Index on multiple columns.
    ```sql
    CREATE INDEX idx_dept_salary ON Employees (department, salary);
    ```
    - Unique Index: Ensures that the indexed column(s) have unique values.
    ```sql
    -- ensures all are unique
    CREATE UNIQUE INDEX idx_unique_email ON Employees (email);
    ```
    - Clustered Index: Sorts and stores the table data based on the indexed column.
    ```sql
    -- Primary keys are usually implemented as clustered indexes by default
    -- sorts and stores the data rows in the table based on the indexed column.
    -- Only one clustered index is allowed per table, usually on the primary key.
    CREATE CLUSTERED INDEX idx_employee_id ON Employees (employee_id);
    ```
    - Non-Clustered Index: Creates a separate structure to store pointers to the data.
    ```sql
    -- creates a separate structure from the data rows and stores pointers to the table rows.
    -- Multiple non-clustered indexes can be created on a table.
    CREATE NONCLUSTERED INDEX idx_salary ON Employees (salary);
    ```
    - Full-Text Index: Used for full-text searches in columns containing large text data.
        - A Full-Text Index is a special type of index used to perform fast text-based searches in columns that contain large textual data, such as articles, descriptions, or product details.
        - Full-text indexes provide better performance than `LIKE` or `ILIKE` operations.
    ```sql
    CREATE TABLE Products (
    product_id INT PRIMARY KEY,
    product_name VARCHAR(100),
    product_description TEXT
    );
    --
    INSERT INTO Products (product_id, product_name, product_description) VALUES
    (1, 'Laptop', 'A high-performance laptop with 16GB RAM and 512GB SSD.'),
    (2, 'Smartphone', 'A smartphone with a powerful camera and 5G connectivity.'),
    (3, 'Headphones', 'Noise-cancelling headphones with high fidelity sound.'),
    (4, 'Smartwatch', 'A smartwatch with fitness tracking and long battery life.');
    --
    CREATE FULLTEXT INDEX idx_description ON Products (product_description);
    --
    SELECT product_name, product_description
    FROM Products
    WHERE MATCH(product_description) AGAINST('laptop');
    ---
    SELECT product_name, product_description
    FROM Products
    WHERE MATCH(product_description)
    AGAINST('high performance laptop' IN NATURAL LANGUAGE MODE);
    -- It returns products based on relevance. If some rows contain all the keywords, they will rank higher than rows that contain partial matches.
    ---
    SELECT product_name, product_description
    FROM Products
    WHERE MATCH(product_description)
    AGAINST('+laptop -"5G connectivity"' IN BOOLEAN MODE);
    -- This query returns only products that mention "laptop" and exclude those that mention "5G connectivity."
    ```

12. What is a view, and operations on it?
    - A view in SQL is a virtual table that is based on the result of a SELECT query. 
    - It does not store data itself but provides a layer of abstraction over one or more underlying tables. 
    - Views make it easier to query complex data by encapsulating frequently used SQL queries into a reusable structure.
    - Encapsulates complex joins, filters, or aggregations into a simple query.
    - You can restrict access to specific columns or rows by exposing only a subset of data through a view.
    ```sql
    CREATE TABLE Employees (
    employee_id INT PRIMARY KEY,
    employee_name VARCHAR(100),
    department VARCHAR(50),
    salary DECIMAL(10, 2)
    );
    ---
    INSERT INTO Employees (employee_id, employee_name, department, salary) VALUES
    (1, 'Alice', 'IT', 60000.00),
    (2, 'Bob', 'Finance', 55000.00),
    (3, 'Charlie', 'HR', 50000.00);
    ---
    CREATE VIEW HighSalaryEmployees AS
    SELECT employee_id, employee_name, salary
    FROM Employees
    WHERE salary > 55000;
    ---
    SELECT * FROM HighSalaryEmployees;
    ```
    - Updating Data Through a View
        - The view references only one table.
        - The update affects columns that exist in the view.
        - The view does not use aggregations or joins.
    ```sql
    UPDATE HighSalaryEmployees
    SET salary = 65000.00
    WHERE employee_id = 1;
    ```

13. What is the difference between `UNION` and `UNION ALL`?
    - `UNION`: Performs an implicit DISTINCT on the result set to eliminate duplicates.
    - `UNION ALL`: Includes all rows, even if they are duplicates.
    
14. What is a sequence, and how do you create one?
    - A sequence is a database object that generates unique numeric values in a sequential order.
    ```sql
    CREATE SEQUENCE emp_seq
    START WITH 1
    INCREMENT BY 1
    MINVALUE 1
    MAXVALUE 1000
    CYCLE; -- When the sequence reaches 1000, it restarts from 1.
    --
    INSERT INTO Employees (employee_id, employee_name, department)
    VALUES (emp_seq.NEXTVAL, 'Alice', 'IT');
    --
    SELECT emp_seq.CURRVAL FROM DUAL; -- current value of sequence
    ```

15. What is the DUAL Table in SQL?
    - The DUAL table is a special one-row, one-column table present in Oracle databases. 
    - It is used primarily to select system values or perform calculations when you don't need data from any specific table.
    - It is a temporary utility table that contains exactly one row with a single column named `DUMMY`;
    - `SELECT 1 + 1 AS result FROM DUAL;`
    - `SELECT SYSDATE AS today FROM DUAL;`
    - `SELECT emp_seq.NEXTVAL FROM DUAL;`

16. How do you use the `CASE` statement in SQL?
    - The CASE statement in SQL is used to create conditional logic within queries.
    ```sql
    SELECT
    employee_name,
    salary,
    CASE
    WHEN salary >= 70000 THEN 'High Salary'
    WHEN salary BETWEEN 50000 AND 69999 THEN 'Medium Salary'
    ELSE 'Low Salary'
    END AS salary_category
    FROM Employees;
    ```

17. How do you perform pattern matching using wildcards in SQL?
    - `%`: Zero or more characters, `LIKE 'A%'` matches `Alice` and `Adam`.
    - `_`: Exactly one character, `LIKE 'B_b'` matches `Bob`.
    - `[ ]`: Any character in brackets, `LIKE 'B[ao]b'` matches `Bab` or `Bob`.
    - `[^ ]`: Any character not in brackets, `LIKE 'B[^o]b'` excludes `Bob`.

18. What are stored procedures in SQL? Provide an example.
    - Store logic once and reuse it multiple times.
    - Precompiled, so it runs faster than executing individual SQL queries.
    - Can restrict access to data by controlling the operations allowed through the procedure.
    - Only the procedure call is sent over the network instead of multiple queries.
    ```sql
    CREATE TABLE Employees (
    employee_id INT PRIMARY KEY,
    employee_name VARCHAR(100),
    department VARCHAR(50),
    salary DECIMAL(10, 2)
    );
    ---
    DELIMITER //
    
    CREATE PROCEDURE AddEmployee (
    IN emp_id INT,
    IN emp_name VARCHAR(100),
    IN emp_department VARCHAR(50),
    IN emp_salary DECIMAL(10, 2)
    )
    BEGIN
    INSERT INTO Employees (employee_id, employee_name, department, salary)
    VALUES (emp_id, emp_name, emp_department, emp_salary);
    END //
    
    DELIMITER ;
    --
    CALL AddEmployee(1, 'Alice', 'IT', 60000.00);
    --
    DELIMITER //
    
    CREATE PROCEDURE GetSalary (
    IN emp_id INT,
    OUT emp_salary DECIMAL(10, 2)
    )
    BEGIN
    SELECT salary INTO emp_salary
    FROM Employees
    WHERE employee_id = emp_id;
    END //
    
    DELIMITER ;
    --
    SET @salary = 0;
    CALL GetSalary(1, @salary);
    SELECT @salary AS EmployeeSalary;
    --
    DELIMITER //

    CREATE PROCEDURE IncrementSalary (INOUT emp_salary DECIMAL(10, 2))
    BEGIN
    SET emp_salary = emp_salary * 1.10;  -- Increase salary by 10%
    END //
    
    DELIMITER ;
    --
    SET @current_salary = 60000.00;
    CALL IncrementSalary(@current_salary);
    SELECT @current_salary AS UpdatedSalary;
    ```

19. What is a trigger, and how is it used?
    - A trigger is a special type of stored procedure that is automatically executed (or fired) when a specific event occurs in the database. 
    - Triggers are associated with tables or views, and they are used to enforce business rules, maintain data integrity, or perform auditing.
    - `BEFORE` Trigger: Executes before the triggering event (INSERT/UPDATE/DELETE).
    ```sql
    DELIMITER //
    
    CREATE TRIGGER before_employee_insert
    BEFORE INSERT ON Employees
    FOR EACH ROW
     BEGIN
       IF NEW.salary < 0 THEN
         SIGNAL SQLSTATE '45000'
         SET MESSAGE_TEXT = 'Salary cannot be negative';
       END IF;
     END //
    
    DELIMITER ;
    ```
    - `AFTER` Trigger: Executes after the triggering event.
    ```sql
    DELIMITER //
    
    CREATE TRIGGER after_employee_insert
    AFTER INSERT ON Employees
    FOR EACH ROW
    BEGIN
      INSERT INTO Employee_Audit (employee_id, employee_name, action, action_time)
      VALUES (NEW.employee_id, NEW.employee_name, 'INSERT', NOW());
    END //
    
    DELIMITER ;
    ```
    - `INSTEAD OF` Trigger: Used with views to replace the standard action with a custom one.
    ```sql
    CREATE VIEW Employee_View AS
    SELECT employee_id, employee_name FROM Employees;
    
    DELIMITER //
    
    CREATE TRIGGER instead_of_view_update
    INSTEAD OF UPDATE ON Employee_View
    FOR EACH ROW
    BEGIN
       UPDATE Employees
       SET employee_name = NEW.employee_name
       WHERE employee_id = OLD.employee_id;
    END //
    
    DELIMITER ;
    ```

20. What are ACID properties in SQL transactions?
    - Atomicity:
        - Atomicity ensures that a transaction is all-or-nothing.
        - If any part of the transaction fails, the entire transaction is rolled back.
        - If the transaction succeeds, all changes are committed to the database.
    - Consistency:
        - Ensures that a transaction takes the database from one consistent state to another.
        - All data integrity constraints (like foreign keys, unique constraints, etc.) must be maintained.
    - Isolation:
        - Ensures that concurrent transactions are executed independently without interfering with each other.
        - Changes made by one transaction are invisible to others until the transaction is committed.
    - Durability:
        - Ensures that once a transaction is committed, the changes are permanently stored, even if the system crashes afterward.

21. What is normalization, and what are the different forms?
    - Normalization is the process of organizing data in a relational database to reduce redundancy and dependency. 
    - It involves dividing large tables into smaller, related tables and linking them through relationships.
    - Normal Forms (NF) in SQL:
        - First Normal Form (1NF):
            - Each column must contain atomic values (indivisible).
            - Each column should contain only one value per row.
            - All entries in a column must be of the same type.
          - Non-1NF Table
      
            | OrderID | CustomerName | Products               |
            |---------|--------------|------------------------|
            | 1       | Alice        | Laptop, Smartphone     |
            | 2       | Bob          | Headphones             |
            
          - 1NF - Solution

            | OrderID | CustomerName | Product    |
            |---------|--------------|------------|
            | 1       | Alice        | Laptop     |
            | 1       | Alice        | Smartphone |
            | 2       | Bob          | Headphones |
        
        - Second Normal Form (2NF):
            - The table must be in 1NF.
            - All non-key columns must be fully dependent on the primary key (not a part of it).
          - Non-2NF Table
          
            | OrderID | ProductID | CustomerName | ProductName | CustomerAddress |
            |---------|-----------|--------------|-------------|-----------------|
            | 1       | 101       | Alice        | Laptop      | 123 Street      |
            | 2       | 102       | Bob          | Headphones  | 456 Avenue      |
          - Issue: CustomerName and CustomerAddress depend only on OrderID, not on both OrderID and ProductID.
          - 2NF Solution:

            | OrderID | CustomerName | CustomerAddress |
            |---------|--------------|-----------------|
            | 1       | Alice        | 123 Street      |
            | 2       | Bob          | 456 Avenue      |

            | OrderID | ProductID | ProductName |
            |---------|-----------|-------------|
            | 1       | 101       | Laptop      |
            | 2       | 102       | Headphones  |
          - The table is split into two, ensuring that non-key attributes are fully dependent on the primary key.
        
        - Third Normal Form (3NF):
            - Eliminate transitive dependencies.
            - The table must be in 2NF.
            - Non-key columns must depend only on the primary key and not on other non-key columns.
          - Non-3NF Table:

            | OrderID | CustomerID | CustomerName | CustomerAddress |
            |---------|------------|--------------|-----------------|
            | 1       | 1          | Alice        | 123 Street      |
            | 2       | 2          | Bob          | 456 Avenue      |
          
          - 3NF Solution:

            | OrderID | CustomerID |
            |---------|------------|
            | 1       | 1          |
            | 2       | 2          |

            | CustomerID | CustomerName | CustomerAddress |
            |------------|--------------|-----------------|
            | 1          | Alice        | 123 Street      |
            | 2          | Bob          | 456 Avenue      |

          - Boyce-Codd Normal Form (BCNF):
            - Handle anomalies not covered by 3NF.
            - The table must be in 3NF.
            - For every functional dependency A -> B, A must be a super key.

            - Non-BCNF Table:

            | Course  | Instructor  | Classroom |
            |---------|-------------|-----------|
            | Math    | Dr. Smith   | Room 101  |
            | Science | Dr. Brown   | Room 102  |
            - If one instructor teaches multiple courses, this setup can lead to inconsistencies.

            - BCNF Solution:

            | Course  | InstructorID |
            |---------|--------------|
            | Math    | 1            |
            | Science | 2            |

            | InstructorID | InstructorName | Classroom |
            |--------------|----------------|-----------|
            | 1            | Dr. Smith      | Room 101  |
            | 2            | Dr. Brown      | Room 102  |

22. What is denormalization? When is it useful?
    - Denormalization helps speed up read-heavy operations by reducing the need for complex joins across multiple tables.

23. How do you find duplicate records in a table?
    - when two or more rows in a table contain the same data in one or more columns.
    ```sql
    SELECT column1, column2, COUNT(*)
    FROM table_name
    GROUP BY column1, column2
    HAVING COUNT(*) > 1;
    ```

24. What is `FOREACH`?
    - In SQL, `FOREACH` is not a native SQL command across all databases but is commonly used in procedural SQL environments
    - `PostgreSQL` supports array iteration with FOREACH loops.
    - `TSQL`(SQL Server): Uses `WHILE` loops or `CURSORS` for similar functionality, as there is no direct FOREACH.
    -  `MySQL` does not support FOREACH directly but offers `LOOP` and `CURSOR` constructs.

25. What is `PARTITION`?
    - `PARTITION` refers to dividing a table or query results into smaller, manageable pieces based on a specific column or set of columns
    - Partitioning Tables (Physical Partitioning)
        - Range Partitioning:
        ```sql
           CREATE TABLE Orders (
           order_id INT,
           order_date DATE,
           amount DECIMAL(10, 2)
           ) PARTITION BY RANGE (order_date);
           --
           CREATE TABLE Orders_2023 PARTITION OF Orders FOR VALUES FROM ('2023-01-01') TO ('2024-01-01');
           CREATE TABLE Orders_2022 PARTITION OF Orders FOR VALUES FROM ('2022-01-01') TO ('2023-01-01');
           -- creates a table Orders that is partitioned by order_date ranges.
           -- Each partition stores orders within a specific year.
        ```
        - List Partitioning:
        ```sql
           CREATE TABLE Employees (
                                      employee_id INT,
                                      employee_name VARCHAR(100),
                                      department VARCHAR(50)
           ) PARTITION BY LIST (department);
        -- 
        CREATE TABLE IT_Employees PARTITION OF Employees FOR VALUES IN ('IT');
        CREATE TABLE HR_Employees PARTITION OF Employees FOR VALUES IN ('HR');
        CREATE TABLE Finance_Employees PARTITION OF Employees FOR VALUES IN ('Finance');
        ```
        - Hash Partitioning:
        ```sql
           CREATE TABLE Orders (
                                   order_id INT,
                                   order_date DATE,
                                   amount DECIMAL(10, 2)
           ) PARTITION BY HASH (order_id);
        -- 
        CREATE TABLE Orders_p0 PARTITION OF Orders FOR VALUES WITH (MODULUS 4, REMAINDER 0);
        CREATE TABLE Orders_p1 PARTITION OF Orders FOR VALUES WITH (MODULUS 4, REMAINDER 1);
        CREATE TABLE Orders_p2 PARTITION OF Orders FOR VALUES WITH (MODULUS 4, REMAINDER 2);
        CREATE TABLE Orders_p3 PARTITION OF Orders FOR VALUES WITH (MODULUS 4, REMAINDER 3);
        -- The Orders table is partitioned by the hash of the order_id.
        -- The modulus and remainder ensure that the rows are evenly distributed across 4 partitions
        ```
        - Composite Partitioning:
        ```sql
           CREATE TABLE Orders (
                                   order_id INT,
                                   order_date DATE,
                                   amount DECIMAL(10, 2)
           ) PARTITION BY RANGE (YEAR(order_date))
        SUBPARTITION BY HASH (order_id);
    
        -- 
        CREATE TABLE Orders_2023 PARTITION OF Orders
            FOR VALUES FROM (2023) TO (2024)
            SUBPARTITIONS 2;
        -- Each yearly partition is further hashed into subpartitions based on the order_id.
        ```
      - Partitioning in SQL Queries using `PARTITION BY` (Logical Partitioning)
        - The PARTITION BY clause is used with window functions (like `ROW_NUMBER()`, `RANK()`, `SUM()`) to group data logically into partitions within a query result.
        ```sql
           SELECT 
          employee_name,
          department,
          salary,
          ROW_NUMBER() OVER (PARTITION BY department ORDER BY salary DESC) AS rank
         FROM Employees;
        ```

        | employee_name | department | salary  | rank |
        |---------------|------------|---------|------|
        | Alice         | IT         | 70000.00 | 1    |
        | Bob           | IT         | 65000.00 | 2    |
        | Charlie       | HR         | 50000.00 | 1    |
        | David         | HR         | 48000.00 | 2    |
      - The `PARTITION BY department` divides the employees into two partitions (IT and HR).
      - Within each partition, `ROW_NUMBER()` assigns a rank based on the salary in descending order.

26. What is `WITH`?
    - The WITH clause in SQL is used to define Common Table Expressions (CTEs). 
    - A CTE is a temporary result set that exists only within the scope of the query. 
    - It can be used to simplify complex queries by breaking them into smaller, more readable parts. 
    - CTEs make it easier to reuse query logic and enhance code readability.
    ```sql
    WITH IT_Employees AS (
        SELECT employee_name, salary 
        FROM Employees 
        WHERE department = 'IT'
    ),
    HR_Employees AS (
        SELECT employee_name, salary 
        FROM Employees 
        WHERE department = 'HR'
    )
    SELECT * 
    FROM IT_Employees
    UNION ALL
    SELECT * 
    FROM HR_Employees;
    ```

27. What are window functions?
    - A window function is a special type of SQL function that performs calculations across a subset of rows (a window), 
    - but does not collapse the result into a single value like aggregate functions (e.g., SUM(), COUNT()
    - Each row retains its original identity in the result set.
    ```sql
    function_name() OVER (
    [PARTITION BY column1, column2, ...]
    [ORDER BY column_name [ASC | DESC]]
    [ROWS or RANGE clause]
    ) AS alias_name
    ```
    - The function to perform, such as ROW_NUMBER(), RANK(), SUM(), etc.
    - Defines the "window" over which the function is applied.
    - Using `LAG()` to Compare Salaries with Previous Employees
    ```sql
    SELECT
    employee_name,
    salary,
    LAG(salary) OVER (ORDER BY salary DESC) AS previous_salary
    FROM Employees;
    ```
      - LAG() retrieves the previous row's salary based on the salary order. For the first row, NULL is returned because there is no previous row.

28. How do you delete duplicate records from a table?
    - If you want to keep only one record from each group of duplicates, you can use the `ROW_NUMBER()` function to assign a unique row number to each duplicate group and delete the extra rows.
    ```sql
    WITH CTE AS (
    SELECT *,
    ROW_NUMBER() OVER (PARTITION BY employee_name, department, salary ORDER BY employee_id) AS row_num
    FROM Employees
    )
    DELETE FROM Employees
    WHERE employee_id IN (
    SELECT employee_id FROM CTE WHERE row_num > 1
    );
    ```

29. What is the difference between correlated and non-correlated subqueries?
30. How do you optimize SQL queries for performance?
31. How do you monitor query execution plans?
32. What is the purpose of the `EXPLAIN` statement?
33. What are materialized views, and how are they different from regular views?
34. What are window functions in SQL? Provide examples.
35. What is the impact of too many indexes on performance?
36. How do you perform data migration between two databases?
37. What are sequences, and how do they generate unique values?
38. How do you manage temporary tables in SQL?
39. How do you perform recursive queries in SQL?
40. What are pivot tables, and how are they created in SQL?
41. What is the purpose of the `MERGE` statement?
42. How do you split a string in SQL?
43. How do you concatenate strings in SQL?
44. How do you prevent SQL injection attacks?
45. What is the difference between OLTP and OLAP?
46. How do you handle NULL values in calculations?
47. How do you use `EXISTS` and `NOT EXISTS`?
48. What is sharding, and how is it implemented?
49. What are partitioned tables, and how do they enhance performance?
50. How do you monitor and troubleshoot deadlocks?
51. How do you handle database replication?
52. How do you implement database backups and restore strategies?
53. What is the difference between vertical and horizontal partitioning?
54. How do you manage database versioning with schema migrations?
55. How do you design a scalable database schema?
56. What is a time-series database, and how does it differ from relational databases?
57. How do you design multi-tenant databases?
58. How do you use SQL with big data technologies?
59. How do you perform ETL (Extract, Transform, Load) operations in SQL?
60. How do you handle schema changes in a production database?
61. What are the best practices for writing complex SQL queries?
62. How do you optimize SQL queries for large datasets?
63. What is the purpose of query hints in SQL?
64. How do you design a reporting system using SQL?
65. What are some common SQL anti-patterns to avoid?
66. How do you secure sensitive data in SQL databases?
67. What is row-level security, and how is it implemented?
68. How do you manage indexes for optimal performance?
69. How do you implement cross-database joins?
70. What is the difference between transactional and analytical databases?
71. How do you perform load testing on SQL databases?
72. How do you implement data encryption at rest and in transit?
73. How do you monitor and manage database performance?
74. How do you handle large data exports and imports?
75. What are some tools for SQL query optimization?
76. How do you migrate data from SQL to NoSQL databases?
77. What is the future of SQL in cloud-based environments?
78. How do you design disaster recovery strategies for SQL databases?
